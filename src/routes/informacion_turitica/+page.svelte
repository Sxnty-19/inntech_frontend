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

  const API = "https://turismo-sm.onrender.com"; // ajusta si corre en otro puerto

  async function cargarEventos() {
    loading.eventos = true;
    error.eventos = null;
    try {
      const res = await fetch(`${API}/evento/`);
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      eventos = await res.json();
    } catch (e) {
      console.error("Error eventos:", e);
      error.eventos = "No se pudieron cargar los eventos.";
      eventos = [];
    } finally {
      loading.eventos = false;
    }
  }

  async function cargarLugares() {
    loading.lugares = true;
    error.lugares = null;
    try {
      const res = await fetch(`${API}/lugar/`);
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      lugares = await res.json();
    } catch (e) {
      console.error("Error lugares:", e);
      error.lugares = "No se pudieron cargar los lugares.";
      lugares = [];
    } finally {
      loading.lugares = false;
    }
  }

  async function cargarServicios() {
    loading.servicios = true;
    error.servicios = null;
    try {
      const res = await fetch(`${API}/servicio/`);
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      servicios = await res.json();
    } catch (e) {
      console.error("Error servicios:", e);
      error.servicios = "No se pudieron cargar los servicios.";
      servicios = [];
    } finally {
      loading.servicios = false;
    }
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
  <h1 class="page-title">Información Turística</h1>

  <!-- EVENTOS -->
  <div class="card">
    <h2 class="card-title">Eventos</h2>

    {#if loading.eventos}
      <p class="subtext">Cargando eventos...</p>
    {:else if error.eventos}
      <p class="error">{error.eventos}</p>
    {:else if eventos.length === 0}
      <p class="subtext">No hay eventos para mostrar.</p>
    {:else}
      <div class="table-wrap">
        <table class="tabla">
          <thead>
            <tr>
              <th>ID</th>
              <th>Nombre</th>
              <th>Fecha</th>
              <th>Ubicación</th>
              <th>Descripción</th>
            </tr>
          </thead>
          <tbody>
            {#each eventos as e}
              <tr>
                <td>{e.id ?? e._id ?? "-"}</td>
                <td>{e.nombre ?? e.title ?? "-"}</td>
                <td>{e.fecha ?? e.date ?? "-"}</td>
                <td>{e.ubicacion ?? e.location ?? "-"}</td>
                <td class="td-desc">{e.descripcion ?? e.description ?? "-"}</td>
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
      <p class="subtext">Cargando lugares...</p>
    {:else if error.lugares}
      <p class="error">{error.lugares}</p>
    {:else if lugares.length === 0}
      <p class="subtext">No hay lugares para mostrar.</p>
    {:else}
      <div class="table-wrap">
        <table class="tabla">
          <thead>
            <tr>
              <th>ID</th>
              <th>Nombre</th>
              <th>Ubicación</th>
              <th>Tipo</th>
              <th>Descripción</th>
            </tr>
          </thead>
          <tbody>
            {#each lugares as l}
              <tr>
                <td>{l.id ?? l._id ?? "-"}</td>
                <td>{l.nombre ?? l.name ?? "-"}</td>
                <td>{l.ubicacion ?? l.location ?? "-"}</td>
                <td>{l.tipo ?? l.type ?? "-"}</td>
                <td class="td-desc">{l.descripcion ?? l.description ?? "-"}</td>
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
      <p class="subtext">Cargando servicios...</p>
    {:else if error.servicios}
      <p class="error">{error.servicios}</p>
    {:else if servicios.length === 0}
      <p class="subtext">No hay servicios para mostrar.</p>
    {:else}
      <div class="table-wrap">
        <table class="tabla">
          <thead>
            <tr>
              <th>ID</th>
              <th>Nombre</th>
              <th>Categoría</th>
              <th>Teléfono / Contacto</th>
              <th>Descripción</th>
            </tr>
          </thead>
          <tbody>
            {#each servicios as s}
              <tr>
                <td>{s.id ?? s._id ?? "-"}</td>
                <td>{s.nombre ?? s.name ?? "-"}</td>
                <td>{s.categoria ?? s.category ?? "-"}</td>
                <td>{s.contacto ?? s.phone ?? "-"}</td>
                <td class="td-desc">{s.descripcion ?? s.description ?? "-"}</td>
              </tr>
            {/each}
          </tbody>
        </table>
      </div>
    {/if}
  </div>
</section>

<Footer />

<style>
  .main {
    min-height: calc(100vh - 160px);
    padding: 36px 20px;
    background: #000;
    color: white;
  }

  .page-title {
    text-align: center;
    font-size: 2.2rem;
    color: orange;
    margin-bottom: 22px;
  }

  .card {
    background: rgba(255,165,0,0.04);
    border: 2px solid orange;
    padding: 18px;
    border-radius: 10px;
    box-shadow: 0 0 12px rgba(255,165,0,0.06);
    margin: 18px auto;
    width: 95%;
    max-width: 1100px;
  }

  .card-title {
    color: orange;
    font-size: 1.25rem;
    margin-bottom: 12px;
  }

  .subtext {
    color: #ccc;
    margin: 12px 0;
  }

  .error {
    color: #ff6b6b;
    font-weight: 700;
    margin: 12px 0;
  }

  .table-wrap {
    overflow-x: auto;
    margin-top: 10px;
  }

  .tabla {
    width: 100%;
    border-collapse: collapse;
    background: #0f0f0f;
    color: #ddd;
  }

  .tabla thead th {
    padding: 12px 10px;
    background: rgba(255,165,0,0.2);
    color: black;
    text-align: left;
    font-weight: 700;
    position: sticky;
    top: 0;
    z-index: 1;
  }

  .tabla th, .tabla td {
    border-bottom: 1px solid rgba(255,165,0,0.08);
    padding: 10px;
    vertical-align: top;
  }

  .td-desc {
    max-width: 40ch;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  tr:hover td {
    background: rgba(255,165,0,0.03);
  }

  /* Responsive tweaks */
  @media (max-width: 800px) {
    .card {
      padding: 14px;
    }
    .page-title {
      font-size: 1.8rem;
    }
    .tabla thead th {
      font-size: 0.95rem;
    }
    .tabla th, .tabla td {
      padding: 8px;
      font-size: 0.95rem;
    }
  }
</style>