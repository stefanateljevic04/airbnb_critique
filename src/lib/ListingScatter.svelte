<script>
	import { max, scaleLinear, scaleOrdinal, scaleSqrt, format, symbol, symbolCircle, symbolSquare, symbolTriangle } from 'd3';

	let { listings = [] } = $props();
	const width = 1000;
	const height = 600;
	const margin = { top: 24, right: 24, bottom: 64, left: 84 };
	const roomTypes = ['Entire home/apt', 'Private room', 'Hotel room'];
	const color = scaleOrdinal(roomTypes, ['#0072B2', '#D55E00', '#009E73']);
	const shape = scaleOrdinal(roomTypes, [symbolCircle, symbolSquare, symbolTriangle]);
	const shapeName = scaleOrdinal(roomTypes, ['circle', 'square', 'triangle']);
	const markPath = (roomType, area) => symbol().type(shape(roomType)).size(area)();
	let selection = $state({ roomType: null, pinned: false });
	let markFill = $derived(scaleOrdinal(roomTypes, roomTypes.map((roomType) =>
		selection.roomType && selection.roomType !== roomType ? '#999999' : color(roomType))));
	let orderedListings = $derived([...listings].sort((a, b) => {
		const selectedOrder = Number(a.room_type === selection.roomType) - Number(b.room_type === selection.roomType);
		return selectedOrder || b.reviews_per_month - a.reviews_per_month;
	}));
	function previewRoomType(roomType) {
		if (!selection.pinned) selection = { roomType, pinned: false };
	}
	function toggleRoomType(roomType) {
		selection = selection.pinned && selection.roomType === roomType
			? { roomType: null, pinned: false }
			: { roomType, pinned: true };
	}
	const dollars = format('$,.0f');
	let x = $derived(scaleSqrt()
		.domain([0, max(listings, (d) => d.number_of_reviews) || 1]).nice()
		.range([margin.left, width - margin.right]));
	let y = $derived(scaleLinear()
		.domain([0, max(listings, (d) => d.price) || 1]).nice()
		.range([height - margin.bottom, margin.top]));
	// Squared radius (and therefore area) varies linearly with review velocity.
	let radius = $derived(scaleSqrt()
		.domain([0, max(listings, (d) => d.reviews_per_month) || 1])
		.range([5, 12]));
</script>

<figure>
	<div class="legend" aria-label="Room type legend">
		{#each roomTypes as roomType}
			<button type="button" aria-pressed={selection.pinned && selection.roomType === roomType}
				class:highlighted={selection.roomType === roomType}
				onmouseenter={() => previewRoomType(roomType)} onmouseleave={() => previewRoomType(null)}
				onfocus={() => previewRoomType(roomType)} onblur={() => previewRoomType(null)}
				onclick={() => toggleRoomType(roomType)}>
				<svg width="24" height="24" aria-hidden="true"><path d={markPath(roomType, 120)} transform="translate(12, 12)" fill={markFill(roomType)} /></svg>{roomType} ({shapeName(roomType)})
			</button>
		{/each}
		<button type="button" onclick={() => selection = { roomType: null, pinned: false }}>Show all</button>
	</div>
	<p class="interaction-help">Hover over or focus a room type to highlight it. Click to keep it highlighted; click it again or choose Show all to reset. Other groups turn gray, making the selected group's prices and review counts easier to follow while keeping the full dataset in view.</p>
	<div class="chart">
		<svg viewBox="0 0 {width} {height}" role="img" aria-labelledby="chart-title chart-description">
			<title id="chart-title">Nightly price versus number of reviews</title>
			<desc id="chart-description">{listings.length} Airbnb listings. Horizontal position shows total reviews; vertical position shows nightly price in USD. Color and shape show room type: circles for entire homes/apartments, squares for private rooms, and triangles for hotel rooms. Point area shows reviews per month.</desc>
			{#each y.ticks(8) as tick}
				<line class="grid" x1={margin.left} x2={width - margin.right} y1={y(tick)} y2={y(tick)} />
				<text x={margin.left - 12} y={y(tick)} dy="0.35em" text-anchor="end">{dollars(tick)}</text>
			{/each}
			{#each x.ticks(10) as tick}
				<line class="axis" x1={x(tick)} x2={x(tick)} y1={height - margin.bottom} y2={height - margin.bottom + 6} />
				<text x={x(tick)} y={height - margin.bottom + 24} text-anchor="middle">{format(',')(tick)}</text>
			{/each}
			<line class="axis" x1={margin.left} x2={width - margin.right} y1={height - margin.bottom} y2={height - margin.bottom} />
			<line class="axis" x1={margin.left} x2={margin.left} y1={margin.top} y2={height - margin.bottom} />
			{#each orderedListings as listing}
				<path class="mark" transform="translate({x(listing.number_of_reviews)}, {y(listing.price)})" d={markPath(listing.room_type, Math.PI * radius(listing.reviews_per_month) ** 2)} fill={markFill(listing.room_type)}>
					<title>{listing.name} — {listing.room_type}; {listing.number_of_reviews} reviews; {dollars(listing.price)}/night; {listing.reviews_per_month} reviews/month</title>
				</path>
			{/each}
			<text class="label" x={(margin.left + width - margin.right) / 2} y={height - 10} text-anchor="middle">Number of reviews (square-root scale)</text>
			<text class="label" transform="translate(22, {(margin.top + height - margin.bottom) / 2}) rotate(-90)" text-anchor="middle">Nightly price (USD)</text>
		</svg>
	</div>
	<div class="legend size-legend" aria-label="Point size legend">
		<strong>Reviews per month:</strong>
		{#each [0, 5, 15] as value}
			<span><svg width="30" height="30" aria-hidden="true"><circle cx="15" cy="15" r={radius(value)} /></svg>{value}</span>
		{/each}
	</div>
	<figcaption>Each point is one listing. Hover over a point for details. Source: Inside Airbnb, June 2026.</figcaption>
</figure>

<style>
	figure { margin: 1.5rem 0; }
	.chart { overflow-x: auto; }
	.chart > svg { display: block; width: 100%; min-width: 640px; }
	text { fill: #333; font-size: 13px; }
	.label { font-size: 16px; font-weight: 600; }
	.grid { stroke: #e5e5e5; }
	.axis { stroke: #333; }
	.legend { display: flex; flex-wrap: wrap; gap: 1rem; align-items: center; color: #333; }
	.legend span { display: inline-flex; align-items: center; gap: 0.4rem; }
	.legend button { display: inline-flex; align-items: center; gap: 0.4rem; font: inherit; color: #333; background: #fff; border: 1px solid #666; border-radius: 4px; padding: 0.25rem 0.5rem; cursor: pointer; }
	.legend button.highlighted { outline: 2px solid #333; outline-offset: 1px; }
	.legend button:focus-visible { outline: 3px solid #0072B2; outline-offset: 3px; }
	.interaction-help { color: #333; font-size: 0.875rem; }
	.mark, .legend path { stroke: #222; stroke-width: 1; stroke-linejoin: round; }
	.mark { vector-effect: non-scaling-stroke; }
	.size-legend { margin-top: 0.75rem; }
	.size-legend circle { fill: #666; stroke: #222; }
	figcaption { margin-top: 1rem; color: #444; font-size: 0.875rem; }
</style>
