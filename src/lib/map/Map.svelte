<script>
	import { select, geoPath, geoMercator } from 'd3';
	import geojson from '../../../static/zuerich.json'
	import population from '../../../static/bevoelkerung_proplz.json'
	import { onMount, afterUpdate, createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	export let zipCode;
	export let highlight;

	let svg;
	let width = 0;
	const height = 500;

	afterUpdate(() => {
		svg.selectAll('path').selectAll(function() {
			const currentZip = parseInt(select(this).attr('data-zip'))
			select(this).attr('fill', getFillColor(currentZip))
		})
	})

	function getFillColor(zip) {
		if (zip === zipCode) { return '#000'; }
		if (highlight.indexOf(`${zip}`) >= 0) {
			return '#e74c3c'
		}
		return '#fff'
	}

	onMount(() => {
		width = Math.min(500, window.innerWidth - 24)
		svg = select(document.getElementById('zuerich-map'))
		const projection = geoMercator()
		const path = geoPath().projection(projection)
		projection.fitSize([width,height],geojson);

		for(const feature of geojson.features) {
			const populationCount = population.filter(p => p.plz === `${feature.properties.postleitzahl}`).reduce((sum, pop) => sum + pop.anzahl, 0)

			svg.append("path")
				.attr("d", path(feature))
				.attr('data-zip', feature.properties.postleitzahl)
				.attr('data-population', populationCount)
				.attr('stroke', '#c0392b')
				.attr('fill', getFillColor(feature.properties.postleitzahl))
				.on('mouseover', function() {
					select(this).attr('fill', getFillColor(feature.properties.postleitzahl))
				})
				.on('mouseout', function() {
					select(this).attr('fill', getFillColor(feature.properties.postleitzahl))
				})
				.on('click', function()  {
					dispatch('zipCodeChange', select(this).attr('data-zip'))
				})
				.append('text')
				.attr('dominant-baseline', 'middle')
				.attr('text-anchor', 'middle')
				.attr('x', '0')
				.attr('y', '0').text(feature.properties.postleitzahl).attr('stroke', '#fff')
		}
	})
</script>

<div class='text-3xl font-extrabold text-gray-500'>{zipCode}</div>

<svg id='zuerich-map' width='500' height='500' style='max-width: calc(100vw - 24px); margin: 0 24px;'></svg>

<style>
</style>
