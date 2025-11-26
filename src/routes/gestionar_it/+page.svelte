<script>
  import { onMount } from "svelte";
  // NOTA: Asumimos que estos componentes existen y se importan correctamente.
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";

  const API = "https://turismo-sm.onrender.com";

  // Controla qué sección se muestra: 'eventos', 'lugares', o 'servicios'
  let currentCategory = "eventos";
  let isSubmitting = false; // Controla el estado de envío global del formulario

  /* ========================= SISTEMA DE MENSAJES ========================= */

  // Variables para el sistema de mensajes/modal personalizado (Reemplazo de alert)
  let message = "";
  let isSuccess = false;
  let showMessage = false;

  // Función para mostrar y ocultar el modal
  function showMessageModal(text, success) {
    message = text;
    isSuccess = success;
    showMessage = true;

    // Ocultar el mensaje automáticamente después de 3 segundos
    setTimeout(() => {
      showMessage = false;
      message = "";
    }, 3000);
  }

  /* ========================= EVENTOS ========================= */

  let eventos = [];
  let loadingEventos = true;
  let crearEvento = false;

  let formEvento = {
    nombre: "",
    fecha: "",
    ubicacion: "",
    descripcion: "",
  };

  async function cargarEventos() {
    try {
      const res = await fetch(`${API}/evento/`);
      const data = await res.json();
      eventos = data.reverse();
    } catch (error) {
      console.error("Error cargando eventos", error);
    } finally {
      loadingEventos = false;
    }
  }

  async function guardarEvento() {
    if (isSubmitting) return;

    if (
      !formEvento.nombre ||
      !formEvento.fecha ||
      !formEvento.ubicacion ||
      !formEvento.descripcion
    ) {
      // Usar el modal para notificar el error
      showMessageModal(
        "Error de validación: Por favor, complete todos los campos del evento.",
        false,
      );
      return;
    }

    isSubmitting = true;

    const body = {
      nombre: formEvento.nombre,
      fecha_inicio: formEvento.fecha,
      ubicacion: formEvento.ubicacion,
      descripcion: formEvento.descripcion,
    };

    try {
      const res = await fetch(`${API}/evento/`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(body),
      });

      if (!res.ok) throw new Error("Error al guardar en la API");

      // ÉXITO: Mostrar mensaje
      showMessageModal(
        `Evento "${formEvento.nombre}" creado exitosamente.`,
        true,
      );

      await cargarEventos();
      crearEvento = false;

      formEvento = { nombre: "", fecha: "", ubicacion: "", descripcion: "" };
    } catch (e) {
      console.error("Error guardando evento:", e);
      // ERROR: Mostrar mensaje
      showMessageModal(`Error al crear el Evento. Intente de nuevo.`, false);
    } finally {
      isSubmitting = false;
    }
  }

  /* ========================= LUGARES ========================= */

  let lugares = [];
  let loadingLugares = true;
  let crearLugar = false;

  let formLugar = {
    nombre: "",
    tipo: "",
    ubicacion: "",
    descripcion: "",
  };

  async function cargarLugares() {
    try {
      const res = await fetch(`${API}/lugar/`);
      const data = await res.json();
      lugares = data.reverse();
    } catch (e) {
      console.error("Error cargando lugares", e);
    } finally {
      loadingLugares = false;
    }
  }

  async function guardarLugar() {
    if (isSubmitting) return;

    if (
      !formLugar.nombre ||
      !formLugar.tipo ||
      !formLugar.ubicacion ||
      !formLugar.descripcion
    ) {
      // Usar el modal para notificar el error
      showMessageModal(
        "Error de validación: Por favor, complete todos los campos del lugar.",
        false,
      );
      return;
    }

    isSubmitting = true;

    const body = {
      nombre: formLugar.nombre,
      tipo: formLugar.tipo,
      ubicacion: formLugar.ubicacion,
      descripcion: formLugar.descripcion,
    };

    try {
      const res = await fetch(`${API}/lugar/`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(body),
      });

      if (!res.ok) throw new Error("Error al guardar en la API");

      // ÉXITO: Mostrar mensaje
      showMessageModal(
        `Lugar Turístico "${formLugar.nombre}" creado exitosamente.`,
        true,
      );

      await cargarLugares();
      crearLugar = false;

      formLugar = { nombre: "", tipo: "", ubicacion: "", descripcion: "" };
    } catch (e) {
      console.error("Error guardando lugar:", e);
      // ERROR: Mostrar mensaje
      showMessageModal(
        `Error al crear el Lugar Turístico. Intente de nuevo.`,
        false,
      );
    } finally {
      isSubmitting = false;
    }
  }

  /* ========================= SERVICIOS ========================= */

  let servicios = [];
  let loadingServicios = true;
  let crearServicio = false;

  let formServicio = {
    nombre: "",
    categoria: "",
    contacto: "",
    descripcion: "",
  };

  async function cargarServicios() {
    try {
      const res = await fetch(`${API}/servicio/`);
      const data = await res.json();
      servicios = data.reverse();
    } catch (e) {
      console.error("Error cargando servicios", e);
    } finally {
      loadingServicios = false;
    }
  }

  async function guardarServicio() {
    if (isSubmitting) return;

    if (
      !formServicio.nombre ||
      !formServicio.categoria ||
      !formServicio.contacto ||
      !formServicio.descripcion
    ) {
      // Usar el modal para notificar el error
      showMessageModal(
        "Error de validación: Por favor, complete todos los campos del servicio.",
        false,
      );
      return;
    }

    isSubmitting = true;

    const body = {
      nombre: formServicio.nombre,
      categoria: formServicio.categoria,
      contacto: formServicio.contacto,
      descripcion: formServicio.descripcion,
    };

    try {
      const res = await fetch(`${API}/servicio/`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(body),
      });

      if (!res.ok) throw new Error("Error al guardar en la API");

      // ÉXITO: Mostrar mensaje
      showMessageModal(
        `Servicio "${formServicio.nombre}" creado exitosamente.`,
        true,
      );

      await cargarServicios();
      crearServicio = false;

      formServicio = {
        nombre: "",
        categoria: "",
        contacto: "",
        descripcion: "",
      };
    } catch (e) {
      console.error("Error guardando servicio:", e);
      // ERROR: Mostrar mensaje
      showMessageModal(`Error al crear el Servicio. Intente de nuevo.`, false);
    } finally {
      isSubmitting = false;
    }
  }

  /* ======== CARGAR TODO AL INICIAR ======== */

  onMount(() => {
    cargarEventos();
    cargarLugares();
    cargarServicios();
  });
