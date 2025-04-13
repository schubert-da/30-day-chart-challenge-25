<script>
	import FractionsChart from '$lib/components/charts/day-01-fractions.svelte';
	import BigOrSmall from '$lib/components/charts/day-04-big-or-small.svelte';
	import Ranking from '$lib/components/charts/day-05-ranking.svelte';
	import Stripes from '$lib/components/charts/day-11-stripes.svelte';
</script>

<main>
	<Stripes></Stripes>
</main>

<style>
	main {
		display: flex;
		align-items: center;
		justify-content: center;

		padding: 3rem;
		background-color: #efefef;
		height: 100vh;
	}
</style>
