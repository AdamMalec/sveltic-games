<svelte:head>
	<title>Sveltic Gam8s: Typing Race</title>
</svelte:head>

<section class="layout">
	<h1>Typing Race</h1>
	<slot />
</section>

<style>
	.layout {
		display: grid;
		grid-template-rows: auto 1fr;
		align-items: center;
		padding: 2rem;
	}
</style>
