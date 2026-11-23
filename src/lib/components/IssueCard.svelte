<script lang="ts">
	import * as Card from './ui/card/index.ts';

	let props: {
		title: string;
		body: string;
		links: { short: string; url: string }[];
	} = $props();
</script>

<Card.Root class="overflow-clip border-2 border-olive-600 bg-red-100">
	<Card.Header>
		<Card.Title>{props.title}</Card.Title>
		<Card.Description>{props.body}</Card.Description>
	</Card.Header>
	<Card.Footer>
		<ul>
			{#each props.links as { short, url } (url)}
				<li><a href={url} rel="noopener noreferrer external" target="_blank">{short}</a></li>
			{/each}
		</ul>
	</Card.Footer>
</Card.Root>
