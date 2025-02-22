<script lang="ts">
	import { createCalendar, melt, type CreateSelectProps } from '$lib';
	import { ChevronRight, ChevronLeft } from 'lucide-svelte';
	import MonthSelect from './MonthSelect.svelte';
	import { writable } from 'svelte/store';
	import { DateFormatter, getLocalTimeZone } from '@internationalized/date';

	const {
		elements: { calendar, heading, grid, cell, prevButton, nextButton },
		states: { months, headingValue, daysOfWeek, placeholder },
		helpers: { isDateDisabled, isDateUnavailable, setMonth },
	} = createCalendar({
		fixedWeeks: true,
	});

	const monthFormatter = new DateFormatter('en', {
		month: 'long',
	});

	const selected: CreateSelectProps<number>['selected'] = writable({
		value: $placeholder.month,
		label: monthFormatter.format($placeholder.toDate(getLocalTimeZone())),
	});

	$: if ($selected.value !== $placeholder.month) {
		selected.set({
			value: $placeholder.month,
			label: monthFormatter.format($placeholder.toDate(getLocalTimeZone())),
		});
	}
</script>

<div class="flex flex-col gap-4">
	<div use:melt={$calendar}>
		<header>
			<button use:melt={$prevButton}>
				<ChevronLeft size={24} />
			</button>
			<div use:melt={$heading}>
				{$headingValue}
			</div>
			<button use:melt={$nextButton}>
				<ChevronRight size={24} />
			</button>
		</header>
		<div>
			{#each $months as month}
				<table use:melt={$grid}>
					<thead aria-hidden="true">
						<tr>
							{#each $daysOfWeek as day}
								<th>
									<div>
										{day}
									</div>
								</th>
							{/each}
						</tr>
					</thead>
					<tbody>
						{#each month.weeks as weekDates}
							<tr>
								{#each weekDates as date}
									<td
										role="gridcell"
										aria-disabled={$isDateDisabled(date) ||
											$isDateUnavailable(date)}
									>
										<div use:melt={$cell(date, month.value)}>
											{date.day}
										</div>
									</td>
								{/each}
							</tr>
						{/each}
					</tbody>
				</table>
			{/each}
		</div>
	</div>
	<MonthSelect
		onSelectedChange={({ next }) => {
			if (next) {
				setMonth(next.value);
			}
			return next;
		}}
		{selected}
	/>
</div>

<style lang="postcss">
	[data-melt-calendar] {
		@apply w-full rounded-lg bg-neutral-800/90 p-3 text-white shadow-sm;
	}

	header {
		@apply flex items-center justify-between pb-2;
	}

	header + div {
		@apply flex items-center gap-6;
	}

	[data-melt-calendar-prevbutton] {
		@apply rounded-lg p-1 transition-all hover:bg-magnum-500/20;
	}

	[data-melt-calendar-nextbutton] {
		@apply rounded-lg p-1 transition-all hover:bg-magnum-500/20;
	}

	[data-melt-calendar-prevbutton][data-disabled] {
		@apply pointer-events-none rounded-lg p-1 opacity-40;
	}

	[data-melt-calendar-nextbutton][data-disabled] {
		@apply pointer-events-none rounded-lg p-1 opacity-40;
	}

	[data-melt-calendar-heading] {
		@apply font-semibold;
	}

	th {
		@apply text-sm font-semibold;

		& div {
			@apply flex h-6 w-6 items-center justify-center p-4;
		}
	}

	[data-melt-calendar-grid] {
		@apply w-full;
	}

	[data-melt-calendar-cell] {
		@apply flex h-6 w-6 cursor-pointer select-none items-center justify-center rounded-lg p-4 hover:bg-magnum-500/20  focus:ring focus:ring-magnum-400;
	}

	[data-melt-calendar-cell][data-disabled] {
		@apply pointer-events-none opacity-40;
	}

	[data-melt-calendar-cell][data-unavailable] {
		@apply pointer-events-none text-red-400 line-through;
	}

	[data-melt-calendar-cell][data-selected] {
		@apply bg-magnum-400 text-neutral-950;
	}

	[data-melt-calendar-cell][data-outside-visible-months] {
		@apply pointer-events-none cursor-default opacity-40 hover:bg-transparent;
	}

	[data-melt-calendar-cell][data-outside-month] {
		@apply pointer-events-none cursor-default opacity-0 hover:bg-transparent;
	}
</style>
