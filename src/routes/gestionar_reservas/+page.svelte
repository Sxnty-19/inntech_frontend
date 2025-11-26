<script>
  import { onMount } from "svelte";
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte"; // Componente Admin
  import Footer from "$lib/components/Footer.svelte";

  // ==============================================
  // ESTADO DE NAVEGACIÓN Y CARGA
  // ==============================================
  let activeTab = "crear"; // 'crear' o 'listado'
  let isLoading = false; // Estado para manejar la carga y deshabilitar botones
  let isSubmitting = false; // Estado para Confirmar Reserva
  let loadingListado = true; // Estado para cargar el listado de TODAS las reservas

  // ==============================================
  // LÓGICA DE RESERVA ADMIN
  // ==============================================
  let id_usuario = ""; // ID del usuario al que se le creará la reserva (ADMIN)
  let date_start = "";
  let date_end = "";
  let availRooms = []; // Habitaciones disponibles para las fechas
  let selectedRooms = []; // Habitaciones seleccionadas para la reserva
  let reservas = []; // TODAS las reservas (Activas, Canceladas, etc.)

  // ==============================================
  // SISTEMA DE NOTIFICACIONES Y MODAL (Replicado de Cliente)
  // ==============================================
  let message = "";
  let isSuccess = false;
  let showMessage = false;

  let isConfirmingCancel = false;
  let reservationToCancel = null;

  function hideNotification() {
    showMessage = false;
    message = "";
  }

  function showNotification(msg, success) {
    hideNotification();
    isSuccess = success;
    message = msg;
    showMessage = true;

    // Ocultar automáticamente después de 4 segundos
    setTimeout(hideNotification, 4000);
  }

  // ==============================================
  // Cargar TODAS las reservas (ADMIN)
  // ==============================================
  async function cargarReservas() {
    loadingListado = true;
    try {
      const res = await fetch(
        "https://inntech-backend.onrender.com/reservas/get_reservas",
      );
      const data = await res.json();
      if (res.ok) reservas = data.data;
      else reservas = [];
    } catch (e) {
      console.error("Error al cargar todas las reservas:", e);
      reservas = [];
      showNotification("Error de conexión al cargar el listado.", false);
    } finally {
      loadingListado = false;
    }
  }

  onMount(() => {
    cargarReservas();
  });

  // ==============================================
  // Buscar habitaciones disponibles
  // ==============================================
  async function buscarHabitaciones() {
    hideNotification();
    availRooms = [];
    isLoading = true;

    try {
      if (!date_start || !date_end) {
        showNotification("Seleccione fecha inicio y fecha fin.", false);
        return;
      }
      if (new Date(date_end) <= new Date(date_start)) {
        showNotification(
          "La fecha fin debe ser posterior a la fecha inicio.",
          false,
        );
        return;
      }

      const res = await fetch(
        `https://inntech-backend.onrender.com/habitaciones/habitaciones_disponibles?date_start=${date_start}&date_end=${date_end}`,
      );
      const data = await res.json();

      if (!res.ok) {
        showNotification(data.detail || "Error al buscar habitaciones.", false);
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

  // ==============================================
  // Agregar y Quitar habitación
  // ==============================================
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
    showNotification(`Habitación ${hab.nombre} agregada.`, true);
  }

  function quitarHab(id) {
    const roomName =
      selectedRooms.find((x) => x.id === id)?.nombre || "Habitación";
    selectedRooms = selectedRooms.filter((x) => x.id !== id);
    showNotification(`${roomName} quitada.`, false);
  }

  // ==============================================
  // Crear reserva ADMIN
  // ==============================================
  async function confirmarReserva() {
    hideNotification();
    isSubmitting = true;

    if (!id_usuario || isNaN(Number(id_usuario))) {
      showNotification("Debe digitar un ID de usuario válido.", false);
      isSubmitting = false;
      return;
    }

    if (!date_start || !date_end) {
      showNotification("Las fechas son obligatorias.", false);
      isSubmitting = false;
      return;
    }

    if (new Date(date_end) <= new Date(date_start)) {
      showNotification("Fecha fin debe ser posterior a fecha inicio.", false);
      isSubmitting = false;
      return;
    }

    if (selectedRooms.length === 0) {
      showNotification("Debe seleccionar al menos una habitación.", false);
      isSubmitting = false;
      return;
    }

    const habitacionesIds = selectedRooms.map((s) => Number(s.id));

    const payload = {
      id_usuario: Number(id_usuario),
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
        `¡Reserva #${data.id_reserva || "Nueva"} creada exitosamente!`,
        true,
      );
      // Limpiar y recargar
      id_usuario = "";
      date_start = "";
      date_end = "";
      selectedRooms = [];
      availRooms = [];
      await cargarReservas();
    } catch (e) {
      console.error(e);
      showNotification(
        "No hay conexión con el servidor al intentar crear la reserva.",
        false,
      );
    } finally {
      isSubmitting = false;
    }
  }

  // ==============================================
  // Modal de Cancelación (ADMIN)
  // ==============================================
  function showCancelConfirmation(id) {
    hideNotification();

    const reserva = reservas.find((r) => r.id_reserva === id);
    if (!reserva) {
      showNotification("Reserva no encontrada.", false);
      return;
    }

    if (reserva.estado !== 1) {
      showNotification("La reserva ya está cancelada.", false);
      return;
    }

    // Almacena la reserva y abre el modal personalizado
    reservationToCancel = reserva;
    isConfirmingCancel = true;
  }

  async function executeCancellation() {
    if (!reservationToCancel) return;

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
      reservationToCancel = null;
    }
  }

  function cancelConfirmation() {
    isConfirmingCancel = false;
    reservationToCancel = null;
    showNotification("Cancelación detenida.", false);
  }
