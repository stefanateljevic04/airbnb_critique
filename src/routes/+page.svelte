<script>
	import { onMount } from 'svelte';
	import { csv } from 'd3';
	import ListingScatter from '#lib/ListingScatter.svelte';

	let listings = $state([]);
	let loading = $state(true);
	let error = $state('');
	const numericColumns = [
		'latitude', 'longitude', 'price', 'minimum_nights', 'number_of_reviews',
		'reviews_per_month', 'availability_365', 'number_of_reviews_ltm'
	];

	onMount(() => {
		let active = true;
		csv('/airbnb_boston_cambridge.csv', (row) => {
			for (const column of numericColumns) row[column] = Number(row[column]);
			return row;
		})
			.then((data) => { if (active) listings = data; })
			.catch(() => { if (active) error = 'Unable to load listings. Please reload the page.'; })
			.finally(() => { if (active) loading = false; });
		return () => { active = false; };
	});
</script>

<svelte:head>
	<title>Airbnb listing comparisons</title>
	<meta name="description" content="Compare Boston and Cambridge Airbnb prices, reviews, and room types." />
</svelte:head>

<main>
	<h1>Boston & Cambridge Airbnb listings</h1>
	<p>Compare nightly prices with total reviews. Point area represents reviews per month.</p>
	{#if loading}
		<p role="status">Loading listings…</p>
	{:else if error}
		<p role="alert">{error}</p>
	{:else}
		<ListingScatter {listings} />
	{/if}
</main>

<style>
	:global(body) { margin: 0; background: #fff; color: #222; font-family: system-ui, sans-serif; }
	main { max-width: 1100px; margin: 0 auto; padding: 2rem 1rem; }
	h1 { font-size: clamp(1.5rem, 4vw, 2rem); }
</style>
