<script lang="ts">
	import * as Card from './ui/card/index.ts';

	let props: {
		variant: 'info' | 'problem' | 'solution';
		title: string;
		body: string;
		links?: { short: string; url: string }[];
	} = $props();

	const colors = {
		info: 'bg-cyan-900',
		problem: 'bg-red-900',
		solution: 'bg-green-800'
	};
</script>

<Card.Root class="pt-0">
	<Card.Header class={`p-4 ${colors[props.variant]}`}>
		<Card.Title class="text-olive-50">{props.title}</Card.Title>

		<div class="flex flex-row gap-2">
			{#each props.links ?? [] as { short, url } (url)}
				<a class="text-olive-50" href={url} rel="noopener noreferrer external" target="_blank">
					{short}
				</a>
			{/each}
		</div>
	</Card.Header>
	<Card.Content class="pl-4">{props.body}</Card.Content>
</Card.Root>
