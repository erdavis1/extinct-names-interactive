<script >
	import favicon from '$lib/assets/favicon.svg';
	export const prerender = true;

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />

	<link
		href="https://fonts.googleapis.com/css2?family=Libre+Franklin:wght@400;500;600;700&display=swap"
		rel="stylesheet"
  	/>

	<link
  href="https://fonts.googleapis.com/css2?family=Libre+Caslon+Text:ital,wght@0,400;0,700;1,400&display=swap"
  rel="stylesheet"
/>
</svelte:head>

{@render children()}
