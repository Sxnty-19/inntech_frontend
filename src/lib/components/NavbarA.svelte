<script>
    import { onMount } from "svelte";
    import { goto } from "$app/navigation";

    // Variables de estado
    let modulos = [];
    let error = "";
    let isLoading = true;
    let isMenuOpen = false; // Estado para controlar si el menú desplegable está abierto

    let user = null;
    let idRol = null;

    onMount(async () => {
        // 1. Cargar usuario y rol
        const stored = localStorage.getItem("user");
        user = stored ? JSON.parse(stored) : null;

        if (!user) {
            error = "Usuario no encontrado";
            isLoading = false;
            return;
        }

        idRol = user.id_rol;

        // 2. Cargar módulos
        await cargarModulos();
        isLoading = false;
    });

    async function cargarModulos() {
        if (!idRol) return;

        try {
            // Implementar retardo con backoff para la API
            const MAX_RETRIES = 3;
            for (let i = 0; i < MAX_RETRIES; i++) {
                const response = await fetch(
                    `https://inntech-backend.onrender.com/modulos_roles/get_modulos_by_rol/${idRol}`,
                );
                const data = await response.json();

                if (response.ok) {
                    modulos = data.data;
                    error = "";
                    return; // Éxito
                }

                if (i < MAX_RETRIES - 1) {
                    // Esperar antes de reintentar
                    await new Promise((resolve) =>
                        setTimeout(resolve, Math.pow(2, i) * 1000),
                    );
                } else {
                    error =
                        data.detail ||
                        "No se pudieron cargar los módulos después de varios intentos";
                }
            }
        } catch (e) {
            console.error("Error al cargar módulos:", e);
            error = "Error de conexión con el servidor";
        }
    }

    function toggleMenu() {
        isMenuOpen = !isMenuOpen;
    }

    function ir(ruta) {
        goto(ruta);
        // Cierra el menú después de la navegación para mejorar la UX en móvil
        if (window.matchMedia("(max-width: 768px)").matches) {
            isMenuOpen = false;
        }
    }
</script>

<!--
  La clase 'open' solo se usa en móvil para el menú desplegable.
  En desktop, el contenido siempre es visible en línea.
