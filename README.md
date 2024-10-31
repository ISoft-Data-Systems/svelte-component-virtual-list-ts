> [!NOTE]  
> This is a straight fork of this repo https://github.com/crocotoast/svelte-virtual-list-ts/issues just with Svelte 5 added as a peer dependency

# svelte-virtual-list-ts ([demo](https://svelte.dev/repl/3ed4b471023045e68b3b5a181dabb59d?version=3.46.2))

Typescript rewrite of [svelte-virtual-list-ce](https://github.com/gitbreaker222/svelte-virtual-list)\.

All of the VirtualList implementation is stolen from [svelte-virtual-list-ce](https://github.com/gitbreaker222/svelte-virtual-list) and/or [svelte-virtual-list](https://github.com/sveltejs/svelte-virtual-list). I just created a SvelteKit project with typescript and added some types to the source code.
The items prop aswell as the let:item binding is generic. If items has a time the types of let:item will be inferred automatically.

A virtual list component for Svelte apps. Instead of rendering all your data, `<VirtualList>` just renders the bits that are visible, keeping your page nice and light.

## Installation

```bash
npm install svelte-virtual-list-ts
```

## Usage

```html
<script>
	import VirtualList from 'svelte-virtual-list-ts';

	const things = [
		// these can be any values you like
		{ name: 'one', number: 1 },
		{ name: 'two', number: 2 },
		{ name: 'three', number: 3 },
		// ...
		{ name: 'six thousand and ninety-two', number: 6092 }
	];
</script>

<VirtualList items="{things}" let:item>
	<!-- this will be rendered for each currently visible item -->
	<!-- item.number and item.name should be known because they are inferred from the things const-->
	<p>{item.number}: {item.name}</p>
</VirtualList>
```

## `start` and `end`

You can track which rows are visible at any given by binding to the `start` and `end` values:

```html
<VirtualList items="{things}" bind:start bind:end>
	<p>{item.number}: {item.name}</p>
</VirtualList>

<p>showing {start}-{end} of {things.length} rows</p>
```

You can rename them with e.g. `bind:start={a} bind:end={b}`.

## `height`

By default, the `<VirtualList>` component will fill the vertical space of its container. You can specify a different height by passing any CSS length:

```html
<VirtualList height="500px" items="{things}" let:item>
	<p>{item.number}: {item.name}</p>
</VirtualList>
```

## `itemHeight`

You can optimize initial display and scrolling when the height of items is known in advance. This should be a number representing a pixel value.

```html
<VirtualList itemHeight="{48}" items="{things}" let:item>
	<p>{item.number}: {item.name}</p>
</VirtualList>
```

## `scrollToIndex()`

You can jump to anywhere in the list, by calling `scrollToIndex` with one of the list items index.

```html
<script>
	let scrollToIndex;

	function handleButtonClick(event) {
		// ... define your index variable here
		scrollToIndex(index);
	}
</script>

<VirtualList bind:scrollToIndex items="{things}">
	<p>{item.number}: {item.name}</p>
</VirtualList>
```

You can also `export let scrollToIndex` to call it from outside. In this case it should be initialized with `undefined`, to prevent warnings in the logs:

```html
<!-- InnerComponent.svelte -->
<script>
	export let scrollToIndex = undefined;
</script>

<!-- OuterComponent.svelte -->
<script>
	let scrollToIndex;
	function someLogic() {
		scrollToIndex(index);
	}
</script>

<InnerComponent bind:scrollToIndex></InnerComponent>
```

## Configuring webpack

If you're using webpack with [svelte-loader](https://github.com/sveltejs/svelte-loader), make sure that you add `"svelte"` to [`resolve.mainFields`](https://webpack.js.org/configuration/resolve/#resolve-mainfields) in your webpack config. This ensures that webpack imports the uncompiled component (`src/index.html`) rather than the compiled version (`index.mjs`) — this is more efficient.

If you're using Rollup with [rollup-plugin-svelte](https://github.com/rollup/rollup-plugin-svelte), this will happen automatically.

## License

[LIL](https://github.com/sveltejs/svelte-virtual-list/blob/master/LICENSE)
