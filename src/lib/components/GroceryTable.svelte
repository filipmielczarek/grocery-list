<script lang="ts">
	import {
		createColumnHelper,
		createTable,
		FlexRender,
		rowSelectionFeature,
		tableFeatures
	} from '@tanstack/svelte-table';
	import { ingredients } from '../consts/ingredients';

	type GroceryItem = {
		kategoria: string;
		nazwa: string;
		gramatura: number;
	};

	const data: GroceryItem[] = Object.entries(ingredients).flatMap(([kategoria, items]) =>
		items.map((item) => ({ ...item, kategoria }))
	);

	const features = tableFeatures({ rowSelectionFeature });
	const columnHelper = createColumnHelper<typeof features, GroceryItem>();

	const gramFormatter = new Intl.NumberFormat('pl-PL', { maximumFractionDigits: 1 });

	const columns = columnHelper.columns([
		columnHelper.accessor('nazwa', { header: 'Produkt' }),
		columnHelper.accessor('gramatura', {
			header: 'Gramatura',
			cell: (info) => `${gramFormatter.format(info.getValue())} g`
		})
	]);

	const table = createTable({
		features,
		columns,
		get data() {
			return data;
		}
	});

	const selection = $derived(table.atoms.rowSelection.get());
	const selectedCount = $derived(Object.keys(selection).length);
	const progress = $derived(data.length === 0 ? 0 : (selectedCount / data.length) * 100);

	const groups = $derived.by(() => {
		type GroceryRow = ReturnType<typeof table.getRowModel>['rows'][number];
		const grouped: Record<string, GroceryRow[]> = {};

		for (const row of table.getRowModel().rows) {
			(grouped[row.original.kategoria] ??= []).push(row);
		}

		return Object.entries(grouped).map(([kategoria, rows]) => ({ kategoria, rows }));
	});
</script>

<div class="min-h-screen bg-white text-teal-800">
	<main class="mx-auto w-full max-w-3xl px-4 py-10 sm:px-6 lg:py-16">
		<header class="mb-8">
			<p class="text-xs font-semibold tracking-[0.25em] text-orange-500 uppercase">Grocery list</p>
			<h1 class="mt-2 text-3xl font-bold tracking-tight text-teal-800 sm:text-4xl">
				Lista zakupów
			</h1>
			<p class="mt-2 text-sm text-teal-800/70">Odznacz produkty, które już masz w domu.</p>

			<div class="mt-6 flex flex-wrap items-center gap-3">
				<div class="h-2 min-w-40 flex-1 overflow-hidden rounded-full bg-lime-100">
					<div
						class="h-full rounded-full bg-teal-800 transition-[width] duration-300"
						style:width="{progress}%"
					></div>
				</div>
				<span class="text-sm font-semibold text-teal-800 tabular-nums">
					{selectedCount} / {data.length}
				</span>
				{#if selectedCount > 0}
					<button
						type="button"
						class="rounded-full border border-teal-800/20 px-3 py-1 text-sm font-medium text-teal-800 transition-colors hover:bg-lime-100"
						onclick={() => table.resetRowSelection()}
					>
						Wyczyść
					</button>
				{/if}
			</div>
		</header>

		<div class="overflow-hidden rounded-2xl border border-teal-800/15">
			<table class="w-full border-collapse text-left">
				<thead>
					{#each table.getHeaderGroups() as headerGroup (headerGroup.id)}
						<tr class="border-b border-teal-800/15 bg-white">
							<th
								scope="col"
								class="w-12 px-4 py-3 text-xs font-semibold tracking-wider text-teal-800/50 uppercase"
							>
								Mam
							</th>
							{#each headerGroup.headers as header (header.id)}
								<th
									scope="col"
									class="px-4 py-3 text-xs font-semibold tracking-wider text-teal-800/50 uppercase"
									class:text-right={header.column.id === 'gramatura'}
								>
									{#if !header.isPlaceholder}
										<FlexRender {header} />
									{/if}
								</th>
							{/each}
						</tr>
					{/each}
				</thead>

				{#each groups as group (group.kategoria)}
					{@const bought = group.rows.filter((row) => row.getIsSelected()).length}
					<tbody>
						<tr class="bg-teal-800 text-white">
							<th scope="rowgroup" colspan={columns.length + 1} class="px-4 py-2.5 text-left">
								<div class="flex items-center justify-between gap-4">
									<span
										class="flex items-center gap-2 text-xs font-semibold tracking-wider uppercase"
									>
										<span class="h-2 w-2 rounded-full bg-orange-500"></span>
										{group.kategoria}
									</span>
									<span
										class="rounded-full bg-white/10 px-2.5 py-0.5 text-xs font-medium tabular-nums"
									>
										{bought}/{group.rows.length}
									</span>
								</div>
							</th>
						</tr>

						{#each group.rows as row (row.id)}
							<tr
								class="cursor-pointer border-t border-teal-800/10 transition-colors {row.getIsSelected()
									? 'bg-lime-100'
									: 'bg-white hover:bg-lime-100'}"
								onclick={() => row.toggleSelected()}
							>
								<td class="w-12 px-4 py-3">
									<input
										type="checkbox"
										class="h-5 w-5 cursor-pointer rounded border-teal-800/40 accent-orange-500"
										checked={row.getIsSelected()}
										onclick={(event) => event.stopPropagation()}
										onchange={() => row.toggleSelected()}
										aria-label="Oznacz {row.original.nazwa} jako kupione"
									/>
								</td>
								{#each row.getAllCells() as cell (cell.id)}
									<td
										class="px-4 py-3 text-sm {cell.column.id === 'gramatura'
											? 'text-right text-teal-800/50 tabular-nums'
											: ''} {cell.column.id === 'nazwa' ? 'font-medium' : ''}"
									>
										<span
											class:line-through={row.getIsSelected()}
											class:opacity-60={row.getIsSelected()}
										>
											<FlexRender {cell} />
										</span>
									</td>
								{/each}
							</tr>
						{/each}
					</tbody>
				{/each}
			</table>
		</div>

		{#if selectedCount === data.length}
			<p class="mt-6 text-center text-sm font-medium text-orange-500">
				Wszystko masz. Miłych zakupów!
			</p>
		{/if}
	</main>
</div>
