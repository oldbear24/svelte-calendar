<script lang="ts">
	import type { CalendarEvent } from './types.js';

	interface Props {
		event: CalendarEvent;
		onClose: () => void;
	}

	let { event, onClose }: Props = $props();

	let dialog: HTMLDialogElement | undefined = $state();

	$effect(() => {
		if (dialog) dialog.showModal();
	});

	function formatDate(date: Date): string {
		return date.toLocaleDateString(undefined, {
			weekday: 'long',
			year: 'numeric',
			month: 'long',
			day: 'numeric'
		});
	}

	function formatTime(date: Date): string {
		return date.toLocaleTimeString(undefined, { hour: '2-digit', minute: '2-digit' });
	}
</script>

<!-- svelte-ignore a11y_click_events_have_key_events a11y_no_noninteractive_element_interactions -->
<dialog
	bind:this={dialog}
	class="modal"
	aria-labelledby="event-modal-title"
	onclick={(e) => {
		if (e.target === dialog) onClose();
	}}
>
	<div class="modal-box max-w-md">
		<div class="mb-4 flex items-start justify-between">
			<div class="flex items-center gap-3">
				<span
					class="mt-1 h-3 w-3 shrink-0 rounded-full"
					style:background-color={event.color ?? '#3b82f6'}
					aria-hidden="true"
				></span>
				<h3 id="event-modal-title" class="text-lg font-bold">{event.title}</h3>
			</div>
			<button class="btn btn-ghost btn-sm btn-circle" onclick={onClose} aria-label="Close">✕</button>
		</div>

		<div class="space-y-3 text-sm">
			{#if event.allDay}
				<div class="flex items-center gap-2">
					<span class="text-base-content/60 w-20 shrink-0">Date</span>
					<span>{formatDate(event.start)}</span>
				</div>
			{:else}
				<div class="flex items-center gap-2">
					<span class="text-base-content/60 w-20 shrink-0">Start</span>
					<span>{formatDate(event.start)} at {formatTime(event.start)}</span>
				</div>
				<div class="flex items-center gap-2">
					<span class="text-base-content/60 w-20 shrink-0">End</span>
					<span>{formatDate(event.end)} at {formatTime(event.end)}</span>
				</div>
			{/if}

			{#if event.description}
				<div class="flex items-start gap-2">
					<span class="text-base-content/60 w-20 shrink-0">Details</span>
					<p class="flex-1">{event.description}</p>
				</div>
			{/if}
		</div>

		<div class="modal-action">
			<button class="btn btn-primary btn-sm" onclick={onClose}>Close</button>
		</div>
	</div>
</dialog>
