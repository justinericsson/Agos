<script lang="ts">
	import { Map as MapLibre, setWorkerUrl } from 'maplibre-gl';
	import { onDestroy, onMount } from 'svelte';
	import 'maplibre-gl/dist/maplibre-gl.css';
	import workerUrl from 'maplibre-gl/dist/maplibre-gl-worker.mjs?worker&url';

	setWorkerUrl(workerUrl);

	import countryData from '#lib/assets/geojson/country/lowres/country.0.001.json';

	let mapContainer: HTMLDivElement;
	let map: MapLibre;

	const emptyFeatureCollection = { type: 'FeatureCollection', features: [] as object[] };

	onMount(() => {
		map = new MapLibre({
			container: mapContainer,
			style: {
				version: 8,
				sources: {},
				layers: [
					{ id: 'background', type: 'background', paint: { 'background-color': 'lightblue' } }
				],
				center: [122.0, 12.5],
				zoom: 4.5
			}
		});

		map.on('load', () => {
			if (!map) return;

			// 1. Regions
			map.addSource('regions', { type: 'geojson', data: countryData, generateId: true });
			map.addLayer({
				id: 'regions-fill',
				type: 'fill',
				source: 'regions',
				paint: { 'fill-color': 'white' }
			});
			map.addLayer({
				id: 'regions-line',
				type: 'line',
				source: 'regions',
				paint: { 'line-color': 'black', 'line-width': 1 }
			});

			// 2. Provinces & NCR districts
			map.addSource('provinces', {
				type: 'geojson',
				data: emptyFeatureCollection,
				generateId: true
			});
			map.addLayer({
				id: 'provinces-fill',
				type: 'fill',
				source: 'provinces',
				paint: { 'fill-color': 'white' }
			});
			map.addLayer({
				id: 'provinces-line',
				type: 'line',
				source: 'provinces',
				paint: { 'line-color': 'black', 'line-width': 1 }
			});

			// 3. Cities and municipalities
			map.addSource('cities', { type: 'geojson', data: emptyFeatureCollection, generateId: true });
			map.addLayer({
				id: 'cities-fill',
				type: 'fill',
				source: 'cities',
				paint: { 'fill-color': 'white' }
			});
			map.addLayer({
				id: 'cities-line',
				type: 'line',
				source: 'cities',
				paint: { 'line-color': 'black', 'line-width': 18 }
			});
		});
	});

	onDestroy(() => {
		if (map) map.remove();
	});
</script>

<div bind:this={mapContainer} class="h-dvh"></div>
