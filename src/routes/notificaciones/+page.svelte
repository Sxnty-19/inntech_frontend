<script>
  import { onMount } from "svelte";
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";

  const API = "https://inntech-backend.onrender.com/notificaciones";

  let notificaciones = [];
  let loading = true;
  let error = null;

  let numero_habitacion = "";
  let descripcion = "";

  // -- FIX AQUÍ --
  let user = null;
  let id_usuario = null;

  if (typeof localStorage !== "undefined") {
    const stored = localStorage.getItem("user");
    if (stored) {
      user = JSON.parse(stored);
      id_usuario = user.id_usuario;
    }
  }

  async function cargarNotificaciones() {
    loading = true;

    try {
      const res = await fetch(`${API}/get_notificaciones`);

      if (!res.ok) throw new Error("Error cargando notificaciones");

      const data = await res.json();

      if (!data.success) throw new Error(data.detail ?? "Error");

      notificaciones = data.data;
    } catch (e) {
      console.error(e);
      error = "No se pudieron cargar las notificaciones.";
    } finally {
      loading = false;
    }
  }

  async function crearNotificacion() {
    if (!numero_habitacion || !descripcion) {
      alert("Complete todos los campos.");
      return;
    }

    try {
      const formData = new FormData();
      formData.append("id_usuario", Number(id_usuario));
      formData.append("numero_habitacion", numero_habitacion);
      formData.append("descripcion", descripcion);
      formData.append("estado", 1);

      const res = await fetch(`${API}/crear_por_numero`, {
        method: "POST",
        body: formData
      });

      if (!res.ok) throw new Error("Error en la petición");

      const data = await res.json();

      if (!data.success) throw new Error(data.detail ?? "Error al crear notificación");

      alert("Notificación creada correctamente.");
      numero_habitacion = "";
      descripcion = "";

      cargarNotificaciones();

    } catch (e) {
      console.error(e);
      alert("No se pudo crear la notificación.");
    }
  }

  onMount(() => {
    cargarNotificaciones();
  });
</script>

<Navbar />
<NavbarA />

<section class="main">

  <h1 class="page-title">Notificaciones VIP</h1>

  <div class="card">

    <h2 class="subtitulo">Crear Notificación</h2>

    <div class="form">
      <input
        type="text"
        placeholder="Número de habitación"
        bind:value={numero_habitacion}
      />

      <textarea
        placeholder="Descripción"
        bind:value={descripcion}
      ></textarea>

      <button class="btn-crear" on:click={crearNotificacion}>
        Crear notificación
      </button>
    </div>

    <hr class="divisor" />

    <h2 class="subtitulo">Todas las Notificaciones</h2>

    {#if loading}
      <p class="subtext">Cargando notificaciones...</p>

    {:else if error}
      <p class="error">{error}</p>

    {:else}
      <div class="table-wrap">
        <table class="tabla">
          <thead>
            <tr>
              <th>ID</th>
              <th>Usuario</th>
              <th>Habitación</th>
              <th>Descripción</th>
              <th>Estado</th>
              <th>Fecha</th>
            </tr>
          </thead>
          <tbody>
            {#each notificaciones as n}
              <tr>
                <td>{n.id_notificacion}</td>
                <td>{n.id_usuario}</td>
                <td>{n.id_habitacion}</td>
                <td>{n.descripcion}</td>
                <td>
                  {#if n.estado === 1}
                    <span class="estado-activa">Activa</span>
                  {:else}
                    <span class="estado-cerrada">Cerrada</span>
                  {/if}
                </td>
                <td>{n.date_created}</td>
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

  .subtitulo {
    color: orange;
    font-size: 1.3rem;
    margin-bottom: 10px;
  }

  .form input,
  .form textarea {
    width: 100%;
    padding: 10px;
    margin-bottom: 12px;
    background: #0f0f0f;
    color: #fff;
    border: 1px solid orange;
    border-radius: 6px;
  }

  .form textarea {
    height: 90px;
  }

  .btn-crear {
    background: orange;
    color: black;
    border: none;
    padding: 10px 14px;
    border-radius: 6px;
    font-weight: bold;
    cursor: pointer;
    transition: 0.3s;
  }

  .btn-crear:hover {
    background: #ffaa2b;
  }

  .divisor {
    border: 0;
    border-bottom: 1px solid rgba(255,165,0,0.2);
    margin: 25px 0;
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

  .estado-activa {
    color: #00ff99;
    font-weight: bold;
  }

  .estado-cerrada {
    color: #ff6b6b;
    font-weight: bold;
  }

  tr:hover td {
    background: rgba(255,165,0,0.03);
  }

</style>