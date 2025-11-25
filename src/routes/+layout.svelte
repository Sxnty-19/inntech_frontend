<script>
	import favicon from "$lib/assets/favicon.svg";

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
	<link
		rel="stylesheet"
		href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css"
		integrity="sha512-SnH5WK+bZxgPHs44uWIX+LLMD/CD+5N+yEStRjS1K48xSgH44n/B+xJ1A40k8c7oE9a9u5pG2Gq0wF04iP5w=="
		crossorigin="anonymous"
		referrerpolicy="no-referrer"
	/>
	<title>H. Santadereano Del Sur</title>
</svelte:head>

{@render children()}

<style>
	/* ---------------------------------- */
	/* RESET GLOBAL: FIX BORDE BLANCO Y FUENTE FOOTER */
	/* ---------------------------------- */
	:global(html),
	:global(body) {
		/* 1. ELIMINA EL BORDE BLANCO */
		margin: 0;
		padding: 0;
		box-sizing: border-box;
		/* 2. ASEGURA LA FUENTE EN TODO EL DOCUMENTO (Incluido Footer) */
		font-family: "Inter", sans-serif;
		/* 3. FONDO GLOBAL PARA EVITAR BORDES BLANCOS VISIBLES */
		background-color: var(--color-bg-primary);
	}
</style>
