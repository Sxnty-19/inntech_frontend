<script>
  import { onMount } from "svelte";
  // NOTA: Asumimos que estos componentes existen y se importan correctamente.
  // @ts-ignore
  import Navbar from "$lib/components/Navbar.svelte";
  // @ts-ignore
  import NavbarA from "$lib/components/NavbarA.svelte";
  // @ts-ignore
  import Footer from "$lib/components/Footer.svelte";

  // ================================
  // GESTIÓN DE NOTIFICACIONES PERSONALIZADAS (MODAL/ALERT)
  // ================================
  let showMessage = false;
  let message = "";
  let isSuccess = false;

  /**
   * Muestra el mensaje personalizado (Modal/Alert)
   * @param {string} text - Contenido del mensaje.
   * @param {boolean} success - Si es verdadero (éxito) o falso (error/info).
   */
  function showCustomMessage(text, success) {
    message = text;
    isSuccess = success;
    showMessage = true;

    // El mensaje desaparece después de 4 segundos
    setTimeout(() => {
      showMessage = false;
    }, 4000);
  }

  // ================================
  // ESTADOS DE DATOS
  // ================================
  /** @type {Array<{id_thabitacion: number, nombre: string, descripcion: string, capacidad_max: number, estado: number}>} */
  let tipos = [];
  /** @type {Array<{id_habitacion: number, id_thabitacion: number, numero: string, estado: number}>} */
  let habitaciones = [];
  let loadingTipos = true;
  let loadingHabitaciones = true;
  let errorTipos = null;
  let errorHabitaciones = null;
  let isSubmittingTipo = false;
  let isSubmittingHabitacion = false;

  // Controla la vista activa ('tipos' o 'habitaciones')
  let activeView = "tipos";

  // ================================
  // FORMULARIOS
  // ================================
  let nuevoTipo = {
    nombre: "",
    descripcion: "",
    capacidad_max: "",
    estado: 1,
  };

  let nuevaHabitacion = {
    id_thabitacion: "",
    numero: "",
    estado: 1,
  };

  // ---------------------------------------------------
  // Utilidad de Fetch
  // ---------------------------------------------------
  /**
   * Realiza una petición fetch segura y maneja errores y mensajes de forma consistente.
   * @param {string} url - La URL del endpoint.
   * @param {RequestInit} options - Opciones de la petición fetch.
   * @returns {Promise<any>} Los datos de la respuesta JSON.
   */
  async function safeFetch(url, options = {}) {
    try {
      const response = await fetch(url, options);
      if (!response.ok) {
        const errorData = await response.json().catch(() => ({}));
        // Muestra error de la API con tu diseño, forzando isSuccess=false
        showCustomMessage(
          errorData.detail || `Error HTTP: ${response.status} en la acción.`,
          false,
        );
        throw new Error(
          errorData.detail || `Error HTTP! status: ${response.status}`,
        );
      }
      return response.json();
    } catch (error) {
      console.error("Error en la petición:", error);
      // Muestra error general de conexión/servidor si el error no fue un 4xx/5xx
      if (!error.message.startsWith("Error HTTP:")) {
        showCustomMessage(
          "Error de conexión o servidor al intentar realizar la acción.",
          false,
        );
      }
      throw error;
    }
  }

  // ================================
  // CARGAR TIPOS DE HABITACION
  // =================================
  async function cargarTipos() {
    loadingTipos = true;
    errorTipos = null;
    try {
      const data = await safeFetch(
        "https://inntech-backend.onrender.com/tipos_habitacion/get_tipos_habitacion",
      );
      tipos = data.data || [];
    } catch (err) {
      errorTipos = "No se pudieron cargar los tipos de habitación.";
    } finally {
      loadingTipos = false;
    }
  }

  // ================================
  // CARGAR HABITACIONES
  // ================================
  async function cargarHabitaciones() {
    loadingHabitaciones = true;
    errorHabitaciones = null;
    try {
      const data = await safeFetch(
        "https://inntech-backend.onrender.com/habitaciones/get_habitaciones",
      );
      habitaciones = data.data || [];
    } catch (err) {
      errorHabitaciones = "No se pudieron cargar las habitaciones.";
    } finally {
      loadingHabitaciones = false;
    }
  }

  // ================================
  // CREAR TIPO HABITACION
  // ================================
  async function guardarTipo() {
    if (!nuevoTipo.nombre || !nuevoTipo.capacidad_max) {
      // Usa tu mensaje, forzando isSuccess=false para que use el color de error
      showCustomMessage(
        "Por favor, complete el nombre y la capacidad máxima.",
        false,
      );
      return;
    }

    isSubmittingTipo = true;
    try {
      await safeFetch(
        "https://inntech-backend.onrender.com/tipos_habitacion/create_tipo_habitacion",
        {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(nuevoTipo),
        },
      );

      // 1. Mensaje de éxito (isSuccess=true)
      showCustomMessage(
        `Tipo "${nuevoTipo.nombre}" creado exitosamente.`,
        true,
      );

      // 2. Resetear y recargar
      nuevoTipo = { nombre: "", descripcion: "", capacidad_max: "", estado: 1 };
      await cargarTipos();
    } catch (err) {
      // El mensaje de error ya se muestra en safeFetch
    } finally {
      isSubmittingTipo = false;
    }
  }

  // ================================
  // CREAR HABITACION
  // ================================
  async function guardarHabitacion() {
    if (!nuevaHabitacion.id_thabitacion || !nuevaHabitacion.numero) {
      // Usa tu mensaje, forzando isSuccess=false
      showCustomMessage(
        "Por favor, seleccione el tipo y el número de habitación.",
        false,
      );
      return;
    }

    isSubmittingHabitacion = true;
    try {
      await safeFetch(
        "https://inntech-backend.onrender.com/habitaciones/create_habitacion",
        {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(nuevaHabitacion),
        },
      );

      // 1. Mensaje de éxito (isSuccess=true)
      showCustomMessage(
        `Habitación ${nuevaHabitacion.numero} registrada.`,
        true,
      );

      // 2. Resetear y recargar
      nuevaHabitacion = { id_thabitacion: "", numero: "", estado: 1 };
      await cargarHabitaciones();
    } catch (err) {
      // El mensaje de error ya se muestra en safeFetch
    } finally {
      isSubmittingHabitacion = false;
    }
  }

  // ================================
  // CAMBIAR ESTADO HABITACION
  // ================================
  /**
   *
   * @param {{id_habitacion: number, id_thabitacion: number, numero: string, estado: number}} h
   */
  async function toggleEstado(h) {
    let nuevo = h.estado === 1 ? 0 : 1;
    let nuevoEstadoTexto = nuevo === 1 ? "LIMPIA" : "SUCIA";

    const formData = new FormData();
    formData.append("numero", h.numero);
    formData.append("estado", nuevo.toString());

    try {
      await safeFetch(
        "https://inntech-backend.onrender.com/habitaciones/actualizar_estado",
        {
          method: "PUT",
          body: formData,
        },
      );

      // 1. Mensaje de éxito al cambiar estado (isSuccess=true)
      showCustomMessage(
        `Habitación ${h.numero} marcada como ${nuevoEstadoTexto}.`,
        true,
      );

      // 2. Recargar
      await cargarHabitaciones();
    } catch (err) {
      // El mensaje de error ya se muestra en safeFetch
    }
  }

  // Ejecutar carga inicial al montar
  onMount(() => {
    cargarTipos();
    cargarHabitaciones();
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
    <h1 class="page-title">Gestión de Habitaciones</h1>

    <!-- ---------------------------------- -->
    <!-- NAVEGACIÓN TIPO PESTAÑA -->
    <!-- ---------------------------------- -->
    <div class="category-tabs">
      <button
        class="tab-btn"
        class:active={activeView === "tipos"}
        on:click={() => (activeView = "tipos")}
      >
        <i class="fas fa-bed"></i>
        Tipos de Habitación
      </button>

      <button
        class="tab-btn"
        class:active={activeView === "habitaciones"}
        on:click={() => (activeView = "habitaciones")}
      >
        <i class="fas fa-door-open"></i>
        Habitaciones Registradas
      </button>
    </div>

    <!-- ---------------------------------- -->
    <!-- VISTA 1: TIPOS DE HABITACIÓN -->
    <!-- ---------------------------------- -->
    {#if activeView === "tipos"}
      <div class="card card-tipos">
        <h2 class="subtitulo">
          <i class="fas fa-tags"></i> Crear y Administrar Tipos
        </h2>

        <!-- Formulario de Creación de Tipo -->
        <div class="form-container">
          <h3 class="form-title">Nuevo Tipo de Habitación</h3>
          <div class="form-grid-tipo">
            <input
              placeholder="Nombre (Ej: Doble, Suite)"
              bind:value={nuevoTipo.nombre}
              class="input-field"
            />
            <input
              placeholder="Capacidad Máxima (Ej: 4)"
              type="number"
              min="1"
              bind:value={nuevoTipo.capacidad_max}
              class="input-field"
            />
            <textarea
              placeholder="Descripción detallada del tipo de habitación..."
              bind:value={nuevoTipo.descripcion}
              class="input-field textarea-full"
            ></textarea>
          </div>

          <button
            class="btn-crear"
            on:click={guardarTipo}
            disabled={isSubmittingTipo}
          >
            <i class="fas fa-save"></i>
            {isSubmittingTipo ? "Guardando Tipo..." : "Guardar Tipo"}
          </button>
        </div>

        <!-- Tabla de Tipos de Documento -->
        <h3 class="section-title mt-40">Tipos Existentes ({tipos.length})</h3>

        {#if loadingTipos}
          <div class="loading-container-bar">
            <div class="loading-bar"></div>
          </div>
        {:else if errorTipos}
          <p class="error-text">
            <i class="fas fa-exclamation-triangle icon-error"></i>
            {errorTipos}
          </p>
        {:else if tipos.length === 0}
          <p class="subtext">
            <i class="fas fa-info-circle icon-info"></i>
            No hay tipos de habitación registrados. Cree uno para empezar.
          </p>
        {:else}
          <!-- Contenedor con overflow-x para hacer la tabla scrollable si es necesario, pero manteniendo la responsividad en móvil -->
          <div class="table-wrap table-short">
            <table class="tabla tabla-tipos">
              <thead>
                <tr>
                  <th class="col-id">ID</th>
                  <th>Nombre</th>
                  <th class="col-capacidad">Capacidad</th>
                  <th class="col-descripcion">Descripción</th>
                  <th>Estado</th>
                </tr>
              </thead>

              <tbody>
                {#each tipos as t}
                  <tr>
                    <td class="col-id" data-label="ID">{t.id_thabitacion}</td>
                    <td class="resaltado" data-label="Nombre">{t.nombre}</td>
                    <td data-label="Capacidad">{t.capacidad_max}</td>
                    <td class="descripcion-celda" data-label="Descripción">
                      {t.descripcion || "N/A"}
                    </td>
                    <td data-label="Estado">
                      {#if t.estado === 1}
                        <span class="estado estado-activa">Activo</span>
                      {:else}
                        <span class="estado estado-cerrada">Inactivo</span>
                      {/if}
                    </td>
                  </tr>
                {/each}
              </tbody>
            </table>
          </div>
        {/if}
      </div>
    {/if}

    <!-- ---------------------------------- -->
    <!-- VISTA 2: HABITACIONES REGISTRADAS -->
    <!-- ---------------------------------- -->
    {#if activeView === "habitaciones"}
      <div class="card card-habitaciones">
        <h2 class="subtitulo">
          <i class="fas fa-hotel"></i> Crear y Administrar Habitaciones
        </h2>

        <!-- Formulario de Creación de Habitación -->
        <div class="form-container">
          <h3 class="form-title">Nueva Habitación</h3>
          <div class="form-grid-habitacion">
            <select
              bind:value={nuevaHabitacion.id_thabitacion}
              class="input-field"
            >
              <option value="">Seleccione tipo de habitación</option>
              {#each tipos as t}
                <option value={t.id_thabitacion}>{t.nombre}</option>
              {/each}
            </select>

            <input
              placeholder="Número de Habitación (Ej: 101)"
              bind:value={nuevaHabitacion.numero}
              class="input-field"
              type="text"
            />
          </div>

          <button
            class="btn-crear"
            on:click={guardarHabitacion}
            disabled={isSubmittingHabitacion || tipos.length === 0}
          >
            <i class="fas fa-save"></i>
            {isSubmittingHabitacion
              ? "Guardando Habitación..."
              : "Guardar Habitación"}
          </button>
          {#if tipos.length === 0}
            <p class="error-text mt-10">
              <i class="fas fa-exclamation-circle"></i> Debe crear un Tipo de Habitación
              primero.
            </p>
          {/if}
        </div>

        <!-- Tabla de Habitaciones Registradas -->
        <h3 class="section-title mt-40">
          Habitaciones Registradas ({habitaciones.length})
        </h3>

        {#if loadingHabitaciones}
          <div class="loading-container-bar">
            <div class="loading-bar"></div>
          </div>
        {:else if errorHabitaciones}
          <p class="error-text">
            <i class="fas fa-exclamation-triangle icon-error"></i>
            {errorHabitaciones}
          </p>
        {:else if habitaciones.length === 0}
          <p class="subtext">
            <i class="fas fa-info-circle icon-info"></i>
            No hay habitaciones registradas.
          </p>
        {:else}
          <div class="table-wrap">
            <table class="tabla tabla-documentos">
              <thead>
                <tr>
                  <th class="col-id">ID</th>
                  <th>Número</th>
                  <th>Tipo</th>
                  <th>Estado Actual</th>
                  <th>Acción</th>
                </tr>
              </thead>

              <tbody>
                {#each habitaciones as h}
                  <tr class:active-row={h.estado === 1}>
                    <td class="col-id" data-label="ID">{h.id_habitacion}</td>
                    <td class="resaltado" data-label="Número">{h.numero}</td>
                    <td data-label="Tipo">
                      <!-- Mostrar nombre del tipo -->
                      {#each tipos as t}
                        {#if t.id_thabitacion === h.id_thabitacion}
                          {t.nombre}
                        {/if}
                      {/each}
                    </td>

                    <td data-label="Estado Actual">
                      {#if h.estado === 1}
                        <span class="estado estado-limpia">
                          <i class="fas fa-check-circle"></i> Limpia
                        </span>
                      {:else}
                        <span class="estado estado-sucia">
                          <i class="fas fa-broom"></i> Sucia
                        </span>
                      {/if}
                    </td>

                    <td data-label="Acción">
                      <button
                        class="btn-toggle"
                        class:btn-toggle-dirty={h.estado === 1}
                        class:btn-toggle-clean={h.estado === 0}
                        on:click={() => toggleEstado(h)}
                      >
                        {#if h.estado === 1}
                          <i class="fas fa-soap"></i> Marcar Sucia
                        {:else}
                          <span class="ok">Habitación Lista</span>
                        {/if}
                      </button>
                    </td>
                  </tr>
                {/each}
              </tbody>
            </table>
          </div>
        {/if}
      </div>
    {/if}
  </div>
</section>

<Footer />

<!-- Notificación Personalizada (Reemplazo de alert) -->
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
  /* ---------------------------------- */
  /* Paleta de Colores (Dark Mode - Moderno) */
  /* ---------------------------------- */
  :root {
    --color-bg-primary: #0f172a; /* Fondo principal muy oscuro (slate-900) */
    --color-card-bg: #1e293b; /* Fondo de tarjeta/componente (slate-800) */
    --color-table-bg: #1a202c; /* Fondo de tabla muy oscuro (gray-900) */
    --color-accent-gold: #fcd34d; /* Amarillo principal (Dorado/Naranja) */
    --color-accent-blue: #60a5fa; /* Azul para highlights (blue-400) */
    --color-text-light: #e2e8f0; /* Texto claro (slate-200) */
    --color-text-muted: #94a3b8; /* Texto sutil (slate-400) */
    --color-border: #334155; /* Borde sutil (slate-700) */
    --color-primary: #1e3a8a; /* Azul (para encabezado tabla) */
    --color-success: #10b981; /* Verde (Activa/Limpia) */
    --color-danger: #ef4444; /* Rojo (Cerrada/Sucia/Error) */
    --color-error: #ef4444; /* Rojo para mensajes de error */
    --color-info: #3b82f6; /* Azul para información/Advertencia */
    --shadow-card: 0 10px 25px rgba(0, 0, 0, 0.4);
    --shadow-btn: 0 5px 15px rgba(252, 211, 77, 0.3);

    /* COLORES ESPECÍFICOS DE ESTADO */
    --color-clean: var(--color-success);
    --color-dirty: var(--color-danger);
    --color-toggle-dirty-bg: #fbbf24; /* Amarillo oscuro para marcar sucia */
    --color-toggle-clean-bg: #3b82f6; /* Azul para marcar limpia */
    --color-dirty: #ef4444; /* Rojo para Sucia (red-500) */

    /* Scrollbar */
    --scrollbar-track: #2d3748;
    --scrollbar-thumb: #4a5568;
    --scrollbar-thumb-hover: #6b7280;
  }

  /* ---------------------------------- */
  /* ESTILOS DEL MENSAJE PERSONALIZADO (REEMPLAZO DE ALERT) */
  /* ---------------------------------- */
  .message-modal {
    position: fixed;
    top: 20px;
    right: 20px;
    z-index: 1000;
    animation: slideIn 0.5s forwards;
    pointer-events: none; /* Asegura que no bloquee clics */
  }

  .message-content {
    padding: 15px 25px;
    border-radius: 8px;
    color: white;
    /* Usamos --color-info (azul) como default para mensajes que no son éxito o error */
    background-color: var(--color-info);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
    display: flex;
    align-items: center;
    gap: 15px;
    font-weight: 600;
    animation: bounce 0.5s ease-out; /* Animación de rebote al aparecer */
    pointer-events: auto;
  }

  .message-content.success {
    background-color: var(--color-success);
  }

  .message-content i {
    font-size: 1.2rem;
  }

  /* Animación de entrada */
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

  /* Animación de rebote sutil */
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
    max-width: 800px;
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
    min-width: 250px;
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

  .tab-btn.active i {
    color: #111;
  }

  .tab-btn:not(.active) i {
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
    width: 100%;
  }

  .card-tipos {
    max-width: 900px;
  }
  .card-habitaciones {
    max-width: 1100px;
  }
  .card-form {
    max-width: 900px;
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

  .section-title {
    color: var(--color-accent-gold);
    font-size: 1.5rem;
    font-weight: 600;
    margin-bottom: 15px;
    padding-bottom: 5px;
    border-bottom: 1px dashed var(--color-border);
  }

  .mt-40 {
    margin-top: 40px;
  }
  .mt-10 {
    margin-top: 10px;
  }

  /* ---------------------------------- */
  /* Formulario (Contenedor y Campos) */
  /* ---------------------------------- */

  .form-container {
    background: var(--color-card-bg);
    padding: 20px;
    border-radius: 10px;
    border: 1px solid #2a3547;
    margin-bottom: 20px;
  }

  .form-title {
    color: var(--color-accent-gold);
    font-size: 1.2rem;
    margin-top: 0;
    margin-bottom: 15px;
    padding-bottom: 5px;
    border-bottom: 1px solid var(--color-border);
  }

  .form-grid-tipo {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 15px;
  }

  .form-grid-habitacion {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 15px;
    max-width: 600px; /* Limita ancho para este formulario más simple */
    margin: 0 auto;
  }

  .textarea-full {
    grid-column: span 2;
    min-height: 100px;
  }

  @media (max-width: 640px) {
    .form-grid-tipo,
    .form-grid-habitacion {
      grid-template-columns: 1fr;
    }
    .textarea-full {
      grid-column: span 1;
    }
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

  .input-field option {
    background: var(--color-table-bg); /* Estilo para el dropdown */
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
    margin-top: 20px;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    width: 100%;
    max-width: 400px; /* Limita el ancho del botón */
    margin-left: auto;
    margin-right: auto;
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
  /* Tabla (General) - Estilos para Responsividad */
  /* ---------------------------------- */
  .table-wrap {
    /* El overflow-x es la clave para permitir el scroll horizontal
       solo cuando la tabla es más ancha que su contenedor (en desktop o tablet).
       En móvil, el media query transformará el layout y este scroll ya no será necesario.
    */
    overflow-x: auto;
    max-height: 70vh; /* Aumento la altura máxima para que las tablas largas puedan scroll verticalmente */
    overflow-y: auto;
    border: 1px solid #334155;
    border-radius: 8px;
    margin: 0 auto;
    /* Asegura que el contenedor de la tabla no se desborde del padre */
    width: 100%;
  }

  /* Scrollbar - Estilos heredados */
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

  .tabla {
    /* min-width garantiza que la tabla no se comprima más allá de un punto */
    width: 100%;
    min-width: 650px;
    border-collapse: collapse;
    background: var(--color-table-bg);
  }

  .tabla-documentos {
    min-width: 800px;
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

  /* Celdas con texto potencialmente largo (DESKTOP/TABLET) */
  .descripcion-celda {
    /* Limita el ancho de la celda de descripción para que no ocupe demasiado espacio */
    max-width: 300px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    color: var(--color-text-muted);
  }

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

  .col-id {
    max-width: 50px;
    text-align: center;
  }

  /* Estilos de Estado */
  .estado {
    padding: 6px 12px;
    border-radius: 20px;
    font-weight: 700;
    font-size: 0.75rem;
    text-transform: uppercase;
    display: inline-block;
  }

  .estado-activa,
  .estado-limpia {
    background: rgba(16, 185, 129, 0.2);
    color: var(--color-clean);
  }

  .estado-cerrada,
  .estado-sucia {
    /* Fondo rojo claro, texto rojo oscuro */
    background: rgba(239, 68, 68, 0.05);
    color: var(--color-dirty);
  }

  /* Botón Toggle de Estado */
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
  }

  .btn-toggle-dirty {
    background: var(--color-toggle-dirty-bg);
    color: #111;
  }

  .btn-toggle-dirty:hover {
    background: #eab308;
  }

  .btn-toggle-clean {
    background: var(--color-primary);
    color: white;
  }

  .btn-toggle-clean:hover {
    background: #2563eb;
  }

  /* ---------------------------------- */
  /* Mensajes y Errores (Loading/Empty) */
  /* ---------------------------------- */
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

  /* Animación de Carga (Barra) - Heredado */
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

  /* ---------------------------------------------------- */
  /* Responsividad de Tablas (Móvil) - REGLAS CLAVE */
  /* ---------------------------------------------------- */
  @media (max-width: 768px) {
    .main {
      padding: 30px 10px;
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
      min-width: unset;
      flex: 1 1 45%;
    }

    /* Ocultar columnas menos importantes en móvil */
    .col-id {
      display: none;
    }

    /* En móvil, permitimos que el texto largo se envuelva en lugar de truncarse */
    .descripcion-celda {
      white-space: normal;
      overflow: visible;
      text-overflow: clip;
      max-width: none;
    }

    /* -- Transformación de Tabla (Filas a Bloques) -- */

    .tabla {
      min-width: 100%; /* La tabla ocupa todo el ancho disponible ahora */
    }

    .tabla,
    .tabla tbody,
    .tabla tr,
    .tabla td {
      display: block;
      width: 100%;
    }

    .tabla thead {
      display: none; /* Oculta los encabezados de columna */
    }

    .tabla tr {
      margin-bottom: 15px;
      border: 1px solid var(--color-border);
      border-radius: 8px;
      overflow: hidden;
      background: var(--color-table-bg);
    }

    .tabla td {
      border: none;
      border-bottom: 1px solid var(--color-border);
      text-align: right;
      padding-left: 50%; /* Espacio para el data-label */
      position: relative;
    }

    .tabla td:last-child {
      border-bottom: 0;
    }

    /* Regla clave: mostrar la etiqueta del encabezado usando data-label */
    .tabla td::before {
      content: attr(data-label);
      position: absolute;
      left: 10px;
      width: calc(50% - 20px);
      padding-right: 10px;
      white-space: nowrap;
      font-weight: 700;
      color: var(--color-accent-gold);
      text-align: left;
    }

    /* Ajuste para el botón de acción en móvil */
    .card-habitaciones .tabla td:last-child {
      /* Permite que el botón se centre o tenga mejor espaciado */
      text-align: center;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
    }
    .card-habitaciones .tabla td:last-child::before {
      /* Mueve la etiqueta de acción arriba del botón */
      position: relative;
      left: 0;
      width: 100%;
      text-align: center;
      margin-bottom: 10px;
      padding: 0;
    }
    .btn-toggle {
      width: 100%;
      max-width: 200px; /* Limita el ancho del botón de acción */
    }
  }
</style>
