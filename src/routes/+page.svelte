<script lang="ts">
	import Map, { type MapLocation } from '#lib/components/Map.svelte';
	import Category from '#lib/components/Category.svelte';
	import Entry from '#lib/components/Entry.svelte';

	type JSONData = {
		city: string;
		info: EntryData[];
		problems: EntryData[];
		solutions: EntryData[];
	};

	type EntryData = {
		title: string;
		description: string;
		links: {
			short: string;
			url: string;
		}[];
		location: string;
	};

	const data = import.meta.glob('#lib/assets/data/*/*/*.json', { import: 'default' });

	let info = $state<EntryData[]>([]);
	let problems = $state<EntryData[]>([]);
	let solutions = $state<EntryData[]>([]);

	async function handleLocationChange(loc: MapLocation) {
		const { region, province, city } = loc;

		if (!region) {
			info = [];
			problems = [];
			solutions = [];
		}

		const promises = Object.entries(data)
			.filter(([url]) => {
				if (!url.includes(`/region-${region}/`)) return false;
				if (province !== null && !url.includes(`/province-${province}/`)) return false;
				if (city !== null && !url.endsWith(`/${city}.json`)) return false;
				return true;
			})
			.map(([, promise]) => promise);

		const files = (await Promise.all(promises.map((promises) => promises()))) as JSONData[];

		info = files.flatMap((f) => f.info ?? []);
		problems = files.flatMap((f) => f.problems ?? []);
		solutions = files.flatMap((f) => f.solutions ?? []);
	}
</script>

<div class="flex min-h-dvh flex-col gap-8 bg-neutral-50 p-8">
	<header class="bg-neutral-100 p-4 shadow-lg">
		<h1 class="text-center font-serif text-4xl font-bold">Agos</h1>
	</header>
	<div class="flex flex-1 flex-col gap-8">
		<Map
			class="h-[65dvh] w-full min-w-0 rounded-lg bg-neutral-100 shadow-lg"
			onLocationChange={handleLocationChange}
		/>
		<div class="flex flex-col gap-8 lg:basis-1/2">
			{#if info.length > 0}
				<Category title="The Ecosystem">
					{#each info as i (i.title + i.location)}
						<Entry variant="info" {...i}></Entry>
					{/each}
				</Category>
			{/if}
			{#if problems.length > 0}
				<Category title="Issues and Problems">
					{#each problems as p (p.title + p.location)}
						<Entry variant="problem" {...p}></Entry>
					{/each}
				</Category>
			{/if}
			{#if solutions.length > 0}
				<Category title="Mitigation Strategies and Solutions">
					{#each solutions as s (s.title + s.location)}
						<Entry variant="solution" {...s}></Entry>
					{/each}
				</Category>
			{/if}
		</div>
	</div>
</div>

<title> Agos </title>
