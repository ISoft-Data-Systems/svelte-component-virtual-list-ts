<script lang="ts">
	import VirtualList from '$lib/VirtualList.svelte';
	const generateThings = (length: number) =>
		Array.from({ length }, (_, index) => ({ number: index, name: 'test' }));
</script>

<div>
	<h1>svelte-virtual-list-ts</h1>
	<h2>Test with 5 Items</h2>
	<div style="height: 200px;">
		<VirtualList items={generateThings(5)} >
			{#snippet children({ item })}
						<!-- this will be rendered for each currently visible item -->
				<p>{item.number}: {item.name}</p>
								{/snippet}
				</VirtualList>
	</div>
	<h2>Test with 100 Items</h2>
	<div style="height: 200px;">
		<VirtualList items={generateThings(100)} >
			{#snippet children({ item })}
						<!-- this will be rendered for each currently visible item -->
				<p>{item.number}: {item.name}</p>
								{/snippet}
				</VirtualList>
	</div>
	<h2>Test with 10.000 Items</h2>
	<div style="height: 200px;">
		<VirtualList items={generateThings(10000)} >
			{#snippet children({ item })}
						<!-- this will be rendered for each currently visible item -->
				<p>{item.number}: {item.name}</p>
								{/snippet}
				</VirtualList>
	</div>
</div>
