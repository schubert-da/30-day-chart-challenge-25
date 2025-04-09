<script>
	import { onMount } from 'svelte';
	import * as d3 from 'd3';
	import { gsap } from 'gsap';
	import { ScrollToPlugin } from 'gsap/dist/ScrollToPlugin.js';

	gsap.registerPlugin(ScrollToPlugin);

	const data = [
		{
			name: 'Lantern Waste',
			areaSqMi: 1200,
			description: 'Dense, magical forest, sparsely populated. First place Lucy enters.'
		},
		{
			name: 'Great Eastern Forest',
			areaSqMi: 2500,
			description: 'Vast wilderness with deep woods and creatures. Borders the Eastern Sea.'
		},
		{
			name: 'Cair Paravel & Surroundings',
			areaSqMi: 800,
			description:
				'Coastal royal capital and its grounds. Includes castle, port, and nearby settlements.'
		},
		{
			name: 'Beaversdam',
			areaSqMi: 300,
			description: 'River valley with forest and lakes; inhabited by talking animals.'
		},
		{
			name: 'Stone Table Region',
			areaSqMi: 600,
			description: 'Central, hilly area considered sacred. Rocky and less inhabited.'
		},
		{
			name: 'The Dancing Lawn',
			areaSqMi: 200,
			description: 'Grassy glade used for gatherings. Symbolically important.'
		},
		{
			name: 'Western Wild',
			areaSqMi: 1000,
			description: 'Remote mountains and highlands, less traveled and wild.'
		},
		{
			name: 'Northern Moors & Ettinsmoor Foothills',
			areaSqMi: 1500,
			description: 'Rocky, colder terrain, home to giants nearby.'
		}
	];

	$: fontScale = layout
		? d3
				.scaleQuantize()
				.domain(d3.extent(layout, (d) => d.value))
				.range([0.75, 0.85, 0.9, 1, 1.1, 1.4])
		: {};

	let treemapHeight, treemapWidth;

	let svgEl;
	let root = null;

	function createTreemap() {
		if (!treemapWidth || !treemapHeight) return;
		const width = treemapWidth;
		const height = treemapHeight;

		const root = d3.hierarchy({ children: data }).sum((d) => d.areaSqMi);
		d3.treemap().size([width, height]).padding(8)(root);

		console.log(root.leaves());
		layout = root.leaves().map((node, index) => ({
			x: node.x0,
			y: node.y0,
			width: node.x1 - node.x0,
			height: node.y1 - node.y0,
			name: node.data.name,
			description: node.data.description,
			fill: '#CDCBC9',
			value: node.value
		}));

		layout = layout.sort((a, b) => b.value - a.value);
	}

	$: layout = null;

	onMount(() => {
		createTreemap;
	});

	$: if (treemapWidth && treemapHeight) {
		createTreemap();
	}

	function scrollToHovered(tile) {
		const trackContainerElement = document.querySelector('.track-container');
		const tileId = formatId(tile);
		console.log('tileId', tileId);

		gsap.to(trackContainerElement, {
			duration: 0.8,
			scrollTo: { y: `#${tileId}`, offsetY: 200 },
			ease: 'back.out(0.8)'
		});

		gsap.to(`.element:not(#${tileId})`, {
			duration: 0.2,
			opacity: 0.75,
			ease: 'power2.out'
		});

		gsap.to(`#${tileId}`, {
			duration: 0.2,
			opacity: 1,
			ease: 'power2.out'
		});
	}

	function revertScroll() {
		gsap.set(`.element`, {
			opacity: 1,
			ease: 'power2.out'
		});
	}

	function formatId(name) {
		return name.replaceAll('&', '').replaceAll(/\s+/g, '-').toLowerCase();
	}
</script>

<div class="viz-container mx-auto flex h-150 w-[1200px] flex-row gap-1 px-[32px] py-8 text-gray-50">
	<div
		class="treemap bg-gray-[#252A30] relative h-full w-[75%]"
		bind:clientHeight={treemapHeight}
		bind:clientWidth={treemapWidth}
	>
		<div class="viz relative h-full w-full bg-[#252A30] font-sans text-[#222]">
			{#each layout as region}
				<!-- svelte-ignore a11y_no_static_element_interactions -->
				<div
					class="absolute overflow-hidden rounded-md border border-white p-2 hover:!bg-[#DC5D5D]"
					style="
			  top: {region.y}px;
			  left: {region.x}px;
			  width: {region.width}px;
			  height: {region.height}px;
			  background-color: {region.fill};
			  --font-scale: {fontScale(region.value)};"
					on:mouseenter={() => {
						scrollToHovered(region.name);
					}}
					on:mouseleave={() => {
						revertScroll();
					}}
				>
					<div class="region pointer-events-none text-base leading-tight font-bold">
						{region.name}
					</div>
					<div class="value pointer-events-none text-sm">
						{region.value}
					</div>
				</div>
			{/each}
		</div>
	</div>

	<div class="track-container bg-gray-[#252A30] w-[25%] overflow-auto">
		<div class="track flex flex-col items-center justify-center gap-1">
			{#each layout as region, index}
				<div
					id={formatId(region.name)}
					class="element flex w-full flex-col items-center justify-between gap-2 rounded-md bg-gray-700 px-4 py-3 text-gray-50"
				>
					<div
						class="title-row font-semibol mb-2 flex w-full items-start gap-2 text-[1.5rem]"
						on:hover
					>
						<span>{index + 1}.</span>
						<div class="title">{region.name}</div>
					</div>
					<div class="viz-row mb-2 flex w-full flex-row justify-between gap-2">
						<div class="mini-viz flex h-25 w-25 items-center justify-center bg-gray-500">
							<div
								class="rect border-2 bg-[#DC5D5D]"
								style="width: {(region.width * 100) / layout[0].width}%; height: {(region.height *
									100) /
									layout[0].height}%"
							></div>
						</div>
						<div class="value mt-2 flex flex-col items-end gap-0">
							<span class="text-[2rem] font-medium"> {region.value}</span>
							<span>square miles</span>
						</div>
					</div>
					<div class="description">{region.description}</div>
				</div>
			{/each}
		</div>
	</div>
</div>

<style>
	.viz .region {
		margin-bottom: 0px;

		font-size: calc(1.5rem * var(--font-scale));
		line-height: 1.1;
	}

	.viz .value {
		font-size: calc(1.5rem * var(--font-scale));
	}
</style>
