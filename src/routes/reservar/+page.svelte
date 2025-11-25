<script>
  import { onMount } from "svelte";
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";

  // ==============================================
  // NUEVO: Estado para la navegación de pestañas
  // ==============================================
  let activeTab = "crear"; // 'crear' o 'activas'

  // ==============================================
  // Lógica de Reserva
  // ==============================================
  let date_start = "";
  let date_end = "";
  let availRooms = [];
  let selectedRooms = [];
  let reservasActivas = [];

  let user = null; // Variable para almacenar la información del usuario
  let isLoading = false; // Estado para manejar la carga y deshabilitar botones

  // VARIABLES PARA EL SISTEMA DE NOTIFICACIONES PERSONALIZADO
  let message = "";
  let isSuccess = false;
  let showMessage = false;

  // VARIABLES PARA EL MODAL DE CANCELACIÓN PERSONALIZADO
  let isConfirmingCancel = false;
  let reservationToCancel = null; // Almacena el objeto de reserva a cancelar

  // FUNCIÓN PARA OCULTAR LA NOTIFICACIÓN
  function hideNotification() {
    showMessage = false;
    message = "";
  }

  // FUNCIÓN PARA MOSTRAR LA NOTIFICACIÓN
  function showNotification(msg, success) {
    hideNotification();
    isSuccess = success;
    message = msg;
    showMessage = true;

    // Ocultar automáticamente después de 4 segundos
    setTimeout(hideNotification, 4000);
  }

  // Cargar reservas activas del usuario
  async function cargarReservas() {
    if (!user || !user.id_usuario) return;
    try {
      const res = await fetch(
        `https://inntech-backend.onrender.com/reservas/activas/${user.id_usuario}`,
      );
      const data = await res.json();
      if (res.ok) reservasActivas = data.data;
      else reservasActivas = [];
    } catch (e) {
      console.error("Error al cargar reservas:", e);
      reservasActivas = [];
    }
  }

  onMount(() => {
    // Carga del usuario desde localStorage al montar el componente
    const userString = localStorage.getItem("user");
    if (userString) {
      try {
        user = JSON.parse(userString);
        cargarReservas();
      } catch (e) {
        console.error("Error al parsear el usuario de localStorage:", e);
      }
    }
  });

  // ========================
  // Buscar habitaciones disponibles
  // ========================
  async function buscarHabitaciones() {
    hideNotification();
    availRooms = [];
    isLoading = true;

    try {
      if (!date_start || !date_end) {
        showNotification("Seleccione fecha inicio y fecha fin.", false);
        isLoading = false;
        return;
      }
      if (new Date(date_end) <= new Date(date_start)) {
        showNotification(
          "La fecha fin debe ser mayor que la fecha inicio.",
          false,
        );
        isLoading = false;
        return;
      }

      const res = await fetch(
        `https://inntech-backend.onrender.com/habitaciones/habitaciones_disponibles?date_start=${date_start}&date_end=${date_end}`,
      );
      const data = await res.json();

      if (!res.ok) {
        showNotification(data.detail || "Error al buscar habitaciones.", false);
        isLoading = false;
        return;
      }

      availRooms = data.data.map((h) => ({
        id: h.id_habitacion ?? h.id ?? h.id_h,
        nombre: h.nombre ?? h.numero ?? `#${h.id_habitacion ?? h.id ?? h.id_h}`,
        raw: h,
      }));

      if (availRooms.length === 0) {
        showNotification(
          "No hay habitaciones disponibles en esas fechas.",
          false,
        );
      } else {
        showNotification(
          `Se encontraron ${availRooms.length} habitaciones disponibles.`,
          true,
        );
      }
    } catch (e) {
      console.error(e);
      showNotification("No hay conexión con el servidor.", false);
    } finally {
      isLoading = false;
    }
  }

  // ========================
  // Agregar y Quitar habitación
  // ========================
  function agregarHab(id) {
    hideNotification();
    const hab = availRooms.find((r) => r.id === id);
    if (!hab) {
      showNotification("Habitación no encontrada.", false);
      return;
    }
    if (selectedRooms.some((s) => s.id === hab.id)) {
      showNotification("La habitación ya fue seleccionada.", false);
      return;
    }
    selectedRooms = [...selectedRooms, hab];
    showNotification(`Habitación ${hab.nombre} agregada a la selección.`, true);
  }

  function quitarHab(id) {
    const roomName =
      selectedRooms.find((x) => x.id === id)?.nombre || "Habitación";
    selectedRooms = selectedRooms.filter((x) => x.id !== id);
    showNotification(`${roomName} quitada de la selección.`, false);
  }

  // ========================
  // Confirmar reserva
  // ========================
  async function confirmarReserva() {
    hideNotification();
    isLoading = true;

    if (!user) {
      showNotification("No autenticado. Por favor, inicie sesión.", false);
      isLoading = false;
      return;
    }

    if (!date_start || !date_end) {
      showNotification("Las fechas son obligatorias.", false);
      isLoading = false;
      return;
    }

    if (selectedRooms.length === 0) {
      showNotification("Debe seleccionar al menos una habitación.", false);
      isLoading = false;
      return;
    }

    const habitacionesIds = selectedRooms.map((s) => Number(s.id));

    const payload = {
      id_usuario: user.id_usuario,
      date_start,
      date_end,
      habitaciones: habitacionesIds,
    };

    try {
      const res = await fetch(
        "https://inntech-backend.onrender.com/reservas/create_with_rooms",
        {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(payload),
        },
      );

      const data = await res.json();

      if (!res.ok) {
        showNotification(
          data.detail ||
            (data.message ? data.message : "Error al crear la reserva."),
          false,
        );
        return;
      }

      showNotification(
        "¡Reserva creada exitosamente! Puedes verla en Reservas Activas.",
        true,
      );
      selectedRooms = [];
      availRooms = [];
      date_start = "";
      date_end = "";
      await cargarReservas();
      // Opcional: Cambiar a la pestaña de reservas activas después de crear una
      // activeTab = 'activas';
    } catch (e) {
      console.error(e);
      showNotification(
        "No hay conexión con el servidor al intentar crear la reserva.",
        false,
      );
    } finally {
      isLoading = false;
    }
  }

  // ==============================================================
  // FUNCIÓN PARA MOSTRAR EL MODAL DE CANCELACIÓN (Reemplaza a confirm())
  // ==============================================================
  function showCancelConfirmation(id) {
    hideNotification();

    const reserva = reservasActivas.find((r) => r.id_reserva === id);
    if (!reserva) {
      showNotification("Reserva no encontrada o ya ha sido cancelada.", false);
      return;
    }

    // SIMULACIÓN DE LA REGLA DE 24 HORAS
    const today = new Date();
    const startDate = new Date(reserva.date_start);
    // Calcular la diferencia en milisegundos
    const diffTime = startDate.getTime() - today.getTime();
    // 24 horas en milisegundos
    const twentyFourHours = 24 * 60 * 60 * 1000;

    if (diffTime < twentyFourHours) {
      showNotification(
        "Esta reserva no se puede cancelar. Debe ser cancelada con al menos 24 horas de antelación.",
        false,
      );
      return;
    }

    // Almacena la reserva y abre el modal personalizado
    reservationToCancel = reserva;
    isConfirmingCancel = true;
  }

  // ==============================================================
  // FUNCIÓN QUE EJECUTA LA CANCELACIÓN (API Call)
  // ==============================================================
  async function executeCancellation() {
    if (!reservationToCancel) return; // Protección

    isLoading = true;
    const id = reservationToCancel.id_reserva;

    // Cierra el modal inmediatamente para mostrar el estado de carga
    isConfirmingCancel = false;

    try {
      const res = await fetch(
        `https://inntech-backend.onrender.com/reservas/cancelar/${id}`,
        {
          method: "PUT",
        },
      );

      const data = await res.json();

      if (!res.ok) {
        showNotification(
          data.detail || "No se pudo cancelar la reserva.",
          false,
        );
        return;
      }

      showNotification("Reserva cancelada exitosamente.", true);
      await cargarReservas();
    } catch (e) {
      showNotification(
        "Error de conexión al intentar cancelar la reserva.",
        false,
      );
    } finally {
      isLoading = false;
      reservationToCancel = null; // Reinicia el estado de la reserva
    }
  }

  // Función para cerrar el modal sin cancelar
  function cancelConfirmation() {
    isConfirmingCancel = false;
    reservationToCancel = null;
    showNotification("Cancelación detenida.", false);
  }
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

