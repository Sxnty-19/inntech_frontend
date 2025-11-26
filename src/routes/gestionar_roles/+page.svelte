<script>
  import { onMount } from "svelte";
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";

  // Datos
  let roles = [];
  let modulos = []; // módulos disponibles (combo)
  let assignedModules = []; // módulos asignados al rol abierto en modal

  // Estados UI
  let cargandoRoles = false;
  let cargandoModulos = false;
  let cargandoAsignados = false;
  let creandoRol = false;
  let asignando = false;
  let errorMsg = "";

  // Form nuevo rol
  let nuevoRol = {
    nombre: "",
    descripcion: "",
    estado: 1,
  };

  // Modal crear rol
  let showCreateModal = false;
  // Modal asignar módulos
  let showAssignModal = false;
  let currentRole = null; // rol seleccionado para asignar módulos
  let selectedModuloToAssign = null;
  let assigningBusy = false;

  // ----------------------------------
  // ESTADOS PARA MENSAJES FLOTANTES Y CONFIRMACIÓN (REEMPLAZANDO alert/confirm)
  // ----------------------------------
  // Notificación flotante
  let message = "";
  let isSuccess = false;
  let showMessage = false;
  let isModalActive = false;

  // Modal de confirmación
  let showConfirmModal = false;
  let confirmMessage = "";
  let confirmAction = () => {};
  // ----------------------------------

  // -------------------------
  // RUTAS (ajusta si hace falta)
  // -------------------------
  const BASE = "https://inntech-backend.onrender.com";
  const R_GET_ROLES = `${BASE}/roles/get_roles`;
  const R_CREATE_ROL = `${BASE}/roles/create_rol`;
  const R_ACTIVAR_ROL = `${BASE}/roles/activar`; // PUT /roles/activar/{id}
  const R_DESACTIVAR_ROL = `${BASE}/roles/desactivar`; // PUT /roles/desactivar/{id}`

  const R_GET_MODULOS = `${BASE}/modulos/get_modulos`;
  const R_GET_MODULOS_BY_ROL = `${BASE}/modulos_roles/get_modulos_by_rol`; // /{id_rol}
  const R_CREATE_MODULOROL = `${BASE}/modulos_roles/create_modulorol`; // POST

  // ----------------------------------
  // UTILIDAD: MENSAJES FLOTANTES
  // ----------------------------------

  // Oculta el modal con animación de salida
  function hideMessageWithTransition(duration = 300) {
    isModalActive = false;
    return new Promise((resolve) =>
      setTimeout(() => {
        showMessage = false;
        resolve();
      }, duration),
    );
  }

  // Muestra mensajes de forma temporal
  function showFloatingMessage(type, text) {
    if (showMessage) {
      hideMessageWithTransition(100);
    }

    message = text;
    isSuccess = type === "success";
    showMessage = true;
    isModalActive = true;

    // Ocultar automáticamente después de 4 segundos
    setTimeout(() => {
      hideMessageWithTransition(300);
    }, 4000);
  }

  // -------------------------
  // CARGAS
  // -------------------------
  async function cargarRoles() {
    cargandoRoles = true;
    errorMsg = "";
    try {
      const res = await fetch(R_GET_ROLES);
      const data = await res.json();
      // Asegúrate de que los roles se carguen de manera consistente
      roles = data.data && Array.isArray(data.data) ? data.data : [];
    } catch (err) {
      console.error("Error cargando roles:", err);
      errorMsg = "Error cargando roles (ver consola).";
      showFloatingMessage("error", "No se pudieron cargar los roles.");
    } finally {
      cargandoRoles = false;
    }
  }

  async function cargarModulos() {
    cargandoModulos = true;
    try {
      const res = await fetch(R_GET_MODULOS);
      const data = await res.json();
      modulos = data.data && Array.isArray(data.data) ? data.data : [];
    } catch (err) {
      console.error("Error cargando módulos:", err);
      modulos = [];
      showFloatingMessage(
        "error",
        "No se pudieron cargar los módulos disponibles.",
      );
    } finally {
      cargandoModulos = false;
    }
  }

  async function cargarModulosAsignados(id_rol) {
    cargandoAsignados = true;
    assignedModules = [];
    try {
      const res = await fetch(`${R_GET_MODULOS_BY_ROL}/${id_rol}`);
      const data = await res.json();
      // El backend parece devolver una lista directa de módulos asignados o vacía
      assignedModules = data.data && Array.isArray(data.data) ? data.data : [];
    } catch (err) {
      console.error("Error cargando módulos asignados:", err);
      assignedModules = [];
      showFloatingMessage("error", "Error cargando módulos asignados al rol.");
    } finally {
      cargandoAsignados = false;
    }
  }

  // -------------------------
  // ACCIONES SOBRE ROLES
  // -------------------------

  async function crearRol() {
    if (!nuevoRol.nombre.trim()) {
      showFloatingMessage("error", "El nombre del rol es obligatorio.");
      return;
    }

    creandoRol = true;
    if (showMessage) await hideMessageWithTransition(100); // Oculta el mensaje anterior

    try {
      const res = await fetch(R_CREATE_ROL, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(nuevoRol),
      });
      const data = await res.json();

      if (!res.ok) {
        console.error("Error crear rol:", data);
        showFloatingMessage(
          "error",
          data.detail || data.message || "Error al crear rol",
        );
      } else {
        showFloatingMessage(
          "success",
          `Rol "${nuevoRol.nombre}" creado con éxito.`,
        );
        showCreateModal = false;
        limpiarNuevoRol();
        await cargarRoles();
      }
    } catch (err) {
      console.error("Error creando rol:", err);
      showFloatingMessage("error", "Error creando rol (ver consola).");
    } finally {
      creandoRol = false;
    }
  }

  function limpiarNuevoRol() {
    nuevoRol = {
      nombre: "",
      descripcion: "",
      estado: 1,
    };
  }

  // Función que prepara el modal de confirmación para cambiar el estado
  function handleToggleEstado(rol) {
    const confirmMsg =
      rol.estado === 1
        ? `¿Seguro que quieres DESACTIVAR el rol "${rol.nombre}"? Esto afectará a los usuarios asignados.`
        : `¿Seguro que quieres ACTIVAR el rol "${rol.nombre}"?`;

    confirmMessage = confirmMsg;
    confirmAction = async () => {
      showConfirmModal = false; // Cerrar antes de ejecutar la acción
      await toggleEstadoRol(rol);
    };
    showConfirmModal = true;
  }

  async function toggleEstadoRol(rol) {
    // La confirmación ya se hizo en handleToggleEstado
    const url =
      rol.estado === 1
        ? `${R_DESACTIVAR_ROL}/${rol.id_rol}`
        : `${R_ACTIVAR_ROL}/${rol.id_rol}`;

    try {
      const res = await fetch(url, { method: "PUT" }); // tus rutas usan PUT
      const data = await res.json();
      if (!res.ok) {
        console.error("Error al cambiar estado rol:", data);
        showFloatingMessage(
          "error",
          data.detail || data.message || "Error al cambiar estado",
        );
      } else {
        showFloatingMessage(
          "success",
          `Estado del rol "${rol.nombre}" cambiado a ${rol.estado === 1 ? "Inactivo" : "Activo"}.`,
        );
        // refrescar
        await cargarRoles();
      }
    } catch (err) {
      console.error("Error toggleEstadoRol:", err);
      showFloatingMessage("error", "Error al cambiar estado (ver consola).");
    }
  }

  // -------------------------
  // ASIGNACIÓN DE MÓDULOS
  // -------------------------
  function abrirModalAsignar(rol) {
    currentRole = rol;
    selectedModuloToAssign = null;
    assignedModules = [];
    showAssignModal = true;
    cargarModulosAsignados(rol.id_rol);
  }

  // Función que prepara el modal de confirmación para asignar módulo
  function handleAsignarModulo() {
    if (!currentRole) return;
    if (!selectedModuloToAssign) {
      showFloatingMessage("error", "Selecciona un módulo para asignar.");
      return;
    }

    // Buscamos el nombre del módulo para el mensaje
    const moduloNombre =
      modulos.find((m) => m.id_modulo == selectedModuloToAssign)?.nombre ||
      "Módulo Desconocido";

    confirmMessage = `¿Seguro que deseas ASIGNAR el módulo "${moduloNombre}" al rol "${currentRole.nombre}"?`;
    confirmAction = async () => {
      showConfirmModal = false; // Cerrar antes de ejecutar la acción
      await asignarModulo();
    };
    showConfirmModal = true;
  }

  async function asignarModulo() {
    if (!currentRole || !selectedModuloToAssign) return; // Ya se validó antes de abrir confirmación

    assigningBusy = true;
    const moduloNombre =
      modulos.find((m) => m.id_modulo == selectedModuloToAssign)?.nombre ||
      "Módulo Desconocido";
    if (showMessage) await hideMessageWithTransition(100);

    try {
      const payload = {
        id_modulo: +selectedModuloToAssign,
        id_rol: +currentRole.id_rol,
        estado: 1,
      };

      const res = await fetch(R_CREATE_MODULOROL, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(payload),
      });

      const data = await res.json();
      if (!res.ok) {
        console.error("Error asignando módulo:", data);
        showFloatingMessage(
          "error",
          data.detail || data.message || "Error al asignar módulo",
        );
      } else {
        showFloatingMessage(
          "success",
          `Módulo "${moduloNombre}" asignado con éxito a ${currentRole.nombre}.`,
        );
        await cargarModulosAsignados(currentRole.id_rol);
        selectedModuloToAssign = null; // Limpiar selección
      }
    } catch (err) {
      console.error("Error en asignarModulo:", err);
      showFloatingMessage("error", "Error al asignar módulo (ver consola).");
    } finally {
      assigningBusy = false;
    }
  }

  // -------------------------
  // INIT
  // -------------------------
  onMount(async () => {
    await Promise.all([cargarRoles(), cargarModulos()]);
  });
