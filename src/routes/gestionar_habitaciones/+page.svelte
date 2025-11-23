<script>
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";

  // ================================
  // LISTAS
  // ================================
  let tipos = [];
  let habitaciones = [];

  // ================================
  // FORMULARIOS
  // ================================
  let crearTipo = false;
  let crearHabitacion = false;

  let nuevoTipo = {
    nombre: "",
    descripcion: "",
    capacidad_max: "",
    estado: 1
  };

  let nuevaHabitacion = {
    id_thabitacion: "",
    numero: "",
    estado: 1
  };

  // ================================
  // CARGAR TIPOS DE HABITACION
  // ================================
  async function cargarTipos() {
    try {
      const res = await fetch("https://inntech-backend.onrender.com/tipos_habitacion/get_tipos_habitacion");
      const data = await res.json();
      tipos = data.data || [];
    } catch (err) {
      console.error("Error cargando tipos:", err);
    }
  }

  // ================================
  // CARGAR HABITACIONES
  // ================================
  async function cargarHabitaciones() {
    try {
      const res = await fetch("https://inntech-backend.onrender.com/habitaciones/get_habitaciones");
      const data = await res.json();
      habitaciones = data.data || [];
    } catch (err) {
      console.error("Error cargando habitaciones:", err);
    }
  }

  // ================================
  // CREAR TIPO HABITACION
  // ================================
  async function guardarTipo() {
    try {
      await fetch("https://inntech-backend.onrender.com/tipos_habitacion/create_tipo_habitacion", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(nuevoTipo)
      });

      crearTipo = false;
      nuevoTipo = { nombre: "", descripcion: "", capacidad_max: "", estado: 1 };
      cargarTipos();
    } catch (err) {
      console.error("Error creando tipo:", err);
    }
  }

  // ================================
  // CREAR HABITACION
  // ================================
  async function guardarHabitacion() {
    try {
      await fetch("https://inntech-backend.onrender.com/habitaciones/create_habitacion", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(nuevaHabitacion)
      });

      crearHabitacion = false;
      nuevaHabitacion = { id_thabitacion: "", numero: "", estado: 1 };
      cargarHabitaciones();
    } catch (err) {
      console.error("Error creando habitación:", err);
    }
  }

  // ================================
  // CAMBIAR ESTADO HABITACION
  // ================================
  async function toggleEstado(h) {
    let nuevo = h.estado === 1 ? 0 : 1;

    const formData = new FormData();
    formData.append("numero", h.numero);
    formData.append("estado", nuevo);

    try {
      await fetch("https://inntech-backend.onrender.com/habitaciones/actualizar_estado", {
        method: "PUT",
        body: formData
      });

      cargarHabitaciones();
    } catch (err) {
      console.error("Error cambiando estado:", err);
    }
  }

  cargarTipos();
  cargarHabitaciones();
</script>

<Navbar />
<NavbarA />

<section class="main">
  <div class="welcome-box">
    <h2 class="title">
      Gestión de <span class="highlight">Habitaciones</span>
    </h2>
    <p class="role">Administra tipos, habitaciones y estados.</p>
  </div>

  <!-- BOTONES SUPERIORES -->
  <div style="display:flex; gap:15px; justify-content:center;">
    <button class="btn-create" on:click={() => crearTipo = !crearTipo}>
      {crearTipo ? "Cerrar" : "Crear Tipo"}
    </button>

    <button class="btn-create" on:click={() => crearHabitacion = !crearHabitacion}>
      {crearHabitacion ? "Cerrar" : "Crear Habitación"}
    </button>
  </div>

  <!-- FORMULARIO TIPO -->
  {#if crearTipo}
    <div class="form-box">
      <h3>Nuevo Tipo de Habitación</h3>

      <div class="form-grid">
        <input placeholder="Nombre" bind:value={nuevoTipo.nombre} />
        <input placeholder="Capacidad Máxima" type="number" bind:value={nuevoTipo.capacidad_max} />
        <textarea placeholder="Descripción" bind:value={nuevoTipo.descripcion}></textarea>
      </div>

      <button class="btn-save" on:click={guardarTipo}>Guardar</button>
    </div>
  {/if}

  <!-- FORMULARIO HABITACION -->
  {#if crearHabitacion}
    <div class="form-box">
      <h3>Nueva Habitación</h3>

      <div class="form-grid">
        <select bind:value={nuevaHabitacion.id_thabitacion}>
          <option value="">Seleccione tipo</option>
          {#each tipos as t}
            <option value={t.id_thabitacion}>{t.nombre}</option>
          {/each}
        </select>

        <input placeholder="Número" bind:value={nuevaHabitacion.numero} />
      </div>

      <button class="btn-save" on:click={guardarHabitacion}>Guardar</button>
    </div>
  {/if}

  <!-- ============================= -->
  <!-- TABLA TIPOS DE HABITACION -->
  <!-- ============================= -->
  <h3 style="margin-top:30px;">Tipos de Habitación</h3>

  <table class="tabla">
    <thead>
      <tr>
        <th>ID</th>
        <th>Nombre</th>
        <th>Capacidad</th>
        <th>Estado</th>
      </tr>
    </thead>

    <tbody>
      {#each tipos as t}
        <tr>
          <td>{t.id_thabitacion}</td>
          <td>{t.nombre}</td>
          <td>{t.capacidad_max}</td>
          <td>{t.estado === 1 ? "Activo" : "Inactivo"}</td>
        </tr>
      {/each}
    </tbody>
  </table>

  <!-- ============================= -->
  <!-- TABLA HABITACIONES -->
  <!-- ============================= -->
  <h3 style="margin-top:30px;">Habitaciones</h3>

  <table class="tabla">
    <thead>
      <tr>
        <th>ID</th>
        <th>Número</th>
        <th>Tipo</th>
        <th>Estado</th>
        <th>Acción</th>
      </tr>
    </thead>

    <tbody>
      {#each habitaciones as h}
        <tr>
          <td>{h.id_habitacion}</td>
          <td>{h.numero}</td>
          <td>
            {#each tipos as t}
              {#if t.id_thabitacion === h.id_thabitacion}
                {t.nombre}
              {/if}
            {/each}
          </td>

          <td>{h.estado === 1 ? "Limpia" : "Sucia"}</td>

          <td>
            <button class="btn-toggle" on:click={() => toggleEstado(h)}>
              {h.estado === 1 ? "Marcar Sucia" : "Marcar Limpia"}
            </button>
          </td>
        </tr>
      {/each}
    </tbody>
  </table>
</section>

<Footer />

<style>
  .main {
    padding: 40px;
    background: #000;
    min-height: calc(100vh - 160px);
    color: white;
  }

  .form-box {
    background: rgba(255, 165, 0, 0.1);
    padding: 20px;
    border: 2px solid orange;
    margin-bottom: 25px;
    border-radius: 10px;
  }

  .form-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 10px;
  }

  input,
  select,
  textarea {
    padding: 8px;
    border-radius: 6px;
    border: 1px solid #444;
    background: #111;
    color: white;
  }

  textarea {
    grid-column: span 3;
    height: 80px;
  }

  table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 10px;
  }

  table th,
  table td {
    border: 1px solid #444;
    padding: 10px;
    text-align: center;
  }

  .btn-create,
  .btn-save,
  .btn-toggle {
    background: orange;
    border: none;
    padding: 10px 15px;
    color: black;
    cursor: pointer;
    margin-bottom: 10px;
    border-radius: 6px;
    font-weight: bold;
  }
</style>