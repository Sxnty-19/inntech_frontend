<script>
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";
  import { onMount } from "svelte"; // Importar onMount para el listener de Escape

  // -----------------------------
  // VARIABLES
  // -----------------------------
  let idReserva = "";
  let mensaje = "";
  let estado = ""; // 'ok' o 'error'

  let mostrarInputDocumento = false;
  let numeroDocumento = "";
  let cargandoUsuario = false;
  let errorUsuario = "";
  let usuarioEncontrado = null;

  // ---------------------------------------------------
  // Función para mostrar mensajes (5 SEGUNDOS)
  // ---------------------------------------------------
  function mostrarMensaje(texto, tipo = "ok") {
    mensaje = texto;
    estado = tipo;
  }

  // -----------------------------
  // FUNC: Buscar Reserva (AJUSTADA)
  // -----------------------------
  async function buscarReserva() {
    mensaje = "";
    estado = "";

    const idReservaStr = String(idReserva ?? "").trim();

    if (!idReservaStr) {
      mostrarMensaje("Debe ingresar un ID de reserva.", "error");
      return;
    }

    try {
      const res = await fetch(
        `https://inntech-backend.onrender.com/reservas/get_reserva/${idReservaStr}`,
      );

      if (!res.ok) {
        mostrarMensaje("Reserva no encontrada.", "error");
        return;
      }

      const data = await res.json();

      if (!data || !data.data) {
        mostrarMensaje("No se encontró la reserva.", "error");
        return;
      }

      mostrarMensaje("Reserva encontrada. Continúe con el Check-Out.", "ok");
    } catch (err) {
      mostrarMensaje("Error al conectar con el servidor.", "error");
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
        `https://inntech-backend.onrender.com/documentos/buscar_usuario_por_documento/${numeroDocumento}`,
      );

      const data = await res.json();

      if (!res.ok) {
        errorUsuario = data.detail || "Usuario no encontrado.";
        cargandoUsuario = false;
        return;
      }

      usuarioEncontrado = data.data;
      cargandoUsuario = false;
      errorUsuario = ""; // Limpiar error si se encuentra
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
      id_usuario: usuarioEncontrado.id_usuario,
    };

    try {
      const res = await fetch(
        "https://inntech-backend.onrender.com/usuarios_habitaciones/registrar_salida",
        {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(body),
        },
      );

      const data = await res.json();

      if (!res.ok) {
        mostrarMensaje(
          `Error al registrar salida: ${data.detail || "Operación fallida"}`,
          "error",
        );
        return;
      }

      mostrarMensaje("Salida registrada correctamente", "ok");

      // Resetear la búsqueda de Check-Out
      mostrarInputDocumento = false;
      numeroDocumento = "";
      usuarioEncontrado = null;

      // Opcional: limpiar la reserva para forzar una nueva búsqueda
      idReserva = "";
      estado = "";
      mensaje = "";
    } catch (err) {
      console.error(err);
      mostrarMensaje(
        "Error al conectar con el servidor para registrar la salida.",
        "error",
      );
    }
  }

  // ---------------------------------------------------
  // Lógica de UX (Cerrar con Escape)
  // ---------------------------------------------------
  function handleKeydown(event) {
    if (event.key === "Escape") {
      mostrarInputDocumento = false;
    }
  }

  onMount(() => {
    document.addEventListener("keydown", handleKeydown);
    return () => {
      document.removeEventListener("keydown", handleKeydown);
    };
  });
</script>

<svelte:head>
  <link
    rel="stylesheet"
    href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css"
    crossorigin="anonymous"
    referrerpolicy="no-referrer"
  />
</svelte:head>

<Navbar />
<NavbarA />

