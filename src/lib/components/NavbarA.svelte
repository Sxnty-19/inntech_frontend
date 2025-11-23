<script>
    import { onMount } from "svelte";
    import { goto } from "$app/navigation";

    let modulos = [];
    let error = "";

    let user = null;
    let idRol = null;

    onMount(async () => {
        user = JSON.parse(localStorage.getItem("user"));

        if (!user) {
            error = "Usuario no encontrado";
            return;
        }

        idRol = user.id_rol;

        await cargarModulos();
    });

    async function cargarModulos() {
        try {
            const response = await fetch(`https://inntech-backend.onrender.com/modulos_roles/get_modulos_by_rol/${idRol}`);
            const data = await response.json();

            if (!response.ok) {
                error = data.detail || "No se pudieron cargar los módulos";
                return;
            }

            modulos = data.data;
        } catch (e) {
            console.error(e);
            error = "Error de conexión con el servidor";
        }
    }

    function ir(ruta) {
        goto(ruta);
    }
</script>

<nav class="mod-nav">
    {#if error}
        <p class="error">{error}</p>
    {:else if modulos.length === 0}
        <p class="loading">Cargando módulos...</p>
    {:else}
        {#each modulos as m}
            <button class="mod-btn" on:click={() => ir(m.ruta)}>
                {m.nombre}
            </button>
        {/each}
    {/if}
</nav>

<style>
    .mod-nav {
        width: 100%;
        background: #1f2937;
        border-bottom: 1px solid rgba(255,255,255,0.1);
        padding: 10px 20px;
        display: flex;
        gap: 15px;
        align-items: center;
    }

    .mod-btn {
        background: transparent;
        border: 1px solid orange;
        padding: 6px 12px;
        border-radius: 6px;
        color: orange;
        cursor: pointer;
        transition: .2s;
    }

    .mod-btn:hover {
        background: orange;
        color: black;
    }

    .loading, .error {
        color: white;
        opacity: 0.7;
    }
</style>