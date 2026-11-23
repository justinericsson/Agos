<script lang="ts">
	import { GeoJSONSource, Map as MapLibre, setWorkerUrl, type GeoJSONFeature } from 'maplibre-gl';
	import { onDestroy, onMount } from 'svelte';
	import 'maplibre-gl/dist/maplibre-gl.css';
	import workerUrl from 'maplibre-gl/dist/maplibre-gl-worker.mjs?worker&url';

	import countryData from '#lib/assets/geojson/country/lowres/country.0.001.json';
	import bbox from '@turf/bbox';
	import * as Breadcrumb from '#lib/components/ui/breadcrumb/index.ts';
	import Button from './ui/button/button.svelte';

	let props: { class?: string } = $props();

	type BoundingBox = [[number, number], [number, number]];

	setWorkerUrl(workerUrl);

	const hoverFill = 'oklch(39.1% 0.09 240.876)';
	const defaultFill = 'oklch(92.2% 0 none)';

	const RES = '0.001';
	const geojsonModules = import.meta.glob([
		'#lib/assets/geojson/regions/lowres/*.json',
		'#lib/assets/geojson/provdists/lowres/*.json',
		'#lib/assets/geojson/municities/lowres/*.json'
	]);

	let hoveredName = $state<string | null>(null);
	let selectedRegion = $state<{ code: number; name: string; bounds: BoundingBox } | null>(null);
	let selectedProvince = $state<{ code: number; name: string; bounds: BoundingBox } | null>(null);
	let selectedCity = $state<{ code: number; name: string } | null>(null);
	let loading = $state(false);

	let mapContainer: HTMLDivElement;
	let map: MapLibre;

	const emptyFeatureCollection: GeoJSON.FeatureCollection = {
		type: 'FeatureCollection',
		features: []
	};

	onMount(() => {
		map = new MapLibre({
			container: mapContainer,
			style: {
				version: 8,
				sources: {},
				layers: []
			},
			center: [122.0, 12.5],
			zoom: 4.5,
			minZoom: 4,
			maxZoom: 16,
			attributionControl: false
		});

		map.on('load', () => {
			if (!map) return;

			// 1. Regions
			map.addSource('regions', {
				type: 'geojson',
				data: countryData as GeoJSON.FeatureCollection,
				generateId: true
			});
			map.addLayer({
				id: 'regions-fill',
				type: 'fill',
				source: 'regions',
				paint: {
					'fill-color': [
						'case',
						['boolean', ['feature-state', 'hover'], false],
						hoverFill,
						defaultFill
					],
					'fill-opacity': 1
				}
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
				paint: {
					'fill-color': [
						'case',
						['boolean', ['feature-state', 'hover'], false],
						hoverFill,
						defaultFill
					]
				}
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
				paint: {
					'fill-color': [
						'case',
						['boolean', ['feature-state', 'hover'], false],
						hoverFill,
						defaultFill
					]
				}
			});
			map.addLayer({
				id: 'cities-line',
				type: 'line',
				source: 'cities',
				paint: { 'line-color': 'black', 'line-width': 1 }
			});

			map.addSource('barangays', {
				type: 'geojson',
				data: emptyFeatureCollection,
				generateId: true
			});
			map.addLayer({
				id: 'barangays-fill',
				type: 'fill',
				source: 'barangays',
				paint: {
					'fill-color': [
						'case',
						['boolean', ['feature-state', 'hover'], false],
						hoverFill,
						defaultFill
					]
				}
			});
			map.addLayer({
				id: 'barangays-line',
				type: 'line',
				source: 'barangays',
				paint: { 'line-color': 'black', 'line-width': 1 }
			});

			bindLayerEvents('regions-fill', 'regions', 'adm1_en', (f) => void handleRegionClick(f));
			bindLayerEvents('provinces-fill', 'provinces', 'adm2_en', (f) => void handleProvinceClick(f));
			bindLayerEvents('cities-fill', 'cities', 'adm3_en', (f) => void handleCityClick(f));
			bindLayerEvents('barangays-fill', 'barangays', 'adm4_en', (f) => console.log(f));
		});
	});

	async function loadGeoJson(relativePath: string): Promise<GeoJSON.FeatureCollection> {
		const key = `/src/lib/assets/geojson/${relativePath}`;
		const promise = geojsonModules[key];

		if (!promise) {
			throw new Error(`GeoJSON asset not found at ${key}`);
		}

		loading = true;
		try {
			const mod = await promise();
			return mod.default as GeoJSON.FeatureCollection;
		} finally {
			loading = false;
		}
	}

	function flyToFeature(feature: GeoJSONFeature) {
		if (!map) return;

		const [minLng, minLat, maxLng, maxLat] = bbox(feature);
		map.fitBounds(
			[
				[minLng, minLat],
				[maxLng, maxLat]
			],
			{ padding: 45, duration: 800 }
		);
	}

	async function handleRegionClick(feature: GeoJSONFeature) {
		if (!map) return;

		const code = feature.properties.adm1_psgc as number;
		const name = feature.properties.adm1_en as string;

		const [minLng, minLat, maxLng, maxLat] = bbox(feature);
		const bounds: BoundingBox = [
			[minLng, minLat],
			[maxLng, maxLat]
		];

		selectedRegion = { code, name, bounds };

		// Loads chunk from src/lib/assets/geojson/regions/
		const provData = await loadGeoJson(`regions/lowres/provdists-region-${code}.${RES}.json`);
		await (map.getSource('provinces') as GeoJSONSource).setData(provData);

		// Hide regions completely so borders, coastlines, and click handlers don't bleed through
		map.setLayoutProperty('regions-fill', 'visibility', 'none');
		map.setLayoutProperty('regions-line', 'visibility', 'none');

		// Ensure provinces are visible
		map.setLayoutProperty('provinces-fill', 'visibility', 'visible');
		map.setLayoutProperty('provinces-line', 'visibility', 'visible');

		hoveredName = null;
		flyToFeature(feature);
	}

	async function handleProvinceClick(feature: GeoJSONFeature) {
		if (!map) return;

		const code = feature.properties.adm2_psgc as number;
		const name = feature.properties.adm2_en as string;

		const [minLng, minLat, maxLng, maxLat] = bbox(feature);
		const bounds: BoundingBox = [
			[minLng, minLat],
			[maxLng, maxLat]
		];

		selectedProvince = { code, name, bounds };

		// Loads chunk from src/lib/assets/geojson/provdists/
		const cityData = await loadGeoJson(`provdists/lowres/municities-provdist-${code}.${RES}.json`);
		await (map.getSource('cities') as GeoJSONSource).setData(cityData);

		// Hide provinces completely
		map.setLayoutProperty('provinces-fill', 'visibility', 'none');
		map.setLayoutProperty('provinces-line', 'visibility', 'none');

		map.setLayoutProperty('cities-fill', 'visibility', 'visible');
		map.setLayoutProperty('cities-line', 'visibility', 'visible');

		hoveredName = null;
		flyToFeature(feature);
	}

	async function handleCityClick(feature: GeoJSONFeature) {
		if (!map) return;

		const code = feature.properties.adm3_psgc as number;
		const name = feature.properties.adm3_en as string;

		selectedCity = { code, name };

		const cityData = await loadGeoJson(`municities/lowres/bgysubmuns-municity-${code}.${RES}.json`);
		await (map.getSource('barangays') as GeoJSONSource).setData(cityData);

		map.setLayoutProperty('cities-fill', 'visibility', 'none');
		map.setLayoutProperty('cities-line', 'visibility', 'none');

		map.setLayoutProperty('barangays-fill', 'visibility', 'visible');
		map.setLayoutProperty('barangays-line', 'visibility', 'visible');

		hoveredName = null;
		flyToFeature(feature);
	}

	function bindLayerEvents(
		layerId: string,
		sourceId: string,
		nameProp: string,
		onClick: (feature: GeoJSONFeature) => void
	) {
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

		map.on('click', layerId, (e) => {
			if (!e.features?.length) return;
			onClick(e.features[0]);
		});
	}

	async function resetToCountry() {
		if (!map) return;

		selectedRegion = null;
		selectedProvince = null;
		selectedCity = null;
		hoveredName = null;

		await (map.getSource('provinces') as GeoJSONSource).setData(emptyFeatureCollection);
		await (map.getSource('cities') as GeoJSONSource).setData(emptyFeatureCollection);
		await (map.getSource('barangays') as GeoJSONSource).setData(emptyFeatureCollection);
		// Restore regions provinces, and cities visibility
		map.setLayoutProperty('regions-fill', 'visibility', 'visible');
		map.setLayoutProperty('regions-line', 'visibility', 'visible');
		map.flyTo({ center: [122.0, 12.5], zoom: 4.5, duration: 800 });
	}

	async function resetToRegion() {
		if (!map || !selectedRegion) return;

		selectedProvince = null;
		selectedCity = null;
		hoveredName = null;

		await (map.getSource('cities') as GeoJSONSource).setData(emptyFeatureCollection);
		await (map.getSource('barangays') as GeoJSONSource).setData(emptyFeatureCollection);

		map.setLayoutProperty('provinces-fill', 'visibility', 'visible');
		map.setLayoutProperty('provinces-line', 'visibility', 'visible');
		map.fitBounds(selectedRegion.bounds, { padding: 45, duration: 800 });
	}

	async function resetToProvince() {
		if (!map || !selectedRegion || !selectedProvince) return;

		selectedCity = null;
		hoveredName = null;

		await (map.getSource('barangays') as GeoJSONSource).setData(emptyFeatureCollection);

		map.setLayoutProperty('cities-fill', 'visibility', 'visible');
		map.setLayoutProperty('cities-line', 'visibility', 'visible');
		map.fitBounds(selectedProvince.bounds, { padding: 45, duration: 800 });
	}

	onDestroy(() => {
		if (map) map.remove();
	});
