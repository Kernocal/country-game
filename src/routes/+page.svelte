<script>
	import jsonData from '$lib/data/ne_50m_admin_countries.geo.json';
	// import testData from '$lib/data/test.json';
	import {onMount} from 'svelte';
	import {fly, fade} from 'svelte/transition';
	import {select, zoom, drag, geoMercator, geoOrthographic, geoPath} from 'd3';

	const data = jsonData;
	let countyNames = [];
	let countyName = '';
	let streak = 0;
	let guessText = '';
	let cursor = {x: 0, y:0};

	let baseSvg;
	let zoomResult;
	let width, height;
	let geoGenerator;
	// let projection = geoOrthographic()
	let projection = geoMercator()

	function mouseclickLogic(e, feature) {
		cursor.x = e.x;
		cursor.y = e.y;
		if (countyName === feature.properties.sovereignt) {
			streak += 1
			guessText = "Correct, " + feature.properties.sovereignt;
			getNewCountry();
		} else {
			streak -= 1
			guessText = "Incorrect, " + feature.properties.sovereignt;
		}
		streak = Math.max(streak, 0);
	}

	function getNewCountry() {
		countyName = countyNames[Math.floor(Math.random() * countyNames.length)]
	}

	$: {
		projection.fitSize([width, height], data);
		geoGenerator = geoPath().projection(projection);
	}

	onMount(() => {
		for (let i = 0; i < data.features.length; i++) {
			countyNames = [...countyNames, data.features[i].properties.sovereignt]
		}
		getNewCountry();
		select(baseSvg).call(zoom().on("zoom", (e) => {
			zoomResult = e.transform;
		}));
	});
</script>

<svelte:window bind:innerWidth={width} bind:innerHeight={height} ></svelte:window>

<div class="flex justify-center items-start">
	<div class="fixed bg-yellow-400/80 rounded-xl px-2 py-1 mt-2 text-center pointer-events-none text-xl font-medium">
		<p class="">Find: <span class="text-blue-500">{countyName}</span></p>
		<p class="">Current streak: {streak}</p>
	</div>
</div>
<button 
	class="fixed bg-yellow-400/80 rounded-xl px-2 py-1 mt-2 font-medium text-red-500" 
	title="Get new country." 
	on:click={() => {streak = 0; getNewCountry();}}>
	Give up.
</button>

{#if guessText !== ''}
	<p 
		class={"fixed rounded-full p-2 text-lg font-bold text-white " + (guessText.includes("Incorrect") ? "bg-red-500" : "bg-purple-500")}
		in:fly={{duration: 2500}}
		out:fly on:introend={() => {guessText = ''}}
		style="left: {cursor.x}px; top: {cursor.y}px;">
		{guessText}
	</p>
{/if}

<svg {width} {height} bind:this={baseSvg} class="bg-blue-300 scroll">
	<g transform={zoomResult}>
		{#each data.features as feature}
			<path 
				class="fill-green-600 stroke-light-100 stroke-01 hover:fill-green-500" 
				d={!geoGenerator(feature).includes("NaN") ? geoGenerator(feature) : ""}
				on:click={(e) => {mouseclickLogic(e, feature)}}>
			</path>
		{/each}
	</g>
</svg>


<style>
	.stroke-01 {
		stroke-width: 0.1;
	}

	:root::-webkit-scrollbar {
		display: none;
		width: 0;
		height: 0;
	}
</style>