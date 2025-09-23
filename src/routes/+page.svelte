<script lang="ts">
	import VirtualList from '$lib/VirtualList.svelte'
	const generateThings = (length: number) => Array.from({ length }, (_, index) => ({ number: index, name: 'test' }))

	let show = $state(false)
</script>

<div>
	<h1>svelte-virtual-list-ts</h1>

	<h2>Test with 10,000 Items</h2>
	<div style="height: 200px;">
		<input
			type="checkbox"
			bind:checked={show}
		/>
		{#if show}
			<VirtualList
				items={generateThings(10000)}
				itemHeight={50}
			>
				{#snippet children({ item })}
					<!-- this will be rendered for each currently visible item -->
					<p>{item.number}: {item.name}</p>
				{/snippet}
			</VirtualList>
		{/if}
	</div>
</div>
