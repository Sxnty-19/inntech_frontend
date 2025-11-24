<script>
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";
  import { onMount } from "svelte";
    import Navbar from "$lib/components/Navbar.svelte";

  let id_usuario = "";         // ID del usuario (ADMIN)
  let date_start = "";
  let date_end = "";
  let availRooms = [];
  let selectedRooms = [];
  let reservas = [];
  let error = "";
  let mensaje = "";

  // Cargar TODAS las reservas
  async function cargarReservas() {
    try {
      const res = await fetch("https://inntech-backend.onrender.com/reservas/get_reservas");
      const data = await res.json();
      if (res.ok) reservas = data.data;
      else reservas = [];
    } catch (e) {
      console.error(e);
      reservas = [];
    }
  }

  onMount(() => {
    cargarReservas();
  });

  // Buscar habitaciones disponibles
  async function buscarHabitaciones() {
    error = "";
    mensaje = "";
    availRooms = [];

    if (!date_start || !date_end) {
      error = "Seleccione fechas.";
      return;
    }

    if (new Date(date_end) <= new Date(date_start)) {
      error = "La fecha fin debe ser mayor que la fecha inicio.";
      return;
    }

    try {
      const res = await fetch(
        `https://inntech-backend.onrender.com/habitaciones/habitaciones_disponibles?date_start=${date_start}&date_end=${date_end}`
      );
      const data = await res.json();

      if (!res.ok) {
        error = data.detail || "Error buscando habitaciones.";
        return;
      }

      availRooms = data.data.map(h => ({
        id: h.id_habitacion ?? h.id ?? h.id_h,
        nombre: h.nombre ?? h.numero ?? `#${h.id_habitacion ?? h.id ?? h.id_h}`,
        raw: h
      }));

      if (availRooms.length === 0) {
        mensaje = "No hay habitaciones disponibles en esas fechas.";
      }
    } catch (e) {
      console.error(e);
      error = "No hay conexión con el servidor.";
    }
  }

  function agregarHab(id) {
    const hab = availRooms.find(h => h.id === id);
    if (!hab) {
      error = "Habitación no encontrada.";
      return;
    }
    if (selectedRooms.some(s => s.id === hab.id)) {
      mensaje = "La habitación ya fue seleccionada.";
      return;
    }
    selectedRooms = [...selectedRooms, hab];
    mensaje = "";
  }

  function quitarHab(id) {
    selectedRooms = selectedRooms.filter(s => s.id !== id);
  }

  // Crear reserva ADMIN
  async function confirmarReserva() {
    error = "";
    mensaje = "";

    if (!id_usuario) {
      error = "Debe digitar el ID del usuario.";
      return;
    }

    if (!date_start || !date_end) {
      error = "Complete las fechas.";
      return;
    }

    if (new Date(date_end) <= new Date(date_start)) {
      error = "Fecha fin debe ser mayor a fecha inicio.";
      return;
    }

    if (selectedRooms.length === 0) {
      error = "Seleccione al menos una habitación.";
      return;
    }

    const payload = {
      id_usuario: Number(id_usuario),
      date_start,
      date_end,
      habitaciones: selectedRooms.map(s => Number(s.id))
    };

    try {
      const res = await fetch(
        "https://inntech-backend.onrender.com/reservas/create_with_rooms",
        {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(payload)
        }
      );

      const data = await res.json();

      if (!res.ok) {
        error = data.detail || data.message || "Error creando reserva.";
        return;
      }

      mensaje = "Reserva creada correctamente.";
      id_usuario = "";
      date_start = "";
      date_end = "";
      selectedRooms = [];
      availRooms = [];
      await cargarReservas();
    } catch (e) {
      console.error(e);
      error = "No hay conexión con el servidor.";
    }
  }

  // Cancelar reserva
  async function cancelarReserva(id) {
    error = "";
    mensaje = "";
    try {
      const res = await fetch(
        `https://inntech-backend.onrender.com/reservas/cancelar/${id}`,
        { method: "PUT" }
      );
      const data = await res.json();

      if (!res.ok) {
        error = data.detail || "No se pudo cancelar.";
        return;
      }

      mensaje = "Reserva cancelada.";
      await cargarReservas();
    } catch (e) {
      console.error(e);
      error = "Error de conexión.";
    }
  }
</script>

<Navbar />
<NavbarA />

