<script>
  import { onMount } from "svelte";
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";

  // Se eliminan las importaciones de Font Awesome que causaban el error.
  // import { library } from '@fortawesome/fontawesome-svg-core';
  // import { faCheckCircle, faBroom } from '@fortawesome/free-solid-svg-icons';

  const API = "https://inntech-backend.onrender.com/habitaciones";

  let habitaciones = [];
  let loading = true;
  let error = null;

  // NUEVAS VARIABLES PARA EL SISTEMA DE NOTIFICACIÓN
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
        // Notificación de error del servidor
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
        .sort((a, b) => a.estado - b.estado); // Reordenar para que las sucias sigan arriba

      // Notificación de éxito
      showNotification(
        `Habitación ${numero} marcada como limpia con éxito.`,
        true,
        3000,
      );
    } catch (e) {
      console.error("Error de conexión al marcar como limpia:", e);
      // Notificación de error de conexión
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

<!-- Se necesita Font Awesome para los iconos (Check, Error) -->
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
          No hay habitaciones registradas en el sistema.
        </p>
      {:else}
        <!-- El scroll vertical se aplica aquí -->
        <div class="table-wrap">
          <table class="tabla">
            <thead>
              <tr>
                <th class="w-1/12">Número</th>
                <th class="w-3/12">Tipo</th>
                <th class="w-4/12">Estado</th>
                <th class="w-4/12">Acción</th>
              </tr>
            </thead>
            <tbody>
              {#each habitaciones as h (h.numero)}
                <tr class:row-dirty={h.estado === 0}>
                  <td class="highlight">{h.numero}</td>
                  <td>{h.id_thabitacion}</td>

                  <td>
                    {#if h.estado === 1}
                      <span class="status-badge estado-limpia"> Limpia </span>
                    {:else}
                      <span class="status-badge estado-sucia">
                        Necesita limpieza
                      </span>
                    {/if}
                  </td>

                  <td>
                    {#if h.estado === 0}
                      <button
                        class="btn-limpiar"
                        on:click={() => marcarLimpia(h.numero)}
                      >
                        Marcar como Limpia
                      </button>
                    {:else}
                      <span class="ok">✅</span>
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

<!-- Notificación Personalizada (Copiada del Login) -->
{#if showMessage}
  <div class="message-modal">
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

<style>
  /* ============================================== */
  /* Variables y Resets Globales del Tema Oscuro */
  /* ============================================== */
  :root {
    --color-dark: #0f172a; /* Fondo principal de Nav/Footer y Card */
    --color-body-bg: #1a202c; /* Fondo del contenido */
    --color-primary: #1e3a8a; /* Azul principal (Botones, Headings) */
    --color-secondary: #fcd34d; /* Amarillo/Naranja de acento (Títulos) */
    --color-text: #e2e8f0; /* Texto claro */
    --color-success: #10b981; /* Verde para estado Limpia */
    --color-danger: #ef4444; /* Rojo para errores */
    --color-warning: #f59e0b; /* Naranja para estado Sucia (Mejor que rojo puro para alerta) */
    --color-neutral: #94a3b8; /* Gris neutro */
    --color-table-row-hover: #1e293b; /* Gris oscuro para hover */

    /* Variables de color para el Scrollbar */
    --scrollbar-track: #2d3748;
    --scrollbar-thumb: #4a5568;
    --scrollbar-thumb-hover: #6b7280;
  }

  :global(body),
  :global(html) {
    background-color: var(--color-body-bg);
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
    max-width: 1200px;
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
    border: 1px solid rgba(255, 255, 255, 0.05);
  }

  .subtext {
    color: var(--color-neutral);
    text-align: center;
    padding: 15px 0;
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
    /* --- CAMBIOS PARA SCROLL VERTICAL --- */
    max-height: 60vh; /* Altura máxima para que el contenido no se extienda demasiado */
    overflow-y: auto; /* Habilitar scroll vertical cuando sea necesario */
    border: 1px solid #334155; /* Borde sutil para delimitar el área de scroll */
    border-radius: 8px;
    /* ---------------------------------- */
  }

  .tabla {
    width: 100%;
    border-collapse: separate;
    border-spacing: 0;
    background: transparent;
    color: var(--color-text);
  }

  .tabla thead {
    overflow: hidden;
  }

  .tabla th {
    background: var(--color-primary);
    color: white;
    padding: 14px 15px;
    text-align: left;
    font-weight: 700;
    text-transform: uppercase;
    font-size: 0.85rem;

    /* --- CAMBIOS PARA CABECERA FIJA --- */
    position: sticky;
    top: 0; /* Fija la cabecera en la parte superior del .table-wrap */
    z-index: 10;
    /* ---------------------------------- */
  }

  .tabla td {
    padding: 12px 15px;
    border-bottom: 1px solid #334155; /* Separador oscuro */
    vertical-align: middle;
  }

  .tabla tbody tr:hover td {
    background: var(--color-table-row-hover);
  }

  /* Destacar la fila sucia para atención inmediata */
  .row-dirty {
    background: rgba(245, 158, 11, 0.05); /* Sombra muy sutil para las sucias */
  }
  .row-dirty:hover td {
    background: rgba(245, 158, 11, 0.1);
  }

  .highlight {
    color: var(--color-secondary);
    font-weight: 700;
  }

  /* ---------------------------------- */
  /* STATUS BADGES */
  /* ---------------------------------- */
  .status-badge {
    padding: 6px 10px;
    border-radius: 6px;
    font-weight: 600;
    font-size: 0.9rem;
    display: inline-flex;
    align-items: center;
    gap: 6px;
  }

  .estado-limpia {
    color: var(--color-success);
    background: rgba(16, 185, 129, 0.15);
    border: 1px solid var(--color-success);
  }

  .estado-sucia {
    color: var(--color-warning); /* Usamos naranja para 'Necesita limpieza' */
    background: rgba(245, 158, 11, 0.15);
    border: 1px solid var(--color-warning);
  }

  /* ---------------------------------- */
  /* BOTÓN Y ÉXITO */
  /* ---------------------------------- */
  .btn-limpiar {
    background: var(--color-primary);
    color: white;
    border: none;
    padding: 8px 15px;
    border-radius: 6px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.3);
    font-size: 0.9rem;
  }

  .btn-limpiar:hover {
    background: #102a65;
    transform: translateY(-1px);
    box-shadow: 0 4px 10px rgba(30, 58, 138, 0.4);
  }

  .ok {
    color: var(--color-success);
    font-size: 1.5rem;
    font-weight: bold;
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
    animation: bounce 0.5s ease-out; /* Animación de rebote al aparecer */
  }

  /* Clase para mensajes de éxito */
  .message-content.success {
    background-color: var(--color-success);
  }

  /* Clase para mensajes de error/información (usa el color de peligro del tema oscuro) */
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
  /* FIN ESTILOS DEL MENSAJE PERSONALIZADO */

  /* ---------------------------------- */
  /* RESPONSIVIDAD */
  /* ---------------------------------- */
  @media (max-width: 768px) {
    .main-container {
      padding: 20px 10px;
    }

    .card {
      padding: 15px;
    }

    .tabla th,
    .tabla td {
      padding: 10px;
      font-size: 0.9rem;
    }
  }
</style>
