<script>
  import { onMount } from "svelte";
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";

  let eventos = [];
  let lugares = [];
  let servicios = [];

  let loading = { eventos: true, lugares: true, servicios: true };
  let error = { eventos: null, lugares: null, servicios: null };

  // 1. Nueva variable de estado para controlar la categoría visible
  // Por defecto, muestra 'eventos' al cargar.
  let currentCategory = "eventos";

  const API = "https://turismo-sm.onrender.com";

  // Función genérica para manejar la carga y reintentos
  async function cargarData(endpoint, category) {
    loading[category] = true;
    error[category] = null;
    try {
      // Implementación de reintentos con backoff exponencial
      const MAX_RETRIES = 3;
      for (let i = 0; i < MAX_RETRIES; i++) {
        const res = await fetch(`${API}/${endpoint}/`);

        if (res.ok) {
          const data = await res.json();
          return data;
        }

        if (i < MAX_RETRIES - 1) {
          await new Promise((resolve) =>
            setTimeout(resolve, Math.pow(2, i) * 1000),
          );
        } else {
          // Si el último intento falla
          throw new Error(`Error HTTP ${res.status}. No se pudo conectar.`);
        }
      }
    } catch (e) {
      console.error(`Error al cargar ${category}:`, e);
      error[category] = `No se pudieron cargar los ${category}.`;
      return [];
    } finally {
      loading[category] = false;
    }
  }

  async function cargarEventos() {
    eventos = await cargarData("evento", "eventos");
  }

  async function cargarLugares() {
    lugares = await cargarData("lugar", "lugares");
  }

  async function cargarServicios() {
    servicios = await cargarData("servicio", "servicios");
  }

  onMount(() => {
    cargarEventos();
    cargarLugares();
    cargarServicios();
  });

  /**
   * Helper para obtener el valor de un campo, priorizando nombres en español/inglés.
   * @param {Object} item - Objeto de datos (evento, lugar, servicio).
   * @param {string} esKey - Clave en español (e.g., 'nombre').
   * @param {string} enKey - Clave en inglés (e.g., 'name').
   * @returns {string} El valor formateado o '-' si no existe.
   */
  const getFieldValue = (item, esKey, enKey) =>
    item[esKey] ?? item[enKey] ?? "-";
</script>

<Navbar />
<NavbarA />

