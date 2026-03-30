<script lang="ts">
	import type { CalendarEvent } from './types.js';

	interface Props {
		currentDate: Date;
		events: CalendarEvent[];
		onEventClick: (event: CalendarEvent) => void;
		onSlotClick: (date: Date) => void;
	}

	let { currentDate, events, onEventClick, onSlotClick }: Props = $props();

	const DAYS = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];
	const MAX_VISIBLE_EVENTS = 3;

	const today = new Date();
	today.setHours(0, 0, 0, 0);

	const weeks = $derived.by(() => {
		const year = currentDate.getFullYear();
		const month = currentDate.getMonth();

		const firstDay = new Date(year, month, 1);
		const lastDay = new Date(year, month + 1, 0);

		// Start grid from the Sunday before (or on) the first day
		const startDate = new Date(firstDay);
		startDate.setDate(startDate.getDate() - startDate.getDay());

		// End grid on the Saturday after (or on) the last day
		const endDate = new Date(lastDay);
		endDate.setDate(endDate.getDate() + (6 - endDate.getDay()));

		const weeksArr: Date[][] = [];
		const cursor = new Date(startDate);

		while (cursor <= endDate) {
			const week: Date[] = [];
			for (let d = 0; d < 7; d++) {
				week.push(new Date(cursor));
				cursor.setDate(cursor.getDate() + 1);
			}
			weeksArr.push(week);
		}

		return weeksArr;
	});

	function getEventsForDay(date: Date): CalendarEvent[] {
		const dayStart = new Date(date);
		dayStart.setHours(0, 0, 0, 0);
		const dayEnd = new Date(date);
		dayEnd.setHours(23, 59, 59, 999);

		return events.filter((e) => {
			const start = new Date(e.start);
			const end = new Date(e.end);
			return start <= dayEnd && end >= dayStart;
		});
	}

	function isToday(date: Date): boolean {
		return date.toDateString() === today.toDateString();
	}

	function isCurrentMonth(date: Date): boolean {
		return date.getMonth() === currentDate.getMonth();
	}
</script>

<div class="flex flex-1 flex-col overflow-hidden">
	<!-- Day headers -->
	<div class="grid grid-cols-7 border-b border-base-300">
		{#each DAYS as day}
			<div class="py-2 text-center text-xs font-semibold tracking-wide text-base-content/60 uppercase">
				{day}
			</div>
		{/each}
	</div>

	<!-- Calendar grid -->
	<div class="grid flex-1 grid-cols-7" style="grid-template-rows: repeat({weeks.length}, minmax(0, 1fr))">
		{#each weeks as week}
			{#each week as day}
				{@const dayEvents = getEventsForDay(day)}
				{@const visibleEvents = dayEvents.slice(0, MAX_VISIBLE_EVENTS)}
				{@const overflowCount = dayEvents.length - MAX_VISIBLE_EVENTS}
				<!-- svelte-ignore a11y_click_events_have_key_events a11y_no_noninteractive_element_interactions -->
				<div
					class="flex min-h-20 flex-col border-b border-r border-base-300 p-1 transition-colors hover:bg-base-200/50 {!isCurrentMonth(day) ? 'bg-base-200/30' : ''}"
					onclick={() => onSlotClick(day)}
					role="gridcell"
					tabindex="0"
					aria-label={day.toLocaleDateString()}
				>
					<button
						class="mb-1 flex h-7 w-7 items-center justify-center self-center rounded-full text-sm font-medium transition-colors
							{isToday(day) ? 'bg-primary text-primary-content' : 'hover:bg-base-300'}
							{!isCurrentMonth(day) ? 'text-base-content/30' : 'text-base-content'}"
						onclick={(e) => { e.stopPropagation(); onSlotClick(day); }}
						aria-label={isToday(day) ? `Today, ${day.toLocaleDateString()}` : day.toLocaleDateString()}
					>
						{day.getDate()}
					</button>

					<div class="flex flex-col gap-0.5">
						{#each visibleEvents as event (event.id)}
							<button
								class="w-full truncate rounded px-1.5 py-0.5 text-left text-xs font-medium text-white transition-opacity hover:opacity-80"
								style:background-color={event.color ?? '#3b82f6'}
								onclick={(e) => { e.stopPropagation(); onEventClick(event); }}
								aria-label={event.title}
							>
								{#if !event.allDay}
									<span class="opacity-80"
										>{event.start.toLocaleTimeString(undefined, { hour: '2-digit', minute: '2-digit' })}</span
									>
								{/if}
								{event.title}
							</button>
						{/each}
						{#if overflowCount > 0}
							<button
								class="w-full rounded px-1.5 py-0.5 text-left text-xs font-medium text-base-content/60 transition-colors hover:bg-base-300"
								onclick={(e) => { e.stopPropagation(); onSlotClick(day); }}
							>
								+{overflowCount} more
							</button>
						{/if}
					</div>
				</div>
			{/each}
		{/each}
	</div>
</div>
