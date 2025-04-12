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
		d3.treemap().size([width, height]).padding(4)(root);

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
	$: vizElement = null;

	onMount(() => {
		createTreemap();
	});

	$: if (treemapWidth && treemapHeight) {
		createTreemap();
	}

	$: if (layout) {
		animateTiles();
	}

	let hideRanks = false;

	function animateTiles() {
		console.log('Animating tiles...');
		layout.forEach((tile, index) => {
			const tileEl = tile.element;

			if (tileEl) {
				const tileArea = tile.width * tile.height;

				const timeline = gsap.timeline({
					paused: true
				});

				timeline.set(tileEl.querySelector('.value'), {
					opacity: 0
				});

				const growTween = timeline.fromTo(
					tileEl,
					{
						x: '0%',
						y: '0%',
						left: '0%',
						bottom: '0%',
						width: 100,
						height: 100
					},
					{
						width: Math.sqrt(tileArea) * 1.5,
						height: Math.sqrt(tileArea) * 1.5,
						duration: 1.25,
						ease: 'back.out(1.05)',
						delay: index * 0.2
					}
				);

				timeline.to(
					tileEl.querySelector('.rank'),
					{
						width: 0,
						delay: 1 / (index + 5) + 0.1,
						duration: 0.1
					},
					'<+3'
				);

				const moveTween = timeline.to(
					tileEl,
					{
						x: tile.x,
						y: tile.y,
						width: tile.width,
						height: tile.height,
						left: '0%',
						top: '0%',
						duration: 1.5,
						ease: 'power3.in'
					},
					'<-0.4'
				);

				timeline.to(
					tileEl.querySelector('.value'),
					{
						opacity: 1,
						duration: 0.5
					},
					'<+1.5'
				);

				timeline.to(vizElement, { backgroundColor: '#252A30' }, '<+1.5');

				if (index === layout.length - 1) {
					layout.forEach((tile, index) => {
						const tileEl = tile.element;
						timeline.to(
							tileEl,
							{
								borderColor: 'white'
							},
							index === 0 ? '<-1' : '<'
						);
					});
				}

				timeline.restart();
			}
		});
	}

	function formatId(name) {
		return name.replaceAll('&', '').replaceAll(/\s+/g, '-').toLowerCase();
	}
</script>

<div class="viz-container mx-auto flex h-150 w-[1400px] flex-row gap-8 px-[32px] py-8 text-gray-50">
	<div class="title mb-[4rem] flex flex-col items-end justify-center gap-2 text-right text-[#222]">
		<h2 class="mb-2 text-right text-[3rem]">The Geographical Survey of Narnia</h2>
		<p class="subtitle max-w-[32ch] text-right text-[1.2rem] text-pretty">
			A <em>totally real</em> survey on the land areas of the different territories in the kingdom of
			Narnia. I personally asked Aslan to convert to square miles for me, so you can trust the data.
		</p>
	</div>
	<div
		class="treemap bg-gray-[#252A30] relative h-full w-full"
		bind:clientHeight={treemapHeight}
		bind:clientWidth={treemapWidth}
	>
		<div
			class="viz relative h-full w-full bg-transparent font-sans text-[#222]"
			bind:this={vizElement}
		>
			{#each layout as region, index}
				<!-- svelte-ignore a11y_no_static_element_interactions -->
				<div
					class="absolute overflow-hidden rounded-md border-2 border-[#252A30] bg-[#DC5D5D] p-2"
					id={formatId(region.name)}
					style="
			  width: {region.width}px;
			  height: {region.height}px;
			  --font-scale: {fontScale(region.value)};"
					bind:this={region.element}
				>
					<div
						class="region ali pointer-events-none flex flex-row gap-1 text-base leading-tight font-bold"
					>
						<span class="rank min-w inline-block overflow-hidden">{index + 1} </span>
						{region.name}
					</div>
					<div class="value pointer-events-none text-sm">
						{region.value}
						{#if region.value > 1000}
							square miles
						{/if}
					</div>
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
