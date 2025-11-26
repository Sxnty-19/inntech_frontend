<script>
    import { onMount } from "svelte";
    import { goto } from "$app/navigation";

    // Variables de estado
    let modulos = [];
    let error = "";
    // Estado de carga mejorado
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

<nav class="mod-nav">
    <div class="nav-wrapper">
        <!-- Botón de Toggle visible solo en móvil -->
        {#if modulos.length > 0 && !isLoading}
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
            <!-- SOLO ANIMACIÓN: Usamos un contenedor simplificado para solo mostrar la barra. -->
            <div class="loading-container-bar">
                <div class="loading-bar"></div>
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
        --color-mid-dark: #0f172a; /* Gris oscuro (Fondo de esta barra) */
        --color-text: #e2e8f0;
        --color-danger: #ef4444;
        --color-bg-light: #1a202c; /* Fondo para la barra de carga */
    }

    /* ---------------------------------- */
    /* BARRA DE NAVEGACIÓN SECUNDARIA */
    /* ---------------------------------- */
    .mod-nav {
        width: 100%;
        background: var(--color-mid-dark);
        padding: 8px 40px;
        display: flex;
        align-items: center;
        position: sticky;
        top: 65px;
        z-index: 49;
        box-shadow:
            0 4px 6px -1px rgba(0, 0, 0, 0.1),
            0 2px 4px -2px rgba(0, 0, 0, 0.1);
        font-family: "Inter", sans-serif;
        box-sizing: border-box;
    }

    .nav-wrapper {
        width: 100%;
        max-width: 1400px;
        margin: 0 auto;
        display: flex;
        align-items: center;
        justify-content: center;
        position: relative;
    }

    /* ---------------------------------- */
    /* Indicador de Carga (Solo Barra de Progreso) */
    /* ---------------------------------- */
    .loading-container-bar {
        width: 100%;
        display: flex;
        justify-content: center; /* Centrar la barra */
        padding: 5px 0; /* Espacio para que no se pegue */
    }

    .loading-bar {
        width: 80%; /* Ancho de la barra de progreso */
        height: 3px;
        background: var(--color-bg-light); /* Fondo estático */
        border-radius: 5px;
        overflow: hidden;
    }

    .loading-bar::before {
        content: "";
        display: block;
        height: 100%;
        width: 30%; /* Tamaño visible de la barra animada */
        background: linear-gradient(
            90deg,
            var(--color-primary),
            var(--color-secondary)
        );
        animation: loading-move 1.5s infinite linear;
    }

    @keyframes loading-move {
        0% {
            transform: translateX(-100%);
        }
        100% {
            transform: translateX(
                333%
            ); /* 100% + 2 veces el ancho para salir */
        }
    }

    /* ---------------------------------- */
    /* Contenedor de botones (Lista de Módulos) - Desktop & Móvil Base */
    /* ---------------------------------- */
    .mod-buttons-container {
        display: flex;
        width: 100%;
        justify-content: center;
    }

    /* Contenedor interno para manejar el scroll horizontal en DESKTOP (CLAVE) */
    .mod-scroll-container {
        display: flex;
        gap: 10px;
        /* Esto es CRUCIAL para el scroll horizontal en DESKTOP */
        overflow-x: auto;
        padding-bottom: 5px; /* Espacio para el scrollbar */
        justify-content: flex-start;
        /* Asegura que los botones se mantengan en una línea horizontal */
        flex-wrap: nowrap;

        scrollbar-width: thin; /* Firefox */
        scrollbar-color: var(--color-primary) var(--color-mid-dark);
    }
    /* Ocultar scrollbar en navegadores WebKit (Chrome, Safari) */
    .mod-scroll-container::-webkit-scrollbar {
        height: 6px;
    }
    .mod-scroll-container::-webkit-scrollbar-thumb {
        background: var(--color-primary);
        border-radius: 3px;
    }
    .mod-scroll-container::-webkit-scrollbar-track {
        background: var(--color-mid-dark);
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
    .empty-message {
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
            top: 55px; /* Ajuste si el Navbar principal es más pequeño en móvil */
        }

        /* Cambiar el contenedor principal a columna para que el botón de toggle fluya */
        .nav-wrapper {
            flex-direction: column;
            justify-content: flex-start;
            align-items: flex-start;
        }

        .menu-toggle-btn {
            display: block;
            width: 100%;
            margin-bottom: 8px;
            box-sizing: border-box;
        }

        /* El contenedor de botones se convierte en un menú desplegable vertical */
        .mod-buttons-container {
            width: 100%;
            max-height: 0;
            overflow: hidden;
            opacity: 0;
            transition:
                max-height 0.4s ease-out,
                opacity 0.3s ease-out;

            background: var(--color-mid-dark);
            border-radius: 0 0 8px 8px;
            box-shadow: 0 8px 16px rgba(0, 0, 0, 0.4);
        }

        .mod-buttons-container.open {
            max-height: 500px; /* Suficiente para muchos módulos */
            opacity: 1;
            /* Agregamos un borde superior si no está debajo del botón */
            border-top: 1px solid rgba(255, 255, 255, 0.1);
        }

        /* El scroll container ahora apila verticalmente los botones */
        .mod-scroll-container {
            flex-direction: column;
            align-items: stretch; /* Estira los botones al 100% del ancho */
            padding: 10px 15px;
            overflow-x: hidden;
            width: 100%;
            box-sizing: border-box;
        }

        /* Ocultar scrollbar vertical si el menú es muy largo */
        .mod-scroll-container::-webkit-scrollbar {
            width: 0;
            height: 0;
        }

        .mod-btn {
            padding: 10px;
            text-align: left;
            border-bottom: 1px solid rgba(255, 255, 255, 0.05);
            border-radius: 4px;
        }

        .mod-btn:last-child {
            border-bottom: none;
        }

        .mod-btn:hover:not([aria-current="page"]) {
            background: var(--color-primary);
            color: white;
            opacity: 0.8;
        }

        /* En móvil, la barra de carga debe ocupar todo el ancho */
        .loading-bar {
            width: 100%;
        }
        /* Eliminar estilos del antiguo loading-container para el nuevo contenedor */
        .loading-container-bar {
            width: 100%;
            justify-content: flex-start;
        }
    }
</style>
