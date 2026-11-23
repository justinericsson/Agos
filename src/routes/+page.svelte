<script>
	import Map from '#lib/components/Map.svelte';
	import Category from '#lib/components/Category.svelte';
	import Entry from '#lib/components/Entry.svelte';
</script>

<header class="bg-olive-50 p-4 shadow-lg">
	<h1 class="text-center font-serif text-4xl font-bold">Agos</h1>
</header>
<div class="flex min-h-dvh flex-col gap-8 bg-olive-50 p-8">
	<div class="flex flex-1 flex-col gap-8 lg:flex-row lg:items-start">
		<Map
			class="h-[65dvh] w-full min-w-0 rounded-lg border-2 border-olive-600 lg:h-[75dvh] lg:basis-1/2"
		/>
		<div class="flex flex-col gap-8 lg:basis-1/2">
			<Category title="Ecosystem">
				<Entry
					variant="info"
					title="Corals"
					body="Hello world"
					links={[{ short: 'hello', url: 'https://google.com' }]}
				/>
				<Entry variant="info" title="Corals" body="Hello world" />
			</Category>

			<Category title="Issues and problems">
				<Entry
					variant="problem"
					title="Corals"
					body="Hello world"
					links={[
						{ short: 'hello', url: 'https://google.com' },
						{ short: 'world', url: 'https://duckduckgo.com' }
					]}
				/>
				<Entry variant="problem" title="Corals" body="Hello world" links={[]} />
			</Category>

			<Category title="Solutions and mitigations">
				<Entry
					variant="solution"
					title="Corals"
					body="Hello world"
					links={[
						{ short: 'hello', url: 'https://google.com' },
						{ short: 'world', url: 'https://duckduckgo.com' }
					]}
				/>
				<Entry variant="solution" title="Corals" body="Hello world" links={[]} />
			</Category>
		</div>
	</div>
</div>
