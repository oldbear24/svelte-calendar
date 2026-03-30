<script lang="ts">
	import { Calendar } from '$lib/index.js';
	import type { CalendarEvent } from '$lib/index.js';

	const today = new Date();

	function d(dayOffset: number, hour: number, minute = 0): Date {
		const dt = new Date(today);
		dt.setDate(dt.getDate() + dayOffset);
		dt.setHours(hour, minute, 0, 0);
		return dt;
	}

	const events: CalendarEvent[] = [
		{
			id: '1',
			title: 'Team Standup',
			start: d(0, 9, 0),
			end: d(0, 9, 30),
			color: '#6366f1',
			description: 'Daily sync with the engineering team'
		},
		{
			id: '2',
			title: 'Product Review',
			start: d(0, 14, 0),
			end: d(0, 15, 30),
			color: '#0ea5e9',
			description: 'Q4 product roadmap review'
		},
		{
			id: '3',
			title: 'Lunch with Design',
			start: d(1, 12, 0),
			end: d(1, 13, 0),
			color: '#10b981'
		},
		{
			id: '4',
			title: 'Sprint Planning',
			start: d(2, 10, 0),
			end: d(2, 12, 0),
			color: '#f59e0b',
			description: 'Planning for the next two-week sprint'
		},
		{
			id: '5',
			title: 'All Hands Meeting',
			start: d(3, 15, 0),
			end: d(3, 16, 0),
			color: '#ef4444',
			description: 'Company-wide quarterly update'
		},
		{
			id: '6',
			title: 'Conference',
			start: d(5, 0, 0),
			end: d(7, 23, 59),
			color: '#8b5cf6',
			description: 'Annual tech conference',
			allDay: true
		},
		{
			id: '7',
			title: 'Design Workshop',
			start: d(-1, 13, 0),
			end: d(-1, 16, 0),
			color: '#ec4899'
		},
		{
			id: '8',
			title: 'Release v2.0',
			start: d(10, 0, 0),
			end: d(10, 23, 59),
			color: '#14b8a6',
			allDay: true
		},
		{
			id: '9',
			title: 'Architecture Review',
			start: d(4, 11, 0),
			end: d(4, 12, 0),
			color: '#f97316'
		},
		{
			id: '10',
			title: 'Client Call',
			start: d(1, 16, 0),
			end: d(1, 17, 0),
			color: '#0ea5e9',
			description: 'Demo new features to Acme Corp'
		}
	];

	let lastSlotClick: Date | null = $state(null);
	let lastEventClick: CalendarEvent | null = $state(null);
</script>

<div class="bg-base-200 min-h-screen p-4 md:p-8">
	<div class="mx-auto max-w-6xl">
		<div class="mb-6">
			<h1 class="text-3xl font-bold text-base-content">Svelte Calendar</h1>
			<p class="mt-1 text-base-content/60">A Microsoft Outlook–style calendar built with Svelte 5 + DaisyUI</p>
		</div>

		<!-- Calendar fills the available height -->
		<div class="h-[700px]">
			<Calendar
				{events}
				view="month"
				onEventClick={(e) => (lastEventClick = e)}
				onSlotClick={(d) => (lastSlotClick = d)}
			/>
		</div>

		<!-- Interaction feedback -->
		<div class="mt-4 flex flex-wrap gap-4">
			{#if lastSlotClick}
				<div class="badge badge-outline badge-lg gap-1">
					<span class="opacity-60">Slot clicked:</span>
					{lastSlotClick.toLocaleString()}
				</div>
			{/if}
			{#if lastEventClick}
				<div class="badge badge-primary badge-lg gap-1">
					<span class="opacity-80">Event clicked:</span>
					{lastEventClick.title}
				</div>
			{/if}
		</div>
	</div>
</div>