<section class="main-container">
  <div class="content-wrapper">
    <h1 class="page-title">Administrar Reservas</h1>

    <!-- =========================================================== -->
    <!-- NUEVA NAVEGACIÓN DE PESTAÑAS CON EL ESTILO SOLICITADO -->
    <!-- =========================================================== -->
    <div class="category-tabs">
      <button
        class="tab-btn"
        class:active={activeTab === "crear"}
        on:click={() => (activeTab = "crear")}
      >
        <i class="fas fa-plus-circle mr-2"></i> Crear Reserva
      </button>

      <button
        class="tab-btn"
        class:active={activeTab === "activas"}
        on:click={() => (activeTab = "activas")}
      >
        <i class="fas fa-calendar-check mr-2"></i> Reservas Activas
      </button>
    </div>

    <!-- =========================================================== -->
    <!-- CONTENIDO DE LA PESTAÑA 1: CREAR RESERVA -->
    <!-- =========================================================== -->
    {#if activeTab === "crear"}
      <div class="tab-content">
        <!-- CARD PRINCIPAL: CREAR RESERVA -->
        <div class="card card-two-columns">
          <!-- TÍTULO DE LA SECCIÓN PRINCIPAL -->
          <h2 class="card-title">Crear Nueva Reserva</h2>

          <!-- El layout de dos columnas se vuelve una columna en móvil -->
          <div class="two-column-layout">
            <!-- COLUMNA 1: BÚSQUEDA Y DISPONIBLES -->
            <div class="column-box column-search">
              <h3 class="box-subtitle">1. Buscar Disponibilidad</h3>

              <!-- MODIFICACIÓN: form-grid para las fechas y el botón -->
              <div class="form-grid">
                <div class="input-group">
                  <label for="date-start">Fecha de inicio</label>
                  <input id="date-start" type="date" bind:value={date_start} />
                </div>

                <div class="input-group">
                  <label for="date-end">Fecha de fin</label>
                  <input id="date-end" type="date" bind:value={date_end} />
                </div>

                <!-- El botón de búsqueda ahora está en su propio grupo para manejar el margen -->
                <div class="btn-group-search">
                  <button
                    on:click={buscarHabitaciones}
                    class="btn-accent btn-search"
                    disabled={isLoading}
                  >
                    {#if isLoading}
                      <i class="fas fa-spinner fa-spin mr-2"></i> Buscando...
                    {:else}
                      Buscar Habitaciones
                    {/if}
                  </button>
                </div>
              </div>
              <!-- FIN MODIFICACIÓN -->

              <!-- HABITACIONES DISPONIBLES -->
              <div class="room-list available-rooms mt-6">
                <h4 class="list-title">
                  Habitaciones disponibles ({availRooms.length})
                </h4>
                {#if availRooms.length === 0}
                  <p class="empty-message-list">
                    {!date_start || !date_end
                      ? "Ingresa las fechas para buscar."
                      : "No hay habitaciones disponibles para el periodo seleccionado."}
                  </p>
                {:else}
                  <ul class="list">
                    {#each availRooms as hab (hab.id)}
                      <li class="list-item">
                        <span class="room-name"
                          >{hab.nombre} (ID: {hab.id})</span
                        >
                        <button
                          class="btn-secondary btn-add"
                          on:click={() => agregarHab(hab.id)}
                          disabled={selectedRooms.some((s) => s.id === hab.id)}
                        >
                          {#if selectedRooms.some((s) => s.id === hab.id)}
                            Agregada
                          {:else}
                            Agregar
                          {/if}
                        </button>
                      </li>
                    {/each}
                  </ul>
                {/if}
              </div>
            </div>

            <!-- COLUMNA 2: RESUMEN Y CONFIRMACIÓN -->
            <div class="column-box column-summary">
              <h3 class="box-subtitle">2. Seleccionar y Confirmar</h3>

              <!-- HABITACIONES SELECCIONADAS -->
              <div class="room-list selected-rooms">
                <h4 class="list-title">
                  Habitaciones seleccionadas ({selectedRooms.length})
                </h4>
                {#if selectedRooms.length === 0}
                  <p class="empty-message-list">
                    Añade habitaciones disponibles a tu reserva.
                  </p>
                {:else}
                  <ul class="list">
                    {#each selectedRooms as s (s.id)}
                      <li class="list-item selected">
                        <span class="room-name">{s.nombre} (ID: {s.id})</span>
                        <button
                          class="btn-danger btn-remove"
                          on:click={() => quitarHab(s.id)}
                        >
                          Quitar
                        </button>
                      </li>
                    {/each}
                  </ul>
                {/if}
              </div>

              <!-- RESUMEN DE FECHAS -->
              <div class="summary-details mt-6 p-4 bg-gray-800/50 rounded-lg">
                <p class="summary-item">
                  <span class="font-bold text-lg text-white">Inicio:</span>
                  <span class="text-primary font-semibold"
                    >{date_start || "Seleccionar"}</span
                  >
                </p>
                <p class="summary-item">
                  <span class="font-bold text-lg text-white">Fin:</span>
                  <span class="text-primary font-semibold"
                    >{date_end || "Seleccionar"}</span
                  >
                </p>
                <p class="summary-item">
                  <span class="font-bold text-lg text-white">Cantidad:</span>
                  <span class="text-accent font-extrabold text-xl"
                    >{selectedRooms.length}</span
                  >
                </p>
              </div>

              <!-- BOTÓN DE CONFIRMACIÓN -->
              <div class="confirm-action">
                <button
                  class="btn-primary btn-confirm"
                  on:click={confirmarReserva}
                  disabled={selectedRooms.length === 0 || isLoading}
                >
                  {#if isLoading}
                    <i class="fas fa-spinner fa-spin mr-2"></i> Confirmando...
                  {:else}
                    Confirmar Reserva ({selectedRooms.length})
                  {/if}
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    {/if}

    <!-- =========================================================== -->
    <!-- CONTENIDO DE LA PESTAÑA 2: RESERVAS ACTIVAS -->
    <!-- =========================================================== -->
    {#if activeTab === "activas"}
      <div class="tab-content">
        <!-- CARD: MIS RESERVAS ACTIVAS -->
        <div class="card">
          <h2 class="card-title">Reservas Activas</h2>
          {#if reservasActivas.length === 0}
            <p class="empty-message">No tienes reservas activas registradas.</p>
          {:else}
            <!-- CLASE table-container AHORA TIENE LÍMITE DE ALTURA Y SCROLL VERTICAL -->
            <div class="table-container">
              <table>
                <thead>
                  <tr>
                    <th>ID Reserva</th>
                    <th>Fecha Inicio</th>
                    <th>Fecha Fin</th>
                    <th>Acción</th>
                  </tr>
                </thead>
                <tbody>
                  {#each reservasActivas as r (r.id_reserva)}
                    <tr>
                      <td class="highlight">{r.id_reserva}</td>
                      <td>{r.date_start.substring(0, 10)}</td>
                      <td>{r.date_end.substring(0, 10)}</td>
                      <td>
                        <button
                          class="btn-cancel"
                          on:click={() => showCancelConfirmation(r.id_reserva)}
                          disabled={isLoading}
                        >
                          Cancelar
                        </button>
                      </td>
                    </tr>
                  {/each}
                </tbody>
              </table>
            </div>
          {/if}
        </div>
      </div>
    {/if}
  </div>
</section>

<Footer />

<!-- ============================================== -->
<!-- 1. Notificación Personalizada (Modal Flotante) -->
<!-- ============================================== -->
{#if showMessage}
  <div class="message-modal">
    <div class="message-content" class:success={isSuccess} role="alert">
      <i
        class="fas"
        class:fa-check-circle={isSuccess}
        class:fa-times-circle={!isSuccess}
      ></i>
      <p>{message}</p>
    </div>
  </div>
{/if}

<!-- ============================================== -->
<!-- 2. Modal de Confirmación de Cancelación (NUEVO) -->
<!-- ============================================== -->
{#if isConfirmingCancel && reservationToCancel}
  <div
    class="custom-modal-backdrop"
    role="button"
    tabindex="0"
    on:click={cancelConfirmation}
    on:keydown={(e) => e.key === "Enter" && cancelConfirmation()}
  >
    <div
      class="custom-modal-content"
      role="dialog"
      tabindex="-1"
      aria-modal="true"
      aria-labelledby="modal-title"
      on:click|stopPropagation={() => {}}
    >
      <h3 id="modal-title" class="modal-title">Confirmar Cancelación</h3>

      <p class="modal-body">
        ¿Estás seguro de que deseas cancelar la reserva
        <span class="font-bold text-accent">
          #{reservationToCancel.id_reserva}
        </span>
        con fecha de inicio
        <span class="font-bold text-primary">
          {reservationToCancel.date_start.substring(0, 10)}
        </span>
        ?
        <br />
        Esta acción no se puede deshacer.
      </p>

      <div class="modal-actions">
        <button
          class="btn-danger"
          on:click={executeCancellation}
          disabled={isLoading}
        >
          Sí, Cancelar Ahora
        </button>

        <button
          class="btn-cancel"
          on:click={cancelConfirmation}
          disabled={isLoading}
        >
          No, Mantener Reserva
        </button>
      </div>
    </div>
  </div>
{/if}

<style>
  /*
  ==============================================
  Estilos Generales y Paleta de Colores (Dark Mode)
  ==============================================
  */
  :root {
    /* ---------------------------------------------------- */
    /* FONDOS ACTUALIZADOS */
    /* ---------------------------------------------------- */
    --color-bg-primary: #1a202c; /* FONDO PRINCIPAL: Coincide con NavbarA */
    --color-bg-card: #2d3748; /* FONDO DE TARJETA PRINCIPAL: Ligeramente más claro que el fondo para destacar */
    --color-bg-box: #0f172a; /* NUEVO FONDO DE CAJA DE FORMULARIO (Darkest Blue/Slate) */
    /* ---------------------------------------------------- */
    --color-table-bg: #111827; /* Fondo de tabla muy oscuro */
    --color-text-light: #f1f5f9; /* Slate 100 */
    --color-accent: #fcd34d; /* Amarillo Brillante (Botón Buscar) */
    --color-primary: #3b82f6; /* Azul Primario (Botón Confirmar) */
    --color-success: #10b981; /* Verde (Activa) */
    --color-highlight: #60a5fa; /* Azul suave para IDs y destacados */
    --color-danger: #ef4444; /* Rojo (Quitar/Error) */
    --color-border: #4b5563; /* Borde gris para tablas, listas */
    /* Variables usadas en los botones de pestaña */
    --color-dark: #1f2937;
    --color-neutral: #e5e7eb;

    /* Variables de color para el Scrollbar (Añadidas aquí) */
    --scrollbar-track: #2d3748;
    --scrollbar-thumb: #4a5568;
    --scrollbar-thumb-hover: #6b7280;
  }

  /* Contenedor principal */
  .main-container {
    background: var(--color-bg-primary);
    min-height: calc(100vh - 160px);
    padding: 40px 20px;
  }

  .content-wrapper {
    max-width: 1200px;
    margin: 0 auto;
  }

  .page-title {
    text-align: center;
    font-size: clamp(2rem, 4vw, 2.8rem);
    color: var(--color-accent); /* Corregido a color claro */
    margin-bottom: 30px;
    font-weight: 800;
  }

  /*
  ==============================================
  ESTILOS DEL SCROLLBAR (Aplicado a .table-container)
  ==============================================
  */
  .table-container::-webkit-scrollbar {
    width: 8px;
    height: 8px;
  }

  .table-container::-webkit-scrollbar-track {
    background: var(--scrollbar-track);
    border-radius: 10px;
  }

  .table-container::-webkit-scrollbar-thumb {
    background: var(--scrollbar-thumb);
    border-radius: 10px;
  }

  .table-container::-webkit-scrollbar-thumb:hover {
    background: var(--scrollbar-thumb-hover);
  }
  /*
  ==============================================
  FIN ESTILOS DEL SCROLLBAR
  ==============================================
  */

  /*
  ==============================================
  ESTILOS DE PESTAÑA SOLICITADOS
  ==============================================
  */
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

  .tab-content {
    /* Añade una animación de aparición suave al cambiar */
    animation: fadeIn 0.4s ease-in-out;
  }

  @keyframes fadeIn {
    from {
      opacity: 0;
      transform: translateY(10px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  /*
  ==============================================
  Estilos de Contenido (Cards, Columnas, etc.)
  ==============================================
  */
  /* Tarjetas (Cards) */
  .card {
    /* Usa la nueva variable de fondo de caja, ya que la tarjeta principal ahora es el contenido de la pestaña */
    background: var(--color-bg-box);
    padding: 30px;
    margin-bottom: 35px;
    border-radius: 16px;
    box-shadow: 0 10px 20px rgba(0, 0, 0, 0.5);
    color: var(--color-text-light);
  }

  .card-title {
    color: var(--color-accent);
    font-size: 2rem;
    margin-bottom: 25px;
    border-bottom: 3px solid var(--color-accent);
    padding-bottom: 10px;
    font-weight: 700;
  }

  /* Subtítulos dentro de las columnas */
  .box-subtitle {
    color: var(--color-accent);
    font-size: 1.5rem;
    font-weight: 600;
    margin-bottom: 20px;
    padding-bottom: 8px;
  }

  /* LAYOUT DE DOS COLUMNAS (Se vuelve 1 en móvil) */
  .two-column-layout {
    display: grid;
    /* Default: 1 columna para móviles */
    grid-template-columns: 1fr;
    gap: 30px;
  }

  /* ESTILO ACTUALIZADO PARA LAS CAJAS DE FORMULARIO (SIMILAR A EDITAR PERFIL) */
  .column-box {
    background: var(--color-bg-box);
    padding: 25px;
    border-radius: 12px;
    box-shadow:
      0 4px 6px rgba(0, 0, 0, 0.4),
      inset 0 0 10px rgba(0, 0, 0, 0.2); /* Sombra externa e interna */
  }

  /*
  ==============================================
  Formularios (Búsqueda) - Responsivo
  ==============================================
  */
  .input-group {
    display: flex;
    flex-direction: column;
    flex-grow: 1;
  }

  label {
    margin-bottom: 6px;
    font-size: 0.95rem;
    color: var(--color-text-light);
    font-weight: 500;
  }

  input[type="date"] {
    /* Aumento ligero de padding para mejor estética */
    padding: 12px 14px;
    border-radius: 8px;
    border: 1px solid #3b424c; /* Borde del input consistente */
    /* Color de fondo del input más oscuro que el fondo principal */
    background: #161b22;
    color: var(--color-text-light);
    transition:
      border-color 0.3s,
      box-shadow 0.3s;
    font-size: 1rem;
    width: 100%;
    -webkit-appearance: none;
  }

  input[type="date"]:focus {
    outline: none;
    border-color: var(--color-accent);
    box-shadow: 0 0 0 3px rgba(252, 211, 77, 0.3);
  }

  .form-grid {
    display: grid;
    /* 3 columnas: Fecha Inicio | Fecha Fin | Botón */
    grid-template-columns: repeat(3, 1fr);
    gap: 30px;
    align-items: flex-end; /* Asegura que los items se alineen al final */
  }

  /*
  ==============================================
  Botones de Acción (diferentes a los de Pestaña)
  ==============================================
  */
  button {
    padding: 10px 18px;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    font-weight: 700;
    transition: all 0.3s ease;
    text-transform: uppercase;
    font-size: 0.9rem;
    white-space: nowrap;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.4);
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .btn-primary {
    background: var(--color-primary);
    color: white;
  }

  .btn-primary:hover {
    background: #60a5fa;
    transform: translateY(-2px);
    box-shadow: 0 6px 10px rgba(0, 0, 0, 0.4);
  }

  .btn-accent {
    background: var(--color-accent);
    color: var(--color-bg-box);
    font-weight: 700;
    padding: 2px 8px;
    font-size: 0.85rem;
  }

  .btn-accent:hover {
    background: #fbbd24;
    transform: translateY(-1px);
    box-shadow: 0 3px 6px rgba(0, 0, 0, 0.3);
  }

  .btn-secondary {
    background: var(--color-success);
    color: white;
  }

  .btn-secondary:hover {
    background: #059669;
    transform: scale(1.03);
  }

  .btn-danger {
    background: var(--color-danger);
    color: white;
  }

  .btn-danger:hover {
    background: #b91c1c;
    transform: scale(1.03);
  }

  .btn-cancel {
    background: #4b5563;
    color: white;
  }

  .btn-cancel:hover {
    background: #6b7280;
  }

  button:disabled {
    background: #374151;
    color: #9ca3af;
    cursor: not-allowed;
    opacity: 0.8;
    transform: none;
    box-shadow: none;
  }

  /* MODIFICACIÓN DE ESTILOS DE BÚSQUEDA */
  .btn-group-search {
    /* En escritorio, ocupa la tercera columna */
    grid-column: 3 / 4;
    /* Alineación para que el botón se sienta más separado de los inputs */
    display: flex;
    flex-direction: column;
    justify-content: flex-end;
    /* Agregamos un margen superior solo para el espacio en el diseño de 3 columnas */
    margin-top: 10px; /* Pequeño margen para compensar el padding, se ajusta en MQ */
  }

  .btn-search {
    /* Eliminamos height: 100% que causaba problemas de alineación */
    width: 100%;
    font-size: 1rem;
    padding-top: 14px;
    padding-bottom: 14px;
  }
  /* FIN MODIFICACIÓN DE ESTILOS DE BÚSQUEDA */

  .btn-confirm {
    width: 100%;
    margin-top: 20px;
    font-size: 1.1rem;
    padding: 12px 20px;
  }

  /* Icono de carga en botón */
  .mr-2 {
    margin-right: 8px;
  }

  /*
  ==============================================
  Listas y Selección
  ==============================================
  */
  .list-title {
    font-size: 1.1rem;
    font-weight: 600;
    color: var(--color-highlight);
    margin-bottom: 15px;
  }

  .list {
    list-style: none;
    padding: 0;
    max-height: 250px;
    overflow-y: auto;
    margin-bottom: 0;
  }

  /* Aplicar Scrollbar a la lista de habitaciones disponibles */
  .room-list .list::-webkit-scrollbar {
    width: 8px;
  }
  .room-list .list::-webkit-scrollbar-track {
    background: var(--scrollbar-track);
    border-radius: 10px;
  }
  .room-list .list::-webkit-scrollbar-thumb {
    background: var(--scrollbar-thumb);
    border-radius: 10px;
  }
  .room-list .list::-webkit-scrollbar-thumb:hover {
    background: var(--scrollbar-thumb-hover);
  }

  .list-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    /* Color de item de lista ligeramente más oscuro que la columna */
    background: #2d3748;
    padding: 12px 15px;
    margin-bottom: 8px;
    border-radius: 8px;
    border-left: 5px solid var(--color-accent);
    transition: all 0.2s;
  }

  .list-item.selected {
    border-left-color: var(--color-success);
  }

  .list-item button {
    margin-left: 15px;
    padding: 6px 10px;
    font-size: 0.75rem;
  }

  .summary-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 5px 0;
  }

  /*
  ==============================================
  Tabla de Reservas Activas
  ==============================================
  */

  .table-container {
    /* APLICACIÓN DEL LÍMITE DE ALTURA Y SCROLL VERTICAL */
    max-height: 350px; /* Altura máxima para PC/Escritorio */
    overflow-y: auto; /* Habilita el scroll vertical si excede la altura */
    overflow-x: auto;

    /* Estilos visuales para el contenedor scrollable */
    border-radius: 8px;
    box-shadow: inset 0 0 8px rgba(0, 0, 0, 0.5); /* Sombra interior sutil */
    padding: 1px;
    border: 1px solid var(--color-border);
  }

  table {
    width: 100%;
    min-width: 700px;
    border-collapse: separate;
    border-spacing: 0;
    background: var(--color-table-bg);
    /* Quitamos el border-radius del table ya que se aplica al container */
    overflow: hidden;
  }

  /* CLAVE: Hace que el encabezado se pegue al top del contenedor scrollable */
  th {
    padding: 18px 20px;
    background: var(--color-primary);
    color: var(--color-text-light);
    font-size: 1rem;
    font-weight: 700;
    text-align: left;
    position: sticky;
    top: 0;
    z-index: 10;
    text-transform: uppercase;
  }

  td {
    padding: 15px 20px;
    color: var(--color-text-light);
    border-bottom: 1px solid var(--color-border);
    font-size: 0.95rem;
  }

  tbody tr {
    transition: background 0.3s ease;
  }

  tbody tr:last-child td {
    border-bottom: none;
  }

  tbody tr:hover {
    background: #1e293b; /* Fondo en hover sutil */
  }
  .empty-message,
  .empty-message-list {
    color: #9ca3af;
    font-style: italic;
    padding: 15px;
    border: 1px dashed var(--color-border);
    border-radius: 8px;
    text-align: center;
    margin-top: 15px;
  }

  .highlight {
    color: var(--color-accent);
    font-weight: 600;
  }

  /*
  ==============================================
  1. NOTIFICACIÓN PERSONALIZADA (Modal Flotante)
  ==============================================
  */
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

  .message-content.success {
    background-color: var(--color-success);
  }

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
    60% {
      transform: translateY(-2px);
    }
    80% {
      transform: translateY(0);
    }
  }

  /*
  ==============================================
  2. MODAL DE CONFIRMACIÓN DE CANCELACIÓN
  ==============================================
  */
  .custom-modal-backdrop {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.7);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 2000;
  }

  .custom-modal-content {
    background: var(--color-bg-card);
    padding: 30px;
    border-radius: 12px;
    max-width: 450px;
    width: 90%;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.6);
    animation: scaleIn 0.3s ease-out;
  }

  .modal-title {
    color: var(--color-danger);
    font-size: 1.8rem;
    font-weight: 700;
    margin-bottom: 15px;
    border-bottom: 2px solid var(--color-danger);
    padding-bottom: 10px;
  }

  .modal-body {
    color: var(--color-text-light);
    margin-bottom: 25px;
    line-height: 1.5;
  }

  .modal-actions {
    display: flex;
    justify-content: space-between;
    gap: 10px;
  }

  .modal-actions button {
    flex: 1;
    padding: 12px;
  }

  @keyframes scaleIn {
    from {
      transform: scale(0.8);
      opacity: 0;
    }
    to {
      transform: scale(1);
      opacity: 1;
    }
  }

  /*
  ==============================================
  RESPONSIVIDAD
  ==============================================
  */
  @media (min-width: 768px) {
    .two-column-layout {
      grid-template-columns: 2fr 1.5fr; /* Más espacio para búsqueda */
      gap: 40px;
    }
    /* Ajuste el botón de búsqueda para que se alinee con los inputs en pantallas más grandes */
    .btn-group-search {
      margin-top: 0;
    }
  }

  @media (max-width: 767px) {
    .form-grid {
      grid-template-columns: 1fr; /* Una columna */
      gap: 15px;
    }

    .btn-group-search {
      grid-column: 1 / -1; /* Ocupa todo el ancho */
      margin-top: 0; /* Eliminar margen superior en móvil */
    }

    .card {
      padding: 20px;
    }

    .tab-btn {
      flex-grow: 1;
    }

    .table-container {
      /* Altura ligeramente menor para móviles */
      max-height: 300px;
    }

    .custom-modal-content {
      padding: 20px;
    }
  }
</style>
