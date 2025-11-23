<script>
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";
  import { onMount } from "svelte";

  let reservas = [];
  let error = "";
  let mensaje = "";
  let loading = true;

  let showCreateForm = false;
  let date_start = "";
  let date_end = "";

  let user = null;
  if (typeof localStorage !== "undefined") {
    const storedUser = localStorage.getItem("user");
    if (storedUser) user = JSON.parse(storedUser);
  }

  // ==========================
  // Cargar todas las reservas
  // ==========================
  async function cargarReservas() {
    loading = true;
    try {
      const res = await fetch("https://inntech-backend.onrender.com/reservas/get_reservas");
      const data = await res.json();
      if (!res.ok) throw new Error(data.detail || "Error al obtener reservas.");
      reservas = data.data;
    } catch (err) {
      error = err.message;
    } finally {
      loading = false;
    }
  }

  onMount(() => {
    cargarReservas();
  });

  // ==========================
  // Cancelar reserva
  // ==========================
  async function cancelarReserva(id) {
    if (!confirm("¿Seguro quieres cancelar esta reserva?")) return;

    try {
      const res = await fetch(`https://inntech-backend.onrender.com/reservas/cancelar/${id}`, {
        method: "PUT"
      });
      const data = await res.json();
      if (!res.ok) throw new Error(data.detail || "No se pudo cancelar la reserva.");
      mensaje = "Reserva cancelada correctamente.";
      await cargarReservas();
    } catch (err) {
      error = err.message;
    }
  }

  // ==========================
  // Crear reserva
  // ==========================
  async function crearReserva() {
    if (!date_start || !date_end) {
      error = "Seleccione fecha inicio y fecha fin.";
      return;
    }

    const payload = {
      id_usuario: user.id_usuario,
      date_start,
      date_end
    };

    try {
      const res = await fetch("https://inntech-backend.onrender.com/reservas/create", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(payload)
      });
      const data = await res.json();
      if (!res.ok) throw new Error(data.detail || data.message || "Error al crear reserva.");

      mensaje = "Reserva creada correctamente.";
      date_start = "";
      date_end = "";
      showCreateForm = false;
      await cargarReservas();

    } catch (err) {
      error = err.message;
    }
  }

  function toggleForm() {
    showCreateForm = !showCreateForm;
    error = "";
    mensaje = "";
  }

  function estadoTexto(estado) {
    return estado === 1 ? "Activa" : "Cancelada";
  }
</script>

<Navbar />
<NavbarA />

<section class="main">
  <div class="box">
    <h2 class="title">Gestionar Reservas</h2>

    {#if error}
      <p class="error">{error}</p>
    {/if}
    {#if mensaje}
      <p class="success">{mensaje}</p>
    {/if}

    <button class="btn-toggle" on:click={toggleForm}>
      {showCreateForm ? "Cerrar formulario" : "Crear nueva reserva"}
    </button>

    {#if showCreateForm}
      <div class="create-form">
        <label>Fecha inicio</label>
        <input type="date" bind:value={date_start} />
        <label>Fecha fin</label>
        <input type="date" bind:value={date_end} />
        <button class="btn-confirm" on:click={crearReserva}>Crear reserva</button>
      </div>
    {/if}

    {#if loading}
      <div class="loader"></div>
    {:else}
      {#if reservas.length === 0}
        <p class="no-data">No hay reservas registradas.</p>
      {:else}
        <div class="table-container">
          <table>
            <thead>
              <tr>
                <th>ID Reserva</th>
                <th>ID Usuario</th>
                <th>Inicio</th>
                <th>Fin</th>
                <th>Estado</th>
                <th>Acción</th>
              </tr>
            </thead>
            <tbody>
              {#each reservas as r}
                <tr>
                  <td>{r.id_reserva}</td>
                  <td>{r.id_usuario}</td>
                  <td>{r.date_start}</td>
                  <td>{r.date_end}</td>
                  <td>{estadoTexto(r.estado)}</td>
                  <td>
                    {#if r.estado === 1}
                      <button class="btn-cancel" on:click={() => cancelarReserva(r.id_reserva)}>Cancelar</button>
                    {/if}
                  </td>
                </tr>
              {/each}
            </tbody>
          </table>
        </div>
      {/if}
    {/if}
  </div>
</section>

<Footer />

<style>
  .main {
    min-height: calc(100vh - 160px);
    padding: 50px 20px;
    background: #000;
    color: white;
    display: flex;
    justify-content: center;
  }

  .box {
    width: 100%;
    max-width: 1000px;
    background: rgba(255,165,0,0.08);
    border: 2px solid orange;
    padding: 30px;
    border-radius: 12px;
    box-shadow: 0 0 15px rgba(255,165,0,0.3);
  }

  .title {
    text-align: center;
    font-size: 2rem;
    color: orange;
    margin-bottom: 25px;
    font-weight: 700;
  }

  .btn-toggle {
    background: orange;
    border: none;
    padding: 10px 18px;
    border-radius: 8px;
    font-weight: 700;
    cursor: pointer;
    margin-bottom: 20px;
  }

  .create-form {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    margin-bottom: 20px;
  }

  .create-form label {
    width: 100%;
    text-align: left;
    font-weight: 600;
  }

  .create-form input {
    flex: 1;
    padding: 8px;
    border-radius: 6px;
    border: none;
  }

  .btn-confirm {
    background: orange;
    border: none;
    padding: 10px 18px;
    border-radius: 8px;
    font-weight: 700;
    cursor: pointer;
  }

  .btn-cancel {
    background: red;
    color: white;
    border: none;
    padding: 6px 12px;
    border-radius: 6px;
    font-weight: bold;
    cursor: pointer;
  }

  .loader {
    border: 5px solid rgba(255,165,0,0.3);
    border-top: 5px solid orange;
    border-radius: 50%;
    width: 45px;
    height: 45px;
    animation: spin 0.8s linear infinite;
    margin: 20px auto;
  }

  @keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
  }

  .error {
    color: red;
    text-align: center;
    font-size: 1.1rem;
  }

  .success {
    color: limegreen;
    text-align: center;
    font-size: 1.1rem;
  }

  .no-data {
    text-align: center;
    color: #ddd;
  }

  .table-container {
    overflow-x: auto;
  }

  table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 10px;
  }

  th {
    background: rgba(255,165,0,0.2);
    color: orange;
    padding: 12px;
    border-bottom: 2px solid orange;
    font-weight: 600;
  }

  td {
    padding: 12px;
    border-bottom: 1px solid rgba(255,165,0,0.3);
    color: white;
  }

  tr:hover td {
    background: rgba(255,165,0,0.15);
  }

  @media (max-width: 600px) {
    th, td, .create-form input {
      font-size: 0.85rem;
      padding: 8px;
    }
  }
</style>