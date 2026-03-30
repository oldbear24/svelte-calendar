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

	function isToday(date: Date): boolean {
		return date.toDateString() === today.toDateString();
	}

	function formatHour(h: number): string {
		if (h === 0) return '12 AM';
		if (h < 12) return `${h} AM`;
		if (h === 12) return '12 PM';
		return `${h - 12} PM`;
	}

	const dayEvents = $derived(
		events.filter((e) => {
			if (e.allDay) return false;
			const dayStart = new Date(currentDate);
			dayStart.setHours(0, 0, 0, 0);
			const dayEnd = new Date(currentDate);
			dayEnd.setHours(23, 59, 59, 999);
			return new Date(e.start) <= dayEnd && new Date(e.end) >= dayStart;
		})
	);

	const allDayEvents = $derived(
		events.filter((e) => {
			if (!e.allDay) return false;
			const dayStart = new Date(currentDate);
			dayStart.setHours(0, 0, 0, 0);
			const dayEnd = new Date(currentDate);
			dayEnd.setHours(23, 59, 59, 999);
			return new Date(e.start) <= dayEnd && new Date(e.end) >= dayStart;
		})
	);

	function getEventStyle(event: CalendarEvent): string {
		const dayStart = new Date(currentDate);
		dayStart.setHours(0, 0, 0, 0);
		const dayEnd = new Date(currentDate);
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
</script>

<div class="flex flex-1 flex-col overflow-hidden">
	<!-- Day header -->
	<div class="border-b border-base-300 py-3 text-center">
		<span class="text-sm text-base-content/60">
			{currentDate.toLocaleDateString(undefined, { weekday: 'long' })}
		</span>
		<div
			class="mx-auto mt-0.5 flex h-10 w-10 items-center justify-center rounded-full text-lg font-semibold
			{isToday(currentDate) ? 'bg-primary text-primary-content' : 'text-base-content'}"
		>
			{currentDate.getDate()}
		</div>
	</div>

	<!-- All-day events -->
	{#if allDayEvents.length > 0}
		<div class="border-b border-base-300 p-1">
			<span class="mr-2 text-xs text-base-content/40">All day</span>
			{#each allDayEvents as event (event.id)}
				<button
					class="mr-1 rounded px-2 py-0.5 text-xs font-medium text-white transition-opacity hover:opacity-80"
					style:background-color={event.color ?? '#3b82f6'}
					onclick={() => onEventClick(event)}
				>
					{event.title}
				</button>
			{/each}
		</div>
	{/if}

	<!-- Scrollable time grid -->
	<div class="flex-1 overflow-y-auto">
		<div class="relative" style="grid-template-columns: 64px 1fr; display: grid;">
			<!-- Hour slots -->
			{#each HOURS as hour}
				<div
					class="border-b border-r border-base-300 h-16 px-2 py-0.5 text-right text-xs text-base-content/40"
					style="grid-column: 1"
				>
					{#if hour > 0}{formatHour(hour)}{/if}
				</div>
				<!-- svelte-ignore a11y_click_events_have_key_events a11y_no_noninteractive_element_interactions -->
				<div
					class="border-b border-base-300 h-16 hover:bg-base-200/40 transition-colors"
					style="grid-column: 2"
					onclick={() => {
						const slotDate = new Date(currentDate);
						slotDate.setHours(hour, 0, 0, 0);
						onSlotClick(slotDate);
					}}
					role="gridcell"
					tabindex="0"
					aria-label={`${currentDate.toLocaleDateString()} ${formatHour(hour)}`}
				></div>
			{/each}

			<!-- Events overlay -->
			<div
				class="pointer-events-none absolute"
				style="left: 64px; right: 0; top: 0; height: {24 * 64}px"
			>
				{#each dayEvents as event (event.id)}
					<button
						class="pointer-events-auto absolute right-1 left-1 overflow-hidden rounded px-2 text-left text-xs font-medium text-white transition-opacity hover:opacity-80"
						style="{getEventStyle(event)} background-color: {event.color ?? '#3b82f6'};"
						onclick={() => onEventClick(event)}
						aria-label={event.title}
					>
						<span class="block truncate font-semibold">{event.title}</span>
						<span class="block opacity-80">
							{event.start.toLocaleTimeString(undefined, { hour: '2-digit', minute: '2-digit' })} –
							{event.end.toLocaleTimeString(undefined, { hour: '2-digit', minute: '2-digit' })}
						</span>
						{#if event.description}
							<span class="block truncate opacity-70">{event.description}</span>
						{/if}
					</button>
				{/each}
			</div>
		</div>
	</div>
</div>
