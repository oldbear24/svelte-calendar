<script lang="ts">
	import type { CalendarEvent, CalendarView } from './types.js';
	import MonthView from './MonthView.svelte';
	import WeekView from './WeekView.svelte';
	import DayView from './DayView.svelte';
	import AgendaView from './AgendaView.svelte';
	import EventModal from './EventModal.svelte';
	import { untrack } from 'svelte';
	import { SvelteDate } from 'svelte/reactivity';

	let {
		events = [],
		view = 'month',
		onEventClick,
		onSlotClick
	}: {
		events?: CalendarEvent[];
		view?: CalendarView;
		onEventClick?: (event: CalendarEvent) => void;
		onSlotClick?: (date: Date) => void;
	} = $props();

	let currentView: CalendarView = $state(untrack(() => view));
	let currentDate: Date = $state(new Date());
	let selectedEvent: CalendarEvent | null = $state(null);

	const VIEW_LABELS: Record<CalendarView, string> = {
		month: 'Month',
		week: 'Week',
		day: 'Day',
		agenda: 'Agenda'
	};

	const headerTitle = $derived.by(() => {
		if (currentView === 'month') {
			return currentDate.toLocaleDateString(undefined, { month: 'long', year: 'numeric' });
		}
		if (currentView === 'week') {
			const startOfWeek = new SvelteDate(currentDate);
			startOfWeek.setDate(currentDate.getDate() - currentDate.getDay());
			const endOfWeek = new SvelteDate(startOfWeek);
			endOfWeek.setDate(startOfWeek.getDate() + 6);

			if (startOfWeek.getMonth() === endOfWeek.getMonth()) {
				return startOfWeek.toLocaleDateString(undefined, {
					month: 'long',
					year: 'numeric'
				});
			}
			return (
				startOfWeek.toLocaleDateString(undefined, { month: 'short' }) +
				' – ' +
				endOfWeek.toLocaleDateString(undefined, { month: 'short', year: 'numeric' })
			);
		}
		if (currentView === 'day') {
			return currentDate.toLocaleDateString(undefined, {
				weekday: 'long',
				month: 'long',
				day: 'numeric',
				year: 'numeric'
			});
		}
		// agenda
		return currentDate.toLocaleDateString(undefined, { month: 'long', year: 'numeric' });
	});

	function goToToday() {
		currentDate = new Date();
	}

	function navigate(direction: -1 | 1) {
		const d = new SvelteDate(currentDate);
		if (currentView === 'month') {
			d.setMonth(d.getMonth() + direction);
		} else if (currentView === 'week') {
			d.setDate(d.getDate() + direction * 7);
		} else if (currentView === 'day') {
			d.setDate(d.getDate() + direction);
		} else {
			d.setMonth(d.getMonth() + direction);
		}
		currentDate = d;
	}

	function handleEventClick(event: CalendarEvent) {
		selectedEvent = event;
		onEventClick?.(event);
	}

	function handleSlotClick(date: Date) {
		onSlotClick?.(date);
	}
</script>

<div
	class="flex h-full min-h-96 w-full flex-col overflow-hidden rounded-lg border border-base-300 bg-base-100 shadow-sm"
>
	<!-- Toolbar -->
	<div class="flex shrink-0 items-center gap-2 border-b border-base-300 px-4 py-3">
		<!-- Navigation -->
		<div class="flex items-center gap-1">
			<button
				class="btn btn-circle btn-ghost btn-sm"
				onclick={() => navigate(-1)}
				aria-label="Previous"
			>
				‹
			</button>
			<button class="btn btn-circle btn-ghost btn-sm" onclick={() => navigate(1)} aria-label="Next">
				›
			</button>
		</div>

		<button class="btn btn-outline btn-sm" onclick={goToToday}> Today </button>

		<!-- Title -->
		<h2 class="min-w-48 flex-1 px-2 text-lg font-semibold text-base-content" aria-live="polite">
			{headerTitle}
		</h2>

		<!-- View switcher -->
		<div class="join" role="group" aria-label="Calendar view">
			{#each Object.keys(VIEW_LABELS) as v (v)}
				{@const viewKey = v as CalendarView}
				<button
					class="btn join-item btn-sm {currentView === viewKey ? 'btn-primary' : 'btn-ghost'}"
					onclick={() => (currentView = viewKey)}
					aria-pressed={currentView === viewKey}
				>
					{VIEW_LABELS[viewKey]}
				</button>
			{/each}
		</div>
	</div>

	<!-- View content -->
	<div class="flex flex-1 flex-col overflow-hidden">
		{#if currentView === 'month'}
			<MonthView
				{currentDate}
				{events}
				onEventClick={handleEventClick}
				onSlotClick={handleSlotClick}
			/>
		{:else if currentView === 'week'}
			<WeekView
				{currentDate}
				{events}
				onEventClick={handleEventClick}
				onSlotClick={handleSlotClick}
			/>
		{:else if currentView === 'day'}
			<DayView
				{currentDate}
				{events}
				onEventClick={handleEventClick}
				onSlotClick={handleSlotClick}
			/>
		{:else}
			<AgendaView {currentDate} {events} onEventClick={handleEventClick} />
		{/if}
	</div>
</div>

{#if selectedEvent}
	<EventModal event={selectedEvent} onClose={() => (selectedEvent = null)} />
{/if}
