<script>
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";

  let user = null;
  let fullName = "";
  let rol = "";
  let historial = [];
  let mensaje = "";
  let error = "";

  // ===== Cargar usuario desde localStorage =====
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

  // ===== Obtener historial =====
  async function cargarHistorial() {
    if (!user) return;

    try {
      const res = await fetch(`https://inntech-backend.onrender.com/reservas/terminadas/${user.id_usuario}`);
      const data = await res.json();

      if (!res.ok) {
        mensaje = data.detail ?? "Error al cargar historial.";
        return;
      }

      historial = data.data;
      if (historial.length === 0) {
        mensaje = "No tienes reservas finalizadas.";
      }
    } catch (e) {
      error = "Error de conexión al servidor.";
    }
  }

  cargarHistorial();
</script>

<Navbar />
<NavbarA />

<section class="main">
  <div class="welcome-box">
    <h2 class="title">Historial de Reservas</h2>

  </div>

  {#if error}
    <p class="error">{error}</p>
  {/if}

  {#if mensaje}
    <p class="mensaje">{mensaje}</p>
  {/if}

  {#if historial.length > 0}
    <table class="tabla">
      <thead>
        <tr>
          <th>ID Reserva</th>
          <th>Fecha Inicio</th>
          <th>Fecha Fin</th>
          <th>Estado</th>
        </tr>
      </thead>
      <tbody>
        {#each historial as item}
          <tr>
            <td>{item.id_reserva}</td>
            <td>{item.date_start}</td>
            <td>{item.date_end}</td>
            <td>{item.estado}</td>
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
    background: rgba(255, 165, 0, 0.1);
    padding: 25px 35px;
    border-radius: 12px;
    border: 2px solid orange;
    box-shadow: 0 0 12px rgba(255, 165, 0, 0.3);
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

  .tabla {
    width: 95%;
    max-width: 900px;
    margin: 20px auto;
    border-collapse: collapse;
    background: #111;
    border: 2px solid orange;
  }

  th, td {
    padding: 12px;
    border: 1px solid orange;
  }

  th {
    background: rgba(255, 165, 0, 0.3);
    color: white;
  }

  td {
    color: #ddd;
  }

  tr:hover {
    background: rgba(255, 165, 0, 0.1);
  }

  .mensaje, .error {
    margin-top: 20px;
    font-size: 1.2rem;
    color: orange;
  }
</style>