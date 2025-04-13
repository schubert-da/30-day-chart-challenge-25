<script>
	import { gsap } from 'gsap';

	const data = [
		{
			name: 'insights',
			max: 550,
			values: [410, 300, 228, 400, 300, 250]
		},
		{
			name: 'analytics',
			max: 550,
			values: [200, 400, 380, 450, 375, 325]
		}
	];

	$: currentCategory = 'insights';
	$: currentData = data.find((d) => d.name === currentCategory);
	$: buttonColor = '#923549';

	$: newCategory = 'analytics';

	let chartElement = null;

	function animateBars() {
		const bars = chartElement.querySelectorAll('.bar');
		const labels = chartElement.querySelectorAll('.bar-label');
		const labelText = chartElement.querySelectorAll('.bar-label span');

		let newData = data.find((d) => d.name === newCategory);
		newCategory = newCategory === 'insights' ? 'analytics' : 'insights';
		buttonColor = buttonColor === '#923549' ? '#98bac1' : '#923549';

		const timeline = gsap.timeline({
			paused: true
		});

		// make bars full height
		timeline.to(bars, {
			height: `100%`,
			duration: 0.5,
			ease: 'power3.out',
			stagger: 0.3
		});

		// decrease border width to 0
		timeline.to(
			bars,
			{
				borderWidth: 0,
				duration: 0.25,
				ease: 'power3.out',
				stagger: 0.3
			},
			'<'
		);

		// remove gap between bars
		timeline.to(
			chartElement,
			{
				gap: 0,
				duration: 0.25,
				delay: 0.5,
				ease: 'power2.out'
			},
			'<'
		);

		// change the anchor of the bars to 'top'
		timeline.set(bars, {
			top: '0',
			bottom: 'unset',
			duration: 0
		});

		// change the labels data and the color
		timeline.set(labels, {
			top: (i) => 100 - (newData.values[i] * 100) / newData.max + 2 + '%',
			duration: 0
		});

		timeline.set(labelText, {
			color: '#415458',
			duration: 0
		});

		// add the gaps back in
		timeline.to(
			chartElement,
			{
				gap: 6,
				duration: 0.25,
				delay: 0.75,
				ease: 'power2.out'
			},
			'<'
		);

		// change the height of the bars to the new data
		timeline.to(bars, {
			height: (i) => 100 - (newData.values[i] * 100) / newData.max + '%',
			borderBottomWidth: 6,
			duration: 0.5,
			ease: 'power3.out',
			delay: 0,
			stagger: 0.3
		});

		timeline.restart();
	}
</script>

<div
	class="viz-container mx-20 flex h-[700px] w-[1000px] flex-col gap-4 px-[32px] pt-6 pb-10 text-[#222]"
>
	<div class="text-content w-full">
		<h2 class="chart-title font-display mb-2 text-[2.5rem] leading-[1.2] text-[#222]">
			Leveraging insights and things
		</h2>

		<p class="chart-description max-w-[65ch] text-[1.125rem] leading-[1.4] text-[#222]">
			A <strong>fake dataset</strong> that documents the trivial task of harnessing cross-dimensional
			value narratives through real-time intelligence orchestration and contextual synergy activation.
			Thanks ChatGPT.
		</p>
	</div>

	<button
		class="w-fit cursor-pointer rounded-md border bg-white px-4 py-2 font-semibold text-white"
		style="background-color: {buttonColor}"
		on:click={() => animateBars()}>Toggle dataset</button
	>

	<div
		class="chart-container h-full w-full rounded-2xl border border-[#666] bg-[#efefef] p-3 pb-[2rem] shadow-lg"
	>
		<div
			class="chart flex h-full flex-row items-center justify-center gap-[6px] overflow-hidden rounded-xl border-1"
			bind:this={chartElement}
		>
			{#each currentData.values as value}
				<div class="bar-container relative h-full w-full overflow-hidden bg-[#98bac1]">
					<div
						class="bar bar absolute bottom-0 left-0 z-20 w-full border-t-6 border-[#efefef] bg-[#923549]"
						style="height: {(value * 100) / currentData.max}%;"
					></div>

					<div
						class="bar-label pointer-none: absolute top-0 left-0 z-10 flex w-full items-start justify-center bg-transparent pt-2"
						style="height: {100 - (value * 100) / currentData.max}%;"
					>
						<span class="text-[2rem] font-bold text-[#923549]">{value}</span>
					</div>
				</div>
			{/each}
		</div>
		<div class="x-axis relative flex flex-row">
			{#each currentData.values as value, i}
				<div
					class="x-axis-label flex w-full items-center justify-center py-1 text-center text-[20px] font-bold text-[#5F494E]"
				>
					{2020 + i}
				</div>
			{/each}
		</div>
	</div>
</div>

<style>
	.bar {
		box-shadow:
			rgba(0, 0, 0, 0.19) 0px 10px 20px,
			rgba(0, 0, 0, 0.19) 0px -10px 20px,
			rgba(0, 0, 0, 0.23) 0px 6px 6px;
	}
</style>
