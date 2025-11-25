<script>
  import { onMount } from "svelte";
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";

  // Constantes y Variables de Estado
  const API = "https://inntech-backend.onrender.com/notificaciones";

  let notificaciones = [];
  let loading = true;
  let error = null;

  // NUEVAS VARIABLES PARA EL MODAL FLOTANTE (COMO EN EL LOGIN)
  let message = ""; // Contenido del mensaje
  let isSuccess = false; // Define si es éxito (verde) o error (rojo)
  let showMessage = false; // Controla si el modal está en el DOM
  let isModalActive = false; // Controla la animación de entrada/salida

  let isSubmitting = false; // Para deshabilitar el botón durante la creación

  let numero_habitacion = "";
  let descripcion = "";

  let user = null;
  let id_usuario = null;

  // Controla la vista activa ('create' o 'history')
  let activeView = "create";

  // Cargar usuario
  if (typeof localStorage !== "undefined") {
    const stored = localStorage.getItem("user");
    if (stored) {
      user = JSON.parse(stored);
      id_usuario = user.id_usuario;
    }
  }

  // NUEVA FUNCIÓN: Oculta el modal con animación de salida
  function hideMessageWithTransition(duration = 300) {
    isModalActive = false; // Inicia la animación de fade-out
    return new Promise((resolve) =>
      setTimeout(() => {
        showMessage = false; // Oculta completamente después de la animación
        resolve();
      }, duration),
    );
  }

  // FUNCIÓN ACTUALIZADA: Para mostrar mensajes de forma temporal (usa el nuevo modal)
  function showFloatingMessage(type, text) {
    if (showMessage) {
      hideMessageWithTransition(100); // Rápida transición si ya había uno
    }

    message = text;
    isSuccess = type === "success";
    showMessage = true;
    isModalActive = true; // Activa el modal para la animación de entrada

    // Ocultar automáticamente después de 4 segundos
    setTimeout(() => {
      hideMessageWithTransition(300);
    }, 4000);
  }

  async function cargarNotificaciones() {
    loading = true;
    error = null;

    try {
      const res = await fetch(`${API}/get_notificaciones`);

      if (!res.ok) throw new Error("Error cargando notificaciones");

      const data = await res.json();

      if (!data.success) throw new Error(data.detail ?? "Error");

      // Ordenar notificaciones para mostrar las más recientes primero
      notificaciones = data.data.sort(
        (a, b) => b.id_notificacion - a.id_notificacion,
      );
    } catch (e) {
      console.error(e);
      error = "No se pudieron cargar las notificaciones.";
    } finally {
      loading = false;
    }
  }

  async function crearNotificacion() {
    if (!numero_habitacion || !descripcion) {
      showFloatingMessage(
        "error",
        "Por favor, complete el número de habitación y la descripción.",
      );
      return;
    }

    if (isSubmitting) return; // Evitar doble submit

    isSubmitting = true;
    error = null;

    // Oculta el mensaje si estaba visible antes de la nueva acción
    if (showMessage) await hideMessageWithTransition(100);

    try {
      const formData = new FormData();
      formData.append("id_usuario", Number(id_usuario));
      formData.append("numero_habitacion", numero_habitacion);
      formData.append("descripcion", descripcion);
      formData.append("estado", 1);

      const res = await fetch(`${API}/crear_por_numero`, {
        method: "POST",
        body: formData,
      });

      const data = await res.json();

      if (!res.ok || !data.success)
        throw new Error(data.detail ?? "Error al crear notificación");

      // Muestra la notificación flotante de éxito
      showFloatingMessage(
        "success",
        `Solicitud para Habitación ${numero_habitacion} creada con éxito.`,
      );

      numero_habitacion = "";
      descripcion = "";

      // Recargar la lista y cambiar a la vista de historial después de crear
      await cargarNotificaciones();
      activeView = "history";
    } catch (e) {
      console.error("Error al crear notificación:", e);
      // Muestra la notificación flotante de error
      showFloatingMessage(
        "error",
        "Error al crear la solicitud. Verifique los datos.",
      );
    } finally {
      isSubmitting = false;
    }
  }

  onMount(() => {
    cargarNotificaciones();
  });
