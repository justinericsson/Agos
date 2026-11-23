<script lang="ts">
	import { Map as MapLibre, setWorkerUrl } from 'maplibre-gl';
	import { onDestroy, onMount } from 'svelte';
	import 'maplibre-gl/dist/maplibre-gl.css';
	import workerUrl from 'maplibre-gl/dist/maplibre-gl-worker.mjs?worker&url';

	setWorkerUrl(workerUrl);

	import countryData from '#lib/assets/geojson/country/lowres/country.0.001.json';

	let hoveredName = $state<string | null>(null);

	let mapContainer: HTMLDivElement;
	let map: MapLibre;

	const emptyFeatureCollection = { type: 'FeatureCollection', features: [] as object[] };

	function bindLayerEvents(layerId: string, sourceId: string, nameProp: string) {
		if (!map) return;

		let hoveredId: string | number | undefined;

		map.on('mousemove', layerId, (e) => {
			if (!map || !e.features?.length) return;

			map.getCanvas().style.cursor = 'pointer';
			const feat = e.features[0];
			hoveredName = feat.properties[nameProp] as string;

			if (hoveredId !== undefined) {
				map.setFeatureState({ source: sourceId, id: hoveredId }, { hover: false });
			}
			hoveredId = feat.id;
			map.setFeatureState({ source: sourceId, id: hoveredId }, { hover: true });
		});

		map.on('mouseleave', layerId, () => {
			if (!map) return;

			map.getCanvas().style.cursor = '';
			hoveredName = null;
			if (hoveredId !== undefined) {
				map.setFeatureState({ source: sourceId, id: hoveredId }, { hover: false });
				hoveredId = undefined;
			}
		});
	}

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

			bindLayerEvents('regions-fill', 'regions', 'adm1_en');
		});
	});

	onDestroy(() => {
		if (map) map.remove();
	});
</script>

<div class="flex h-dvh flex-col items-stretch">
	<h1 class="p-2 text-center font-bold">{hoveredName ?? 'Philippines'}</h1>
	<div bind:this={mapContainer} class="basis-full"></div>
</div>