<section class="main">
  <div class="main-content">
    <h1 class="page-title">Información Turística</h1>
    <p class="intro-text">
      Descubra los eventos, lugares y servicios disponibles para enriquecer su
      visita. Toda la información se actualiza en tiempo real desde la
      plataforma de turismo.
    </p>

    <!-- 2. Botones de Navegación/Filtro -->
    <div class="category-tabs">
      <button
        class="tab-btn"
        class:active={currentCategory === "eventos"}
        on:click={() => (currentCategory = "eventos")}
      >
        Eventos Destacados
      </button>
      <button
        class="tab-btn"
        class:active={currentCategory === "lugares"}
        on:click={() => (currentCategory = "lugares")}
      >
        Lugares Turísticos
      </button>
      <button
        class="tab-btn"
        class:active={currentCategory === "servicios"}
        on:click={() => (currentCategory = "servicios")}
      >
        Servicios
      </button>
    </div>

    <!-- 3. Contenedores condicionales según currentCategory -->
    <div class="content-display">
      <!-- EVENTOS SECTION -->
      {#if currentCategory === "eventos"}
        <div class="info-section">
          <h2 class="section-title">Eventos Destacados</h2>

          {#if loading.eventos}
            <div class="loading-bar"></div>
          {:else if error.eventos}
            <p class="message error-msg">{error.eventos}</p>
          {:else if eventos.length === 0}
            <p class="message subtext">No hay eventos próximos para mostrar.</p>
          {:else}
            <div class="card-grid">
              {#each eventos as e}
                <div class="card event-card">
                  <h3 class="card-name">
                    {getFieldValue(e, "nombre", "title")}
                  </h3>
                  <div class="card-detail">
                    <span class="detail-label">Fecha:</span>
                    <span class="detail-value"
                      >{getFieldValue(e, "fecha_inicio", "date")}</span
                    >
                  </div>
                  <div class="card-detail">
                    <span class="detail-label">Ubicación:</span>
                    <span class="detail-value"
                      >{getFieldValue(e, "ubicacion", "location")}</span
                    >
                  </div>
                  <div class="card-description">
                    <p class="description-text">
                      {getFieldValue(e, "descripcion", "description")}
                    </p>
                  </div>
                </div>
              {/each}
            </div>
          {/if}
        </div>
      {/if}

      <!-- LUGARES SECTION -->
      {#if currentCategory === "lugares"}
        <div class="info-section">
          <h2 class="section-title">Lugares Turísticos</h2>

          {#if loading.lugares}
            <div class="loading-bar"></div>
          {:else if error.lugares}
            <p class="message error-msg">{error.lugares}</p>
          {:else if lugares.length === 0}
            <p class="message subtext">
              No hay lugares turísticos para mostrar.
            </p>
          {:else}
            <div class="card-grid">
              {#each lugares as l}
                <div class="card place-card">
                  <h3 class="card-name">
                    {getFieldValue(l, "nombre", "name")}
                  </h3>
                  <div class="card-detail">
                    <span class="detail-label">Tipo:</span>
                    <span class="detail-value"
                      >{getFieldValue(l, "tipo", "type")}</span
                    >
                  </div>
                  <div class="card-detail">
                    <span class="detail-label">Ubicación:</span>
                    <span class="detail-value"
                      >{getFieldValue(l, "ubicacion", "location")}</span
                    >
                  </div>
                  <div class="card-description">
                    <p class="description-text">
                      {getFieldValue(l, "descripcion", "description")}
                    </p>
                  </div>
                </div>
              {/each}
            </div>
          {/if}
        </div>
      {/if}

      <!-- SERVICIOS SECTION -->
      {#if currentCategory === "servicios"}
        <div class="info-section">
          <h2 class="section-title">Servicios</h2>

          {#if loading.servicios}
            <div class="loading-bar"></div>
          {:else if error.servicios}
            <p class="message error-msg">{error.servicios}</p>
          {:else if servicios.length === 0}
            <p class="message subtext">No hay servicios disponibles.</p>
          {:else}
            <div class="card-grid">
              {#each servicios as s}
                <div class="card service-card">
                  <h3 class="card-name">
                    {getFieldValue(s, "nombre", "name")}
                  </h3>
                  <div class="card-detail">
                    <span class="detail-label">Categoría:</span>
                    <span class="detail-value"
                      >{getFieldValue(s, "categoria", "category")}</span
                    >
                  </div>
                  <div class="card-detail">
                    <span class="detail-label">Contacto:</span>
                    <span class="detail-value"
                      >{getFieldValue(s, "contacto", "phone")}</span
                    >
                  </div>
                  <div class="card-description">
                    <p class="description-text">
                      {getFieldValue(s, "descripcion", "description")}
                    </p>
                  </div>
                </div>
              {/each}
            </div>
          {/if}
        </div>
      {/if}
    </div>
  </div>
</section>

<Footer />

<style>
  /* ---------------------------------- */
  /* VARIABLES DE COLOR - UNIFICADAS */
  /* ---------------------------------- */
  :global(:root) {
    --color-dark: #0f172a; /* Fondo muy oscuro (Unificado con Navbars) */
    --color-card-bg: #1a202c; /* Fondo de Tarjeta (Ligeramente más claro) */
    --color-primary: #1e3a8a; /* Azul Principal */
    --color-secondary: #fcd34d; /* Amarillo/Naranja de Resalte */
    --color-text-light: #e2e8f0;
    --color-error: #ef4444;
    --color-neutral: #9ca3af;
    --color-detail-label: #a0a7af; /* Gris para etiquetas de detalle */
    --shadow-card: 0 4px 10px rgba(0, 0, 0, 0.3);
    --color-highlight: #60a5fa; /* Azul suave para IDs y destacados */
  }

  /* FIX CRÍTICO: Aplicamos el fondo oscuro globalmente al viewport. */
  :global(body),
  :global(html) {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    background-color: var(--color-card-bg);
  }

  /* ---------------------------------- */
  /* LAYOUT PRINCIPAL */
  /* ---------------------------------- */
  .main {
    background: var(--color-card-bg);
    min-height: calc(100vh - 160px);
    padding: 30px 20px;
    color: var(--color-text-light);
    font-family: "Inter", sans-serif;
  }
  .main-content {
    max-width: 1200px;
    margin: 0 auto;
  }
  .page-title {
    text-align: center;
    color: var(--color-secondary);
    margin-bottom: 10px;
    font-size: 2.4rem;
    font-weight: 800;
  }
  .intro-text {
    text-align: center;
    color: var(--color-neutral);
    margin-bottom: 40px;
    font-size: 1rem;
    max-width: 700px;
    margin-left: auto;
    margin-right: auto;
  }

  /* ---------------------------------- */
  /* NAVEGACIÓN POR PESTAÑAS (TABS) */
  /* ---------------------------------- */
  .category-tabs {
    display: flex;
    justify-content: center;
    gap: 15px;
    /* Reducimos el margen inferior aquí para que se vea más pegado al contenido */
    margin-bottom: 35px;
    flex-wrap: wrap;
  }

  .tab-btn {
    padding: 12px 25px;
    /* Usamos color-border para el borde, que es un gris oscuro */
    border: 2px solid var(--color-border);
    border-radius: 8px;
    background-color: var(--color-dark);
    color: var(--color-neutral);
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
    font-size: 1rem;
    display: flex;
    align-items: center;
    text-transform: none; /* Quitamos uppercase en los botones de pestaña */
  }

  .tab-btn:hover {
    background-color: #2a3547; /* Un poco más claro en hover */
    color: var(--color-text-light);
    transform: translateY(-2px); /* Pequeño levantamiento en hover */
  }

  .tab-btn.active {
    border-color: var(--color-primary);
    background-color: var(--color-primary);
    color: var(--color-text-light);
    /* Sombra azul brillante solicitada */
    box-shadow: 0 0 15px rgba(59, 130, 246, 0.8);
    transform: translateY(0); /* Evita que el botón activo se levante */
  }

  /* ---------------------------------- */
  /* SECCIONES Y TÍTULOS */
  /* ---------------------------------- */
  .info-section {
    margin-bottom: 40px;
  }

  .section-title {
    color: var(
      --color-secondary
    ); /* Cambiado a secondary para mejor contraste */
    text-align: center;
    font-size: 2rem;
    font-weight: 700;
    margin-bottom: 25px;
    padding-bottom: 10px;
    border-bottom: 2px solid var(--color-card-bg);
    max-width: 500px;
    margin-left: auto;
    margin-right: auto;
  }

  /* ---------------------------------- */
  /* GRID DE TARJETAS (CARDS) */
  /* ---------------------------------- */
  .card-grid {
    display: grid;
    /* 3 columnas en desktop, se ajusta a 1 en móvil */
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: 25px;
  }

  .card {
    background: var(--color-dark);
    padding: 20px;
    border-radius: 12px;
    box-shadow: var(--shadow-card);
    transition:
      transform 0.3s ease,
      box-shadow 0.3s ease;
    border: 1px solid rgba(255, 255, 255, 0.05);
    display: flex;
    flex-direction: column;
  }

  .card:hover {
    transform: translateY(-3px);
    box-shadow: 0 8px 15px rgba(0, 0, 0, 0.5);
  }

  /* ---------------------------------- */
  /* CONTENIDO DE LA TARJETA */
  /* ---------------------------------- */
  .card-name {
    font-size: 1.3rem;
    font-weight: 700;
    color: var(--color-secondary);
    margin-bottom: 15px;
    border-bottom: 1px dashed rgba(255, 255, 255, 0.1);
    padding-bottom: 10px;
  }

  .card-detail {
    display: flex;
    justify-content: space-between;
    padding: 5px 0;
    font-size: 0.95rem;
  }

  .detail-label {
    color: var(--color-detail-label);
    font-weight: 500;
  }

  .detail-value {
    color: var(--color-text-light);
    font-weight: 400;
    text-align: right;
    max-width: 60%; /* Permite que el valor sea más corto si es necesario */
  }

  .card-description {
    margin-top: 15px;
    padding-top: 15px;
    border-top: 1px solid rgba(255, 255, 255, 0.05);
    flex-grow: 1; /* Empuja el footer o contenido restante hacia abajo */
  }

  .description-text {
    font-size: 0.9rem;
    color: var(--color-neutral);
    line-height: 1.5;
  }

  /* ---------------------------------- */
  /* ESTADOS (LOADING/ERROR) */
  /* ---------------------------------- */
  .message {
    padding: 10px 0;
    font-weight: 500;
    text-align: center;
  }
  .error-msg {
    color: var(--color-error);
  }
  .subtext {
    color: var(--color-neutral);
  }

  .loading-bar {
    height: 4px;
    width: 100%;
    border-radius: 2px;
    overflow: hidden;
  }
  .loading-bar:before {
    content: "";
    display: block;
    height: 100%;
    width: 50%;
    background: var(--color-secondary);
    animation: loading-animate 1.5s infinite linear;
  }
  @keyframes loading-animate {
    0% {
      transform: translateX(-100%);
    }
    100% {
      transform: translateX(200%);
    }
  }

  /* ---------------------------------- */
  /* RESPONSIVIDAD (Móvil) */
  /* ---------------------------------- */
  @media (max-width: 768px) {
    .main {
      padding: 15px;
    }

    .category-tabs {
      gap: 10px;
      margin-bottom: 30px;
    }

    .tab-btn {
      padding: 10px 18px;
      font-size: 0.9rem;
    }

    .card-grid {
      /* En móvil, solo una columna por defecto */
      grid-template-columns: 1fr;
      gap: 20px;
    }

    .section-title {
      font-size: 1.5rem;
    }

    .card {
      padding: 15px;
    }

    .card-name {
      font-size: 1.2rem;
    }
  }
</style>
