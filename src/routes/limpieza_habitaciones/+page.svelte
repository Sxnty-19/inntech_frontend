<script>
  import { onMount } from "svelte";
  // NOTA: Asumimos que estos componentes existen y se importan correctamente.
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";

  const API = "https://inntech-backend.onrender.com/habitaciones";

  let habitaciones = [];
  let loading = true;
  let error = null;

  // SISTEMA DE NOTIFICACIÓN
  let message = "";
  let isSuccess = false;
  let showMessage = false;

  /**
   * Muestra una notificación personalizada.
   * @param {string} msg - El mensaje a mostrar.
   * @param {boolean} success - Si es un mensaje de éxito (verde) o error/info (rojo).
   * @param {number} duration - Duración en ms para mostrar el mensaje.
   */
  function showNotification(msg, success, duration = 3000) {
    showMessage = false; // Forzar reinicio de la animación si ya estaba visible
    message = msg;
    isSuccess = success;
    showMessage = true;

    // Ocultar automáticamente después del tiempo especificado
    setTimeout(() => {
      showMessage = false;
    }, duration);
  }

  async function cargarHabitaciones() {
    loading = true;
    error = null;

    try {
      const res = await fetch(`${API}/get_habitaciones`);
      const data = await res.json();

      if (!data.success) throw new Error(data.detail ?? "Error al cargar.");

      // Ordenar: las sucias (0) primero, luego las limpias (1)
      habitaciones = data.data.sort((a, b) => a.estado - b.estado);
    } catch (e) {
      console.error(e);
      error =
        "No se pudieron cargar las habitaciones. Verifique la conexión al servidor.";
      // Notificación de error en caso de fallo de carga
      showNotification(error, false, 4000);
    } finally {
      loading = false;
    }
  }

  // Actualizar el estado de la habitación
  async function marcarLimpia(numero) {
    try {
      const formData = new FormData();
      formData.append("numero", numero);
      formData.append("estado", 1); // 1 = Limpia

      const res = await fetch(`${API}/actualizar_estado`, {
        method: "PUT",
        body: formData,
      });

      const data = await res.json();

      if (!data.success) {
        console.error("Error del servidor al actualizar:", data.detail);
        showNotification(
          `Error al marcar habitación ${numero}: ${data.detail || "Fallo desconocido."}`,
          false,
          4000,
        );
        return;
      }

      // Actualizar visualmente y reordenar
      habitaciones = habitaciones
        .map((h) => (h.numero === numero ? { ...h, estado: 1 } : h))
        .sort((a, b) => a.estado - b.estado);

      // Notificación de éxito
      showNotification(
        `Habitación ${numero} marcada como limpia con éxito.`,
        true,
        3000,
      );
    } catch (e) {
      console.error("Error de conexión al marcar como limpia:", e);
      showNotification(
        "Error de conexión. Inténtalo de nuevo más tarde.",
        false,
        4000,
      );
    }
  }

  onMount(() => {
    cargarHabitaciones();
  });
</script>

<!-- Se necesita Font Awesome para los iconos (Check, Error, Broom) -->
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

