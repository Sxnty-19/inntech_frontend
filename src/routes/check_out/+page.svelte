<script>
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";

  // -----------------------------
  // VARIABLES
  // -----------------------------
  let idReserva = "";
  let mensaje = "";
  let estado = "";

  let mostrarInputDocumento = false;
  let numeroDocumento = "";
  let cargandoUsuario = false;
  let errorUsuario = "";
  let usuarioEncontrado = null;

  // -----------------------------
  // FUNC: Buscar Reserva
  // -----------------------------
  async function buscarReserva() {
    mensaje = "";
    estado = "";

    if (!idReserva.trim()) {
      estado = "error";
      mensaje = "⚠️ Debe ingresar un ID de reserva.";
      return;
    }

    try {
      const res = await fetch(
        `https://inntech-backend.onrender.com/reservas/get_reserva/${idReserva}`
      );

      if (!res.ok) {
        estado = "error";
        mensaje = "❌ Reserva no encontrada.";
        return;
      }

      const data = await res.json();

      if (!data || !data.data) {
        estado = "error";
        mensaje = "❌ No se encontró la reserva.";
        return;
      }

      estado = "ok";
      mensaje = `✔️ Reserva encontrada.`;

    } catch (err) {
      estado = "error";
      mensaje = "❌ Error al conectar con el servidor.";
      console.error(err);
    }
  }

  // -----------------------------
  // BTN: Mostrar input documento
  // -----------------------------
  function abrirCheckOut() {
    mostrarInputDocumento = true;
    numeroDocumento = "";
    usuarioEncontrado = null;
    errorUsuario = "";
  }

  // -----------------------------
  // Buscar usuario por documento
  // -----------------------------
  async function buscarUsuarioPorDocumento() {
    usuarioEncontrado = null;
    errorUsuario = "";

    if (!numeroDocumento.trim()) {
      errorUsuario = "Debe ingresar un número de documento.";
      return;
    }

    cargandoUsuario = true;

    try {
      const res = await fetch(
        `https://inntech-backend.onrender.com/documentos/buscar_usuario_por_documento/${numeroDocumento}`
      );

      const data = await res.json();

      if (!res.ok) {
        errorUsuario = data.detail || "Usuario no encontrado.";
        cargandoUsuario = false;
        return;
      }

      usuarioEncontrado = data.data;
      cargandoUsuario = false;

    } catch (error) {
      console.error(error);
      errorUsuario = "Error al conectar con el servidor.";
      cargandoUsuario = false;
    }
  }

  // -----------------------------
  // Registrar salida
  // -----------------------------
  async function confirmarSalida() {
    if (!usuarioEncontrado) return;

    const body = {
      id_reserva: Number(idReserva),
      id_usuario: usuarioEncontrado.id_usuario
    };

    try {
      const res = await fetch(
        "https://inntech-backend.onrender.com/usuarios_habitaciones/registrar_salida",
        {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(body)
        }
      );

      const data = await res.json();

      if (!res.ok) {
        alert("❌ Error: " + data.detail);
        return;
      }

      alert("✔ Salida registrada correctamente");

      mostrarInputDocumento = false;
      numeroDocumento = "";
      usuarioEncontrado = null;

    } catch (err) {
      console.error(err);
      alert("Error registrando salida");
    }
  }
</script>

<!-- UI -->
<Navbar />
<NavbarA />

<section class="main">
  <div class="welcome-box">
    <h2 class="title">
      Gestión de <span class="highlight">Check-Out</span>
    </h2>
    <p class="role">Registrar salida de huéspedes.</p>
  </div>

  <!-- FORMULARIO: Buscar Reserva -->
  <div class="form-box">
    <h3>Buscar Reserva</h3>
    <input placeholder="ID de la reserva" bind:value={idReserva} />

    <button class="btn-save" on:click={buscarReserva}>Buscar</button>
  </div>

  {#if mensaje}
    <div class="msg {estado}">{mensaje}</div>
  {/if}

  <!-- SI LA RESERVA EXISTE: mostrar el botón de check-out -->
  {#if estado === "ok"}
    <div class="tabla-box" style="text-align:center;">
      <button class="btn-accion" on:click={abrirCheckOut}>
        ➖ Registrar salida
      </button>
    </div>
  {/if}

  <!-- INPUT DOCUMENTO -->
  {#if mostrarInputDocumento}
    <div class="registro-box">
      <h3>Registrar Salida</h3>

      <input
        placeholder="Número de documento del usuario"
        bind:value={numeroDocumento}
      />

      <button class="btn-save" on:click={buscarUsuarioPorDocumento}>
        Buscar usuario
      </button>

      {#if cargandoUsuario}
        <p>Buscando...</p>
      {/if}

      {#if errorUsuario}
        <p style="color:red">{errorUsuario}</p>
      {/if}

      {#if usuarioEncontrado}
        <p style="color:lightgreen">
          Usuario encontrado: {usuarioEncontrado.nombre_completo}
        </p>

        <button class="btn-save" on:click={confirmarSalida}>
          Confirmar salida
        </button>
      {/if}
    </div>
  {/if}
</section>

<Footer />

<style>
  .main {
    padding: 40px;
    background: #000;
    min-height: calc(100vh - 160px);
    color: white;
  }

  .form-box,
  .tabla-box,
  .registro-box {
    background: rgba(255, 165, 0, 0.1);
    padding: 20px;
    border: 2px solid orange;
    border-radius: 10px;
    margin: 20px auto;
    max-width: 600px;
  }

  input {
    padding: 10px;
    width: 100%;
    border-radius: 6px;
    background: #111;
    border: 1px solid #444;
    color: white;
  }

  .btn-save {
    background: orange;
    border: none;
    padding: 12px;
    width: 100%;
    border-radius: 6px;
    margin-top: 10px;
    font-weight: bold;
  }

  .btn-accion {
    background: #ff9800;
    padding: 10px 18px;
    border-radius: 8px;
    border: none;
    cursor: pointer;
    font-weight: bold;
  }
</style>