</script>

<svelte:head>
  <!-- Font Awesome para íconos -->
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
    <!-- Título Principal -->
    <h1 class="page-title">Gestión de Roles y Permisos</h1>

    <!-- Botones de Acción Global -->
    <div class="top-actions">
      <button
        class="btn-primary"
        on:click={() => (showCreateModal = !showCreateModal)}
      >
        <i class="fas fa-plus icon-left"></i>
        {showCreateModal ? "Cerrar Formulario" : "Crear Nuevo Rol"}
      </button>
      <button
        class="btn-secondary"
        on:click={() => cargarRoles()}
        disabled={cargandoRoles}
      >
        <i class="fas fa-sync-alt icon-left" class:fa-spin={cargandoRoles}></i>
        {cargandoRoles ? "Cargando..." : "Refrescar Roles"}
      </button>
    </div>

    {#if errorMsg}
      <p class="error-inline"><i class="fas fa-times-circle"></i> {errorMsg}</p>
    {/if}

    <!-- ---------------------------------- -->
    <!-- MODAL CREAR ROL -->
    <!-- ---------------------------------- -->
    {#if showCreateModal}
      <div
        class="modal-backdrop"
        on:click={() => (showCreateModal = false)}
      ></div>
      <div class="modal large-modal">
        <h3 class="modal-title">Crear Nuevo Rol</h3>

        <div class="form-grid">
          <input
            placeholder="Nombre del Rol (Ej: Gerente de Operaciones)"
            bind:value={nuevoRol.nombre}
            class="input-field"
          />
          <input
            placeholder="Descripción (Ej: Permiso total en Módulos A y B)"
            bind:value={nuevoRol.descripcion}
            class="input-field"
          />
          <div class="input-group">
            <label for="estado-select">Estado Inicial</label>
            <select
              bind:value={nuevoRol.estado}
              id="estado-select"
              class="select-field"
            >
              <option value={1}>Activo</option>
              <option value={0}>Inactivo</option>
            </select>
          </div>
        </div>

        <div class="modal-actions-footer">
          <button class="btn-save" on:click={crearRol} disabled={creandoRol}>
            <i class="fas fa-check icon-left"></i>
            {creandoRol ? "Creando..." : "Guardar Rol"}
          </button>
          <button class="btn-cancel" on:click={() => (showCreateModal = false)}
            >Cancelar</button
          >
        </div>
      </div>
    {/if}

    <!-- ---------------------------------- -->
    <!-- TABLA DE ROLES -->
    <!-- ---------------------------------- -->
    <div class="card card-table">
      <h2 class="card-subtitle">
        <i class="fas fa-users-cog"></i> Lista de Roles
      </h2>
      <div class="table-wrap">
        <table class="tabla">
          <thead>
            <tr>
              <th>ID</th>
              <th>Nombre</th>
              <th>Descripción</th>
              <th>Estado</th>
              <th class="col-actions">Acciones</th>
            </tr>
          </thead>

          <tbody>
            {#if cargandoRoles}
              <!-- Animación de carga sutil para la tabla -->
              <tr>
                <td colspan="5" class="loading-cell">
                  <div class="loading-container-bar">
                    <div class="loading-bar"></div>
                  </div>
                </td>
              </tr>
            {:else if roles.length === 0}
              <tr
                ><td colspan="5" class="empty-cell"
                  >No hay roles registrados en el sistema.</td
                ></tr
              >
            {:else}
              {#each roles as r}
                <tr class:active-row={r.estado === 1}>
                  <td>{r.id_rol}</td>
                  <td class="resaltado">{r.nombre}</td>
                  <td class="descripcion-celda">{r.descripcion}</td>
                  <td>
                    <span
                      class="badge"
                      class:badge-active={r.estado === 1}
                      class:badge-inactive={r.estado === 0}
                    >
                      {r.estado === 1 ? "Activo" : "Inactivo"}
                    </span>
                  </td>
                  <td class="col-actions">
                    <button
                      class="btn-toggle"
                      on:click={() => handleToggleEstado(r)}
                    >
                      <i
                        class="fas"
                        class:fa-power-off={r.estado === 1}
                        class:fa-check-circle={r.estado === 0}
                      ></i>
                      {r.estado === 1 ? "Desactivar" : "Activar"}
                    </button>

                    <button
                      class="btn-assign"
                      on:click={() => abrirModalAsignar(r)}
                    >
                      <i class="fas fa-key"></i> Asignar módulos
                    </button>
                  </td>
                </tr>
              {/each}
            {/if}
          </tbody>
        </table>
      </div>
    </div>

    <!-- ---------------------------------- -->
    <!-- MODAL ASIGNAR MÓDULOS -->
    <!-- ---------------------------------- -->
    {#if showAssignModal && currentRole}
      <div
        class="modal-backdrop"
        on:click={() => (showAssignModal = false)}
      ></div>

      <div class="modal giant-modal">
        <h3 class="modal-title">
          Asignar Módulos a: <span class="highlight-modal"
            >{currentRole.nombre}</span
          >
        </h3>

        <div class="assign-row">
          <div class="assign-left">
            <h4 class="section-header">Asignar Nuevo Permiso</h4>

            <label for="modulo-select" class="label-field"
              >Módulo a asignar</label
            >
            {#if cargandoModulos}
              <p class="loading-text">
                <i class="fas fa-spinner fa-spin"></i> Cargando módulos disponibles...
              </p>
            {:else if modulos.length === 0}
              <p class="error-inline">
                No hay módulos disponibles para asignar.
              </p>
            {:else}
              <select
                bind:value={selectedModuloToAssign}
                id="modulo-select"
                class="select-field"
              >
                <option value={null}>-- Selecciona un módulo --</option>
                {#each modulos as m}
                  <option value={m.id_modulo}>{m.nombre} - ({m.ruta})</option>
                {/each}
              </select>

              <div style="margin-top:15px;">
                <button
                  class="btn-save btn-block"
                  on:click={handleAsignarModulo}
                  disabled={assigningBusy || !selectedModuloToAssign}
                >
                  <i class="fas fa-link icon-left"></i>
                  {assigningBusy ? "Asignando..." : "Asignar Permiso"}
                </button>
              </div>
            {/if}
          </div>

          <div class="assign-right">
            <h4 class="section-header">Módulos Asignados al Rol</h4>

            {#if cargandoAsignados}
              <p class="loading-text">
                <i class="fas fa-spinner fa-spin"></i> Cargando asignaciones actuales...
              </p>
            {:else if assignedModules.length === 0}
              <p class="empty-cell">Este rol aún no tiene módulos asignados.</p>
            {:else}
              <div class="table-mini-wrap">
                <table class="tabla-mini">
                  <thead>
                    <tr
                      ><th class="mini-id">ID</th><th>Nombre Módulo</th><th
                        class="mini-ruta">Ruta</th
                      ></tr
                    >
                  </thead>
                  <tbody>
                    {#each assignedModules as am}
                      <tr>
                        <td class="mini-id">{am.id_modulo ?? am.id_mxr}</td>
                        <td>{am.nombre}</td>
                        <td class="mini-ruta">{am.ruta}</td>
                      </tr>
                    {/each}
                  </tbody>
                </table>
              </div>
            {/if}
          </div>
        </div>

        <div class="modal-actions-footer">
          <button
            class="btn-secondary"
            on:click={() => (showAssignModal = false)}
          >
            <i class="fas fa-times icon-left"></i>Cerrar</button
          >
        </div>
      </div>
    {/if}

    <!-- ---------------------------------- -->
    <!-- MODAL DE CONFIRMACIÓN -->
    <!-- ---------------------------------- -->
    {#if showConfirmModal}
      <div
        class="modal-backdrop"
        on:click={() => (showConfirmModal = false)}
      ></div>
      <div class="modal confirm-modal">
        <h3 class="modal-title warning-title">
          <i class="fas fa-exclamation-triangle"></i> Confirmar Acción
        </h3>
        <p class="confirm-message">{confirmMessage}</p>
        <div class="modal-actions-footer">
          <button class="btn-cancel" on:click={() => (showConfirmModal = false)}
            >Cancelar</button
          >
          <button class="btn-danger" on:click={confirmAction}>Confirmar</button>
        </div>
      </div>
    {/if}
  </div>

  <!-- ---------------------------------- -->
  <!-- NOTIFICACIÓN FLOTANTE (REPLICADO DEL DISEÑO ANTERIOR) -->
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
    --color-accent-gold: #fcd34d; /* Amarillo principal (Dorado) */
    --color-accent-blue: #60a5fa; /* Azul para highlights (blue-400) */
    --color-text-light: #e2e8f0; /* Texto claro (slate-200) */
    --color-text-muted: #94a3b8; /* Texto sutil (slate-400) */
    --color-border: #334155; /* Borde sutil (slate-700) */
    --color-primary-dark: #1e3a8a; /* Azul oscuro (para botón primario) */
    --color-success: #10b981; /* Verde (Activa) */
    --color-danger: #ef4444; /* Rojo (Desactivada) */
    --color-warning: #f59e0b; /* Naranja (Warning/Confirmación) */
    --shadow-card: 0 10px 25px rgba(0, 0, 0, 0.4);
    --shadow-btn-primary: 0 5px 15px rgba(252, 211, 77, 0.3);

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
    margin-bottom: 5px;
    font-weight: 900;
    text-transform: uppercase;
    letter-spacing: 1px;
    text-shadow: 0 0 10px rgba(252, 211, 77, 0.2);
  }

  .top-actions {
    display: flex;
    justify-content: center;
    gap: 15px;
    margin-bottom: 10px;
    flex-wrap: wrap;
    align-self: center;
    max-width: 650px;
    width: 100%;
  }

  /* ---------------------------------- */
  /* Botones de Acción Global */
  /* ---------------------------------- */

  .btn-primary,
  .btn-secondary {
    flex: 1;
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
  }

  .btn-save,
  .btn-cancel,
  .btn-danger {
    padding: 12px 20px;
    border: none;
    border-radius: 8px;
    font-weight: 700;
    cursor: pointer;
    transition: all 0.3s ease;
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 1rem;
  }

  .btn-primary {
    background: var(--color-accent-gold);
    color: #111;
    box-shadow: var(--shadow-btn-primary);
  }

  .btn-primary:hover:not(:disabled) {
    background: #ffaa2b;
    transform: translateY(-2px);
    box-shadow: 0 8px 20px rgba(252, 211, 77, 0.5);
  }

  .btn-secondary {
    background: var(--color-card-bg);
    color: var(--color-text-light);
    border: 1px solid var(--color-border);
  }

  .btn-secondary:hover:not(:disabled) {
    background: #2a3547;
    border-color: var(--color-accent-blue);
  }

  .btn-primary:disabled,
  .btn-secondary:disabled {
    opacity: 0.6;
    cursor: not-allowed;
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

  .card-table {
    overflow: hidden;
    justify-content: center;
  }

  .card-subtitle {
    color: var(--color-accent-blue);
    font-size: clamp(1.4rem, 2.5vw, 1.8rem);
    padding: 20px 30px;
    font-weight: 700;
    display: flex;
    align-items: center;
    gap: 10px;
    border-bottom: 1px solid var(--color-border);
    background: var(--color-bg-primary);
  }

  /* ---------------------------------- */
  /* Inputs y Selects */
  /* ---------------------------------- */
  .input-field,
  .select-field {
    box-sizing: border-box;
    width: 100%;
    padding: 12px 16px;
    background: var(--color-table-bg);
    color: var(--color-text-light);
    border: 1px solid var(--color-border);
    border-radius: 8px;
    transition: all 0.3s;
  }

  .input-field::placeholder {
    color: var(--color-text-muted);
  }

  .input-field:focus,
  .select-field:focus {
    border-color: var(--color-accent-gold);
    box-shadow: 0 0 0 3px rgba(252, 211, 77, 0.4);
    outline: none;
  }

  .input-group label,
  .label-field {
    display: block;
    color: var(--color-text-muted);
    font-size: 0.9rem;
    margin-bottom: 5px;
  }

  .form-grid {
    display: grid;
    gap: 20px;
    margin-top: 20px;
    grid-template-columns: repeat(3, 1fr);
  }

  /* ---------------------------------- */
  /* Tabla de Roles */
  /* ---------------------------------- */
  .table-wrap::-webkit-scrollbar {
    overflow-x: auto;
    max-height: 70vh;
    overflow-y: auto;
  }

  .table-wrap::-webkit-scrollbar-track {
    background: var(--scrollbar-track);
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
    max-height: 70vh;
    overflow-y: auto;
    border: 1px solid #334155; /* Este es el borde explícito alrededor del área de scroll */
    border-radius: 8px;
    margin: 0 auto; /* Centrar el contenedor de scroll */
    max-width: 100%; /* El ancho máximo es el de la tarjeta */
  }

  .tabla {
    width: 100%;
    min-width: 100%;
    border-collapse: collapse;
    background: var(--color-table-bg);
  }

  .tabla th {
    background: var(--color-primary-dark);
    color: var(--color-text-light);
    padding: 15px 15px;
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
    vertical-align: middle;
  }

  .tabla tr:last-child td {
    border-bottom: none;
  }

  .active-row {
    background-color: rgba(96, 165, 250, 0.05); /* Azul muy sutil */
  }

  .active-row:hover td {
    background: rgba(96, 165, 250, 0.1);
  }

  .tabla tbody tr:not(.active-row):hover td {
    background: rgba(255, 255, 255, 0.03);
  }

  .resaltado {
    color: var(--color-accent-gold);
    font-weight: 600;
  }

  .descripcion-celda {
    color: var(--color-text-muted);
  }

  /* Badges de Estado */
  .badge {
    padding: 4px 10px;
    border-radius: 4px;
    font-weight: 700;
    font-size: 0.75rem;
    text-transform: uppercase;
    display: inline-block;
  }

  .badge-active {
    background: rgba(16, 185, 129, 0.2);
    color: var(--color-success);
  }

  .badge-inactive {
    background: rgba(239, 68, 68, 0.2);
    color: var(--color-danger);
  }

  /* Botones de Acción de Fila */
  .col-actions {
    display: flex;
    gap: 8px;
    padding-top: 10px;
    padding-bottom: 10px;
  }

  .btn-toggle,
  .btn-assign {
    padding: 8px 12px;
    border: 1px solid var(--color-border);
    border-radius: 6px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s;
    font-size: 0.85rem;
    display: flex;
    align-items: center;
    gap: 6px;
  }

  .btn-toggle {
    background: var(--color-table-bg);
    color: var(--color-text-light);
  }

  .btn-toggle:hover {
    background: #2a3547;
    color: var(--color-accent-blue);
    border-color: var(--color-accent-blue);
  }

  .btn-assign {
    background: var(--color-accent-gold);
    color: #111;
  }

  .btn-assign:hover {
    background: #ffaa2b;
  }

  .loading-cell,
  .empty-cell {
    text-align: center;
    color: var(--color-text-muted);
    font-style: italic;
    padding: 20px;
  }

  .loading-cell:hover {
    background: transparent !important;
  }

  /* ---------------------------------- */
  /* MODALES (General, Crear, Asignar, Confirmar) */
  /* ---------------------------------- */

  .modal-backdrop {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.8);
    z-index: 100;
  }

  .modal {
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
    max-width: 600px;
    box-shadow: 0 15px 40px rgba(0, 0, 0, 0.9);
    animation: fadeInScale 0.3s ease-out forwards;
  }

  .large-modal {
    max-width: 800px;
  }

  .giant-modal {
    max-width: 1000px;
  }

  .confirm-modal {
    max-width: 450px;
    border-color: var(--color-warning);
    padding: 25px;
  }

  .modal-title {
    color: var(--color-accent-gold);
    font-size: 1.8rem;
    margin-bottom: 15px;
    border-bottom: 1px solid var(--color-border);
    padding-bottom: 10px;
    display: flex;
    align-items: center;
    gap: 10px;
  }

  .highlight-modal {
    color: var(--color-accent-blue);
    font-weight: 800;
  }

  .warning-title {
    color: var(--color-warning);
  }

  .confirm-message {
    color: var(--color-text-light);
    font-size: 1.1rem;
    margin-bottom: 25px;
    line-height: 1.4;
  }

  .modal-actions-footer {
    margin-top: 20px;
    padding-top: 15px;
    border-top: 1px solid var(--color-border);
    display: flex;
    justify-content: flex-end;
    gap: 10px;
  }

  .btn-save {
    background: var(--color-success);
    color: #111;
    font-weight: 800;
  }

  .btn-save:hover {
    background: #10e092;
    transform: translateY(-1px);
  }

  .btn-cancel {
    background: var(--color-accent-gold);
    color: var(--color-table-bg);
  }

  .btn-cancel:hover {
    background: var(--color-accent-gold);
  }

  .btn-danger {
    background: var(--color-danger);
    color: white;
    font-weight: 800;
  }

  .btn-danger:hover {
    background: #ff5c5c;
  }

  /* Estilos del modal de Asignación */
  .assign-row {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 30px;
    margin-top: 10px;
  }

  .assign-left,
  .assign-right {
    padding: 15px;
    background: var(--color-table-bg);
    border-radius: 8px;
    border: 1px solid var(--color-border);
  }

  .section-header {
    color: var(--color-accent-gold);
    font-size: 1.2rem;
    font-weight: 700;
    margin-bottom: 15px;
    padding-bottom: 5px;
    border-bottom: 1px solid var(--color-border);
  }

  .loading-text {
    color: var(--color-accent-blue);
    font-style: italic;
    display: flex;
    align-items: center;
    gap: 8px;
    margin-top: 10px;
  }

  .table-mini-wrap {
    max-height: 300px;
    overflow-y: auto;
    border-radius: 6px;
    border: 1px solid #444;
  }

  .tabla-mini {
    width: 100%;
    border-collapse: collapse;
    font-size: 0.9rem;
  }

  .tabla-mini th {
    background: #334155;
    color: var(--color-text-light);
    padding: 10px 12px;
    font-size: 0.8rem;
    position: sticky;
    top: 0;
  }

  .tabla-mini td {
    padding: 8px 12px;
    border-bottom: 1px solid #334155;
    color: var(--color-text-light);
  }

  .tabla-mini tbody tr:hover td {
    background: #2a3547;
  }

  .tabla-mini .mini-id {
    width: 50px;
  }

  .tabla-mini .mini-ruta {
    color: var(--color-accent-blue);
  }

  .btn-block {
    width: 100%;
    justify-content: center;
  }

  /* ---------------------------------- */
  /* Indicador de Carga (Solo Barra de Progreso) */
  /* ---------------------------------- */

  /* Celda que contiene la barra, reducimos el padding para que la barra use más espacio */
  .loading-cell {
    padding: 10px 20px;
  }

  /* Contenedor de la barra de carga - Ocupa casi todo el ancho y se centra */
  .loading-container-bar {
    width: 95%; /* Ocupa el 95% del ancho de la celda */
    margin: 0 auto; /* Centrar horizontalmente */
    height: px;
    background-color: var(--color-border);
    border-radius: 5px;
    overflow: hidden;
    position: relative;
  }

  /* La barra que se anima */
  .loading-bar {
    position: absolute;
    top: 0;
    left: 0;
    width: 30%; /* Tamaño inicial de la barra */
    height: 100%;
    /* Gradiente con el color de acento para el efecto de movimiento */
    background: linear-gradient(
      90deg,
      rgba(252, 211, 77, 0.2) 0%,
      var(--color-accent-gold) 50%,
      rgba(252, 211, 77, 0.2) 100%
    );
    border-radius: 5px;
    animation: loadingBarAnimation 1.5s infinite linear;
  }

  /* Keyframes para animar la barra de izquierda a derecha */
  @keyframes loadingBarAnimation {
    0% {
      transform: translateX(-100%);
    }
    100% {
      /* Mueve la barra 100% de su ancho más 100% del contenedor. Un 330% es un valor seguro para el ancho 30% */
      transform: translateX(333.33%);
    }
  }

  /* ---------------------------------- */
  /* MENSAJES FLOTANTES */
  /* ---------------------------------- */

  .message-modal {
    position: fixed;
    bottom: 20px;
    right: 20px;
    z-index: 1000;
    max-width: 400px;
    width: 90%;
    pointer-events: none;
    transition:
      transform 0.3s ease-out,
      opacity 0.3s ease-out;
  }

  .slide-in {
    transform: translateY(0);
    opacity: 1;
  }

  .fade-out {
    transform: translateY(20px);
    opacity: 0;
  }

  .message-content {
    padding: 15px 20px;
    border-radius: 8px;
    box-shadow: 0 5px 20px rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    gap: 15px;
    font-weight: 600;
    color: var(--color-text-light);
  }

  .message-content.success {
    background-color: var(--color-modal-success);
  }

  .message-content:not(.success) {
    background-color: var(--color-modal-error);
  }

  .message-content i {
    font-size: 1.4rem;
  }

  /* Animación para el modal general */
  @keyframes fadeInScale {
    from {
      opacity: 0;
      transform: translate(-50%, -40%) scale(0.95);
    }
    to {
      opacity: 1;
      transform: translate(-50%, -50%) scale(1);
    }
  }

  /* Media Queries para Responsividad */
  @media (max-width: 768px) {
    .page-title {
      font-size: 2rem;
    }

    .top-actions {
      flex-direction: column;
      gap: 10px;
    }

    .btn-primary,
    .btn-secondary {
      width: 100%;
    }

    .form-grid {
      grid-template-columns: 1fr;
    }

    .col-actions {
      flex-direction: column;
      align-items: flex-start;
    }

    .tabla td {
      padding: 10px 15px;
    }

    .assign-row {
      grid-template-columns: 1fr;
      gap: 20px;
    }

    .loading-cell {
      padding: 10px 10px; /* Reducimos el padding en móvil */
    }

    .loading-container-bar {
      width: 98%; /* Le damos un poco más de ancho en móvil */
    }

    .modal {
      top: 10px;
      transform: translate(-50%, 0);
      max-height: 95vh;
      overflow-y: auto;
      margin-bottom: 20px;
    }
  }
</style>
