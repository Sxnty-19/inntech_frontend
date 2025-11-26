<script>
  import { onMount } from "svelte";
  // NOTA: Asumimos que estos componentes existen y se importan correctamente.
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";

  // Tipos de documento
  let tipos = [];
  let documentos = [];
  let loadingTipos = true;
  let loadingDocumentos = true;
  let error = null;

  // Formulario para crear tipo
  let nuevoTipo = {
    nombre: "",
    descripcion: "",
  };
  let isSubmitting = false; // Para deshabilitar el botón durante la creación

  // Controla la vista activa ('tipos' o 'documentos')
  let activeView = "tipos";

  // ---------------------------------------------------
  // Mensajes Flotantes (Replicado del diseño de Notificaciones)
  // ---------------------------------------------------
  let message = "";
  let isSuccess = false;
  let showMessage = false;
  let isModalActive = false;

  function hideMessageWithTransition(duration = 300) {
    isModalActive = false;
    return new Promise((resolve) =>
      setTimeout(() => {
        showMessage = false;
        resolve();
      }, duration),
    );
  }

  function showFloatingMessage(type, text) {
    if (showMessage) {
      hideMessageWithTransition(100);
    }

    message = text;
    isSuccess = type === "success";
    showMessage = true;
    isModalActive = true;

    setTimeout(() => {
      hideMessageWithTransition(300);
    }, 4000);
  }

  // ---------------------------------------------------
  // Utilidad de Fetch con Reintento
  // ---------------------------------------------------
  async function fetchWithRetry(url, options = {}, maxRetries = 3) {
    for (let i = 0; i < maxRetries; i++) {
      try {
        const response = await fetch(url, options);
        if (!response.ok) {
          const errorData = await response.json().catch(() => ({}));
          throw new Error(
            errorData.detail || `Error HTTP! status: ${response.status}`,
          );
        }
        return response;
      } catch (error) {
        if (i < maxRetries - 1) {
          const delay = Math.pow(2, i) * 1000;
          await new Promise((resolve) => setTimeout(resolve, delay));
        } else {
          throw error;
        }
      }
    }
  }

  // ---------------------------------------------------
  // Cargar tipos de documento
  // ---------------------------------------------------
  async function cargarTipos() {
    loadingTipos = true;
    try {
      const res = await fetchWithRetry(
        "https://inntech-backend.onrender.com/tipos_documento/get_tipos_documento",
      );
      const data = await res.json();
      if (!data.success)
        throw new Error(data.detail ?? "Error al obtener tipos");
      tipos = data.data || [];
    } catch (err) {
      console.error("Error cargando tipos:", err);
      error = "No se pudieron cargar los tipos de documento.";
    } finally {
      loadingTipos = false;
    }
  }

  // ---------------------------------------------------
  // Cargar documentos completos
  // ---------------------------------------------------
  async function cargarDocumentos() {
    loadingDocumentos = true;
    try {
      const res = await fetchWithRetry(
        "https://inntech-backend.onrender.com/documentos/get_documentos_completo",
      );
      const data = await res.json();
      if (!data.success)
        throw new Error(data.detail ?? "Error al obtener documentos");
      documentos = data.data || [];
    } catch (err) {
      console.error("Error cargando documentos:", err);
      error = "No se pudieron cargar los documentos registrados.";
    } finally {
      loadingDocumentos = false;
    }
  }

  // ---------------------------------------------------
  // Crear tipo de documento
  // ---------------------------------------------------
  async function crearTipo() {
    if (!nuevoTipo.nombre || !nuevoTipo.descripcion) {
      showFloatingMessage(
        "error",
        "Por favor, complete todos los campos para el nuevo tipo.",
      );
      return;
    }

    if (isSubmitting) return;

    isSubmitting = true;

    try {
      const res = await fetchWithRetry(
        "https://inntech-backend.onrender.com/tipos_documento/create_tipo_documento",
        {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(nuevoTipo),
        },
      );

      const data = await res.json();

      if (!data.success) {
        throw new Error(
          data.detail ?? "El tipo ya existe o hubo un error en el servidor.",
        );
      }

      showFloatingMessage(
        "success",
        `Tipo '${nuevoTipo.nombre}' creado con éxito.`,
      );

      limpiarNuevo();
      // Recargar la lista y cambiar a la vista de tipos
      await cargarTipos();
    } catch (err) {
      console.error("Error al crear tipo:", err);
      showFloatingMessage(
        "error",
        err.message || "Error al crear el tipo de documento. Intente de nuevo.",
      );
    } finally {
      isSubmitting = false;
    }
  }

  function limpiarNuevo() {
    nuevoTipo = {
      nombre: "",
      descripcion: "",
    };
  }

  // Calcula cuántos documentos hay de un tipo específico (para la columna "Docs. Registrados")
  function countDocumentsByType(nombreTipo) {
    if (!documentos || documentos.length === 0) return 0;
    return documentos.filter((d) => d.tipo_documento === nombreTipo).length;
  }

  onMount(() => {
    cargarTipos();
    cargarDocumentos();
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
    <h1 class="page-title">Gestión de Documentos</h1>

    <!-- ---------------------------------- -->
    <!-- NAVEGACIÓN TIPO PESTAÑA -->
    <!-- ---------------------------------- -->
    <div class="category-tabs">
      <button
        class="tab-btn"
        class:active={activeView === "tipos"}
        on:click={() => (activeView = "tipos")}
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
          ><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"
          ></path><polyline points="14 2 14 8 20 8"></polyline><line
            x1="16"
            y1="13"
            x2="8"
            y2="13"
          ></line><line x1="16" y1="17" x2="8" y2="17"></line><polyline
            points="10 9 9 9 8 9"
          ></polyline></svg
        >
        Tipos de Documento
      </button>
      <button
        class="tab-btn"
        class:active={activeView === "documentos"}
        on:click={() => (activeView = "documentos")}
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
          ><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"
          ></path><polyline points="14 2 14 8 20 8"></polyline><line
            x1="16"
            y1="13"
            x2="8"
            y2="13"
          ></line><line x1="16" y1="17" x2="8" y2="17"></line><line
            x1="10"
            y1="9"
            x2="9"
            y2="9"
          ></line></svg
        >
        Documentos Registrados
      </button>
    </div>

    <!-- ---------------------------------- -->
    <!-- VISTA 1: GESTIÓN DE TIPOS DE DOCUMENTO -->
    <!-- ---------------------------------- -->
    {#if activeView === "tipos"}
      <div class="card card-tipos">
        <h2 class="subtitulo">Crear y Administrar Tipos de Documento</h2>

        <!-- Formulario de Creación de Tipo -->
        <div class="form-container">
          <h3 class="form-title">Nuevo Tipo</h3>
          <div class="form-grid">
            <input
              placeholder="Nombre del Tipo (Ej: Cédula de Ciudadanía)"
              bind:value={nuevoTipo.nombre}
              class="input-field"
            />
            <input
              placeholder="Descripción breve (Ej: Documento de identidad personal)"
              bind:value={nuevoTipo.descripcion}
              class="input-field"
            />
          </div>
          <button
            class="btn-crear"
            on:click={crearTipo}
            disabled={isSubmitting}
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
              ><path
                d="M19 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11l5 5v11a2 2 0 0 1-2 2z"
              ></path><polyline points="17 21 17 13 7 13 7 21"
              ></polyline><polyline points="7 3 7 8 15 8"></polyline></svg
            >
            {isSubmitting ? "Guardando..." : "Guardar Tipo"}
          </button>
        </div>

        <!-- Tabla de Tipos de Documento -->
        <h3 class="section-title mt-40">Tipos Existentes ({tipos.length})</h3>

        {#if loadingTipos}
          <div class="loading-container-bar">
            <div class="loading-bar"></div>
          </div>
        {:else if error}
          <p class="error-text">
            <i class="fas fa-exclamation-triangle icon-error"></i>
            {error}
          </p>
        {:else}
          <div class="table-wrap table-short">
            <table class="tabla">
              <thead>
                <tr>
                  <th class="col-id">ID</th>
                  <th>Nombre</th>
                  <th>Descripción</th>
                  <th>Docs. Registrados</th>
                </tr>
              </thead>
              <tbody>
                {#each tipos as t}
                  <tr>
                    <td class="col-id">{t.id_tdocumento}</td>
                    <td class="resaltado">{t.nombre}</td>
                    <td class="descripcion-celda">{t.descripcion}</td>
                    <td
                      ><span class="count-badge"
                        >{countDocumentsByType(t.nombre)}</span
                      ></td
                    >
                  </tr>
                {/each}
              </tbody>
            </table>
          </div>
        {/if}
      </div>
    {/if}

    <!-- ---------------------------------- -->
    <!-- VISTA 2: DOCUMENTOS REGISTRADOS -->
    <!-- ---------------------------------- -->
    {#if activeView === "documentos"}
      <div class="card card-documentos">
        <h2 class="subtitulo">Listado de Documentos Registrados</h2>

        {#if loadingDocumentos}
          <div class="loading-container-bar">
            <div class="loading-bar"></div>
          </div>
        {:else if error}
          <p class="error-text">
            <i class="fas fa-exclamation-triangle icon-error"></i>
            {error}
          </p>
        {:else if documentos.length === 0}
          <p class="subtext">
            <i class="fas fa-info-circle icon-info"></i>
            No hay documentos registrados en el sistema.
          </p>
        {:else}
          <div class="table-wrap">
            <table class="tabla tabla-documentos">
              <thead>
                <tr>
                  <th class="col-id">ID</th>
                  <th>Tipo</th>
                  <th>Número</th>
                  <th>Lugar Exp.</th>
                  <th>Usuario</th>
                  <th>Estado</th>
                  <th>Fecha Creado</th>
                </tr>
              </thead>
              <tbody>
                {#each documentos as d}
                  <tr class:active-row={d.estado === 1}>
                    <td class="col-id">{d.id_documento}</td>
                    <td class="resaltado">{d.tipo_documento}</td>
                    <td>{d.numero_documento}</td>
                    <td>{d.lugar_expedicion}</td>
                    <td>{d.nombre_completo}</td>
                    <td>
                      {#if d.estado === 1}
                        <span class="estado estado-activa">Activo</span>
                      {:else}
                        <span class="estado estado-cerrada">Inactivo</span>
                      {/if}
                    </td>
                    <td
                      >{new Date(d.date_created).toLocaleDateString(
                        "es-CO",
                      )}</td
                    >
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
    --color-accent-gold: #fcd34d; /* Amarillo principal (Dorado/Naranja) */
    --color-accent-blue: #60a5fa; /* Azul para highlights (blue-400) */
    --color-text-light: #e2e8f0; /* Texto claro (slate-200) */
    --color-text-muted: #94a3b8; /* Texto sutil (slate-400) */
    --color-border: #334155; /* Borde sutil (slate-700) */
    --color-primary: #1e3a8a; /* Azul (para encabezado tabla) */
    --color-success: #10b981; /* Verde (Activa) */
    --color-danger: #ef4444; /* Rojo (Cerrada/Error) */
    --shadow-card: 0 10px 25px rgba(0, 0, 0, 0.4);
    --shadow-btn: 0 5px 15px rgba(252, 211, 77, 0.3);

    /* COLORES PARA EL MODAL FLOTANTE */
    --color-modal-success: var(--color-success);
    --color-modal-error: var(--color-danger);
    /* Scrollbar */
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
    width: 100%;
  }

  .card-tipos {
    max-width: 900px;
  }

  .card-documentos {
    max-width: 1100px;
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

  /* ---------------------------------- */
  /* Formulario (Creación de Tipo) */
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

  .form-grid {
    display: grid;
    grid-template-columns: 1fr;
    gap: 15px;
  }

  @media (min-width: 640px) {
    .form-grid {
      grid-template-columns: repeat(2, 1fr);
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
  /* Tabla (General) */
  /* ---------------------------------- */
  .table-wrap {
    overflow-x: auto;
    max-height: 60vh;
    overflow-y: auto;
    border: 1px solid #334155;
    border-radius: 8px;
    margin: 0 auto;
  }

  .table-short {
    max-height: 40vh;
  }

  /* Scrollbar - Estilos del componente Notificaciones */
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
    width: 100%;
    min-width: 650px; /* Ajuste del min-width para la tabla de Tipos */
    border-collapse: collapse;
    background: var(--color-table-bg);
  }

  .tabla-documentos {
    min-width: 900px; /* Ajuste del min-width para la tabla de Documentos */
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

  .count-badge {
    background: var(--color-accent-blue);
    color: #111;
    font-weight: 700;
    padding: 4px 8px;
    border-radius: 4px;
    font-size: 0.8rem;
  }

  /* Estilos de Estado (Activo/Inactivo) */
  .estado {
    padding: 6px 12px;
    border-radius: 20px;
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
  /* Mensajes y Errores */
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
      left: 10px;
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
      min-width: unset;
      flex: 1 1 45%;
    }

    /* Ocultar columnas menos importantes en móvil */
    .col-id {
      display: none;
    }

    .tabla th,
    .tabla td {
      padding: 10px;
    }

    .tabla {
      min-width: 550px;
    }

    .tabla-documentos {
      min-width: 700px;
    }
  }
</style>
