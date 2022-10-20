<script>
	import { select, geoPath, geoMercator } from 'd3';
	import geojson from '../../zuerich.json'
	import population from '../../bevoelkerung_proplz.json'
	import { onMount, afterUpdate, createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	export let zipCode;
	export let zone1;
	export let zone2;
	export let zone3;

	let svg;
	let zipCodeGroup;
	let width = 0;
	const height = 500;

	const restaurantIconWidth = 416, restaurantIconHeight = 512;
	const iconScale = 0.03;

	afterUpdate(() => {
		svg.selectAll('path').selectAll(function() {
			const currentZip = parseInt(select(this).attr('data-zip'))
			if(!currentZip) {
				return
			}

			select(this).attr('fill', getFillColor(currentZip))
				// .attr('stroke-width', currentZip === zipCode ? 3 : 1)
				// .attr('stroke', currentZip === zipCode ? '#333' : '#c0392b')

			if(currentZip===zipCode) {
				select(this).raise()
			}

			if (currentZip === zipCode) {
				const centerX = parseInt(select(this).attr('data-centroid-x'))
				const centerY = parseInt(select(this).attr('data-centroid-y'))
				select(document.getElementById('restaurant-icon-group')).attr('transform', `translate(${centerX - restaurantIconWidth*iconScale/2}, ${centerY - restaurantIconHeight*iconScale/2})`)
				select(document.getElementById('restaurant-icon')).attr('transform', `scale(${iconScale}, ${iconScale})`)
			}
		})
	})

	function getFillColor(zip) {
		if (zone1.indexOf(`${zip}`) >= 0) {
			return '#B91C1C'
		}
		if (zone2.indexOf(`${zip}`) >= 0) {
			return '#F87171'
		}
		if (zone3.indexOf(`${zip}`) >= 0) {
			return '#FEE2E2'
		}
		return '#fff'
	}

	onMount(() => {
		width = Math.min(500, window.innerWidth - 24)
		svg = select(document.getElementById('zuerich-map'))
		zipCodeGroup = select(document.getElementById('zip-code-group'))
		const projection = geoMercator()
		const path = geoPath().projection(projection)
		projection.fitSize([width,height],geojson);

		for(const feature of geojson.features) {
			const populationCount = population.filter(p => p.plz === `${feature.properties.postleitzahl}`).reduce((sum, pop) => sum + pop.anzahl, 0)
			const centroid = path.centroid(feature)
			zipCodeGroup.append("path")
				.attr("d", path(feature))
				.attr('data-zip', feature.properties.postleitzahl)
				.attr('data-population', populationCount)
				.attr('stroke', '#c0392b')
				.attr('data-centroid-x', centroid[0])
				.attr('data-centroid-y', centroid[1])
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
		}
	})
</script>
<svg id='zuerich-map' width='500' height='500' style='max-width: calc(100vw - 24px); margin: 0 24px;'>
	<g id='zip-code-group'></g>
	<g transform='' id='restaurant-icon-group'>
		<path fill="#000" id='restaurant-icon' d="M207.9 15.2c.8 4.7 16.1 94.5 16.1 128.8 0 52.3-27.8 89.6-68.9 104.6L168 486.7c.7 13.7-10.2 25.3-24 25.3H80c-13.7 0-24.7-11.5-24-25.3l12.9-238.1C27.7 233.6 0 196.2 0 144 0 109.6 15.3 19.9 16.1 15.2 19.3-5.1 61.4-5.4 64 16.3v141.2c1.3 3.4 15.1 3.2 16 0 1.4-25.3 7.9-139.2 8-141.8 3.3-20.8 44.7-20.8 47.9 0 .2 2.7 6.6 116.5 8 141.8.9 3.2 14.8 3.4 16 0V16.3c2.6-21.6 44.8-21.4 48-1.1zm119.2 285.7l-15 185.1c-1.2 14 9.9 26 23.9 26h56c13.3 0 24-10.7 24-24V24c0-13.2-10.7-24-24-24-82.5 0-221.4 178.5-64.9 300.9z"></path>
	</g>
</svg>

<style>
</style>
