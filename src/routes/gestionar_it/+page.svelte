<script>
  import { onMount } from "svelte";
  import { goto } from "$app/navigation";
    import Footer from "$lib/components/Footer.svelte";
    import Navbar from "$lib/components/Navbar.svelte";
    import NavbarA from "$lib/components/NavbarA.svelte";

  const API = "https://turismo-sm.onrender.com";

  /* ========================= EVENTOS ========================= */

  let eventos = [];
  let loadingEventos = true;
  let crearEvento = false;

  let formEvento = {
    nombre: "",
    fecha: "",
    ubicacion: "",
    descripcion: ""
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
    const body = {
      nombre: formEvento.nombre,
      fecha_inicio: formEvento.fecha,
      ubicacion: formEvento.ubicacion,
      descripcion: formEvento.descripcion
    };

    try {
      const res = await fetch(`${API}/evento/`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(body)
      });

      if (!res.ok) throw new Error();

      await cargarEventos();
      crearEvento = false;

      formEvento = { nombre: "", fecha: "", ubicacion: "", descripcion: "" };
    } catch {
      alert("Error guardando evento");
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
    descripcion: ""
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
    const body = {
      nombre: formLugar.nombre,
      tipo: formLugar.tipo,
      ubicacion: formLugar.ubicacion,
      descripcion: formLugar.descripcion
    };

    try {
      const res = await fetch(`${API}/lugar/`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(body)
      });

      if (!res.ok) throw new Error();

      await cargarLugares();
      crearLugar = false;

      formLugar = { nombre: "",tipo: "", ubicacion: "", descripcion: "" };
    } catch {
      alert("Error guardando lugar");
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
    descripcion: ""
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
    const body = {
      nombre: formServicio.nombre,
      categoria: formServicio.categoria,
      contacto: formServicio.contacto,
      descripcion: formServicio.descripcion
    };

    try {
      const res = await fetch(`${API}/servicio/`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(body)
      });

      if (!res.ok) throw new Error();

      await cargarServicios();
      crearServicio = false;

      formServicio = { nombre: "", categoria: "",contacto: "", descripcion: "" };
    } catch {
      alert("Error guardando servicio");
    }
  }

  /* ======== CARGAR TODO AL INICIAR ======== */

  onMount(() => {
    cargarEventos();
    cargarLugares();
    cargarServicios();
  });
</script>

<Navbar />
<NavbarA />

