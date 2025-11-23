<script>
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";
  import { onMount } from "svelte";

  let date_start = "";
  let date_end = "";
  let availRooms = [];         
  let selectedRooms = [];      
  let reservasActivas = [];
  let error = "";
  let mensaje = "";

  let user = null;   // <--- corregido, antes era const user = ...

  // Cargar reservas activas
  async function cargarReservas() {
    if (!user) return;
    try {
      const res = await fetch(`https://inntech-backend.onrender.com/reservas/activas/${user.id_usuario}`);
      const data = await res.json();
      if (res.ok) reservasActivas = data.data;
      else reservasActivas = [];
    } catch (e) {
      reservasActivas = [];
    }
  }

  onMount(() => {
    user = JSON.parse(localStorage.getItem("user"));   // <--- ahora aquí, CLIENT-SIDE ONLY
    cargarReservas();                                   // <--- ahora seguro
  });

  // ========================
  // Buscar habitaciones
  // ========================
  async function buscarHabitaciones() {
    error = "";
    mensaje = "";
    availRooms = [];
    try {
      if (!date_start || !date_end) {
        error = "Seleccione fecha inicio y fecha fin.";
        return;
      }
      if (new Date(date_end) <= new Date(date_start)) {
        error = "La fecha fin debe ser mayor que la fecha inicio.";
        return;
      }

      const res = await fetch(
        `https://inntech-backend.onrender.com/habitaciones/habitaciones_disponibles?date_start=${date_start}&date_end=${date_end}`
      );
      const data = await res.json();

      if (!res.ok) {
        error = data.detail || "Error al buscar habitaciones.";
        return;
      }

      // Esperamos que cada 'hab' tenga al menos id_habitacion y nombre (ajusta según tu API)
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

  // ========================
  // Agregar habitación individual
  // ========================
  function agregarHab(id) {
    // buscar objeto en availRooms
    const hab = availRooms.find(r => r.id === id);
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
    selectedRooms = selectedRooms.filter((x) => x.id !== id);
  }

  // ========================
  // Confirmar reserva
  // ========================
  async function confirmarReserva() {
    error = "";
    mensaje = "";

    if (!user) {
      error = "No autenticado.";
      return;
    }

    if (!date_start || !date_end) {
      error = "Las fechas son obligatorias.";
      return;
    }

    if (selectedRooms.length === 0) {
      error = "Debe seleccionar al menos una habitación.";
      return;
    }

    const habitacionesIds = selectedRooms.map(s => Number(s.id));

    const payload = {
      id_usuario: user.id_usuario,
      date_start,
      date_end,
      habitaciones: habitacionesIds
    };
    
    console.log(JSON.stringify(payload));


    try {
      const res = await fetch("https://inntech-backend.onrender.com/reservas/create_with_rooms", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(payload)
      });

      const data = await res.json();

      if (!res.ok) {
        error = data.detail || (data.message ? data.message : "Error al crear la reserva.");
        return;
      }

      mensaje = "Reserva creada exitosamente.";
      error = "";
      selectedRooms = [];
      availRooms = [];
      date_start = "";
      date_end = "";
      await cargarReservas();
    } catch (e) {
      console.error(e);
      error = "No hay conexión con el servidor.";
    }
  }

  // ========================
  // Cancelar reserva
  // ========================
  async function cancelarReserva(id) {
    error = "";
    mensaje = "";
    try {
      const res = await fetch(`http://127.0.0.1:8000/reservas/cancelar/${id}`, {
        method: "PUT"
      });

      const data = await res.json();

      if (!res.ok) {
        error = data.detail || "No se pudo cancelar.";
        return;
      }

      mensaje = "Reserva cancelada.";
      await cargarReservas();
    } catch (e) {
      error = "Error de conexión.";
    }
  }
</script>

<Navbar />
<NavbarA />

<section class="main">
  <h1 class="title">Reservas</h1>

  {#if error}
    <div class="error">{error}</div>
  {/if}
  {#if mensaje}
    <div class="success">{mensaje}</div>
  {/if}

  <!-- FORMULARIO DE RESERVA -->
  <div class="card">
    <h2>Crear nueva reserva</h2>

    <div class="form-grid">
      <div>
        <label>Fecha inicio</label>
        <input type="date" bind:value={date_start} />
      </div>

      <div>
        <label>Fecha fin</label>
        <input type="date" bind:value={date_end} />
      </div>

      <button on:click={buscarHabitaciones} class="btn-search">
        Buscar habitaciones
      </button>
    </div>

    <!-- HABITACIONES DISPONIBLES (lista simple con botón agregar) -->
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

    <!-- HABITACIONES SELECCIONADAS -->
    <div style="margin-top:16px;">
      <h3>Habitaciones seleccionadas</h3>
      {#if selectedRooms.length === 0}
        <p>No hay habitaciones seleccionadas.</p>
      {:else}
        <ul class="list">
          {#each selectedRooms as s}
            <li>
              {s.nombre} (ID: {s.id})
              <button class="btn-remove" on:click={() => quitarHab(s.id)}>Quitar</button>
            </li>
          {/each}
        </ul>
      {/if}
      <div style="margin-top:10px;">
        <button class="btn-confirm" on:click={confirmarReserva} disabled={selectedRooms.length === 0}>Confirmar reserva</button>
      </div>
    </div>
  </div>

  <!-- RESERVAS ACTIVAS -->
  <div class="card">
    <h2>Mis Reservas Activas</h2>
    {#if reservasActivas.length === 0}
      <p>No tienes reservas activas.</p>
    {:else}
      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Fecha Inicio</th>
            <th>Fecha Fin</th>
            <th>Estado</th>
            <th>Acción</th>
          </tr>
        </thead>
        <tbody>
          {#each reservasActivas as r}
            <tr>
              <td>{r.id_reserva}</td>
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
    display: flex;
    gap: 15px;
    align-items: center;
  }
  input {
    padding: 8px;
    border-radius: 6px;
    border: none;
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
  .list {
    margin-top: 10px;
  }
  .list li {
    margin-bottom: 8px;
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
  }
  th, td {
    padding: 10px;
    border-bottom: 1px solid orange;
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
</style>