</script>

<svelte:head>
  <link
    rel="stylesheet"
    href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css"
    xintegrity="sha512-SnH5WK+bZxgPHs44uWIX+LLMD/CD+5N+yEStRjS1K48xSgH44n/B+xJ1A40k8c7oE9a9u5pG2Gq0wF04iP5pG2Gq0wF04iP5w=="
    crossorigin="anonymous"
    referrerpolicy="no-referrer"
  />
</svelte:head>

<Navbar />
<NavbarA />

<section class="main-container">
  <div class="content-wrapper">
    <h1 class="page-title">Gestión de Reservas (ADMIN)</h1>

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
        class:active={activeTab === "listado"}
        on:click={() => (activeTab = "listado")}
      >
        <i class="fas fa-list mr-2"></i> Todas las Reservas
      </button>
    </div>

    {#if activeTab === "crear"}
      <div class="tab-content">
        <div class="card card-two-columns">
          <h2 class="card-title">Crear Nueva Reserva para Cliente</h2>

          <div class="two-column-layout">
            <div class="column-box column-search">
              <h3 class="box-subtitle">1. Ingresar Datos y Buscar</h3>

              <div class="form-grid-admin">
                <div class="input-group">
                  <label for="id-usuario">ID Usuario (Obligatorio)</label>
                  <input
                    id="id-usuario"
                    type="number"
                    bind:value={id_usuario}
                    placeholder="Ej: 10"
                  />
                </div>

                <div class="input-group">
                  <label for="date-start">Fecha de inicio</label>
                  <input id="date-start" type="date" bind:value={date_start} />
                </div>

                <div class="input-group">
                  <label for="date-end">Fecha de fin</label>
                  <input id="date-end" type="date" bind:value={date_end} />
                </div>

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

              <div class="room-list available-rooms mt-6">
                <h4 class="list-title">
                  Habitaciones disponibles ({availRooms.length})
                </h4>
                {#if availRooms.length === 0}
                  <p class="empty-message-list">
                    Ingrese fechas y haga clic en Buscar.
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

            <div class="column-box column-summary">
              <h3 class="box-subtitle">2. Resumen y Confirmación</h3>

              <div class="summary-details p-4 bg-gray-800/50 rounded-lg mb-6">
                <p class="summary-item">
                  <span class="font-bold text-lg text-white">ID Cliente:</span>
                  <span class="text-accent font-extrabold text-xl"
                    >{id_usuario || "PENDIENTE"}</span
                  >
                </p>
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
              </div>

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

              <div class="confirm-action">
                <button
                  class="btn-primary btn-confirm"
                  on:click={confirmarReserva}
                  disabled={selectedRooms.length === 0 ||
                    !id_usuario ||
                    isSubmitting}
                >
                  {#if isSubmitting}
                    <i class="fas fa-spinner fa-spin mr-2"></i> Creando Reserva...
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

    {#if activeTab === "listado"}
      <div class="tab-content">
        <div class="card">
          <h2 class="card-title">Listado Global de Reservas</h2>

          {#if loadingListado}
            <div class="loading-message">
              <i class="fas fa-sync fa-spin mr-2"></i> Cargando listado de reservas...
            </div>
          {:else if reservas.length === 0}
            <p class="empty-message">
              No hay reservas registradas en el sistema.
            </p>
          {:else}
            <div class="table-container">
              <table>
                <thead>
                  <tr>
                    <th>ID Reserva</th>
                    <th>ID Usuario</th>
                    <th>Inicio</th>
                    <th>Fin</th>
                    <th>Estado</th>
                    <th>Acción</th>
                  </tr>
                </thead>
                <tbody>
                  {#each reservas as r (r.id_reserva)}
                    <tr
                      class:active-row={r.estado === 1}
                      class:cancelled-row={r.estado !== 1}
                    >
                      <td class="highlight">{r.id_reserva}</td>
                      <td class="text-primary font-bold">{r.id_usuario}</td>
                      <td>{r.date_start.substring(0, 10)}</td>
                      <td>{r.date_end.substring(0, 10)}</td>
                      <td>
                        {#if r.estado === 1}
                          <span class="status status-active">Activa</span>
                        {:else}
                          <span class="status status-cancelled">Cancelada</span>
                        {/if}
                      </td>
                      <td>
                        {#if r.estado === 1}
                          <button
                            class="btn-cancel-admin"
                            on:click={() =>
                              showCancelConfirmation(r.id_reserva)}
                            disabled={isLoading}
                          >
                            Cancelar
                          </button>
                        {:else}
                          <span class="text-danger font-semibold"
                            >Finalizada</span
                          >
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
    {/if}
  </div>
</section>

<Footer />

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
      <h3 id="modal-title" class="modal-title">Confirmar Cancelación ADMIN</h3>

      <p class="modal-body">
        ¿Estás seguro de cancelar la reserva
        <span class="font-bold text-accent">
          #{reservationToCancel.id_reserva}
        </span>
        del usuario ID
        <span class="font-bold text-primary">
          {reservationToCancel.id_usuario}
        </span>
        ?
        <br />
        Esta cancelación se hará con privilegios de administrador.
      </p>

      <div class="modal-actions">
        <button
          class="btn-danger"
          on:click={executeCancellation}
          disabled={isLoading}
        >
          Sí, Cancelar Reserva
        </button>

        <button
          class="btn-cancel-modal"
          on:click={cancelConfirmation}
          disabled={isLoading}
        >
          No, Mantener
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
    --color-bg-card: #1a202c; /* FONDO DE TARJETA PRINCIPAL: Ligeramente más claro que el fondo para destacar */
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
    background: var(--color-bg-card);
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
    color: var(--color-accent);
    margin-bottom: 30px;
    font-weight: 800;
  }

  /*
  ==============================================
  ESTILOS DEL SCROLLBAR
  ==============================================
  */
  .table-container::-webkit-scrollbar,
  .room-list .list::-webkit-scrollbar {
    width: 8px;
    height: 8px;
  }

  .table-container::-webkit-scrollbar-track,
  .room-list .list::-webkit-scrollbar-track {
    background: var(--scrollbar-track);
    border-radius: 10px;
  }

  .table-container::-webkit-scrollbar-thumb,
  .room-list .list::-webkit-scrollbar-thumb {
    background: var(--scrollbar-thumb);
    border-radius: 10px;
  }

  .table-container::-webkit-scrollbar-thumb:hover,
  .room-list .list::-webkit-scrollbar-thumb:hover {
    background: var(--scrollbar-thumb-hover);
  }

  /*
  ==============================================
  ESTILOS DE PESTAÑA
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
  .card {
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

  .box-subtitle {
    color: var(--color-accent);
    font-size: 1.5rem;
    font-weight: 600;
    margin-bottom: 20px;
    padding-bottom: 8px;
  }

  .two-column-layout {
    display: grid;
    grid-template-columns: 1fr;
    gap: 30px;
  }

  .column-box {
    background: var(--color-bg-box);
    padding: 25px;
    border-radius: 12px;
    box-shadow:
      0 4px 6px rgba(0, 0, 0, 0.4),
      inset 0 0 10px rgba(0, 0, 0, 0.2);
    border: 1px solid var(--color-border);
  }

  /*
  ==============================================
  Formularios (Búsqueda) - Admin
  ==============================================
  */

  input[type="date"],
  input[type="number"] {
    padding: 12px 14px;
    border-radius: 8px;
    border: 1px solid #3b424c;
    background: #161b22;
    color: var(--color-text-light);
    transition:
      border-color 0.3s,
      box-shadow 0.3s;
    font-size: 1rem;
    width: 100%;
    -webkit-appearance: none;
  }

  input[type="date"]:focus,
  input[type="number"]:focus {
    outline: none;
    border-color: var(--color-accent);
    box-shadow: 0 0 0 3px rgba(252, 211, 77, 0.3);
  }

  /* GRID DE ADMIN: 4 COLUMNAS (ID USUARIO | INICIO | FIN | BOTÓN) */
  .form-grid-admin {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 20px;
    align-items: flex-end;
  }

  /*
  ==============================================
  Botones
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

  .btn-cancel-admin {
    /* Estilo para el botón Cancelar en la tabla */
    background: var(--color-danger);
    color: white;
  }
  .btn-cancel-admin:hover {
    background: #b91c1c;
  }
  .btn-cancel-modal {
    /* Estilo para el botón No, Mantener en el modal */
    background: #4b5563;
    color: white;
  }

  button:disabled {
    background: #374151;
    color: #9ca3af;
    cursor: not-allowed;
    opacity: 0.8;
    transform: none;
    box-shadow: none;
  }

  .btn-group-search {
    grid-column: 4 / 5;
    display: flex;
    flex-direction: column;
    justify-content: flex-end;
    margin-top: 10px;
  }

  .btn-search {
    width: 100%;
    font-size: 1rem;
    padding-top: 14px;
    padding-bottom: 14px;
  }

  .btn-confirm {
    width: 100%;
    margin-top: 20px;
    font-size: 1.1rem;
    padding: 12px 20px;
  }

  /*
  ==============================================
  Listas y Selección
  ==============================================
  */
  .mt-6 {
    margin-top: 1.5rem;
  }
  .mb-6 {
    margin-bottom: 1.5rem;
  }
  .p-4 {
    padding: 1rem;
  }
  .rounded-lg {
    border-radius: 0.5rem;
  }
  .font-bold {
    font-weight: 700;
  }
  .font-semibold {
    font-weight: 600;
  }
  .font-extrabold {
    font-weight: 800;
  }
  .text-lg {
    font-size: 1.125rem;
  }
  .text-xl {
    font-size: 1.25rem;
  }
  .text-white {
    color: white;
  }
  .text-primary {
    color: var(--color-primary);
  }
  .text-accent {
    color: var(--color-accent);
  }
  .text-danger {
    color: var(--color-danger);
  }
  .bg-gray-800\/50 {
    background-color: rgba(31, 41, 55, 0.5);
  }

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

  .list-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background: var(--color-bg-card);
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
    border-bottom: 1px dashed #4b5563;
  }
  .summary-item:last-child {
    border-bottom: none;
  }

  .empty-message-list {
    text-align: center;
    padding: 20px 10px;
    color: #94a3b8;
    font-style: italic;
  }

  .empty-message {
    text-align: center;
    padding: 40px 10px;
    color: #94a3b8;
    font-size: 1.1rem;
    font-style: italic;
  }

  .loading-message {
    text-align: center;
    padding: 40px 10px;
    color: var(--color-highlight);
    font-size: 1.1rem;
    font-weight: 600;
  }

  /*
  ==============================================
  Tabla de Reservas Activas (Listado Global)
  ==============================================
  */

  .table-container {
    max-height: 500px;
    overflow-y: auto;
    overflow-x: auto;
    border-radius: 8px;
    box-shadow: inset 0 0 8px rgba(0, 0, 0, 0.5);
    padding: 1px;
    border: 1px solid var(--color-border);
  }

  table {
    width: 100%;
    min-width: 850px;
    border-collapse: separate;
    border-spacing: 0;
    background: var(--color-table-bg);
    overflow: hidden;
  }

  thead tr {
    background-color: var(--color-primary);
    color: white;
    overflow: hidden;
  }

  th {
    padding: 14px 20px;
    text-align: left;
    font-size: 0.9rem;
    font-weight: 700;
    text-transform: uppercase;
    position: sticky;
    top: 0;
    z-index: 10;
  }

  td {
    padding: 12px 20px;
    border-bottom: 1px solid #374151;
    color: var(--color-text-light);
    font-size: 0.95rem;
  }

  tr:hover {
    background-color: rgba(255, 255, 255, 0.05);
  }

  .active-row {
    /* Estilo sutil para filas activas */
    background-color: rgba(16, 185, 129, 0.05);
  }
  .cancelled-row {
    /* Estilo sutil para filas canceladas o cerradas */
    color: #9ca3af;
  }

  .highlight {
    color: var(--color-accent);
    font-weight: 700;
  }

  .status {
    padding: 4px 10px;
    border-radius: 4px;
    font-weight: 600;
    font-size: 0.8rem;
    display: inline-block;
  }

  .status-active {
    background-color: rgba(16, 185, 129, 0.2);
    color: var(--color-success);
  }

  .status-cancelled {
    background-color: rgba(239, 68, 68, 0.2);
    color: var(--color-danger);
  }

  /*
  ==============================================
  Modal de Notificación y Confirmación
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
    background-color: var(--color-danger);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
    display: flex;
    align-items: center;
    gap: 15px;
    font-weight: 600;
  }

  .message-content.success {
    background-color: var(--color-success);
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

  /* Modal de Confirmación */
  .custom-modal-backdrop {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.7);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 999;
    /* Animación de fade-in */
    opacity: 0;
    animation: fadeInModal 0.3s forwards;
  }

  .custom-modal-content {
    background: var(--color-bg-box);
    padding: 30px;
    border-radius: 12px;
    max-width: 450px;
    width: 90%;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.8);
    color: var(--color-text-light);
    /* Animación de zoom-in */
    transform: scale(0.9);
    animation: zoomIn 0.3s 0.1s forwards;
    opacity: 0;
  }

  .modal-title {
    color: var(--color-accent);
    font-size: 1.5rem;
    margin-bottom: 15px;
    border-bottom: 2px solid var(--color-accent);
    padding-bottom: 10px;
  }

  .modal-body {
    margin-bottom: 25px;
    line-height: 1.6;
  }

  .modal-actions {
    display: flex;
    justify-content: space-between;
    gap: 15px;
  }

  .modal-actions button {
    flex-grow: 1;
  }

  @keyframes fadeInModal {
    to {
      opacity: 1;
    }
  }

  @keyframes zoomIn {
    to {
      transform: scale(1);
      opacity: 1;
    }
  }

  /*
  ==============================================
  Responsividad
  ==============================================
  */

  @media (min-width: 768px) {
    /* Layout de dos columnas en escritorio */
    .two-column-layout {
      grid-template-columns: 1fr 1fr;
    }

    /* Layout de 4 columnas en escritorio para el grid admin */
    .form-grid-admin {
      grid-template-columns: 1.2fr 1fr 1fr 0.8fr; /* ID | Start | End | Button */
    }
  }

  @media (max-width: 767px) {
    .page-title {
      font-size: 2rem;
    }

    .card {
      padding: 20px 15px;
    }

    .form-grid-admin {
      /* Se convierte en una columna apilada en móvil */
      grid-template-columns: 1fr;
      gap: 15px;
    }

    .btn-group-search {
      grid-column: 1 / -1; /* Ocupa todo el ancho */
      margin-top: 0;
    }

    /* Adaptar tabla para móvil */
    .table-container {
      max-height: 400px;
    }

    table,
    thead,
    tbody,
    th,
    td,
    tr {
      display: block;
    }

    thead {
      display: none;
    }

    tr {
      border: 1px solid var(--color-border);
      margin-bottom: 10px;
      padding: 10px;
      border-radius: 8px;
      background: var(--color-bg-card);
    }

    td {
      border: none;
      position: relative;
      padding-left: 50%;
      text-align: right;
      display: block;
    }

    td::before {
      content: attr(data-label);
      position: absolute;
      left: 10px;
      width: 45%;
      padding-right: 10px;
      white-space: nowrap;
      text-align: left;
      font-weight: 700;
      color: var(--color-highlight);
    }

    td:nth-child(1)::before {
      content: "ID:";
    }
    td:nth-child(2)::before {
      content: "Usuario:";
    }
    td:nth-child(3)::before {
      content: "Inicio:";
    }
    td:nth-child(4)::before {
      content: "Fin:";
    }
    td:nth-child(5)::before {
      content: "Estado:";
    }
    td:nth-child(6)::before {
      content: "Acción:";
    }
    td:nth-child(6) {
      text-align: center;
    } /* Botón centrado */

    .list-item {
      flex-direction: column;
      align-items: flex-start;
    }

    .list-item button {
      align-self: flex-end;
    }
  }
</style>
