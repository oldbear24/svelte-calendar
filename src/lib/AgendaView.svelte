<script lang="ts">
	import type { CalendarEvent } from './types.js';
	import { SvelteDate, SvelteMap } from 'svelte/reactivity';

	interface Props {
		currentDate: Date;
		events: CalendarEvent[];
		onEventClick: (event: CalendarEvent) => void;
	}

	let { currentDate, events, onEventClick }: Props = $props();

	const today = new Date();

	// Show events from current date forward (up to 90 days)
	const upcomingEvents = $derived.by(() => {
		const start = new SvelteDate(currentDate);
		start.setHours(0, 0, 0, 0);
		const end = new SvelteDate(start);
		end.setDate(end.getDate() + 90);

		return events
			.filter((e) => new Date(e.end) >= start && new Date(e.start) <= end)
			.sort((a, b) => new Date(a.start).getTime() - new Date(b.start).getTime());
	});

	// Group events by date label
	const groupedEvents = $derived.by(() => {
		const groups = new SvelteMap<string, { label: string; date: Date; events: CalendarEvent[] }>();

		for (const event of upcomingEvents) {
			const eventDate = new SvelteDate(event.start);
			eventDate.setHours(0, 0, 0, 0);
			const key = eventDate.toDateString();

			if (!groups.has(key)) {
				groups.set(key, { label: formatDateLabel(eventDate), date: eventDate, events: [] });
			}
			groups.get(key)!.events.push(event);
		}

		return [...groups.values()];
	});

	function isToday(date: Date): boolean {
		return date.toDateString() === today.toDateString();
	}

	function formatDateLabel(date: Date): string {
		if (isToday(date)) return 'Today';
		const tomorrow = new SvelteDate(today);
		tomorrow.setDate(today.getDate() + 1);
		if (date.toDateString() === tomorrow.toDateString()) return 'Tomorrow';

		return date.toLocaleDateString(undefined, {
			weekday: 'long',
			month: 'long',
			day: 'numeric',
			year: date.getFullYear() !== today.getFullYear() ? 'numeric' : undefined
		});
	}

	function formatTime(date: Date): string {
		return date.toLocaleTimeString(undefined, { hour: '2-digit', minute: '2-digit' });
	}

	function formatDuration(event: CalendarEvent): string {
		if (event.allDay) return 'All day';
		return `${formatTime(new Date(event.start))} – ${formatTime(new Date(event.end))}`;
	}
</script>

<div class="flex-1 overflow-y-auto px-4 py-2">
	{#if groupedEvents.length === 0}
		<div class="py-16 text-center">
			<p class="text-base-content/40">No upcoming events</p>
		</div>
	{:else}
		{#each groupedEvents as group (group.date.toDateString())}
			<div class="mb-4">
				<!-- Date header -->
				<div class="mb-2 flex items-center gap-3">
					<div
						class="flex h-12 w-12 shrink-0 flex-col items-center justify-center rounded-full text-center
						{isToday(group.date) ? 'bg-primary text-primary-content' : 'bg-base-200 text-base-content'}"
					>
						<span class="text-xs leading-none font-medium">
							{group.date.toLocaleDateString(undefined, { month: 'short' })}
						</span>
						<span class="text-lg leading-none font-bold">{group.date.getDate()}</span>
					</div>
					<div>
						<h3 class="font-semibold {isToday(group.date) ? 'text-primary' : 'text-base-content'}">
							{group.label}
						</h3>
						<p class="text-xs text-base-content/50">
							{group.date.toLocaleDateString(undefined, { weekday: 'long' })}
						</p>
					</div>
				</div>

				<!-- Events list -->
				<div class="ml-15 space-y-1 pl-3">
					{#each group.events as event (event.id)}
						<button
							class="flex w-full items-start gap-3 rounded-lg border border-base-300 bg-base-100 px-3 py-2 text-left transition-colors hover:bg-base-200"
							onclick={() => onEventClick(event)}
							aria-label={event.title}
						>
							<span
								class="mt-1 h-2.5 w-2.5 shrink-0 rounded-full"
								style:background-color={event.color ?? '#3b82f6'}
								aria-hidden="true"
							></span>
							<div class="min-w-0 flex-1">
								<p class="truncate font-medium text-base-content">{event.title}</p>
								<p class="text-xs text-base-content/50">{formatDuration(event)}</p>
								{#if event.description}
									<p class="mt-0.5 truncate text-xs text-base-content/40">{event.description}</p>
								{/if}
							</div>
						</button>
					{/each}
				</div>
			</div>
		{/each}
	{/if}
</div>