</script>

<div class={`overflow-hidde flex flex-col items-stretch justify-stretch ${props.class}`}>
	<Breadcrumb.Root class="m-2 self-center">
		<Breadcrumb.List>
			<Breadcrumb.Item>
				<Button variant="link" onclick={resetToCountry}>Philippines</Button>
			</Breadcrumb.Item>
			{#if selectedRegion}
				<Breadcrumb.Separator />
				<Breadcrumb.Item>
					<Button variant="link" onclick={resetToRegion}>{selectedRegion.name}</Button>
				</Breadcrumb.Item>
			{/if}
			{#if selectedProvince}
				<Breadcrumb.Separator />
				<Breadcrumb.Item>
					<Button variant="link" onclick={resetToProvince}>{selectedProvince.name}</Button>
				</Breadcrumb.Item>
			{/if}
			{#if selectedCity}
				<Breadcrumb.Separator />
				<Breadcrumb.Item>
					<Button variant="link">{selectedCity.name}</Button>
				</Breadcrumb.Item>
			{/if}
		</Breadcrumb.List>
	</Breadcrumb.Root>

	<div bind:this={mapContainer} class="relative min-h-0 flex-1"></div>
	<p class="m-4 min-h-6 text-center">{loading ? 'Loading ' : (hoveredName ?? '')}</p>
</div>
