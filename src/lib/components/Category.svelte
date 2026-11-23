<script lang="ts">
	import type { Snippet } from 'svelte';
	import Separator from './ui/separator/separator.svelte';

	let props: { title: string; children: Snippet } = $props();
</script>

<div class="flex flex-col gap-4">
	<p class="font-serif text-xl font-semibold">{props.title}</p>
	<Separator class=" bord bg-olive-600" />

	<div class="grid grid-cols-1 gap-4 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-2">
		{@render props.children()}
	</div>
</div>
