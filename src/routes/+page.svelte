<script>
	import jsonData from '$lib/data/ne_50m_admin_countries.geo.json';
	// import testData from '$lib/data/test.json';
	import {onMount} from 'svelte';
	import {geoMercator, geoOrthographic, geoPath, select, zoom} from 'd3';

	const data = jsonData;
	let countyNames = [];
	let countyName = '';
	let streak = 0;

	let baseSvg;
	let zoomResult;
	let width, height;
	let geoGenerator;
	// let projection = d3.geoOrthographic()
	let projection = geoMercator()

	function mouseclickLogic(feature) {
		let guessText
		if (countyName === feature.properties.sovereignt) {
			streak += 1
			guessText = "Correct!, ";
		} else {
			streak -= 1
			guessText = "Incorrect, ";
		}
		streak = Math.max(streak, 0);
		alert(guessText + countyName);
		countyName = getNewCountry();
	}

	function getNewCountry() {
		return countyNames[Math.floor(Math.random() * countyNames.length)]
	}

	$: {
		projection.fitSize([width, height], data);
		geoGenerator = geoPath().projection(projection);
	}

	onMount(() => {
		for (let i = 0; i < data.features.length; i++) {
			countyNames = [...countyNames, data.features[i].properties.sovereignt]
		}
		countyName = getNewCountry();
		select(baseSvg).call(zoom().on("zoom", (e) => {zoomResult = e.transform;}));
	});
</script>

<svelte:window bind:innerWidth={width} bind:innerHeight={height} ></svelte:window>

<div class="flex justify-center">
	<div class="fixed bg-yellow-400/80 rounded-xl px-2 py-1 mt-2 text-center pointer-events-none text-black-900 text-lg">
		<p class="">Find: {countyName}</p>
		<p class="">Current streak: {streak}</p>
	</div>
</div>
<svg {width} {height} bind:this={baseSvg} class="bg-blue-300 scroll">
	<g transform={zoomResult}>
		{#each data.features as feature}
			<path 
				class="fill-green-500 hover:fill-green-600/90" 
				d={!geoGenerator(feature).includes("NaN") ? geoGenerator(feature) : ""} 
				on:click={() => {mouseclickLogic(feature)}}>
			</path>
		{/each}
	</g>
</svg>

<style>
	:root::-webkit-scrollbar {
		display: none;
		width: 0;
		height: 0;
	}
</style>