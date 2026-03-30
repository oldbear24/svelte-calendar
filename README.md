# svelte-calendar

A Microsoft Outlook–style calendar component for Svelte 5, styled with [DaisyUI](https://daisyui.com/) v5 and [Tailwind CSS](https://tailwindcss.com/) v4.

## Features

- **Four view modes** — Month, Week, Day, and Agenda
- **Pagination** — Prev / Next / Today navigation for every view
- **Events** — Display events as colored chips (month) or time-positioned blocks (week/day); grouped list in agenda
- **Event detail modal** — Click any event to open a DaisyUI dialog with full details
- **Slot click callback** — Click an empty calendar slot to receive the target date
- **All-day events** — Rendered in a dedicated all-day row in week/day views
- **Overflow handling** — Month view shows "+N more" when a day has too many events
- **Accessible** — ARIA roles, labels, and keyboard navigation throughout
- **TypeScript** — Fully typed props and exported types

## Installation

```sh
npm install svelte-calendar
```

> **Peer dependencies:** `svelte ^5.0.0`
>
> The component uses Tailwind CSS v4 + DaisyUI v5. Make sure both are configured in your project:
>
> ```css
> /* app.css */
> @import 'tailwindcss';
> @plugin 'daisyui';
> ```

## Usage

```svelte
<script lang="ts">
	import { Calendar } from 'svelte-calendar';
	import type { CalendarEvent } from 'svelte-calendar';

	const events: CalendarEvent[] = [
		{
			id: '1',
			title: 'Team Standup',
			start: new Date('2026-03-30T09:00:00'),
			end: new Date('2026-03-30T09:30:00'),
			color: '#6366f1',
			description: 'Daily sync with the engineering team'
		},
		{
			id: '2',
			title: 'Conference',
			start: new Date('2026-04-04'),
			end: new Date('2026-04-04'),
			allDay: true,
			color: '#8b5cf6'
		}
	];
</script>

<Calendar
	{events}
	view="month"
	onEventClick={(event) => console.log('clicked', event)}
	onSlotClick={(date) => console.log('slot', date)}
/>
```

## Props

| Prop           | Type                             | Default   | Description                                               |
| -------------- | -------------------------------- | --------- | --------------------------------------------------------- |
| `events`       | `CalendarEvent[]`                | `[]`      | Array of events to display                                |
| `view`         | `CalendarView`                   | `'month'` | Initial view: `'month'`, `'week'`, `'day'`, or `'agenda'` |
| `onEventClick` | `(event: CalendarEvent) => void` | —         | Called when the user clicks an event                      |
| `onSlotClick`  | `(date: Date) => void`           | —         | Called when the user clicks an empty calendar slot        |

## Types

```typescript
interface CalendarEvent {
	id: string;
	title: string;
	start: Date;
	end: Date;
	color?: string; // CSS color string, defaults to indigo
	description?: string;
	allDay?: boolean;
}

type CalendarView = 'month' | 'week' | 'day' | 'agenda';
```

## Development

```sh
npm install
npm run dev        # start the demo app
npm run build      # build the library + demo
npm run test       # run unit & component tests
npm run lint       # prettier + eslint check
```

## Publishing

```sh
npm run prepack    # builds the dist/ package output
npm publish
```