<section class="main-container">
  <div class="main-content">
    <h1 class="page-title">Limpieza de Habitaciones</h1>
    <p class="intro-text">
      Gestión rápida del estado de limpieza. Las habitaciones pendientes
      aparecen primero.
    </p>

    <div class="card">
      {#if loading}
        <div class="loading-bar"></div>
        <p class="subtext">Cargando datos de habitaciones...</p>
      {:else if error}
        <p class="message error-msg">{error}</p>
      {:else if habitaciones.length === 0}
        <p class="message subtext">
          <i class="fas fa-info-circle"></i> No hay habitaciones registradas en el
          sistema.
        </p>
      {:else}
        <div class="table-wrap">
          <table class="tabla">
            <thead>
              <tr>
                <th class="w-1/12 col-numero">Número</th>
                <th class="w-3/12 col-tipo">Tipo</th>
                <th class="w-4/12 col-estado">Estado</th>
                <th class="w-4/12 col-accion">Acción</th>
              </tr>
            </thead>
            <tbody>
              {#each habitaciones as h (h.numero)}
                <!-- row-dirty aplica estilos a la fila de habitaciones sucias -->
                <tr class:row-dirty={h.estado === 0}>
                  <td class="highlight" data-label="Número">{h.numero}</td>
                  <td data-label="Tipo">{h.id_thabitacion}</td>

                  <td data-label="Estado">
                    {#if h.estado === 1}
                      <!-- CLASE ACTUALIZADA: .estado en lugar de .status-badge -->
                      <span class="estado estado-limpia">
                        <i class="fas fa-check-circle"></i> Limpia
                      </span>
                    {:else}
                      <!-- CLASE ACTUALIZADA: .estado en lugar de .status-badge -->
                      <span class="estado estado-sucia">
                        <i class="fas fa-broom"></i> Sucia
                      </span>
                    {/if}
                  </td>

                  <td data-label="Acción">
                    {#if h.estado === 0}
                      <!-- CLASE ACTUALIZADA: Usando el estilo btn-toggle-clean para el botón de limpiar -->
                      <button
                        class="btn-toggle btn-toggle-clean"
                        on:click={() => marcarLimpia(h.numero)}
                      >
                        <i class="fas fa-magic"></i> Marcar como lista
                      </button>
                    {:else}
                      <span class="ok">Habitación lista</span>
                    {/if}
                  </td>
                </tr>
              {/each}
            </tbody>
          </table>
        </div>
      {/if}
    </div>
  </div>
</section>

<Footer />

<!-- Notificación Personalizada -->
{#if showMessage}
  <div class="message-modal">
    <div class="message-content" class:success={isSuccess}>
      <i
        class="fas"
        class:fa-check-circle={isSuccess}
        class:fa-times-circle={!isSuccess}
      ></i>
      <p>{message}</p>
    </div>
  </div>
{/if}

<style>
  /* ============================================== */
  /* Variables y Resets Globales del Tema Oscuro */
  /* ============================================== */
  :root {
    --color-dark: #0f172a; /* Fondo principal de Nav/Footer y Card (slate-900) */
    --color-body-bg: #1a202c; /* Fondo del contenido (gray-900) */
    --color-primary: #1e3a8a; /* Azul principal (Botones, Headings) */
    --color-secondary: #fcd34d; /* Amarillo/Naranja de acento (Títulos) */
    --color-text: #e2e8f0; /* Texto claro (slate-200) */
    --color-danger: #ef4444; /* Rojo para errores (red-500) */
    --color-neutral: #94a3b8; /* Gris neutro (slate-400) */
    --color-table-row-hover: #1e293b; /* Gris oscuro para hover (slate-800) */
    --color-border: #334155; /* Borde sutil (slate-700) */

    /* NUEVAS VARIABLES para los estados y botones */
    --color-clean: #10b981; /* Verde esmeralda para Limpia (emerald-500) */
    --color-dirty: #ef4444; /* Rojo para Sucia (red-500) */
    --color-toggle-clean-bg: #2563eb; /* Azul para el botón de Limpiar */
    --color-toggle-dirty-bg: #facc15; /* Amarillo para el botón de Sucia (no usado aquí, pero definido) */

    /* Variables de color para el Scrollbar */
    --scrollbar-track: #2d3748;
    --scrollbar-thumb: #4a5568;
    --scrollbar-thumb-hover: #6b7280;
  }

  :global(body),
  :global(html) {
    background-color: var(--color-body-bg);
    font-family: "Inter", sans-serif;
  }

  /* ---------------------------------- */
  /* DISEÑO DEL SCROLLBAR (solo WebKit) */
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
  /* ---------------------------------- */

  /* ---------------------------------- */
  /* LAYOUT PRINCIPAL */
  /* ---------------------------------- */
  .main-container {
    min-height: calc(100vh - 110px);
    background: var(--color-body-bg);
    padding: 40px 20px;
    color: var(--color-text);
  }

  .main-content {
    max-width: 1000px;
    margin: 0 auto;
  }

  .page-title {
    text-align: center;
    font-size: clamp(2rem, 4vw, 2.8rem);
    color: var(--color-secondary);
    margin-bottom: 10px;
    font-weight: 800;
  }

  .intro-text {
    text-align: center;
    color: var(--color-neutral);
    margin-bottom: 30px;
    font-size: 1rem;
    max-width: 700px;
    margin-left: auto;
    margin-right: auto;
  }

  /* ---------------------------------- */
  /* CAJA / CARD */
  /* ---------------------------------- */
  .card {
    background: var(--color-dark);
    padding: 30px;
    border-radius: 12px;
    box-shadow: 0 10px 25px rgba(0, 0, 0, 0.5);
    border: 1px solid var(--color-border);
  }

  .subtext {
    color: var(--color-neutral);
    text-align: center;
    padding: 15px 0;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
  }

  .message {
    text-align: center;
    padding: 20px;
    border-radius: 6px;
    font-weight: 600;
  }

  .error-msg {
    color: var(--color-danger);
    background: rgba(239, 68, 68, 0.1);
    border: 1px solid var(--color-danger);
  }

  /* ---------------------------------- */
  /* TABLA */
  /* ---------------------------------- */
  .table-wrap {
    overflow-x: auto;
    max-height: 65vh;
    overflow-y: auto;
    border: 1px solid var(--color-border);
    border-radius: 8px;
  }

  .tabla {
    width: 100%;
    min-width: 600px;
    border-collapse: collapse;
    background: transparent;
    color: var(--color-text);
  }

  .tabla th {
    background: var(--color-primary);
    color: white;
    padding: 14px 15px;
    text-align: left;
    font-weight: 700;
    text-transform: uppercase;
    font-size: 0.85rem;
    position: sticky;
    top: 0;
    z-index: 10;
  }

  .tabla thead tr:first-child th:first-child {
    border-top-left-radius: 8px;
  }
  .tabla thead tr:first-child th:last-child {
    border-top-right-radius: 8px;
  }

  .tabla td {
    padding: 12px 15px;
    border-bottom: 1px solid #334155;
    vertical-align: middle;
  }

  .tabla tbody tr:last-child td {
    border-bottom: none;
  }

  .tabla tbody tr:hover td {
    background: var(--color-table-row-hover);
  }

  /* Destacar la fila sucia para atención inmediata (usa color de sucio/rojo) */
  .row-dirty {
    background: rgba(239, 68, 68, 0.05);
  }
  .row-dirty:hover td {
    background: rgba(239, 68, 68, 0.1);
  }

  .highlight {
    color: var(--color-text);
    font-weight: 700;
  }

  .col-numero {
    width: 10%;
  }
  .col-tipo {
    width: 30%;
  }
  .col-estado {
    width: 30%;
  }
  .col-accion {
    width: 30%;
  }

  /* ---------------------------------- */
  /* ESTILOS DE ESTADO (NUEVOS) */
  /* ---------------------------------- */
  .estado {
    padding: 6px 12px;
    border-radius: 20px; /* Borde más redondeado/Pill shape */
    font-weight: 700;
    font-size: 0.75rem;
    text-transform: uppercase;
    display: inline-flex; /* Usar flex para alinear icono y texto */
    align-items: center;
    gap: 5px;
  }

  .estado-limpia {
    /* Fondo verde claro, texto verde oscuro */
    background: rgba(16, 185, 129, 0.2);
    color: var(--color-clean);
  }

  .estado-sucia {
    /* Fondo rojo claro, texto rojo oscuro */
    background: rgba(239, 68, 68, 0.2);
    color: var(--color-dirty);
  }

  /* ---------------------------------- */
  /* BOTÓN DE ACCIÓN (NUEVO ESTILO DE TOGGLE) */
  /* ---------------------------------- */
  .btn-toggle {
    border: none;
    padding: 8px 15px;
    cursor: pointer;
    border-radius: 6px;
    font-weight: 600;
    transition: all 0.2s;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 5px;
    font-size: 0.9rem;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
  }

  /* Usamos el estilo 'clean' para el botón de Marcar Limpio */
  .btn-toggle-clean {
    background: var(--color-primary);
    color: white;
  }

  .btn-toggle-clean:hover {
    background: #1d4ed8; /* Azul oscuro */
    transform: translateY(-1px);
    box-shadow: 0 4px 8px rgba(37, 99, 235, 0.5);
  }

  /* Este estilo no se usa en esta vista, pero se deja definido por si se necesita */
  .btn-toggle-dirty {
    background: var(--color-toggle-dirty-bg);
    color: #111; /* Color de texto oscuro para fondo claro */
  }

  .btn-toggle-dirty:hover {
    background: #eab308; /* Amarillo oscuro */
  }

  .ok {
    color: var(--color-neutral);
    font-weight: 600;
    font-size: 0.95rem;
  }

  /* ---------------------------------- */
  /* LOADING ANIMATION */
  /* ---------------------------------- */
  .loading-bar {
    height: 4px;
    width: 100%;
    border-radius: 2px;
    overflow: hidden;
    margin: 10px 0;
  }
  .loading-bar:before {
    content: "";
    display: block;
    height: 100%;
    width: 50%;
    background: var(--color-secondary);
    animation: loading-animate 1.5s infinite linear;
  }
  @keyframes loading-animate {
    0% {
      transform: translateX(-100%);
    }
    100% {
      transform: translateX(200%);
    }
  }

  /* ---------------------------------- */
  /* ESTILOS DEL MENSAJE PERSONALIZADO */
  /* ---------------------------------- */
  .message-modal {
    position: fixed;
    top: 20px;
    right: 20px;
    z-index: 1000;
    animation: slideIn 0.5s forwards;
    pointer-events: none;
  }

  .message-content {
    padding: 15px 25px;
    border-radius: 8px;
    color: white;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
    display: flex;
    align-items: center;
    gap: 15px;
    font-weight: 600;
    animation: bounce 0.5s ease-out;
    pointer-events: auto;
  }

  /* Clase para mensajes de éxito */
  .message-content.success {
    background-color: var(--color-clean); /* Usa el color verde de 'clean' */
  }

  /* Clase para mensajes de error/información */
  .message-content:not(.success) {
    background-color: var(--color-danger);
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
  }
  /* FIN ESTILOS DEL MENSAJE PERSONALIZADO */

  /* ---------------------------------- */
  /* RESPONSIVIDAD (Móvil) */
  /* ---------------------------------- */
  @media (max-width: 768px) {
    .main-container {
      padding: 20px 10px;
    }

    .card {
      padding: 15px;
    }

    .tabla {
      min-width: 450px;
    }

    .tabla thead {
      display: none;
    }

    .tabla,
    .tabla tbody,
    .tabla tr,
    .tabla td {
      display: block;
      width: 100%;
    }

    .tabla tr {
      margin-bottom: 15px;
      border: 1px solid var(--color-border);
      border-radius: 8px;
      overflow: hidden;
      background: var(--color-dark);
    }

    .tabla td {
      border: none;
      border-bottom: 1px solid var(--color-border);
      text-align: right;
      padding-left: 50%;
      position: relative;
    }

    .tabla td:last-child {
      border-bottom: 0;
      text-align: center;
      padding-left: 10px;
    }

    .tabla td::before {
      content: attr(data-label);
      position: absolute;
      left: 10px;
      width: calc(50% - 20px);
      padding-right: 10px;
      white-space: nowrap;
      font-weight: 700;
      color: var(--color-secondary);
      text-align: left;
    }

    .btn-toggle {
      width: 100%;
    }
  }
</style>