<div class="page-container">

  <!-- ===================================================== -->
  <!-- ====================== EVENTOS ====================== -->
  <!-- ===================================================== -->
  <div class="card">
    <div class="card-header">
      <h2 class="card-title">Eventos</h2>
      <button class="btn-create" on:click={() => (crearEvento = !crearEvento)}>
        {crearEvento ? "Cerrar" : "+ Crear Evento"}
      </button>
    </div>

    {#if crearEvento}
      <div class="formulario-crear">
        <h3 class="form-title">Nuevo Evento</h3>

        <div class="form-grid">
          <label>Nombre: <input type="text" bind:value={formEvento.nombre} /></label>
          <label>Fecha: <input type="date" bind:value={formEvento.fecha} /></label>
          <label>Ubicación: <input type="text" bind:value={formEvento.ubicacion} /></label>

          <label class="desc-full">
            Descripción:
            <textarea bind:value={formEvento.descripcion}></textarea>
          </label>
        </div>

        <button class="btn-save" on:click={guardarEvento}>Guardar Evento</button>
      </div>
    {/if}

    {#if loadingEventos}
      <p class="loading">Cargando eventos...</p>
    {:else}
      <table class="table">
        <thead>
          <tr>
            <th>Nombre</th><th>Fecha</th><th>Ubicación</th>
          </tr>
        </thead>
        <tbody>
          {#each eventos as item}
            <tr>
              <td>{item.nombre}</td>
              <td>{item.fecha_inicio}</td>
              <td>{item.ubicacion}</td>
            </tr>
          {/each}
        </tbody>
      </table>
    {/if}
  </div>


  <!-- ===================================================== -->
  <!-- ====================== LUGARES ====================== -->
  <!-- ===================================================== -->
  <div class="card">
    <div class="card-header">
      <h2 class="card-title">Lugares</h2>
      <button class="btn-create" on:click={() => (crearLugar = !crearLugar)}>
        {crearLugar ? "Cerrar" : "+ Crear Lugar"}
      </button>
    </div>

    {#if crearLugar}
      <div class="formulario-crear">
        <h3 class="form-title">Nuevo Lugar</h3>

        <div class="form-grid">
          <label>Nombre: <input type="text" bind:value={formLugar.nombre} /></label>
          <label>Tipo: <input type="text" bind:value={formLugar.tipo} /></label>
          <label>Ubicación: <input type="text" bind:value={formLugar.ubicacion} /></label>

          <label class="desc-full">
            Descripción:
            <textarea bind:value={formLugar.descripcion}></textarea>
          </label>
        </div>

        <button class="btn-save" on:click={guardarLugar}>Guardar Lugar</button>
      </div>
    {/if}

    {#if loadingLugares}
      <p class="loading">Cargando lugares...</p>
    {:else}
      <table class="table">
        <thead>
          <tr>
            <th>Nombre</th><th>Tipo</th><th>Ubicación</th>
          </tr>
        </thead>
        <tbody>
          {#each lugares as item}
            <tr>
              <td>{item.nombre}</td>
              <td>{item.tipo}</td>
              <td>{item.ubicacion}</td>
            </tr>
          {/each}
        </tbody>
      </table>
    {/if}
  </div>


  <!-- ===================================================== -->
  <!-- ===================== SERVICIOS ===================== -->
  <!-- ===================================================== -->
  <div class="card">
    <div class="card-header">
      <h2 class="card-title">Servicios</h2>
      <button class="btn-create" on:click={() => (crearServicio = !crearServicio)}>
        {crearServicio ? "Cerrar" : "+ Crear Servicio"}
      </button>
    </div>

    {#if crearServicio}
      <div class="formulario-crear">
        <h3 class="form-title">Nuevo Servicio</h3>

        <div class="form-grid">
          <label>Nombre: <input type="text" bind:value={formServicio.nombre} /></label>
          <label>Categoria: <input type="text" bind:value={formServicio.categoria} /></label>
          <label>Contacto: <input type="number" bind:value={formServicio.contacto} /></label>

          <label class="desc-full">
            Descripción:
            <textarea bind:value={formServicio.descripcion}></textarea>
          </label>
        </div>

        <button class="btn-save" on:click={guardarServicio}>Guardar Servicio</button>
      </div>
    {/if}

    {#if loadingServicios}
      <p class="loading">Cargando servicios...</p>
    {:else}
      <table class="table">
        <thead>
          <tr>
            <th>Nombre</th><th>Categoria</th><th>Contacto</th>
          </tr>
        </thead>
        <tbody>
          {#each servicios as item}
            <tr>
              <td>{item.nombre}</td>
              <td>{item.categoria}</td>
              <td>{item.contacto}</td>
            </tr>
          {/each}
        </tbody>
      </table>
    {/if}
  </div>

</div>
 <Footer />
<style>
  .page-container { padding: 20px; color: white; }

  .card {
    background: #111827;
    padding: 20px;
    border-radius: 12px;
    border: 1px solid rgba(255,255,255,0.1);
    margin-bottom: 30px;
  }

  .card-header { display: flex; justify-content: space-between; align-items: center; }

  .btn-create {
    background: #3b82f6; padding: 10px 14px; border-radius: 8px;
    color: white; border: none; cursor: pointer; font-weight: bold;
  }

  .btn-create:hover { background: #60a5fa; }

  .table { width: 100%; margin-top: 20px; border-collapse: collapse; }

  .table th, .table td {
    padding: 10px;
    border-bottom: 1px solid rgba(255,255,255,0.1);
  }

  .formulario-crear {
    background: #1b2534; padding: 20px; border-radius: 10px;
    border: 1px solid rgba(255,255,255,0.1); margin-top: 20px;
  }

  .form-grid { display: grid; grid-template-columns: repeat(2,1fr); gap: 15px; }

  .form-grid input, .form-grid textarea {
    margin-top: 5px; padding: 8px; border-radius: 6px;
    border: none; background: #0f172a; color: white;
  }

  .desc-full { grid-column: span 2; }

  .btn-save {
    width: 100%; padding: 12px; background: #3b82f6;
    color: white; border: none; border-radius: 8px; font-weight: bold;
    margin-top: 15px;
  }

  .btn-save:hover { background: #60a5fa; }
</style>