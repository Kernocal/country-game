<script>
	import { onMount } from 'svelte';
	import * as d3 from 'd3';
	import {transition} from 'd3-transition';

	let el
	let countyName
	let countyNames = []
	let testT = [1,2,3]

	function mouseoverLogic(e, d) {
		d3.select(d)
		if (countyName == d.properties.name) {
			alert("Correct!", countyName, d.properties.name)
		}
		countyName = generateName()
	}

	function generateName() {
		return countyNames[Math.floor(Math.random() * countyNames.length)]
	}

	onMount(() => {
		d3.json('ne_50m_admin_countries.geo.json').then(function(bb) {
			for (let i = 0; i < bb.features.length; i++) {
				countyNames = [...countyNames, bb.features[i].properties.sovereignt]
			}
			countyName = generateName()
			let width = window.innerWidth;
			let height = window.innerHeight;
			// let projection = d3.geoOrthographic()
			let projection = d3.geoMercator()
			projection.fitSize([width, height], bb);
			let geoGenerator = d3.geoPath().projection(projection);

			let svg = d3.select("body").append('svg').attr("width", width).attr("height", height);

			svg.append('g').selectAll('path')
			.data(bb.features)
			.join('path')
			.attr("class", "fill-blue hover:fill-red")
			.attr('d', geoGenerator)
			.on("click", mouseoverLogic)

			svg.call(d3.zoom().on("zoom", function () {
    			svg.select("g").attr("transform", d3.zoomTransform(this))
  			}))
		});
	});
</script>
<!-- {#each countyNames as countyName}
	<p class="fill-yellow-500 fixed">{countyName}</p>
{/each} -->
<p class="fill-yellow-500 fixed">Select: {countyName ? countyName : ''}</p>
<div bind:this={el} class="chart"></div>

<style>
	:global(.fill-blue) {
		fill: darkslateblue;
	}
	:global(.hover\:fill-red:hover) {
		fill: red;
	}
</style>