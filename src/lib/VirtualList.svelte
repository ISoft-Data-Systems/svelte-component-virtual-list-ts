<script
	lang="ts"
	generics="T"
>
	import { onMount, tick, untrack, type Snippet } from 'svelte'

	interface Props {
		items: Array<T>
		height?: string
		itemHeight?: undefined | number
		/** Index of first item currently shown in list */
		start?: number
		/** Index of last item currently shown in list*/
		end?: number
		children: Snippet<[{ item: T }]>
	}

	let {
		items,
		height = '100%',
		itemHeight = undefined,
		start = $bindable(0),
		end = $bindable(0),
		children,
	}: Props = $props()

	let heightMap: Array<number> = []
	let rows: HTMLCollectionOf<HTMLElement>
	let viewport: Element | undefined = $state.raw()
	let contents: Element | undefined = $state.raw()
	let viewportHeight = $state(0)
	let visible: Array<{ index: number; data: T }> = $derived(
		items.slice(start, end).map((data, i) => ({ index: i + start, data })),
	)
	let mounted: boolean = $state(false)

	let paddingTop = $state(0)
	let paddingBottom = $state(0)
	let averageHeight: number

	$inspect('viewportHeight', viewportHeight)

	async function refresh(items: Array<T>, viewportHeight: number, itemHeight: number | undefined) {
		$inspect.trace('refresh')
		const isStartOverflow = items.length < start

		if (isStartOverflow) {
			await scrollToIndex(items.length - 1, { behavior: 'auto' })
		}

		await tick() // wait until the DOM is up to date

		console.warn('getting viewport scrollTop')
		let contentHeight = paddingTop - (viewport?.scrollTop ?? 0)
		let i = start

		while (contentHeight < viewportHeight && i < items.length) {
			let row = rows[i - start]

			if (!row) {
				end = i + 1
				await tick() // render the newly visible row
				row = rows[i - start]
			}

			if (!itemHeight) {
				console.warn('will get offsetHeight')
			}
			// getting offsetHeight here will trigger Layout for every row that is rendered when this is run on mount
			const rowHeight = (heightMap[i] = itemHeight || row.offsetHeight)
			contentHeight += rowHeight
			i += 1
		}

		end = i

		const remaining = items.length - end
		averageHeight = (paddingTop + contentHeight) / end

		paddingBottom = remaining * averageHeight
		heightMap.length = items.length
	}

	async function handleScroll() {
		console.warn('getting viewport scrollTop')
		const scrollTop = viewport?.scrollTop ?? 0

		for (let v = 0; v < rows.length; v += 1) {
			if (!itemHeight) {
				console.warn('will get offsetHeight')
			}
			heightMap[start + v] = itemHeight || rows[v].offsetHeight
		}

		let i = 0
		let y = 0

		while (i < items.length) {
			const rowHeight = heightMap[i] || averageHeight
			if (y + rowHeight > scrollTop) {
				start = i
				paddingTop = y

				break
			}

			y += rowHeight
			i += 1
		}

		while (i < items.length) {
			y += heightMap[i] || averageHeight
			i += 1

			if (y > scrollTop + viewportHeight) {
				break
			}
		}

		end = i

		const remainingItems = items.length - end
		averageHeight = y / end

		while (i < items.length) {
			heightMap[i++] = averageHeight
		}
		paddingBottom = remainingItems * averageHeight

		// TODO if we overestimated the space these
		// rows would occupy we may need to add some
		// more. maybe we can just call handleScroll again?
	}

	export async function scrollToIndex(index: number, opts: ScrollToOptions) {
		const itemsDelta = index - start
		const _itemHeight = itemHeight || averageHeight
		const distance = itemsDelta * _itemHeight

		console.warn('scrolling viewport')
		viewport?.scrollTo({
			left: 0,
			top: (viewport?.scrollTop ?? 0) + distance,
			behavior: 'smooth',
			...opts,
		})
	}

	// Invalidate heightmap if any of these values change
	$effect(() => {
		items
		itemHeight
		viewportHeight
		if (mounted) {
			// Untrack this so that state read *within* refresh does not cause it to run again
			untrack(() => refresh(items, viewportHeight, itemHeight))
		}
	})

	// trigger initial refresh
	onMount(() => {
		rows = contents?.getElementsByTagName('svelte-virtual-list-row') as HTMLCollectionOf<HTMLElement>
		mounted = true
	})
</script>

<svelte-virtual-list-viewport
	bind:this={viewport}
	bind:offsetHeight={viewportHeight}
	onscroll={handleScroll}
	style:height
	tabindex="-1"
>
	<svelte-virtual-list-contents
		bind:this={contents}
		style:padding-top="{paddingTop}px"
		style:padding-bottom="{paddingBottom}px"
	>
		{#each visible as row (row.index)}
			<svelte-virtual-list-row>
				{@render children({ item: row.data })}
			</svelte-virtual-list-row>
		{/each}
	</svelte-virtual-list-contents>
</svelte-virtual-list-viewport>

<style>
	svelte-virtual-list-viewport {
		position: relative;
		overflow-y: auto;
		-webkit-overflow-scrolling: touch;
		display: block;
	}

	svelte-virtual-list-contents,
	svelte-virtual-list-row {
		display: block;
	}

	svelte-virtual-list-row {
		overflow: hidden;
	}
</style>