<section class="main">
  <div class="content-container">
    <h1 class="page-title">Gestión de Check-Out</h1>

    <div class="card card-search">
      <h2 class="card-subtitle">
        <i class="fas fa-search"></i> Buscar Reserva para Salida
      </h2>

      <div class="form-grid-search">
        <input
          placeholder="ID de la reserva"
          bind:value={idReserva}
          class="input-field"
          type="number"
        />
        <button class="btn-primary" on:click={buscarReserva}>
          <i class="fas fa-search icon-left"></i>
          Buscar Reserva
        </button>
      </div>
    </div>

    {#if mensaje}
      <div
        class="msg-inline"
        class:msg-success={estado === "ok"}
        class:msg-error={estado === "error"}
      >
        <i
          class="fas"
          class:fa-check-circle={estado === "ok"}
          class:fa-times-circle={estado === "error"}
        ></i>
        {mensaje}
      </div>
    {/if}

    {#if estado === "ok"}
      <div class="card card-actions">
        <h2 class="card-subtitle">
          <i class="fas fa-sign-out-alt"></i> Opciones de Salida
        </h2>
        <div class="actions-wrapper">
          <button class="btn-accion large-btn" on:click={abrirCheckOut}>
            <i class="fas fa-minus-circle icon-left"></i> Registrar Salida de Huésped
          </button>
        </div>
      </div>
    {/if}

    {#if mostrarInputDocumento}
      <div
        class="modal-backdrop"
        on:click={() => (mostrarInputDocumento = false)}
      ></div>
      <div class="registro-box">
        <h3 class="modal-title">
          <i class="fas fa-user-minus"></i> Registrar Salida de Huésped
        </h3>

        <input
          placeholder="Número de documento del huésped"
          bind:value={numeroDocumento}
          class="input-field"
        />

        <div
          class="modal-actions-footer"
          style="border-top: none; padding-top: 15px; margin-top: 0;"
        >
          <button
            class="btn-primary"
            on:click={buscarUsuarioPorDocumento}
            disabled={cargandoUsuario}
          >
            <i class="fas fa-search icon-left" class:fa-spin={cargandoUsuario}
            ></i>
            {cargandoUsuario ? "Buscando..." : "Buscar Huésped"}
          </button>
        </div>

        {#if errorUsuario}
          <p class="error-inline">
            <i class="fas fa-times-circle"></i>
            {errorUsuario}
          </p>
        {/if}

        {#if usuarioEncontrado}
          <div class="found-user-box">
            <p class="success-inline">
              <i class="fas fa-check-circle"></i> Huésped encontrado: **{usuarioEncontrado.nombre_completo}**
            </p>
          </div>
        {/if}

        <div class="modal-actions-footer">
          <button
            class="btn-cancel"
            on:click={() => (mostrarInputDocumento = false)}>Cancelar</button
          >
          {#if usuarioEncontrado}
            <button class="btn-save" on:click={confirmarSalida}>
              <i class="fas fa-check-double icon-left"></i> Confirmar Salida
            </button>
          {/if}
        </div>
      </div>
    {/if}
  </div>
</section>

<Footer />

<style>
  /* ---------------------------------- */
  /* Paleta de Colores (Dark Mode - Moderno) */
  /* ---------------------------------- */
  :root {
    --color-bg-primary: #0f172a; /* Fondo principal muy oscuro (slate-900) */
    --color-card-bg: #1e293b; /* Fondo de tarjeta/componente (slate-800) */
    --color-table-bg: #1a202c; /* Fondo de tabla muy oscuro (gray-900) */
    --color-accent-gold: #fcd34d; /* Amarillo principal (Dorado) */
    --color-accent-blue: #60a5fa; /* Azul para highlights (blue-400) */
    --color-text-light: #e2e8f0; /* Texto claro (slate-200) */
    --color-text-muted: #94a3b8; /* Texto sutil (slate-400) */
    --color-border: #334155; /* Borde sutil (slate-700) */
    --color-primary-dark: #1e3a8a; /* Azul oscuro (para encabezados/tabla) */
    --color-success: #10b981; /* Verde (Activa) */
    --color-danger: #ef4444; /* Rojo (Desactivada) */
    --color-warning: #f59e0b; /* Naranja (Warning/Confirmación) */
    --shadow-card: 0 10px 25px rgba(0, 0, 0, 0.4);
    --shadow-btn-primary: 0 5px 15px rgba(252, 211, 77, 0.3);
  }

  /* ---------------------------------- */
  /* Estilos principales y Layout */
  /* ---------------------------------- */

  .main {
    min-height: calc(100vh - 160px);
    padding: 60px 20px;
    background: var(--color-table-bg);
    color: var(--color-text-light);
    font-family: "Inter", sans-serif;
  }

  .content-container {
    max-width: 1200px;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
    gap: 30px;
  }

  .page-title {
    text-align: center;
    font-size: clamp(2.2rem, 4vw, 3.2rem);
    color: var(--color-accent-gold);
    margin-bottom: 5px;
    font-weight: 900;
    text-transform: uppercase;
    letter-spacing: 1px;
    text-shadow: 0 0 10px rgba(252, 211, 77, 0.2);
  }

  /* ---------------------------------- */
  /* Tarjeta Contenedora (Card) */
  /* ---------------------------------- */

  .card {
    background: var(--color-bg-primary);
    border: 1px solid var(--color-border);
    padding: clamp(20px, 4vw, 30px);
    border-radius: 12px;
    box-shadow: var(--shadow-card);
    width: 100%;
  }

  .card-search {
    max-width: 650px;
    align-self: center;
    padding: 30px;
  }

  .card-actions {
    max-width: 650px;
    align-self: center;
    padding: 25px 30px;
  }

  .actions-wrapper {
    display: flex;
    justify-content: center;
    padding-top: 15px;
  }

  .card-subtitle {
    color: var(--color-accent-blue);
    font-size: clamp(1.4rem, 2.5vw, 1.8rem);
    padding: 0 0 15px 0;
    font-weight: 700;
    display: flex;
    align-items: center;
    gap: 10px;
    border-bottom: 1px solid var(--color-border);
    margin-bottom: 20px;
  }

  /* ---------------------------------- */
  /* Inputs y Formularios */
  /* ---------------------------------- */

  /* Inputs generales dentro del componente, incluyendo los enlazados en el script */
  input:not(.btn-accion, .btn-primary) {
    box-sizing: border-box;
    width: 100%;
    padding: 12px 16px;
    background: var(--color-table-bg);
    color: var(--color-text-light);
    border: 1px solid var(--color-border);
    border-radius: 8px;
    transition: all 0.3s;
  }

  input::placeholder {
    color: var(--color-text-muted);
  }

  input:focus {
    border-color: var(--color-accent-gold);
    box-shadow: 0 0 0 3px rgba(252, 211, 77, 0.4);
    outline: none;
  }

  .form-grid-search {
    display: grid;
    gap: 15px;
    grid-template-columns: 2fr 1fr;
  }

  /* ---------------------------------- */
  /* Botones de Acción Global */
  /* ---------------------------------- */

  .btn-primary,
  .btn-save,
  .btn-cancel,
  .btn-accion {
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 12px 25px;
    border: none;
    border-radius: 8px;
    font-weight: 700;
    cursor: pointer;
    transition: all 0.3s ease;
    gap: 8px;
    font-size: 1rem;
    white-space: nowrap;
  }

  /* Botón de acción grande para la tarjeta de Check-Out */
  .btn-accion {
    background: var(--color-danger);
    color: var(--color-text-light);
    padding: 15px 30px !important;
    font-size: 1.1rem !important;
    font-weight: 800 !important;
  }

  .btn-accion:hover {
    background: #c03939; /* Rojo un poco más oscuro */
  }

  /* Sobrescribe .btn-save del código original */
  .btn-save {
    background: var(--color-success);
    color: #111;
    font-weight: 800;
    padding: 12px 20px;
    width: auto !important;
  }

  .btn-primary {
    background: var(--color-accent-gold);
    color: #111;
    box-shadow: var(--shadow-btn-primary);
    padding: 12px 25px;
  }

  .btn-primary:hover:not(:disabled) {
    background: #ffaa2b;
    transform: translateY(-1px);
    box-shadow: 0 8px 20px rgba(252, 211, 77, 0.5);
  }

  .btn-primary:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }

  .btn-save:hover {
    background: #10e092;
    transform: translateY(-1px);
  }

  .btn-cancel {
    background: var(--color-card-bg);
    color: var(--color-text-light);
    border: 1px solid var(--color-border);
    padding: 12px 20px;
  }

  .btn-cancel:hover {
    background: #2a3547;
    border-color: var(--color-accent-blue);
  }

  /* ---------------------------------- */
  /* MODALES (registro-box) */
  /* ---------------------------------- */

  .modal-backdrop {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.8);
    z-index: 100;
  }

  .registro-box {
    position: fixed;
    z-index: 101;
    left: 50%;
    transform: translate(-50%, -50%);
    top: 50%;
    background: var(--color-bg-primary);
    padding: 30px;
    border-radius: 12px;
    border: 2px solid var(--color-accent-gold);
    width: 95%;
    max-width: 650px;
    box-shadow: 0 15px 40px rgba(0, 0, 0, 0.9);
    animation: fadeInScale 0.3s ease-out forwards;
    display: flex; /* Añadido para mejor disposición interna */
    flex-direction: column; /* Añadido para mejor disposición interna */
    gap: 15px; /* Añadido para espaciado consistente */
  }

  .modal-title {
    color: var(--color-accent-gold);
    font-size: 1.8rem;
    margin-bottom: 5px; /* Reducido para usar el gap */
    border-bottom: 1px solid var(--color-border);
    padding-bottom: 10px;
    display: flex;
    align-items: center;
    gap: 10px;
  }

  .modal-actions-footer {
    margin-top: 5px;
    padding-top: 15px;
    border-top: 1px solid var(--color-border);
    display: flex;
    justify-content: flex-end;
    gap: 10px;
  }

  .found-user-box {
    padding: 15px;
    border: 1px solid var(--color-success);
    border-radius: 8px;
    background: rgba(16, 185, 129, 0.1);
  }

  /* ---------------------------------- */
  /* Mensajes Inline y de Error */
  /* ---------------------------------- */

  .msg-inline {
    position: fixed;
    top: 20px;
    right: 20px;
    z-index: 1000;

    max-width: 650px;
    padding: 15px 25px;
    margin: -15px auto 15px auto;

    display: flex;
    align-items: center;
    gap: 15px;

    font-weight: 600;
    color: white;
    background-color: var(--color-error);
    border-radius: 8px;

    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
    transition: all 0.3s;

    animation: bounce 0.5s ease-out; /* Animación de aparición */
  }

  .msg-success {
    color: var(--color-text-light);
    background: var(--color-success);
    border: 1px solid var(--color-success);
  }

  .msg-error {
    color: var(--color-text-light);
    background: var(--color-danger);
    border: 1px solid var(--color-danger);
  }

  /* Inline simples */
  .error-inline,
  .success-inline {
    margin-top: 5px;
    padding: 10px 15px;

    display: flex;
    align-items: center;
    gap: 8px;

    font-weight: 600;
    border-radius: 6px;
  }

  .error-inline {
    color: var(--color-text-light);
    background: var(--color-danger);
    border: 1px solid var(--color-danger);
  }

  .success-inline {
    color: var(--color-text-light);
    background: var(--color-success);
    border: 1px solid var(--color-success);
  }

  /* Animación Modal */
  @keyframes fadeInScale {
    from {
      opacity: 0;
      transform: translate(-50%, -50%) scale(0.9);
    }
    to {
      opacity: 1;
      transform: translate(-50%, -50%) scale(1);
    }
  }

  /* Media Queries */
  @media (max-width: 768px) {
    .form-grid-search {
      grid-template-columns: 1fr;
    }

    .registro-box {
      width: 90%;
      padding: 20px;
    }

    .btn-accion {
      font-size: 1rem !important;
      padding: 12px 20px !important;
    }
  }

  /* --- SOBRESCRITURAS DEL CÓDIGO ORIGINAL (Clean up) --- */

  /* Ocultar/Reemplazar elementos originales obsoletos para usar los nuevos */
  .welcome-box,
  .role,
  .msg,
  .tabla-box,
  .form-box {
    display: none;
  }

  /* Asegurar que el registro-box original use los nuevos estilos de modal */
  .registro-box {
    background: var(--color-bg-primary) !important;
    padding: 30px !important;
    border: 2px solid var(--color-accent-gold) !important;
    border-radius: 12px !important;
    max-width: 650px !important;
    margin: 0 auto !important;
    box-shadow: var(--shadow-card) !important;
    position: fixed;
    z-index: 101;
    left: 50%;
    transform: translate(-50%, -50%);
    top: 50%;
  }

  .registro-box h3 {
    display: block; /* Asegurar que el título del modal se muestre */
    margin-bottom: 0;
  }
</style>
