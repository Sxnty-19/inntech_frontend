<script>
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";

  let user = null;
  let fullName = "";
  let rol = "";

  let numeroHabitacion = "";
  let descripcion = "";

  let notificaciones = [];
  let mensaje = "";
  let error = "";

  // ===== Cargar usuario del LocalStorage =====
  if (typeof localStorage !== "undefined") {
    const storedUser = localStorage.getItem("user");
    if (storedUser) {
      user = JSON.parse(storedUser);

      fullName = `
        ${user.primer_nombre}
        ${user.segundo_nombre ?? ""}
        ${user.primer_apellido}
        ${user.segundo_apellido ?? ""}
      `.replace(/\s+/g, " ").trim();

      rol = user.rol;
    }
  }

  // ===== Crear notificación =====
  async function crearNotificacion() {
    if (!numeroHabitacion || !descripcion) {
      error = "Todos los campos son obligatorios.";
      return;
    }

    error = "";
    mensaje = "";

    const formData = new FormData();
    formData.append("id_usuario", user.id_usuario);
    formData.append("numero_habitacion", numeroHabitacion);
    formData.append("descripcion", descripcion);
    formData.append("estado", 1);

    try {
      const res = await fetch("https://inntech-backend.onrender.com/notificaciones/crear_por_numero", {
        method: "POST",
        body: formData
      });

      const data = await res.json();

      if (!res.ok) {
        error = data.detail ?? "Error al crear notificación.";
      } else {
        mensaje = "Notificación creada correctamente!";
        numeroHabitacion = "";
        descripcion = "";
        cargarNotificaciones();
      }

    } catch (e) {
      error = "Error de conexión con el servidor.";
    }
  }

  // ===== Cargar notificaciones del usuario =====
  async function cargarNotificaciones() {
    if (!user) return;

    try {
      const res = await fetch(
        `https://inntech-backend.onrender.com/notificaciones/usuario/${user.id_usuario}`
      );
      const data = await res.json();

      if (!res.ok) {
        mensaje = data.detail ?? "No hay notificaciones.";
        notificaciones = [];
      } else {
        notificaciones = data.data;
      }
    } catch (e) {
      error = "Error al cargar notificaciones.";
    }
  }

  cargarNotificaciones();
</script>

<Navbar />
<NavbarA />

<section class="main">
  <div class="welcome-box">
    <h2 class="title">Notificaciones</h2>
  </div>

  <div class="form-box">
    <h3 class="form-title">Crear Notificación</h3>

    {#if error}
      <p class="error">{error}</p>
    {/if}

    {#if mensaje}
      <p class="mensaje">{mensaje}</p>
    {/if}

    <div class="form-group">
      <label>Número de habitación</label>
      <input bind:value={numeroHabitacion} placeholder="Ej: 203" />
    </div>

    <div class="form-group">
      <label>Descripción</label>
      <textarea bind:value={descripcion} placeholder="Descripción del problema..."></textarea>
    </div>

    <button class="btn" on:click={crearNotificacion}>Enviar Notificación</button>
  </div>

  {#if notificaciones.length > 0}
    <h3 class="tabla-title">Mis Notificaciones</h3>

    <table class="tabla">
      <thead>
        <tr>
          <th>ID</th>
          <th>ID Habitación</th>
          <th>Descripción</th>
          <th>Estado</th>
          <th>Fecha</th>
        </tr>
      </thead>
      <tbody>
        {#each notificaciones as item}
          <tr>
            <td>{item.id_notificacion}</td>
            <td>{item.id_habitacion}</td>
            <td>{item.descripcion}</td>
            <td>{item.estado}</td>
            <td>{item.date_created}</td>
          </tr>
        {/each}
      </tbody>
    </table>
  {/if}
</section>

<Footer />

<style>
  .main {
    min-height: calc(100vh - 160px);
    padding: 40px 20px;
    background: #000;
    color: white;
    text-align: center;
  }

  .welcome-box {
    margin-bottom: 30px;
    background: rgba(255,165,0,0.1);
    padding: 25px 35px;
    border-radius: 12px;
    border: 2px solid orange;
    box-shadow: 0 0 12px rgba(255,165,0,0.3);
  }

  .title {
    font-size: 2rem;
    color: orange;
    margin-bottom: 10px;
  }

  .highlight {
    color: white;
    font-weight: 700;
  }

  .form-box {
    width: 95%;
    max-width: 600px;
    margin: auto;
    background: rgba(255,165,0,0.05);
    padding: 25px;
    border: 2px solid orange;
    border-radius: 12px;
    box-shadow: 0 0 10px rgba(255,165,0,0.2);
  }

  .form-title {
    font-size: 1.5rem;
    color: orange;
    margin-bottom: 15px;
  }

  .form-group {
    text-align: left;
    margin-bottom: 15px;
  }

  label {
    font-weight: bold;
    color: orange;
  }

  input, textarea {
    width: 100%;
    padding: 10px;
    margin-top: 5px;
    background: #111;
    border: 1px solid orange;
    color: white;
    border-radius: 8px;
  }

  textarea {
    height: 120px;
  }

  .btn {
    width: 100%;
    padding: 12px;
    margin-top: 10px;
    background: orange;
    color: black;
    font-weight: bold;
    border: none;
    border-radius: 8px;
    cursor: pointer;
  }

  .btn:hover {
    background: #ffae00;
  }

  .tabla {
    width: 95%;
    max-width: 900px;
    margin: 30px auto;
    border-collapse: collapse;
    border: 2px solid orange;
    background: #111;
  }

  th, td {
    padding: 12px;
    border: 1px solid orange;
  }

  th {
    background: rgba(255,165,0,0.3);
  }

  tr:hover {
    background: rgba(255,165,0,0.1);
  }

  .error {
    color: red;
    margin-bottom: 10px;
  }

  .mensaje {
    color: orange;
    margin-bottom: 10px;
  }
</style>
