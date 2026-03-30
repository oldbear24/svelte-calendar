<script lang="ts">
	import type { CalendarEvent } from './types.js';

	interface Props {
		currentDate: Date;
		events: CalendarEvent[];
		onEventClick: (event: CalendarEvent) => void;
		onSlotClick: (date: Date) => void;
	}

	let { currentDate, events, onEventClick, onSlotClick }: Props = $props();

	const HOURS = Array.from({ length: 24 }, (_, i) => i);
	const today = new Date();

	const weekDays = $derived.by(() => {
		const startOfWeek = new Date(currentDate);
		startOfWeek.setDate(currentDate.getDate() - currentDate.getDay());
		startOfWeek.setHours(0, 0, 0, 0);

		return Array.from({ length: 7 }, (_, i) => {
			const d = new Date(startOfWeek);
			d.setDate(startOfWeek.getDate() + i);
			return d;
		});
	});

	const DAY_LABELS = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];

	function isToday(date: Date): boolean {
		return date.toDateString() === today.toDateString();
	}

	function formatHour(h: number): string {
		if (h === 0) return '12 AM';
		if (h < 12) return `${h} AM`;
		if (h === 12) return '12 PM';
		return `${h - 12} PM`;
	}

	function getEventsForDayHour(date: Date): CalendarEvent[] {
		const dayStart = new Date(date);
		dayStart.setHours(0, 0, 0, 0);
		const dayEnd = new Date(date);
		dayEnd.setHours(23, 59, 59, 999);

		return events.filter((e) => {
			if (e.allDay) return false;
			const start = new Date(e.start);
			const end = new Date(e.end);
			return start <= dayEnd && end >= dayStart;
		});
	}

	function getEventStyle(event: CalendarEvent, date: Date): string {
		const dayStart = new Date(date);
		dayStart.setHours(0, 0, 0, 0);
		const dayEnd = new Date(date);
		dayEnd.setHours(24, 0, 0, 0);

		const eventStart = new Date(event.start) < dayStart ? dayStart : new Date(event.start);
		const eventEnd = new Date(event.end) > dayEnd ? dayEnd : new Date(event.end);

		const startMinutes = eventStart.getHours() * 60 + eventStart.getMinutes();
		const endMinutes = eventEnd.getHours() * 60 + eventEnd.getMinutes();
		const durationMinutes = Math.max(endMinutes - startMinutes, 30);

		const top = (startMinutes / 60) * 64;
		const height = (durationMinutes / 60) * 64;

		return `top: ${top}px; height: ${height}px;`;
	}

	function getAllDayEvents(date: Date): CalendarEvent[] {
		const dayStart = new Date(date);
		dayStart.setHours(0, 0, 0, 0);
		const dayEnd = new Date(date);
		dayEnd.setHours(23, 59, 59, 999);

		return events.filter((e) => {
			if (!e.allDay) return false;
			const start = new Date(e.start);
			const end = new Date(e.end);
			return start <= dayEnd && end >= dayStart;
		});
	}
</script>

<div class="flex flex-1 flex-col overflow-hidden">
	<!-- Day headers -->
	<div class="grid border-b border-base-300" style="grid-template-columns: 64px repeat(7, minmax(0, 1fr))">
		<div class="border-r border-base-300"></div>
		{#each weekDays as day, i}
			<div
				class="border-r border-base-300 py-2 text-center last:border-r-0"
				class:bg-base-200={isToday(day)}
			>
				<span class="text-xs text-base-content/60">{DAY_LABELS[i]}</span>
				<div
					class="mx-auto mt-0.5 flex h-8 w-8 items-center justify-center rounded-full text-sm font-semibold
					{isToday(day) ? 'bg-primary text-primary-content' : 'text-base-content'}"
				>
					{day.getDate()}
				</div>
			</div>
		{/each}
	</div>

	<!-- All-day row -->
	{#if weekDays.some((d) => getAllDayEvents(d).length > 0)}
		<div
			class="grid border-b border-base-300"
			style="grid-template-columns: 64px repeat(7, minmax(0, 1fr))"
		>
			<div class="border-r border-base-300 px-1 py-1 text-right text-xs text-base-content/40">
				All day
			</div>
			{#each weekDays as day}
				<div class="min-h-6 border-r border-base-300 p-0.5 last:border-r-0">
					{#each getAllDayEvents(day) as event (event.id)}
						<button
							class="w-full truncate rounded px-1.5 py-0.5 text-left text-xs font-medium text-white transition-opacity hover:opacity-80"
							style:background-color={event.color ?? '#3b82f6'}
							onclick={() => onEventClick(event)}
						>
							{event.title}
						</button>
					{/each}
				</div>
			{/each}
		</div>
	{/if}

	<!-- Scrollable time grid -->
	<div class="flex-1 overflow-y-auto">
		<div
			class="relative grid"
			style="grid-template-columns: 64px repeat(7, minmax(0, 1fr))"
		>
			<!-- Hour rows -->
			{#each HOURS as hour}
				<div
					class="border-b border-base-300 border-r border-r-base-300 h-16 px-1 py-0.5 text-right text-xs text-base-content/40"
					style="grid-column: 1"
				>
					{#if hour > 0}{formatHour(hour)}{/if}
				</div>
				{#each weekDays as _day, di}
					<!-- svelte-ignore a11y_click_events_have_key_events a11y_no_noninteractive_element_interactions -->
					<div
						class="border-b border-r border-base-300 h-16 last:border-r-0 hover:bg-base-200/40 transition-colors"
						style="grid-column: {di + 2}"
						onclick={() => {
							const slotDate = new Date(weekDays[di]);
							slotDate.setHours(hour, 0, 0, 0);
							onSlotClick(slotDate);
						}}
						role="gridcell"
						tabindex="0"
						aria-label={`${weekDays[di].toLocaleDateString()} ${formatHour(hour)}`}
					></div>
				{/each}
			{/each}

			<!-- Events overlay per day column -->
			{#each weekDays as day, di}
				{@const dayEvents = getEventsForDayHour(day)}
				<div
					class="pointer-events-none absolute top-0"
					style="left: calc(64px + {di} * ((100% - 64px) / 7)); width: calc((100% - 64px) / 7); height: {24 * 64}px"
				>
					{#each dayEvents as event (event.id)}
						<button
							class="pointer-events-auto absolute right-0.5 left-0.5 overflow-hidden rounded px-1.5 text-left text-xs font-medium text-white transition-opacity hover:opacity-80"
							style="{getEventStyle(event, day)} background-color: {event.color ?? '#3b82f6'};"
							onclick={() => onEventClick(event)}
							aria-label={event.title}
						>
							<span class="block truncate font-semibold">{event.title}</span>
							<span class="block opacity-80">
								{event.start.toLocaleTimeString(undefined, { hour: '2-digit', minute: '2-digit' })}
							</span>
						</button>
					{/each}
				</div>
			{/each}
		</div>
	</div>
</div>
