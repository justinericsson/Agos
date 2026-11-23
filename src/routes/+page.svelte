<script>
	import Map from '#lib/components/Map.svelte';
	import InfoCard from '#lib/components/InfoCard.svelte';
</script>

<header class="bg-olive-50 p-4 shadow-lg">
	<h1 class="text-center text-4xl font-bold">Agos</h1>
</header>
<div class="flex min-h-dvh flex-col gap-8 bg-olive-50 p-8">
	<div class="flex flex-1 flex-col gap-8 md:flex-row md:items-start">
		<Map
			class="h-[65dvh] w-full min-w-0 rounded-lg border-2 border-olive-600 md:h-[75dvh] md:basis-2/3"
		/>
		<div class="grid grid-cols-2 gap-4 md:basis-1/3 md:grid-cols-1 lg:grid-cols-2">
			<InfoCard
				title="Corals"
				body="Hello world"
				links={[{ short: 'hello', url: 'https://google.com' }]}
			/>
			<InfoCard title="Corals" body="Hello world" links={[]} />
		</div>
	</div>
</div>
