<script>
	import favicon from "$lib/assets/favicon.svg";

	let { children } = $props();
</script>

<svelte:head>
	<meta name="viewport" content="width=device-width, initial-scale=1" />

	<link rel="icon" href={favicon} />
	<link
		rel="stylesheet"
		href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css"
		integrity="sha512-SnH5WK+bZxgPHs44uWIX+LLMD/CD+5N+yEStRjS1K48xSgH44n/B+xJ1A40k8c7oE9a9u5pG2Gq0wF04iP5w=="
		crossorigin="anonymous"
		referrerpolicy="no-referrer"
	/>
	<title>H. Santadereano Del Sur</title>

	<link
		href="https://fonts.googleapis.com/css2?family=Inter:wght@100..900&display=swap"
		rel="stylesheet"
	/>
</svelte:head>

{@render children()}

<style>
	/* ---------------------------------- */
	/* DEFINICIÓN DE PALETA DE COLORES (Variables CSS) */
	/* ---------------------------------- */
	:global(:root) {
		/* Variables definidas en tu CSS anterior */
		--color-bg-primary: #0f172a;
		--color-table-bg: #1a202c;
		--color-accent-gold: #fcd34d;
		--color-text-light: #e2e8f0;
		/* Puedes añadir aquí todas las variables que uses comúnmente */
	}

	/* ---------------------------------- */
	/* RESET GLOBAL Y REGLAS DE RESPONSIVIDAD BASE */
	/* ---------------------------------- */
	:global(html) {
		/* Tamaño de fuente base responsive (opcional) */
		font-size: 16px;
	}

	:global(html),
	:global(body) {
		/* 1. MODELO DE CAJA GLOBAL (Clave para evitar desbordamientos) */
		margin: 0;
		padding: 0;
		/* box-sizing: border-box es CRUCIAL para que padding y borde no sumen al width */
		box-sizing: border-box;

		/* 2. TIPOGRAFÍA Y ALTURA MÍNIMA */
		font-family: "Inter", sans-serif;
		background-color: var(--color-bg-primary);
		min-height: 100vh; /* Asegura que el cuerpo ocupe al menos toda la altura de la vista */
	}

	/* 3. Evita el desbordamiento horizontal accidental en todos los elementos */
	:global(*) {
		box-sizing: border-box;
	}

	/* 4. CLAVES PARA RESPONSIVIDAD GENERAL */
	:global(img),
	:global(video),
	:global(picture) {
		/* Asegura que las imágenes nunca se salgan de su contenedor */
		max-width: 100%;
		height: auto;
		display: block; /* Elimina espacio extra bajo las imágenes */
	}
</style>
