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

    <!-- EVENTOS -->
    <div class="card">
      <h2 class="card-title">Eventos Destacados</h2>

      {#if loading.eventos}
        <div class="loading-bar"></div>
      {:else if error.eventos}
        <p class="message error-msg">{error.eventos}</p>
      {:else if eventos.length === 0}
        <p class="message subtext">No hay eventos próximos para mostrar.</p>
      {:else}
        <div class="table-wrap">
          <table class="tabla">
            <thead>
              <tr>
                <th>Nombre</th>
                <th>Fecha</th>
                <th>Ubicación</th>
                <th class="td-desc-header">Descripción</th>
              </tr>
            </thead>
            <tbody>
              {#each eventos as e}
                <tr>
                  <td data-label="Nombre">{e.nombre ?? e.title ?? "-"}</td>
                  <td data-label="Fecha">{e.fecha ?? e.date ?? "-"}</td>
                  <td data-label="Ubicación"
                    >{e.ubicacion ?? e.location ?? "-"}</td
                  >
                  <td data-label="Descripción" class="td-desc"
                    >{e.descripcion ?? e.description ?? "-"}</td
                  >
                </tr>
              {/each}
            </tbody>
          </table>
        </div>
      {/if}
    </div>

    <!-- LUGARES -->
    <div class="card">
      <h2 class="card-title">Lugares Turísticos</h2>

      {#if loading.lugares}
        <div class="loading-bar"></div>
      {:else if error.lugares}
        <p class="message error-msg">{error.lugares}</p>
      {:else if lugares.length === 0}
        <p class="message subtext">No hay lugares turísticos para mostrar.</p>
      {:else}
        <div class="table-wrap">
          <table class="tabla">
            <thead>
              <tr>
                <th>Nombre</th>
                <th>Tipo</th>
                <th>Ubicación</th>
                <th class="td-desc-header">Descripción</th>
              </tr>
            </thead>
            <tbody>
              {#each lugares as l}
                <tr>
                  <td data-label="Nombre">{l.nombre ?? l.name ?? "-"}</td>
                  <td data-label="Tipo">{l.tipo ?? l.type ?? "-"}</td>
                  <td data-label="Ubicación"
                    >{l.ubicacion ?? l.location ?? "-"}</td
                  >
                  <td data-label="Descripción" class="td-desc"
                    >{l.descripcion ?? l.description ?? "-"}</td
                  >
                </tr>
              {/each}
            </tbody>
          </table>
        </div>
      {/if}
    </div>

    <!-- SERVICIOS -->
    <div class="card">
      <h2 class="card-title">Servicios</h2>

      {#if loading.servicios}
        <div class="loading-bar"></div>
      {:else if error.servicios}
        <p class="message error-msg">{error.servicios}</p>
      {:else if servicios.length === 0}
        <p class="message subtext">No hay servicios disponibles.</p>
      {:else}
        <div class="table-wrap">
          <table class="tabla">
            <thead>
              <tr>
                <th>Nombre</th>
                <th>Categoría</th>
                <th>Contacto</th>
                <th class="td-desc-header">Descripción</th>
              </tr>
            </thead>
            <tbody>
              {#each servicios as s}
                <tr>
                  <td data-label="Nombre">{s.nombre ?? s.name ?? "-"}</td>
                  <td data-label="Categoría"
                    >{s.categoria ?? s.category ?? "-"}</td
                  >
                  <td data-label="Contacto">{s.contacto ?? s.phone ?? "-"}</td>
                  <td data-label="Descripción" class="td-desc"
                    >{s.descripcion ?? s.description ?? "-"}</td
                  >
                </tr>
              {/each}
            </tbody>
          </table>
        </div>
      {/if}
    </div>
  </div>
</section>

<Footer />

<style>
  /* Variables de Color (CSS Puro) - Mismo estilo que Reservas/Perfil */
  :root {
    --color-dark: #1f2937; /* Fondo de la página principal */
    --color-card: #0f172a; /* Fondo de las tarjetas */
    --color-primary: #1e3a8a; /* Azul Principal */
    --color-secondary: #fcd34d; /* Amarillo/Naranja de Resalte */
    --color-text-light: #e2e8f0;
    --color-error: #ef4444;
    --color-success: #10b981;
    --color-neutral: #9ca3af;
  }

  /* ---------------------------------- */
  /* LAYOUT PRINCIPAL */
  /* ---------------------------------- */
  .main {
    background: var(--color-dark);
    min-height: calc(100vh - 160px);
    padding: 30px 20px;
    color: var(--color-text-light);
    font-family: Arial, sans-serif;
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
    font-weight: bold;
  }
  .intro-text {
    text-align: center;
    color: var(--color-neutral);
    margin-bottom: 30px;
    font-size: 1rem;
  }

  /* ---------------------------------- */
  /* CARDS Y MENSAJES */
  /* ---------------------------------- */
  .card {
    background: var(--color-card);
    padding: 25px;
    margin-bottom: 30px;
    border-radius: 12px;
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
    border-top: 5px solid var(--color-primary);
  }
  .card-title {
    color: var(--color-primary);
    font-size: 1.5rem;
    margin-bottom: 20px;
    border-bottom: 1px solid rgba(255, 255, 255, 0.1);
    padding-bottom: 10px;
  }
  .message {
    padding: 10px 0;
    font-weight: 500;
  }
  .error-msg {
    color: var(--color-error);
  }
  .subtext {
    color: var(--color-neutral);
  }

  /* Loading State */
  .loading-bar {
    height: 4px;
    width: 100%;
    background: #374151;
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
  /* TABLAS (Misma Estructura Responsiva que Reservas) */
  /* ---------------------------------- */
  .table-wrap {
    overflow-x: auto;
    width: 100%;
  }
  .tabla {
    width: 100%;
    min-width: 600px;
    border-collapse: separate;
    border-spacing: 0 10px; /* Espacio entre filas */
    color: var(--color-text-light);
  }
  .tabla thead tr {
    background: #2d3748;
    font-size: 0.9rem;
    text-transform: uppercase;
    letter-spacing: 0.05em;
  }
  .tabla th,
  .tabla td {
    padding: 12px 15px;
    text-align: left;
    border: none;
  }
  .tabla tbody tr {
    background: #253141;
    border-radius: 8px;
    transition: background 0.2s;
  }
  .tabla tbody tr:hover {
    background: #2d3748;
  }

  /* Estilo para la columna de descripción */
  .td-desc-header {
    width: 30%; /* Ocupa más espacio para la descripción */
  }
  .td-desc {
    max-width: 250px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  /* ---------------------------------- */
  /* RESPONSIVIDAD (Media Queries para Móvil) */
  /* ---------------------------------- */
  @media (max-width: 768px) {
    .main {
      padding: 15px;
    }
    .page-title {
      font-size: 2rem;
    }
    .card {
      padding: 15px;
    }

    /* Ocultar el ID en móvil si es irrelevante para la vista compacta,
       pero lo mantendremos para evitar cambios en la estructura de datos
       que no se ve en el código original. Aquí se han quitado los IDs
       innecesarios en la vista principal para simplificar la tabla. */

    /* TABLA: Modo Responsivo (Tarjetas por Fila) */
    .tabla {
      min-width: 100%;
      display: block;
    }
    .tabla thead {
      display: none; /* Oculta el encabezado en móvil */
    }
    .tabla tbody,
    .tabla tr {
      display: block;
    }
    .tabla tr {
      margin-bottom: 15px;
      border: 1px solid rgba(255, 255, 255, 0.1);
      border-radius: 8px;
    }

    .tabla td {
      display: block;
      text-align: right;
      padding-left: 50%;
      position: relative;
    }

    .tabla td:before {
      /* Muestra la etiqueta del encabezado (th) */
      content: attr(data-label);
      position: absolute;
      left: 10px;
      width: 45%;
      padding-right: 10px;
      white-space: nowrap;
      text-align: left;
      font-weight: 600;
      color: var(--color-secondary);
    }

    /* La descripción debe poder envolver texto */
    .td-desc {
      white-space: normal;
      overflow: visible;
      text-overflow: clip;
      max-width: 100%; /* Asegura que la celda usa todo el ancho disponible */
    }
    .td-desc:before {
      /* Asegura que la etiqueta de descripción esté visible */
      line-height: 1.5;
      top: 10px;
    }
  }
</style>