-->
<nav class="mod-nav">
    <div class="nav-wrapper">
        <!-- Botón de Toggle visible solo en móvil -->
        {#if modulos.length > 0}
            <button
                class="menu-toggle-btn"
                on:click={toggleMenu}
                aria-expanded={isMenuOpen}
                aria-controls="module-list"
            >
                <!-- Icono de Hamburguesa (☰) o Cerrar (✕) -->
                {isMenuOpen ? "✕ Cerrar Módulos" : "☰ Módulos"}
            </button>
        {/if}

        {#if error}
            <p class="error">{error}</p>
        {:else if isLoading}
            <div class="loading-indicator">
                <!-- Reemplazo de Font Awesome con un spinner SVG simple por robustez -->
                <svg class="spinner" viewBox="0 0 50 50">
                    <circle cx="25" cy="25" r="20" fill="none" stroke-width="5"
                    ></circle>
                </svg>
                <p>Cargando módulos...</p>
            </div>
        {:else if modulos.length > 0}
            <!-- Contenedor de botones. En móvil, se convierte en el menú desplegable. -->
            <div
                class="mod-buttons-container"
                class:open={isMenuOpen}
                id="module-list"
            >
                <!-- Contenedor interno para manejar el scroll horizontal en desktop -->
                <div class="mod-scroll-container">
                    {#each modulos as m (m.ruta)}
                        <button
                            class="mod-btn"
                            on:click={() => ir(m.ruta)}
                            aria-current={location.pathname === m.ruta
                                ? "page"
                                : undefined}
                        >
                            {m.nombre}
                        </button>
                    {/each}
                </div>
            </div>
        {:else}
            <!-- Mensaje si no hay módulos (ni error, ni cargando) -->
            <p class="empty-message">No hay módulos asignados para tu rol.</p>
        {/if}
    </div>
</nav>

<style>
    /* ---------------------------------- */
    /* VARIABLES (Consistencia de marca) */
    /* ---------------------------------- */
    :root {
        --color-primary: #1e3a8a; /* Azul oscuro elegante */
        --color-secondary: #fcd34d; /* Amarillo/Naranja sutil */
        --color-mid-dark: #1f2937; /* Gris oscuro (Fondo de esta barra) */
        --color-text: #e2e8f0;
        --color-danger: #ef4444;
    }

    /* ---------------------------------- */
    /* BARRA DE NAVEGACIÓN SECUNDARIA */
    /* ---------------------------------- */
    .mod-nav {
        width: 100%;
        background: var(--color-mid-dark);
        border-bottom: 2px solid var(--color-primary);
        padding: 8px 40px;
        display: flex;
        align-items: center;
        position: sticky;
        top: 65px; /* Altura aproximada del Navbar principal */
        z-index: 49;
        box-shadow:
            0 4px 6px -1px rgba(0, 0, 0, 0.1),
            0 2px 4px -2px rgba(0, 0, 0, 0.1);
        font-family: "Inter", sans-serif;
        /* FIX CLAVE: Asegurar que el padding no cause desbordamiento */
        box-sizing: border-box;
        /* FIX CLAVE: Ocultar cualquier elemento que se desborde internamente */
        overflow-x: hidden;
    }

    .nav-wrapper {
        width: 100%;
        max-width: 1400px; /* Limite el ancho en pantallas ultra anchas */
        margin: 0 auto;
        display: flex;
        align-items: center;
        justify-content: center;
        position: relative;
    }

    /* ---------------------------------- */
    /* Contenedor de botones (Lista de Módulos) - Desktop & Móvil Base */
    /* ---------------------------------- */
    .mod-buttons-container {
        display: flex;
        /* En desktop, este contenedor controla la visibilidad y centrado */
        width: 100%;
        justify-content: center;
    }

    /* Contenedor interno para manejar el scroll horizontal en DESKTOP */
    .mod-scroll-container {
        display: flex;
        gap: 10px;
        /* Esto es CRUCIAL para el scroll horizontal en DESKTOP */
        overflow-x: auto;
        padding-bottom: 5px;
        justify-content: flex-start;
        /* Ocultar scrollbar en navegadores WebKit (Chrome, Safari) */
        scrollbar-width: none; /* Firefox */
        -ms-overflow-style: none; /* IE/Edge */
    }

    .mod-scroll-container::-webkit-scrollbar {
        display: none; /* WebKit */
    }

    /* Botones de Módulo (simulando pestañas) */
    .mod-btn {
        background: transparent;
        border: none;
        padding: 8px 18px;
        border-radius: 6px;
        color: var(--color-text);
        font-weight: 500;
        cursor: pointer;
        transition: all 0.25s ease;
        /* Esto es CRUCIAL para el scroll horizontal */
        flex-shrink: 0;
        text-transform: capitalize;
        font-size: 0.95rem;
    }

    /* Estado ACTIVO */
    .mod-btn[aria-current="page"] {
        background: var(--color-primary);
        color: white;
        font-weight: 600;
        box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
    }

    /* Estado HOVER */
    .mod-btn:hover:not([aria-current="page"]) {
        background: rgba(255, 255, 255, 0.1);
        color: var(--color-secondary);
    }

    /* ---------------------------------- */
    /* INDICADORES DE ESTADO */
    /* ---------------------------------- */
    .error,
    .empty-message,
    .loading-indicator {
        width: 100%;
        text-align: center;
        padding: 5px 0;
        color: var(--color-text);
        font-size: 0.9rem;
    }

    .error {
        color: var(--color-danger);
        font-weight: 600;
    }
    .loading-indicator {
        display: flex;
        align-items: center;
        gap: 10px;
        justify-content: center;
    }

    .spinner {
        animation: spin 1.2s linear infinite;
        height: 18px;
        width: 18px;
        stroke: var(--color-secondary);
        stroke-linecap: round;
    }

    @keyframes spin {
        0% {
            transform: rotate(0deg);
        }
        100% {
            transform: rotate(360deg);
        }
    }

    /* ---------------------------------- */
    /* Botón de Toggle (Menú Hambuger) */
    /* ---------------------------------- */
    .menu-toggle-btn {
        display: none; /* Oculto por defecto, visible solo en móvil */
        background: var(--color-primary);
        color: white;
        border: none;
        padding: 8px 15px;
        border-radius: 6px;
        font-weight: 600;
        cursor: pointer;
        transition: background 0.2s;
        gap: 8px;
        width: 100%; /* Ocupa todo el ancho en móvil */
        text-align: center;
    }

    .menu-toggle-btn:hover {
        opacity: 0.9;
    }

    /* ---------------------------------- */
    /* RESPONSIVIDAD: Menú Desplegable en Móvil (max-width: 768px) */
    /* ---------------------------------- */
    @media (max-width: 768px) {
        .mod-nav {
            padding: 8px 15px;
            top: 55px;
        }

        /* FIX CLAVE: Cambiar el contenedor principal a columna */
        .nav-wrapper {
            flex-direction: column;
            justify-content: flex-start;
            align-items: flex-start;
        }

        .menu-toggle-btn {
            display: block;
            width: 100%; /* El botón ocupa todo el ancho */
            margin-bottom: 8px;
            box-sizing: border-box; /* FIX BOX-SIZING */
        }

        /* FIX CLAVE: El contenedor del menú ahora fluye */
        .mod-buttons-container {
            position: static; /* Quitamos absolute para que fluya */
            top: auto;
            left: auto;
            right: auto;
            width: 100%; /* CLAVE: Ocupa el 100% del nav-wrapper */

            background: var(--color-mid-dark);
            border: 1px solid var(--color-primary);
            z-index: 10;
            box-shadow: 0 8px 16px rgba(0, 0, 0, 0.4);
            border-top: none;
            border-radius: 8px; /* Redondeamos las esquinas completamente */

            /* Animación de altura */
            max-height: 0;
            overflow: hidden;
            opacity: 0;
            transition:
                max-height 0.4s ease-out,
                opacity 0.3s ease-out;

            padding: 0;
            box-sizing: border-box; /* FIX BOX-SIZING */
        }

        .mod-buttons-container.open {
            max-height: 500px;
            opacity: 1;
        }

        /* El scroll container ahora se apila verticalmente y agrega el padding interno */
        .mod-scroll-container {
            flex-direction: column;
            align-items: stretch; /* CLAVE: Estira los elementos al 100% del ancho del contenedor */
            padding: 10px 15px; /* Agregamos el padding interno aquí */
            overflow-x: hidden;
            width: 100%; /* Ocupa el 100% del contenedor padre */
            box-sizing: border-box; /* FIX BOX-SIZING */
        }

        /* El botón se ajusta automáticamente gracias a align-items: stretch */
        .mod-btn {
            padding: 10px;
            text-align: left;
            border-bottom: 1px solid rgba(255, 255, 255, 0.05);
            border-radius: 4px;
        }

        .mod-btn:hover:not([aria-current="page"]) {
            background: var(--color-primary);
            color: white;
            opacity: 0.8;
        }

        .mod-btn[aria-current="page"] {
            opacity: 1;
        }

        /* En móvil, los indicadores se alinean a la izquierda */
        .loading-indicator {
            justify-content: flex-start;
        }

        .error,
        .empty-message {
            text-align: left;
        }
    }
</style>
