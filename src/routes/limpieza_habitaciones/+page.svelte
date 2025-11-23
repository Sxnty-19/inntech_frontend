<script>
  import { onMount } from "svelte";
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";

  const API = "https://inntech-backend.onrender.com/habitaciones";

  let habitaciones = [];
  let loading = true;
  let error = null;

  async function cargarHabitaciones() {
    loading = true;
    error = null;

    try {
      const res = await fetch(`${API}/get_habitaciones`);
      const data = await res.json();

      if (!data.success) throw new Error(data.detail ?? "Error");

      habitaciones = data.data;
    } catch (e) {
      console.error(e);
      error = "No se pudieron cargar las habitaciones.";
    } finally {
      loading = false;
    }
  }

  // ✅ CORREGIDO: enviando FormData como lo requiere FastAPI
  async function marcarLimpia(numero) {
    try {
      const formData = new FormData();
      formData.append("numero", numero);
      formData.append("estado", 1);

      const res = await fetch(`${API}/actualizar_estado`, {
        method: "PUT",
        body: formData
      });

      const data = await res.json();

      if (!data.success) {
        throw new Error(data.detail ?? "Error al actualizar.");
      }

      // Actualizar visualmente
      habitaciones = habitaciones.map(h =>
        h.numero === numero ? { ...h, estado: 1 } : h
      );

    } catch (e) {
      console.error(e);
      alert("No se pudo actualizar el estado.");
    }
  }

  onMount(() => {
    cargarHabitaciones();
  });
</script>

<Navbar />
<NavbarA />

<section class="main">

  <h1 class="page-title">Limpieza de Habitaciones</h1>

  <div class="card">

    {#if loading}
      <p class="subtext">Cargando habitaciones...</p>

    {:else if error}
      <p class="error">{error}</p>

    {:else}
      <div class="table-wrap">
        <table class="tabla">
          <thead>
            <tr>
              <th>Número</th>
              <th>Tipo</th>
              <th>Estado</th>
              <th>Acción</th>
            </tr>
          </thead>
          <tbody>
            {#each habitaciones as h}
              <tr>
                <td>{h.numero}</td>
                <td>{h.id_thabitacion}</td>

                <td>
                  {#if h.estado === 1}
                    <span class="estado-limpia">Limpia</span>
                  {:else}
                    <span class="estado-sucia">Necesita limpieza</span>
                  {/if}
                </td>

                <td>
                  {#if h.estado === 0}
                    <button class="btn-limpiar"
                      on:click={() => marcarLimpia(h.numero)}>
                      Marcar como limpia
                    </button>
                  {:else}
                    <span class="ok">✔</span>
                  {/if}
                </td>
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
    padding: 20px;
    border-radius: 10px;
    width: 95%;
    max-width: 1100px;
    margin: auto;
  }

  .subtext {
    color: #ccc;
    text-align: center;
  }

  .error {
    color: #ff6b6b;
    font-weight: bold;
    text-align: center;
  }

  .table-wrap {
    overflow-x: auto;
  }

  .tabla {
    width: 100%;
    border-collapse: collapse;
    background: #0f0f0f;
    color: #ddd;
  }

  .tabla th {
    background: rgba(255,165,0,0.2);
    color: black;
    padding: 12px;
    text-align: left;
  }

  .tabla td {
    padding: 10px;
    border-bottom: 1px solid rgba(255,165,0,0.1);
  }

  .estado-limpia {
    color: #00ff99;
    font-weight: bold;
  }

  .estado-sucia {
    color: #ff6b6b;
    font-weight: bold;
  }

  .btn-limpiar {
    background: orange;
    color: black;
    border: none;
    padding: 7px 10px;
    border-radius: 6px;
    font-weight: bold;
    cursor: pointer;
    transition: 0.3s;
  }

  .btn-limpiar:hover {
    background: #ffaa2b;
  }

  .ok {
    color: #00ff99;
    font-size: 1.3rem;
    font-weight: bold;
  }

  tr:hover td {
    background: rgba(255,165,0,0.03);
  }
</style>