</script>

<!-- Se necesita Font Awesome para los iconos de éxito/error -->
<svelte:head>
  <link
    rel="stylesheet"
    href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css"
    crossorigin="anonymous"
    referrerpolicy="no-referrer"
  />
  <!-- Se necesita la fuente Inter -->
  <link
    rel="stylesheet"
    href="https://fonts.googleapis.com/css2?family=Inter:wght@100..900&display=swap"
    crossorigin="anonymous"
  />
</svelte:head>

<Navbar />
<NavbarA />

<section class="main">
  <div class="content-container">
    <h1 class="page-title">Administración de Contenido Turístico</h1>

    <!-- ---------------------------------- -->
    <!-- NAVEGACIÓN TIPO PESTAÑA (TABS) -->
    <!-- ---------------------------------- -->
    <div class="category-tabs">
      <button
        class="tab-btn"
        class:active={currentCategory === "eventos"}
        on:click={() => (currentCategory = "eventos")}
      >
        <!-- Icono de Calendario (Eventos) -->
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="20"
          height="20"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2.5"
          stroke-linecap="round"
          stroke-linejoin="round"
        >
          <path d="M8 2v4" />
          <path d="M16 2v4" />
          <rect width="18" height="18" x="3" y="4" rx="2" />
          <path d="M3 10h18" />
        </svg>
        Eventos
      </button>
      <button
        class="tab-btn"
        class:active={currentCategory === "lugares"}
        on:click={() => (currentCategory = "lugares")}
      >
        <!-- Icono de Map Pin (Lugares) -->
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="20"
          height="20"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2.5"
          stroke-linecap="round"
          stroke-linejoin="round"
        >
          <path d="M20 10c0 6-8 12-8 12s-8-6-8-12a8 8 0 0 1 16 0Z" />
          <circle cx="12" cy="10" r="3" />
        </svg>
        Lugares Turísticos
      </button>
      <button
        class="tab-btn"
        class:active={currentCategory === "servicios"}
        on:click={() => (currentCategory = "servicios")}
      >
        <!-- Icono de Maletín (Servicios) -->
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="20"
          height="20"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2.5"
          stroke-linecap="round"
          stroke-linejoin="round"
        >
          <rect width="20" height="14" x="2" y="7" rx="2" ry="2" />
          <path d="M16 21V5a2 2 0 0 0-2-2h-4a2 2 0 0 0-2 2v16" />
        </svg>
        Servicios
      </button>
    </div>

    <!-- ===================================================== -->
    <!-- ====================== EVENTOS ====================== -->
    <!-- ===================================================== -->
    {#if currentCategory === "eventos"}
      <div class="card card-eventos">
        <div class="card-header">
          <h2 class="subtitulo">Administrar Eventos</h2>
          <button
            class="btn-toggle"
            on:click={() => (crearEvento = !crearEvento)}
          >
            {crearEvento ? "Cancelar" : "+ Crear Evento"}
          </button>
        </div>

        {#if crearEvento}
          <div class="form-container">
            <h3 class="form-title">Nuevo Evento</h3>

            <div class="form-grid">
              <input
                placeholder="Nombre del Evento"
                type="text"
                bind:value={formEvento.nombre}
                class="input-field"
              />
              <label>
                Fecha:
                <input
                  type="date"
                  bind:value={formEvento.fecha}
                  class="input-field mt-5"
                />
              </label>
              <input
                placeholder="Ubicación"
                type="text"
                bind:value={formEvento.ubicacion}
                class="input-field"
              />
              <textarea
                placeholder="Descripción detallada del evento..."
                bind:value={formEvento.descripcion}
                class="input-field desc-full"
              />
            </div>

            <button
              class="btn-crear"
              on:click={guardarEvento}
              disabled={isSubmitting}
            >
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="20"
                height="20"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="2.5"
                stroke-linecap="round"
                stroke-linejoin="round"
                ><path
                  d="M19 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11l5 5v11a2 2 0 0 1-2 2z"
                ></path><polyline points="17 21 17 13 7 13 7 21"
                ></polyline><polyline points="7 3 7 8 15 8"></polyline></svg
              >
              {isSubmitting ? "Guardando..." : "Guardar Evento"}
            </button>
          </div>
        {/if}

        <h3 class="section-title mt-40">
          Listado de Eventos ({eventos.length})
        </h3>

        {#if loadingEventos}
          <div class="loading-container-bar">
            <div class="loading-bar"></div>
          </div>
        {:else}
          <div class="table-wrap">
            <table class="tabla">
              <thead>
                <tr>
                  <th>Nombre</th>
                  <th>Fecha</th>
                  <th>Ubicación</th>
                </tr>
              </thead>
              <tbody>
                {#each eventos as item}
                  <tr>
                    <td class="resaltado">{item.nombre}</td>
                    <td>{item.fecha_inicio}</td>
                    <td>{item.ubicacion}</td>
                  </tr>
                {/each}
              </tbody>
            </table>
          </div>
        {/if}
      </div>
    {/if}

    <!-- ===================================================== -->
    <!-- ====================== LUGARES ====================== -->
    <!-- ===================================================== -->
    {#if currentCategory === "lugares"}
      <div class="card card-lugares">
        <div class="card-header">
          <h2 class="subtitulo">Administrar Lugares Turísticos</h2>
          <button
            class="btn-toggle"
            on:click={() => (crearLugar = !crearLugar)}
          >
            {crearLugar ? "Cancelar" : "+ Crear Lugar"}
          </button>
        </div>

        {#if crearLugar}
          <div class="form-container">
            <h3 class="form-title">Nuevo Lugar</h3>

            <div class="form-grid">
              <input
                placeholder="Nombre del Lugar"
                type="text"
                bind:value={formLugar.nombre}
                class="input-field"
              />
              <input
                placeholder="Tipo (Ej: Playa, Museo, Parque)"
                type="text"
                bind:value={formLugar.tipo}
                class="input-field"
              />
              <input
                placeholder="Ubicación"
                type="text"
                bind:value={formLugar.ubicacion}
                class="input-field"
              />
              <textarea
                placeholder="Descripción detallada del lugar..."
                bind:value={formLugar.descripcion}
                class="input-field desc-full"
              />
            </div>

            <button
              class="btn-crear"
              on:click={guardarLugar}
              disabled={isSubmitting}
            >
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="20"
                height="20"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="2.5"
                stroke-linecap="round"
                stroke-linejoin="round"
                ><path
                  d="M19 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11l5 5v11a2 2 0 0 1-2 2z"
                ></path><polyline points="17 21 17 13 7 13 7 21"
                ></polyline><polyline points="7 3 7 8 15 8"></polyline></svg
              >
              {isSubmitting ? "Guardando..." : "Guardar Lugar"}
            </button>
          </div>
        {/if}

        <h3 class="section-title mt-40">
          Listado de Lugares ({lugares.length})
        </h3>

        {#if loadingLugares}
          <div class="loading-container-bar">
            <div class="loading-bar"></div>
          </div>
        {:else}
          <div class="table-wrap">
            <table class="tabla">
              <thead>
                <tr>
                  <th>Nombre</th>
                  <th>Tipo</th>
                  <th>Ubicación</th>
                </tr>
              </thead>
              <tbody>
                {#each lugares as item}
                  <tr>
                    <td class="resaltado">{item.nombre}</td>
                    <td>{item.tipo}</td>
                    <td>{item.ubicacion}</td>
                  </tr>
                {/each}
              </tbody>
            </table>
          </div>
        {/if}
      </div>
    {/if}

    <!-- ===================================================== -->
    <!-- ===================== SERVICIOS ===================== -->
    <!-- ===================================================== -->
    {#if currentCategory === "servicios"}
      <div class="card card-servicios">
        <div class="card-header">
          <h2 class="subtitulo">Administrar Servicios</h2>
          <button
            class="btn-toggle"
            on:click={() => (crearServicio = !crearServicio)}
          >
            {crearServicio ? "Cancelar" : "+ Crear Servicio"}
          </button>
        </div>

        {#if crearServicio}
          <div class="form-container">
            <h3 class="form-title">Nuevo Servicio</h3>

            <div class="form-grid">
              <input
                placeholder="Nombre del Servicio"
                type="text"
                bind:value={formServicio.nombre}
                class="input-field"
              />
              <input
                placeholder="Categoría (Ej: Hotel, Restaurante, Transporte)"
                type="text"
                bind:value={formServicio.categoria}
                class="input-field"
              />
              <input
                placeholder="Contacto (Teléfono/Email)"
                type="text"
                bind:value={formServicio.contacto}
                class="input-field"
              />
              <textarea
                placeholder="Descripción detallada del servicio..."
                bind:value={formServicio.descripcion}
                class="input-field desc-full"
              />
            </div>

            <button
              class="btn-crear"
              on:click={guardarServicio}
              disabled={isSubmitting}
            >
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="20"
                height="20"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="2.5"
                stroke-linecap="round"
                stroke-linejoin="round"
                ><path
                  d="M19 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11l5 5v11a2 2 0 0 1-2 2z"
                ></path><polyline points="17 21 17 13 7 13 7 21"
                ></polyline><polyline points="7 3 7 8 15 8"></polyline></svg
              >
              {isSubmitting ? "Guardando..." : "Guardar Servicio"}
            </button>
          </div>
        {/if}

        <h3 class="section-title mt-40">
          Listado de Servicios ({servicios.length})
        </h3>

        {#if loadingServicios}
          <div class="loading-container-bar">
            <div class="loading-bar"></div>
          </div>
        {:else}
          <div class="table-wrap">
            <table class="tabla">
              <thead>
                <tr>
                  <th>Nombre</th>
                  <th>Categoría</th>
                  <th>Contacto</th>
                </tr>
              </thead>
              <tbody>
                {#each servicios as item}
                  <tr>
                    <td class="resaltado">{item.nombre}</td>
                    <td>{item.categoria}</td>
                    <td>{item.contacto}</td>
                  </tr>
                {/each}
              </tbody>
            </table>
          </div>
        {/if}
      </div>
    {/if}
  </div>

  <!-- Notificación Personalizada (Reemplazo de alert) -->
  {#if showMessage}
    <div class="message-modal">
      <div class="message-content" class:success={isSuccess}>
        <!-- Iconos dinámicos según el estado -->
        <i
          class="fas"
          class:fa-check-circle={isSuccess}
          class:fa-times-circle={!isSuccess}
        ></i>
        <p>{message}</p>
      </div>
    </div>
  {/if}
</section>

<Footer />

<style>
  /* ---------------------------------- */
  /* Paleta de Colores (Dark Mode - Moderno) */
  /* ---------------------------------- */
  :root {
    --color-bg-primary: #0f172a; /* Fondo principal muy oscuro (slate-900) */
    --color-card-bg: #1e293b; /* Fondo de tarjeta/componente (slate-800) */
    --color-table-bg: #1a202c; /* Fondo de tabla muy oscuro (gray-900) */
    --color-accent-gold: #fcd34d; /* Amarillo principal (Dorado/Naranja) */
    --color-accent-blue: #60a5fa; /* Azul para highlights (blue-400) */
    --color-text-light: #e2e8f0; /* Texto claro (slate-200) */
    --color-text-muted: #94a3b8; /* Texto sutil (slate-400) */
    --color-border: #334155; /* Borde sutil (slate-700) */
    --color-primary: #1e3a8a; /* Azul (para encabezado tabla) */
    --color-success: #10b981; /* Verde */
    --color-danger: #ef4444; /* Rojo */
    --shadow-card: 0 10px 25px rgba(0, 0, 0, 0.4);
    --shadow-btn-gold: 0 5px 15px rgba(252, 211, 77, 0.3);
    --shadow-btn-toggle: 0 2px 8px rgba(59, 130, 246, 0.4);

    /* Scrollbar */
    --scrollbar-track: #2d3748;
    --scrollbar-thumb: #4a5568;
    --scrollbar-thumb-hover: #6b7280;
  }

  /* Estilos globales para el body */
  :global(body) {
    font-family: "Inter", sans-serif;
    background-color: var(--color-table-bg);
    margin: 0;
  }

  /* ---------------------------------- */
  /* Estilos principales y Layout */
  /* ---------------------------------- */

  .main {
    min-height: calc(100vh - 160px);
    padding: 60px 20px;
    background: var(--color-table-bg);
    color: var(--color-text-light);
    font-family: "Inter", sans-serif;
  }

  .content-container {
    max-width: 1200px;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
    gap: 30px;
  }

  .page-title {
    text-align: center;
    font-size: clamp(2.2rem, 4vw, 3.2rem);
    color: var(--color-accent-gold);
    margin-bottom: 20px;
    font-weight: 900;
    text-transform: uppercase;
    letter-spacing: 1px;
    text-shadow: 0 0 10px rgba(252, 211, 77, 0.2);
  }

  /* ---------------------------------- */
  /* NAVEGACIÓN TIPO PESTAÑA (TABS) */
  /* ---------------------------------- */
  .category-tabs {
    display: flex;
    justify-content: center;
    gap: 15px;
    margin-bottom: 30px;
    flex-wrap: wrap;
    align-self: center;
    max-width: 900px;
    width: 100%;
  }

  .tab-btn {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    padding: 12px 25px;
    border: 2px solid var(--color-border);
    border-radius: 8px;
    background-color: var(--color-card-bg);
    color: var(--color-text-muted);
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
    font-size: 1rem;
    min-width: 180px; /* Ajuste para que se vea bien en 3 columnas */
  }

  .tab-btn:hover {
    background-color: #2a3547;
    color: var(--color-text-light);
    border-color: var(--color-accent-blue);
  }

  .tab-btn.active {
    border-color: var(--color-accent-gold);
    background-color: var(--color-accent-gold);
    color: #111;
    font-weight: 700;
    box-shadow: 0 0 15px rgba(252, 211, 77, 0.5);
  }

  .tab-btn.active svg {
    color: #111;
  }

  .tab-btn:not(.active) svg {
    color: var(--color-accent-gold);
  }

  /* ---------------------------------- */
  /* Tarjeta Contenedora (Card) */
  /* ---------------------------------- */

  .card {
    background: var(--color-bg-primary);
    border: 1px solid var(--color-border);
    padding: clamp(20px, 4vw, 30px);
    border-radius: 12px;
    box-shadow: var(--shadow-card);
    align-self: center;
    width: 100%;
    max-width: 900px; /* Límite para el contenido de administración */
  }

  .card-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-bottom: 15px;
    margin-bottom: 15px;
    border-bottom: 1px solid var(--color-border);
  }

  .subtitulo {
    color: var(--color-accent-blue);
    font-size: clamp(1.4rem, 2.5vw, 1.8rem);
    margin: 0;
    font-weight: 700;
    display: flex;
    align-items: center;
    gap: 10px;
  }

  .section-title {
    color: var(--color-accent-gold);
    font-size: 1.2rem;
    font-weight: 600;
    margin-bottom: 15px;
    padding-bottom: 5px;
    border-bottom: 1px dashed var(--color-border);
  }

  .mt-40 {
    margin-top: 40px;
  }

  /* Botón de alternar (Crear/Cancelar) */
  .btn-toggle {
    background: var(--color-accent-gold);
    color: #111;
    border: none;
    padding: 14px 25px;
    border-radius: 8px;
    font-weight: 800;
    cursor: pointer;
    transition:
      background 0.3s,
      transform 0.1s,
      box-shadow 0.3s;
    font-size: 1rem;
    box-shadow: var(--shadow-btn);
  }
  .btn-toggle:hover:not(:disabled) {
    background: #ffaa2b;
    transform: translateY(-2px);
    box-shadow: 0 8px 20px rgba(252, 211, 77, 0.5);
  }

  .btn-toggle:disabled {
    background: var(--color-border);
    color: var(--color-text-muted);
    cursor: not-allowed;
    opacity: 0.7;
    box-shadow: none;
  }
  /* ---------------------------------- */
  /* Formulario */
  /* ---------------------------------- */

  .form-container {
    background: var(--color-card-bg);
    padding: 20px;
    border-radius: 10px;
    border: 1px solid #2a3547;
    margin-bottom: 20px;
  }

  .form-title {
    color: var(--color-accent-gold);
    font-size: 1.2rem;
    margin-top: 0;
    margin-bottom: 15px;
    padding-bottom: 5px;
    border-bottom: 1px solid var(--color-border);
  }

  .form-grid {
    display: grid;
    grid-template-columns: 1fr;
    gap: 15px;
  }

  @media (min-width: 640px) {
    .form-grid {
      grid-template-columns: repeat(2, 1fr);
    }
  }

  /* Estilos comunes para inputs y textareas */
  .input-field {
    box-sizing: border-box;
    width: 100%;
    padding: 14px 18px;
    background: var(--color-table-bg);
    color: var(--color-text-light);
    border: 1px solid var(--color-border);
    border-radius: 8px;
    transition: all 0.3s;
  }

  .input-field::placeholder {
    color: var(--color-text-muted);
  }

  .input-field:focus {
    border-color: var(--color-accent-gold);
    box-shadow: 0 0 0 3px rgba(252, 211, 77, 0.4);
    outline: none;
  }

  .desc-full {
    grid-column: 1 / -1; /* Ocupa todas las columnas */
    min-height: 100px;
    resize: vertical;
  }

  .mt-5 {
    margin-top: 5px;
  }

  /* Botón de Guardar Formulario */
  .btn-crear {
    background: var(--color-accent-gold);
    color: #111;
    border: none;
    padding: 14px 25px;
    border-radius: 8px;
    font-weight: 800;
    cursor: pointer;
    transition:
      background 0.3s,
      transform 0.1s,
      box-shadow 0.3s;
    font-size: 1rem;
    box-shadow: var(--shadow-btn-gold);
    margin-top: 20px;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    width: 100%;
  }

  .btn-crear:hover:not(:disabled) {
    background: #ffaa2b;
    transform: translateY(-2px);
    box-shadow: 0 8px 20px rgba(252, 211, 77, 0.5);
  }

  .btn-crear:disabled {
    background: var(--color-border);
    color: var(--color-text-muted);
    cursor: not-allowed;
    opacity: 0.7;
    box-shadow: none;
  }

  /* ---------------------------------- */
  /* Tabla */
  /* ---------------------------------- */
  .table-wrap {
    overflow-x: auto;
    max-height: 60vh;
    overflow-y: auto;
    border: 1px solid #334155;
    border-radius: 8px;
    margin: 0 auto;
  }

  /* Scrollbar - Sigue el estilo del ejemplo */
  .table-wrap::-webkit-scrollbar {
    width: 8px;
    height: 8px;
  }

  .table-wrap::-webkit-scrollbar-track {
    background: var(--scrollbar-track);
    border-radius: 10px;
  }

  .table-wrap::-webkit-scrollbar-thumb {
    background: var(--scrollbar-thumb);
    border-radius: 10px;
  }

  .table-wrap::-webkit-scrollbar-thumb:hover {
    background: var(--scrollbar-thumb-hover);
  }

  .tabla {
    width: 100%;
    min-width: 650px;
    border-collapse: collapse;
    background: var(--color-table-bg);
  }

  .tabla th {
    background: var(--color-primary);
    color: var(--color-text-light);
    padding: 15px;
    text-align: left;
    font-weight: 700;
    font-size: 0.95rem;
    text-transform: uppercase;
    position: sticky;
    top: 0;
    z-index: 5;
    letter-spacing: 0.5px;
  }

  .tabla td {
    padding: 12px 15px;
    border-bottom: 1px solid var(--color-border);
    font-size: 0.95rem;
    color: var(--color-text-light);
  }

  tr:hover td {
    background: rgba(255, 255, 255, 0.03);
  }

  .resaltado {
    color: var(--color-accent-gold);
    font-weight: 700;
  }

  /* Carga */
  .loading-container-bar {
    width: 100%;
    padding: 20px 0;
    display: flex;
    justify-content: center;
  }

  .loading-bar {
    width: 80%;
    max-width: 400px;
    height: 4px;
    background: var(--color-border);
    border-radius: 5px;
    overflow: hidden;
  }

  .loading-bar::before {
    content: "";
    display: block;
    height: 100%;
    width: 30%;
    background: linear-gradient(
      90deg,
      var(--color-accent-gold),
      var(--color-accent-blue)
    );
    animation: loading-move 1.5s infinite linear;
  }

  @keyframes loading-move {
    0% {
      transform: translateX(-100%);
    }
    100% {
      transform: translateX(333%);
    }
  }

  /* ---------------------------------- */
  /* ESTILOS DEL MENSAJE FLOTANTE (REPLICADO DEL LOGIN) */
  /* ---------------------------------- */
  .message-modal {
    position: fixed;
    top: 20px;
    right: 20px;
    z-index: 1000;
    animation: slideIn 0.5s forwards;
  }

  .message-content {
    padding: 15px 25px;
    border-radius: 8px;
    color: white;
    background-color: var(--color-danger); /* Usa el color de peligro */
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    gap: 15px;
    font-weight: 600;
    animation: bounce 0.5s ease-out; /* Animación de rebote al aparecer */
  }

  .message-content.success {
    background-color: var(--color-success); /* Usa el color de éxito */
  }

  .message-content i {
    font-size: 1.2rem;
  }

  @keyframes slideIn {
    from {
      transform: translateX(100%);
      opacity: 0;
    }
    to {
      transform: translateX(0);
      opacity: 1;
    }
  }

  /* Animación de rebote para el modal */
  @keyframes bounce {
    0%,
    100% {
      transform: translateY(0);
    }
    20% {
      transform: translateY(-5px);
    }
    40% {
      transform: translateY(0);
    }
    60% {
      transform: translateY(-2px);
    }
    80% {
      transform: translateY(0);
    }
  }

  /* ---------------------------------- */
  /* Responsividad (Móvil) */
  /* ---------------------------------- */
  @media (max-width: 768px) {
    .main {
      padding: 30px 10px;
    }

    .page-title {
      font-size: 1.8rem;
    }

    .card {
      padding: 20px 15px;
    }

    .subtitulo {
      font-size: 1.2rem;
    }

    .category-tabs {
      gap: 10px;
      width: 100%;
    }

    .tab-btn {
      font-size: 0.9rem;
      padding: 10px 15px;
      min-width: unset;
      flex: 1 1 45%;
    }

    .tabla th,
    .tabla td {
      padding: 10px;
    }

    .tabla {
      min-width: 550px;
    }
  }
</style>
