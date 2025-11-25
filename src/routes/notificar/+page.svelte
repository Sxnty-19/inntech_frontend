<script>
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";
  import { onMount } from "svelte";

  // Constantes y Variables de Estado
  const API_URL = "https://inntech-backend.onrender.com/notificaciones";

  let user = null;
  let fullName = "";
  let rol = "";

  let numeroHabitacion = "";
  let descripcion = "";

  let notificaciones = [];
  let error = "";
  let isSubmitting = false;

  // Estado para controlar la vista activa: 'create' o 'history'
  let activeView = "create";

  // ===============================================
  // ESTADOS DEL MODAL FLOTANTE (Toast Notification)
  // ===============================================
  let message = ""; // Contenido del mensaje de la notificación
  let isSuccess = false; // Define si es éxito (verde) o error (rojo)
  let showMessage = false; // Controla si el modal está en el DOM
  let isModalActive = false; // Controla la animación de entrada/salida

  // ===== Cargar usuario del LocalStorage =====
  if (typeof localStorage !== "undefined") {
    const storedUser = localStorage.getItem("user");
    if (storedUser) {
      user = JSON.parse(storedUser);

      // Limpieza de nombre
      const primer_nombre = user.primer_nombre || "";
      const segundo_nombre = user.segundo_nombre
        ? ` ${user.segundo_nombre}`
        : "";
      const primer_apellido = user.primer_apellido || "";
      const segundo_apellido = user.segundo_apellido
        ? ` ${user.segundo_apellido}`
        : "";

      fullName =
        `${primer_nombre}${segundo_nombre} ${primer_apellido}${segundo_apellido}`
          .replace(/\s+/g, " ")
          .trim();

      rol = user.rol;
    }
  }

  // ===============================================
  // FUNCIONES DEL MODAL FLOTANTE (Toast Notification)
  // ===============================================

  // Oculta el modal con animación de salida
  function hideMessageWithTransition(duration = 300) {
    isModalActive = false; // Inicia la animación de fade-out
    return new Promise((resolve) =>
      setTimeout(() => {
        showMessage = false; // Oculta completamente después de la animación
        resolve();
      }, duration),
    );
  }

  // FUNCIÓN ACTUALIZADA: Para mostrar mensajes de forma temporal (usa el nuevo modal flotante)
  async function showFloatingMessage(type, text) {
    if (showMessage) {
      await hideMessageWithTransition(100); // Rápida transición si ya había uno
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

  // ===== Crear notificación =====
  async function crearNotificacion() {
    if (!numeroHabitacion || !descripcion) {
      showFloatingMessage(
        "error",
        "Por favor, complete el número de habitación y la descripción.",
      );
      return;
    }

    if (isSubmitting) return;

    isSubmitting = true;
    error = "";

    // Oculta el mensaje si estaba visible antes de la nueva acción
    if (showMessage) await hideMessageWithTransition(100);

    const formData = new FormData();
    formData.append("id_usuario", user.id_usuario);
    formData.append("numero_habitacion", numeroHabitacion);
    formData.append("descripcion", descripcion);
    formData.append("estado", 1);

    try {
      const res = await fetch(`${API_URL}/crear_por_numero`, {
        method: "POST",
        body: formData,
      });

      const data = await res.json();

      if (!res.ok || !data.success) {
        showFloatingMessage(
          "error",
          data.detail ?? "Error al crear notificación.",
        );
      } else {
        showFloatingMessage(
          "success",
          `Notificación creada correctamente para Habitación ${numeroHabitacion}! Estado: Activa`,
        );
        numeroHabitacion = "";
        descripcion = "";
        await cargarNotificaciones();
        // Mover a la vista de historial después de crear
        activeView = "history";
      }
    } catch (e) {
      error = "Error de conexión con el servidor.";
      showFloatingMessage(
        "error",
        "Error de conexión con el servidor. Intente de nuevo.",
      );
    } finally {
      isSubmitting = false;
    }
  }

  // ===== Cargar notificaciones del usuario =====
  async function cargarNotificaciones() {
    if (!user) return;

    try {
      const res = await fetch(`${API_URL}/usuario/${user.id_usuario}`);
      const data = await res.json();

      if (!res.ok) {
        error = data.detail ?? "No hay notificaciones o error al cargar.";
        notificaciones = [];
      } else {
        // Ordenar por ID o fecha de creación descendente para ver las últimas
        notificaciones = data.data.sort(
          (a, b) => b.id_notificacion - a.id_notificacion,
        );
      }
    } catch (e) {
      error = "Error al cargar notificaciones.";
    }
  }

  // onMount para cargar al inicio
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
  <div class="main-content">
    <div class="welcome-box">
      <h1 class="page-title">Centro de Solicitudes</h1>
      <p class="subtext">
        Gestiona tus peticiones de servicio y revisa su estado en tiempo real.
      </p>
    </div>

    <!-- EL MENSAJE EN BLOQUE ANTERIOR HA SIDO ELIMINADO -->

    <!-- NAVEGACIÓN TIPO PESTAÑA -->
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
        Ver Historial
      </button>
    </div>

    <div class="content-display">
      <!-- VISTA 1: CREAR SOLICITUD -->
      {#if activeView === "create"}
        <div class="form-box card">
          <h2 class="section-title">Nueva Petición de Servicio</h2>

          {#if error}
            <p class="error-text">{error}</p>
          {/if}

          <div class="form-group">
            <label for="room-number">Número de habitación</label>
            <input
              type="number"
              id="room-number"
              bind:value={numeroHabitacion}
              placeholder="Ej: 203"
              class="input-field"
              min="1"
            />
          </div>

          <div class="form-group">
            <label for="description">Descripción de la solicitud</label>
            <textarea
              id="description"
              bind:value={descripcion}
              placeholder="Necesito toallas extra, el aire acondicionado no funciona, etc."
              class="input-field"
            ></textarea>
          </div>

          <button
            class="btn btn-primary"
            on:click={crearNotificacion}
            disabled={isSubmitting}
          >
            {isSubmitting ? "Enviando..." : "Enviar Solicitud"}
          </button>
        </div>
      {/if}

      <!-- VISTA 2: HISTORIAL DE SOLICITUDES -->
      {#if activeView === "history"}
        <div class="history-box card">
          <h2 class="section-title">Historial de Solicitudes</h2>

          {#if notificaciones.length > 0}
            <div class="table-wrap">
              <table class="tabla">
                <thead>
                  <tr>
                    <th class="col-id">ID</th>
                    <th>Habitación</th>
                    <th>Descripción</th>
                    <th>Estado</th>
                    <th>Fecha</th>
                  </tr>
                </thead>
                <tbody>
                  {#each notificaciones as item}
                    <tr>
                      <td class="highlight-id col-id">{item.id_notificacion}</td
                      >
                      <td class="highlight-room">{item.id_habitacion}</td>
                      <td class="description-cell">{item.descripcion}</td>
                      <td>
                        <!-- Mostrar estado con estilo visual -->
                        {#if item.estado === 1}
                          <span class="estado estado-activa">Activa</span>
                        {:else}
                          <span class="estado estado-cerrada">Cerrada</span>
                        {/if}
                      </td>
                      <td>{item.date_created.substring(0, 10)}</td>
                    </tr>
                  {/each}
                </tbody>
              </table>
            </div>
          {:else if !error}
            <p class="subtext-center">
              No tienes solicitudes de servicio registradas.
            </p>
          {/if}
        </div>
      {/if}
    </div>
  </div>

  <!-- ---------------------------------- -->
  <!-- NOTIFICACIÓN FLOTANTE (Toast Notification) -->
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
    --color-dark: #0f172a; /* Fondo muy oscuro */
    --color-card-bg: #1a202c; /* Fondo de Tarjeta (Ligeramente más claro) */

    --color-accent: #fcd34d; /* Amarillo principal */
    --color-primary: #1e3a8a; /* Azul (para botón activo) */
    --color-text-light: #e2e8f0; /* Texto claro */
    --color-text-muted: #94a3b8; /* Texto sutil */
    --color-border: #334155; /* Borde sutil */
    --color-success: #10b981; /* Verde (Activa) */
    --color-danger: #ef4444; /* Rojo (Cerrada/Error) */
    --color-highlight: #60a5fa; /* Azul claro para títulos */
    --shadow-card: 0 5px 15px rgba(0, 0, 0, 0.3); /* Sombra de tarjeta más profunda */

    /* COLORES PARA EL MODAL FLOTANTE */
    --color-modal-success: var(--color-success); /* Verde para éxito */
    --color-modal-error: var(--color-danger); /* Rojo para error */
  }

  :global(body),
  :global(html) {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    background-color: var(--color-dark);
  }

  /* ---------------------------------- */
  /* Estilos principales */
  /* ---------------------------------- */

  .main {
    background: var(--color-card-bg);
    min-height: calc(100vh - 160px);
    padding: 30px 20px;
    color: var(--color-text-light);
    font-family: "Inter", sans-serif;
  }
  .main-content {
    max-width: 1200px;
    margin: 0 auto;
  }

  .welcome-box {
    text-align: center;
    margin-bottom: 40px;
  }

  .page-title {
    font-size: clamp(2rem, 4vw, 3rem);
    color: var(--color-accent);
    margin-bottom: 5px;
    font-weight: 800;
  }

  .subtext {
    color: var(--color-text-muted);
    font-size: 1.1rem;
  }

  .subtext-center {
    text-align: center;
    color: var(--color-text-muted);
    padding: 30px 10px;
  }

  .content-display {
    padding-top: 10px;
  }

  /* ---------------------------------- */
  /* NAVEGACIÓN POR PESTAÑAS (TABS) */
  /* ---------------------------------- */
  .category-tabs {
    display: flex;
    justify-content: center;
    gap: 15px;
    /* Reducimos el margen inferior aquí para que se vea más pegado al contenido */
    margin-bottom: 35px;
    flex-wrap: wrap;
  }

  .tab-btn {
    padding: 12px 25px;
    /* Usamos color-border para el borde, que es un gris oscuro */
    border: 2px solid var(--color-border);
    border-radius: 8px;
    background-color: var(--color-dark);
    color: var(--color-neutral);
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
    font-size: 1rem;
    display: flex;
    align-items: center;
    text-transform: none; /* Quitamos uppercase en los botones de pestaña */
  }

  .tab-btn:hover {
    background-color: #2a3547; /* Un poco más claro en hover */
    color: var(--color-text-light);
    transform: translateY(-2px); /* Pequeño levantamiento en hover */
  }

  .tab-btn.active {
    border-color: var(--color-primary);
    background-color: var(--color-primary);
    color: var(--color-text-light);
    /* Sombra azul brillante solicitada */
    box-shadow: 0 0 15px rgba(59, 130, 246, 0.8);
    transform: translateY(0); /* Evita que el botón activo se levante */
  }

  /* ---------------------------------- */
  /* Tarjeta Contenedora (Card) */
  /* ---------------------------------- */

  .card {
    background: var(--color-dark);
    border: 1px solid var(--color-border);
    padding: clamp(20px, 4vw, 30px);
    border-radius: 12px;
    box-shadow: var(--shadow-card);
    margin: 0 auto;
  }

  /* MEJORA CLAVE: Aumentar el ancho del formulario */
  .form-box {
    max-width: 650px; /* Ancho más amplio para el formulario */
    min-height: 400px;
    transition: all 0.3s;
  }

  .history-box {
    max-width: 900px;
    min-height: 400px;
    transition: all 0.3s;
  }

  .section-title {
    color: var(--color-highlight);
    font-size: clamp(1.4rem, 2.5vw, 1.8rem);
    margin-bottom: 25px;
    font-weight: 700;
    text-align: center;
    border-bottom: 1px solid var(--color-border);
    padding-bottom: 10px;
    max-width: 400px;
    margin: 0 auto 30px auto; /* Espaciado extra después del título */
  }

  /* ---------------------------------- */
  /* Formulario (Creación) - MEJORAS APLICADAS */
  /* ---------------------------------- */

  .form-group {
    text-align: left;
    margin-bottom: 25px; /* Más espacio entre campos */
  }

  label {
    display: block;
    font-weight: 600;
    color: var(--color-text-light);
    margin-bottom: 8px; /* Más espacio entre etiqueta y campo */
    font-size: 1rem;
  }

  .input-field {
    /* FIX CRÍTICO: Asegura que el padding y el border se incluyan dentro del 100% del ancho */
    box-sizing: border-box;
    width: 100%;
    padding: 14px 18px; /* Aumentar el padding para ser más prominente */
    background: var(--color-table-bg);
    color: var(--color-text-light);
    border: 1px solid var(--color-border);
    border-radius: 10px; /* Bordes un poco más redondeados */
    transition: all 0.3s;
  }

  .input-field:focus {
    border-color: var(--color-accent);
    box-shadow: 0 0 0 3px rgba(252, 211, 77, 0.4); /* Sombra de enfoque más visible */
    outline: none;
  }

  textarea {
    height: 120px;
    resize: vertical;
  }

  .btn {
    width: 100%;
    padding: 14px; /* Aumentar padding del botón */
    margin-top: 30px; /* Más margen superior */
    font-weight: 800; /* Más peso a la fuente */
    border: none;
    border-radius: 10px;
    cursor: pointer;
    transition:
      background 0.3s,
      transform 0.1s,
      box-shadow 0.3s;
    font-size: 1.05rem;
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
  }

  .btn-primary {
    background: var(--color-accent);
    color: #111; /* Color de texto más oscuro para alto contraste */
  }

  .btn-primary:hover:not(:disabled) {
    background: #ffaa2b;
    transform: translateY(-2px);
    box-shadow: 0 6px 15px rgba(0, 0, 0, 0.3);
  }

  .btn:disabled {
    background: var(--color-border);
    color: var(--color-text-muted);
    cursor: not-allowed;
    opacity: 0.7;
  }

  /* ---------------------------------- */
  /* Tabla (Historial) */
  /* ---------------------------------- */

  .history-box {
    padding-top: 20px;
  }

  .table-wrap {
    overflow-x: auto;
    border-radius: 8px;
    max-height: 400px;
    overflow-y: auto;
    border: 1px solid var(--color-border);
  }

  .tabla {
    width: 100%;
    min-width: 500px;
    border-collapse: separate;
    border-spacing: 0;
    background: var(--color-table-bg);
    border-radius: 8px;
    overflow: hidden;
  }

  .tabla th {
    background: var(--color-card-bg);
    color: var(--color-highlight);
    padding: 15px;
    text-align: left;
    font-weight: 700;
    font-size: 0.9rem;
    text-transform: uppercase;
    position: sticky;
    top: 0;
    z-index: 5;
    border-bottom: 2px solid var(--color-border);
  }

  .tabla th:first-child,
  .tabla td:first-child {
    /* ID solo visible en desktop */
    display: table-cell;
  }

  .tabla td {
    padding: 12px 15px;
    border-bottom: 1px solid rgba(255, 255, 255, 0.05);
    font-size: 0.9rem;
    color: var(--color-text-light);
  }

  .description-cell {
    max-width: 250px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    color: var(--color-text-muted);
  }

  .highlight-id {
    color: var(--color-highlight);
    font-weight: 600;
  }

  .highlight-room {
    color: var(--color-accent);
    font-weight: 600;
  }

  tr:hover td {
    background: rgba(255, 255, 255, 0.03);
  }

  /* Estilos de Estado */
  .estado {
    padding: 4px 10px;
    border-radius: 4px;
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
  /* Mensajes de App (Éxito/Error) - ELIMINADO EL ESTILO .app-message */
  /* ---------------------------------- */

  .error-text {
    color: var(--color-danger);
    text-align: center;
    margin-bottom: 15px;
  }

  /* ---------------------------------- */
  /* ESTILOS DEL MENSAJE FLOTANTE (Toast Notification) */
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

    .category-tabs {
      gap: 10px;
      margin-bottom: 30px;
      width: 100%;
      justify-content: space-between;
    }

    .tab-btn {
      flex: 1;
      padding: 10px 5px;
      font-size: 0.9rem;
    }

    .card {
      padding: 20px 15px;
    }

    .form-box {
      max-width: 100%; /* El formulario vuelve a ocupar todo el ancho en móvil */
    }

    .tabla th,
    .tabla td {
      padding: 10px;
    }

    /* Ocultar la columna ID en móvil */
    .col-id {
      display: none !important;
    }

    /* Responsividad del modal flotante en móvil */
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
  }
</style>
