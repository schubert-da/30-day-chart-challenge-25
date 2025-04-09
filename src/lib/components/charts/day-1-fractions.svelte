<script>
	import { gsap } from 'gsap';
	import FractionsMorning from '$lib/assets/images/fractions-morning.jpg';
	import FractionsEvening from '$lib/assets/images/fractions-evening.jpg';
	import FractionsEvening2 from '$lib/assets/images/fractions-evening-2.jpg';

	const data = [
		{
			name: 'Morning',
			content:
				'Nearly half of all respondents <strong>chose the morning as their favorite</strong> time to work. They highlighted feeling the most alert and motivated at the start of the day, often using the quiet hours to tackle complex tasks with clarity.',
			value: 0.45,
			height: 115,
			image: [FractionsMorning]
		},
		{
			name: 'Afternoon',
			content:
				'Only a small portion of participants preferred working in the afternoon. Those who did <strong>appreciated having time to ease into the day</strong>.',
			value: 0.15,
			height: 180
		},
		{
			name: 'Evening',
			content:
				'A surprising <strong>40%  said they work best in the evening</strong>, describing a boost in creativity and fewer interruptions later in the day.',
			value: 0.4,
			height: 115,
			image: [FractionsEvening, FractionsEvening2]
		}
	];

	let hoverTimeline = gsap.timeline({ paused: true });

	function playHoverAnimation(event, scaleFactor) {
		let oldScale = 100 / scaleFactor + 0.08;
		console.log('oldScale', oldScale);

		gsap.set('.bar-content', { opacity: 0 });
		const barContentElements = event.target.querySelector('.bar-content');

		hoverTimeline = gsap.timeline().fromTo(
			barContentElements,
			{ opacity: 0, scaleY: oldScale, scaleX: 0.975 },
			{
				opacity: 1,
				scaleY: 1,
				scaleX: 1,
				duration: 0.45,
				ease: 'power2.in'
			}
		);

		if (hoverTimeline.isActive() === false) {
			console.log('hoverTimeline is not active, restarting...');
			hoverTimeline.restart();
		}
	}

	function reverseHoverAnimation(event) {
		const barContentElements = event.target.querySelector('.bar-content');
		gsap.to(barContentElements, {
			opacity: 0,
			duration: 0.1,
			ease: 'power2.out'
		});
	}
</script>

<div
	class="viz-container mx-20 flex h-150 w-[800px] flex-row gap-8 bg-gray-800 px-[32px] py-8 text-gray-50"
>
	<div class="chart-container w-[33%]">
		<div class="chart h-full w-50 divide-y divide-gray-200 border-2 border-gray-200">
			{#each data as row, index}
				<div class="bar-container relative p-2" style="height: {row.value * 100}%">
					<!-- svelte-ignore a11y_no_static_element_interactions -->
					<!-- svelte-ignore a11y_mouse_events_have_key_events -->
					<div
						class="bar relative h-full text-gray-50"
						on:mouseenter={(evt) => {
							playHoverAnimation(evt, row.height);
						}}
						on:mouseleave={reverseHoverAnimation}
					>
						<h3 class="title pointer-events-none relative z-20 text-[32px] uppercase">
							{row.name}
						</h3>
						<p class="value pointer-events-none relative z-20">
							{row.value * 100}%
							{#if index === 0}
								<span> of <span class="font-semibold">respondents</span> </span>
							{/if}
						</p>

						<div class="bar-content shadow" style="height: {row.height}%">
							<h3
								class="title pointer-events-none relative z-20 text-[32px] text-transparent uppercase"
							>
								{row.name}
							</h3>

							<p class="pointer-events-none w-[40ch] text-[18px] text-[#444]">
								{@html row.content}
							</p>

							{#if row.image}
								<div
									class="image-container flex max-h-full flex-row gap-1"
									style="max-width: {180 * row.image.length}px"
								>
									{#each row.image as image}
										<img
											class="max-[100%] pointer-events-none rounded-md object-contain"
											style="max-width: {100 / row.image.length}%"
											src={image}
											alt="placeholder since there are no insights from this data"
										/>
									{/each}
								</div>
							{/if}
						</div>
					</div>
				</div>
			{/each}
		</div>
	</div>

	<div class="text-content w-1/2">
		<h2 class="chart-title font-display mb-4 text-[48px] leading-[1.2]">When Do We Work?</h2>

		<p class="chart-description text-[20px] leading-[1.4] text-gray-200">
			<strong>Disclaimer</strong>: This data is 100% made up — no respondents were harmed (or even
			questioned) in the making of this chart. This was a fun exercise to practice interaction
			design.

			<br /><br />
			The (make believe) data highlights a
			<strong class="text-white">clear preference for working in the morning</strong>, with nearly
			half of respondents identifying it as their most productive time of day.
		</p>
	</div>
</div>

<style>
	.bar-content {
		position: absolute;
		top: 50%;
		left: -16px;
		transform: translate(0, -50%);
		display: flex;
		flex-direction: row;
		align-items: flex-start;
		gap: 32px;
		opacity: 0;
		z-index: 1;

		width: 750px;
		height: 120%;
		padding: 16px;
		background-color: #fefefe;
		border-radius: 8px;
		z-index: 10;
		pointer-events: none;

		color: #222;
	}

	.bar:hover .bar-content {
		/* opacity: 1; */
	}

	.bar:hover > h3,
	.bar:hover > p {
		color: #222;
	}
</style>