<section class="main">
  <h1 class="title">Gestión de Reservas (ADMIN)</h1>

  {#if error}
    <div class="error">{error}</div>
  {/if}

  {#if mensaje}
    <div class="success">{mensaje}</div>
  {/if}

  <!-- FORMULARIO ADMIN -->
  <div class="card">
    <h2>Crear Reserva</h2>

    <div class="form-grid">
      <div>
        <label>ID Usuario</label>
        <input type="number" bind:value={id_usuario} placeholder="Ej: 10" />
      </div>

      <div>
        <label>Fecha inicio</label>
        <input type="date" bind:value={date_start} />
      </div>

      <div>
        <label>Fecha fin</label>
        <input type="date" bind:value={date_end} />
      </div>

      <div style="align-self:end;">
        <button class="btn-search" on:click={buscarHabitaciones}>
          Buscar Habitaciones
        </button>
      </div>
    </div>

    {#if availRooms.length > 0}
      <h3>Habitaciones disponibles</h3>
      <ul class="list">
        {#each availRooms as hab}
          <li>
            <button class="btn-add" on:click={() => agregarHab(hab.id)}>Agregar</button>
            {hab.nombre} (ID: {hab.id})
          </li>
        {/each}
      </ul>
    {/if}

    <h3>Habitaciones seleccionadas</h3>
    {#if selectedRooms.length === 0}
      <p>No seleccionadas.</p>
    {:else}
      <ul class="list">
        {#each selectedRooms as s}
          <li>
            {s.nombre} (ID {s.id})
            <button class="btn-remove" on:click={() => quitarHab(s.id)}>
              Quitar
            </button>
          </li>
        {/each}
      </ul>
    {/if}

    <div style="margin-top:12px;">
      <button class="btn-confirm" on:click={confirmarReserva} disabled={selectedRooms.length === 0 || !id_usuario}>
        Confirmar Reserva
      </button>
    </div>
  </div>

  <!-- TODAS LAS RESERVAS -->
  <div class="card">
    <h2>Todas las reservas</h2>

    {#if reservas.length === 0}
      <p>No hay reservas.</p>
    {:else}
      <table>
        <thead>
          <tr>
            <th>ID</th>
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
              <td>{r.estado}</td>
              <td>
                <button class="btn-cancel" on:click={() => cancelarReserva(r.id_reserva)}>
                  Cancelar
                </button>
              </td>
            </tr>
          {/each}
        </tbody>
      </table>
    {/if}
  </div>
</section>

<Footer />

<style>
  .main {
    padding: 40px;
    background: #000;
    min-height: calc(100vh - 160px);
    color: white;
  }
  .title {
    text-align: center;
    color: orange;
    margin-bottom: 20px;
  }
  .card {
    background: rgba(255, 255, 255, 0.05);
    padding: 25px;
    margin: 20px auto;
    border-radius: 10px;
    border: 1px solid orange;
    width: 90%;
    max-width: 900px;
  }
  .form-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
    gap: 15px;
    align-items: center;
  }
  input[type="date"],
  input[type="number"] {
    padding: 8px;
    border-radius: 6px;
    border: none;
    background: #111;
    color: white;
    width: 100%;
  }
  .btn-search,
  .btn-confirm,
  .btn-add,
  .btn-remove,
  .btn-cancel {
    padding: 8px 15px;
    border: none;
    background: orange;
    color: black;
    border-radius: 6px;
    cursor: pointer;
    font-weight: bold;
  }
  .btn-remove,
  .btn-cancel {
    background: red;
    color: white;
  }
  .btn-confirm[disabled] {
    opacity: 0.5;
    cursor: not-allowed;
  }
  .list {
    margin-top: 10px;
  }
  .list li {
    margin-bottom: 8px;
    display:flex;
    gap:10px;
    align-items:center;
  }
  .error {
    background: red;
    padding: 10px;
    border-radius: 6px;
    font-weight: bold;
    text-align: center;
  }
  .success {
    background: green;
    padding: 10px;
    border-radius: 6px;
    font-weight: bold;
    text-align: center;
  }
  table {
    width: 100%;
    border-collapse: collapse;
    color: white;
    margin-top: 12px;
  }
  th, td {
    padding: 10px;
    border-bottom: 1px solid orange;
    text-align: left;
  }
  select {
    background: #111;
    color: white;
    border: 1px solid #444;
    border-radius: 6px;
  }
  option {
    background: #111;
  }

  /* responsive */
  @media (max-width: 700px) {
    .form-grid { grid-template-columns: 1fr; }
    table, thead, tbody, th, td, tr { display: block; }
    thead { display: none; }
    tr { margin-bottom: 12px; border: 1px solid rgba(255,255,255,0.03); padding: 10px; border-radius:6px; }
    td { border: none; padding: 6px 0; }
  }
</style>