</script>

<!-- Se necesita Font Awesome para los iconos -->
<svelte:head>
  <link
    rel="stylesheet"
    href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css"
    xintegrity="sha512-SnH5WK+bZxgPHs44uWIX+LLMD/CD+5N+yEStRjS1K48xSgH44n/B+xJ1A40k8c7oE9a9u5pG2Gq0wF04iP5w=="
    crossorigin="anonymous"
    referrerpolicy="no-referrer"
  />
</svelte:head>

<Navbar />
<NavbarA />

<section class="main">
  <div class="content-container">
    <h1 class="page-title">Gestion Notificaciones</h1>

    <!-- ---------------------------------- -->
    <!-- NAVEGACIÓN TIPO PESTAÑA -->
    <!-- ---------------------------------- -->
    <div class="category-tabs">
      <button
        class="tab-btn"
        class:active={activeView === "create"}
        on:click={() => (activeView = "create")}
      >
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="20"
          height="20"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2.5"
          stroke-linecap="round"
          stroke-linejoin="round"
          ><path d="M12 5v14" /><path d="M5 12h14" /></svg
        >
        Crear Solicitud
      </button>
      <button
        class="tab-btn"
        class:active={activeView === "history"}
        on:click={() => (activeView = "history")}
      >
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="20"
          height="20"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2.5"
          stroke-linecap="round"
          stroke-linejoin="round"
          ><path d="M12 2v10" /><path d="M5 9l-3 3 3 3" /><path
            d="M19 9l3 3-3 3"
          /><rect x="2" y="16" width="20" height="4" rx="2" /></svg
        >
        Ver Historial y Estado
      </button>
    </div>

    <!-- ---------------------------------- -->
    <!-- VISTA 1: CREAR SOLICITUD -->
    <!-- ---------------------------------- -->
    {#if activeView === "create"}
      <div class="card card-create">
        <h2 class="subtitulo">Crear Nueva Petición de Servicio</h2>

        <div class="form">
          <input
            type="number"
            placeholder="Número de habitación (Ej: 305)"
            bind:value={numero_habitacion}
            class="input-field"
            min="1"
          />

          <textarea
            placeholder="Describa la necesidad o solicitud especial (Ej: Necesitamos toallas extra y un tazón de frutas)."
            bind:value={descripcion}
            class="input-field"
          ></textarea>

          <button
            class="btn-crear"
            on:click={crearNotificacion}
            disabled={isSubmitting}
          >
            {isSubmitting ? "Enviando..." : "Crear Notificación"}
          </button>
        </div>
      </div>
    {/if}

    <!-- ---------------------------------- -->
    <!-- VISTA 2: HISTORIAL Y ESTADO -->
    <!-- ---------------------------------- -->
    {#if activeView === "history"}
      <div class="card card-history">
        <h2 class="subtitulo">Historial de Solicitudes</h2>

        {#if loading}
          <!-- Animación de carga sutil para la tabla -->
          <div class="loading-container-bar">
            <div class="loading-bar"></div>
          </div>
        {:else if error}
          <p class="error-text">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="24"
              height="24"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
              class="icon-error"
              ><circle cx="12" cy="12" r="10" /><path d="M15 9l-6 6" /><path
                d="M9 9l6 6"
              /></svg
            >
            {error}
          </p>
        {:else if notificaciones.length === 0}
          <p class="subtext">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="20"
              height="20"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
              class="icon-info"
              ><circle cx="12" cy="12" r="10" /><path d="M12 16v-4" /><path
                d="M12 8h.01"
              /></svg
            >
            No hay notificaciones registradas en este momento.
          </p>
        {:else}
          <div class="table-wrap">
            <table class="tabla">
              <thead>
                <tr>
                  <th class="col-id">ID</th>
                  <th class="col-user">Usuario ID</th>
                  <th>Habitación</th>
                  <th>Descripción</th>
                  <th>Estado</th>
                  <th>Fecha Creación</th>
                </tr>
              </thead>
              <tbody>
                {#each notificaciones as n}
                  <tr class:active-row={n.estado === 1}>
                    <td class="col-id">{n.id_notificacion}</td>
                    <td class="col-user">{n.id_usuario}</td>
                    <td class="resaltado">{n.id_habitacion}</td>
                    <td class="descripcion-celda">{n.descripcion}</td>
                    <td>
                      {#if n.estado === 1}
                        <span class="estado estado-activa">Activa</span>
                      {:else}
                        <span class="estado estado-cerrada">Cerrada</span>
                      {/if}
                    </td>
                    <td>{n.date_created.substring(0, 10)}</td>
                  </tr>
                {/each}
              </tbody>
            </table>
          </div>
        {/if}
      </div>
    {/if}
  </div>

  <!-- ---------------------------------- -->
  <!-- NOTIFICACIÓN FLOTANTE (COMO EN EL LOGIN) -->
  <!-- ---------------------------------- -->
  {#if showMessage}
    <div
      class="message-modal"
      class:fade-out={!isModalActive}
      class:slide-in={isModalActive}
    >
      <!-- Animación de rebote aplicada en message-content -->
      <div class="message-content" class:success={isSuccess}>
        <!-- Iconos dinámicos según el estado -->
        <i
          class="fas"
          class:fa-check-circle={isSuccess}
          class:fa-times-circle={!isSuccess}
        ></i>
        <p>{message}</p>
      </div>
    </div>
  {/if}
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
    --color-primary: #1e3a8a; /* Azul (para botón activo) */
    --color-success: #10b981; /* Verde (Activa) */
    --color-danger: #ef4444; /* Rojo (Cerrada/Error) */
    --shadow-card: 0 10px 25px rgba(0, 0, 0, 0.4);
    --shadow-btn: 0 5px 15px rgba(252, 211, 77, 0.3);

    /* COLORES PARA EL MODAL FLOTANTE */
    --color-modal-success: var(--color-success); /* Verde para éxito */
    --color-modal-error: var(--color-danger); /* Rojo para error */
    /* Variables de color para el Scrollbar */
    --scrollbar-track: #2d3748;
    --scrollbar-thumb: #4a5568;
    --scrollbar-thumb-hover: #6b7280;
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
    margin-bottom: 20px;
    font-weight: 900;
    text-transform: uppercase;
    letter-spacing: 1px;
    text-shadow: 0 0 10px rgba(252, 211, 77, 0.2);
  }

  /* ---------------------------------- */
  /* NAVEGACIÓN TIPO PESTAÑA (TABS) */
  /* ---------------------------------- */
  .category-tabs {
    display: flex;
    justify-content: center;
    gap: 15px;
    margin-bottom: 10px;
    flex-wrap: wrap;
    align-self: center;
    max-width: 650px;
    width: 100%;
  }

  .tab-btn {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    padding: 12px 25px;
    border: 2px solid var(--color-border);
    border-radius: 8px;
    background-color: var(--color-card-bg);
    color: var(--color-text-muted);
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
    font-size: 1rem;
  }

  .tab-btn:hover {
    background-color: #2a3547;
    color: var(--color-text-light);
    border-color: var(--color-accent-blue);
  }

  .tab-btn.active {
    border-color: var(--color-accent-gold);
    background-color: var(--color-accent-gold);
    color: #111;
    font-weight: 700;
    box-shadow: 0 0 15px rgba(252, 211, 77, 0.5);
  }

  .tab-btn.active svg {
    color: #111;
  }

  .tab-btn:not(.active) svg {
    color: var(--color-accent-gold);
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
    align-self: center;
  }

  .card-create {
    max-width: 600px;
    width: 100%;
  }

  .card-history {
    max-width: 1000px;
    width: 100%;
  }

  .subtitulo {
    color: var(--color-accent-blue);
    font-size: clamp(1.4rem, 2.5vw, 1.8rem);
    margin-bottom: 25px;
    font-weight: 700;
    display: flex;
    align-items: center;
    gap: 10px;
    padding-bottom: 10px;
    border-bottom: 1px solid var(--color-border);
  }

  /* ---------------------------------- */
  /* Formulario (Creación) */
  /* ---------------------------------- */

  .form {
    display: grid;
    gap: 20px;
  }

  .input-field {
    box-sizing: border-box;
    width: 100%;
    padding: 14px 18px;
    background: var(--color-table-bg);
    color: var(--color-text-light);
    border: 1px solid var(--color-border);
    border-radius: 8px;
    transition: all 0.3s;
  }

  .input-field::placeholder {
    color: var(--color-text-muted);
  }

  .input-field:focus {
    border-color: var(--color-accent-gold);
    box-shadow: 0 0 0 3px rgba(252, 211, 77, 0.4);
    outline: none;
  }

  .form textarea {
    height: 100px;
    resize: vertical;
  }

  .btn-crear {
    background: var(--color-accent-gold);
    color: #111;
    border: none;
    padding: 14px 25px;
    border-radius: 8px;
    font-weight: 800;
    cursor: pointer;
    transition:
      background 0.3s,
      transform 0.1s,
      box-shadow 0.3s;
    font-size: 1rem;
    box-shadow: var(--shadow-btn);
  }

  .btn-crear:hover:not(:disabled) {
    background: #ffaa2b;
    transform: translateY(-2px);
    box-shadow: 0 8px 20px rgba(252, 211, 77, 0.5);
  }

  .btn-crear:disabled {
    background: var(--color-border);
    color: var(--color-text-muted);
    cursor: not-allowed;
    opacity: 0.7;
    box-shadow: none;
  }

  /* ---------------------------------- */
  /* Tabla (Historial) */
  /* ---------------------------------- */

  .table-wrap::-webkit-scrollbar {
    width: 8px;
    height: 8px;
  }

  .table-wrap::-webkit-scrollbar-track {
    background: var(--scrollbar-track);
    border-radius: 10px;
  }

  .table-wrap::-webkit-scrollbar-thumb {
    background: var(--scrollbar-thumb);
    border-radius: 10px;
  }

  .table-wrap::-webkit-scrollbar-thumb:hover {
    background: var(--scrollbar-thumb-hover);
  }

  /* Tabla de Historial (Diseño Responsivo y Elegante) */
  .table-wrap {
    overflow-x: auto;
    /* Scroll vertical es para esta caja, que está contenida */
    max-height: 60vh;
    overflow-y: auto;
    border: 1px solid #334155; /* Este es el borde explícito alrededor del área de scroll */
    border-radius: 8px;
    margin: 0 auto; /* Centrar el contenedor de scroll */
    max-width: 1000px; /* Limita el ancho del contenedor */
  }
  .tabla {
    width: 100%;
    min-width: 850px;
    border-collapse: collapse;
    background: var(--color-table-bg);
  }

  .tabla th {
    background: var(--color-primary);
    color: var(--color-text-light);
    padding: 15px;
    text-align: left;
    font-weight: 700;
    font-size: 0.95rem;
    text-transform: uppercase;
    position: sticky;
    top: 0;
    z-index: 5;
    letter-spacing: 0.5px;
  }

  .tabla td {
    padding: 12px 15px;
    border-bottom: 1px solid var(--color-border);
    font-size: 0.95rem;
    color: var(--color-text-light);
  }

  .descripcion-celda {
    max-width: 350px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    color: var(--color-text-muted);
  }

  /* Fila activa (Estado = 1) */
  .active-row {
    background-color: rgba(96, 165, 250, 0.1);
  }

  .active-row:hover td {
    background: rgba(96, 165, 250, 0.15);
  }

  tr:hover:not(.active-row) td {
    background: rgba(255, 255, 255, 0.03);
  }

  .resaltado {
    color: var(--color-accent-gold);
    font-weight: 700;
  }

  /* Estilos de Estado */
  .estado {
    padding: 6px 12px;
    border-radius: 20px; /* Pill shape */
    font-weight: 700;
    font-size: 0.75rem;
    text-transform: uppercase;
    display: inline-block;
  }

  .estado-activa {
    background: rgba(16, 185, 129, 0.2);
    color: var(--color-success);
  }

  .estado-cerrada {
    background: rgba(239, 68, 68, 0.2);
    color: var(--color-danger);
  }

  /* ---------------------------------- */
  /* Mensajes y Errores (ANTERIOR: YA NO SE USA) */
  /* ---------------------------------- */
  /*
  .app-message, .error-text, .subtext {
    ...
  }
  */

  .error-text,
  .subtext {
    text-align: center;
    padding: 20px;
    font-size: 1rem;
    color: var(--color-text-muted);
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
  }

  .icon-error {
    color: var(--color-danger);
  }
  .icon-info {
    color: var(--color-accent-blue);
  }

  /* ---------------------------------- */
  /* Animación de Carga (Barra) */
  /* ---------------------------------- */
  .loading-container-bar {
    width: 100%;
    padding: 20px 0;
    display: flex;
    justify-content: center;
  }

  .loading-bar {
    width: 80%;
    max-width: 400px;
    height: 4px;
    background: var(--color-border);
    border-radius: 5px;
    overflow: hidden;
  }

  .loading-bar::before {
    content: "";
    display: block;
    height: 100%;
    width: 30%;
    background: linear-gradient(
      90deg,
      var(--color-accent-gold),
      var(--color-accent-blue)
    );
    animation: loading-move 1.5s infinite linear;
  }

  @keyframes loading-move {
    0% {
      transform: translateX(-100%);
    }
    100% {
      transform: translateX(333%);
    }
  }

  /* ---------------------------------- */
  /* ESTILOS DEL MENSAJE FLOTANTE (REPLICADO DEL LOGIN) */
  /* ---------------------------------- */
  .message-modal {
    position: fixed;
    top: 20px;
    right: 20px;
    z-index: 1000;
    transition:
      opacity 0.3s ease-out,
      transform 0.3s ease-out;
  }

  /* Animación de entrada: slide-in */
  .message-modal.slide-in {
    animation: slideIn 0.5s forwards;
  }

  /* Animación de salida: fade-out + slide-out */
  .message-modal.fade-out {
    opacity: 0;
    transform: translateX(100%);
    transition:
      opacity 0.3s ease-in,
      transform 0.3s ease-in;
  }

  .message-content {
    padding: 15px 25px;
    border-radius: 8px;
    color: white;
    background-color: var(--color-modal-error);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
    display: flex;
    align-items: center;
    gap: 15px;
    font-weight: 600;
    animation: none;
  }

  /* Aplicar la animación de rebote solo si el modal está activo (entrando) */
  .slide-in .message-content {
    animation: bounce 0.5s ease-out;
  }

  .message-content.success {
    background-color: var(--color-modal-success);
  }

  .message-content i {
    font-size: 1.2rem;
  }

  @keyframes slideIn {
    from {
      transform: translateX(100%);
      opacity: 0;
    }
    to {
      transform: translateX(0);
      opacity: 1;
    }
  }

  /* Animación de rebote para el modal */
  @keyframes bounce {
    0%,
    100% {
      transform: translateY(0);
    }
    20% {
      transform: translateY(-5px);
    }
    40% {
      transform: translateY(0);
    }
    60% {
      transform: translateY(-2px);
    }
    80% {
      transform: translateY(0);
    }
  }

  /* ---------------------------------- */
  /* Responsividad (Móvil) */
  /* ---------------------------------- */
  @media (max-width: 768px) {
    .main {
      padding: 30px 10px;
    }

    .message-modal {
      top: 10px;
      right: 10px;
      left: 10px; /* Ocupa casi todo el ancho en móvil */
    }

    .message-content {
      padding: 10px 15px;
      font-size: 0.9rem;
      text-align: left;
    }

    .page-title {
      font-size: 1.8rem;
    }

    .card {
      padding: 20px 15px;
    }

    .subtitulo {
      font-size: 1.2rem;
    }

    .category-tabs {
      gap: 10px;
      width: 100%;
    }

    .tab-btn {
      font-size: 0.9rem;
      padding: 10px 15px;
    }

    /* Ocultar columnas menos importantes en móvil */
    .col-user,
    .col-id {
      display: none;
    }

    .descripcion-celda {
      max-width: 150px;
    }

    .tabla th,
    .tabla td {
      padding: 10px;
    }

    .tabla {
      min-width: 550px;
    }
  }
</style>
