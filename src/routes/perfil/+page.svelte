<script>
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";
  import { onMount } from "svelte";
  // Importar la transición 'fly' de Svelte para la animación
  import { fly } from "svelte/transition";

  // Controla la vista activa: 'personales', 'registro', 'documentos'
  let currentView = "personales";

  let user = null;

  // Datos usuario
  let primer_nombre = "";
  let segundo_nombre = "";
  let primer_apellido = "";
  let segundo_apellido = "";
  let telefono = "";

  // Documento
  let tiposDoc = [];
  let id_tdocumento = "";
  let numero_documento = "";
  let lugar_expedicion = "";
  let estado_doc = 1;

  // Documentos existentes
  let documentos = [];

  // Variables para la nueva Notificación Flotante (LISTO)
  let notificationText = "";
  let notificationType = ""; // Puede ser 'success' o 'error'

  onMount(async () => {
    const stored = localStorage.getItem("user");
    if (!stored) return;
    user = JSON.parse(stored);

    // Cargar datos usuario al form
    primer_nombre = user.primer_nombre;
    segundo_nombre = user.segundo_nombre;
    primer_apellido = user.primer_apellido;
    segundo_apellido = user.segundo_apellido;
    telefono = user.telefono;

    await cargarTiposDoc();
    await cargarDocumentos();
  });

  /**
   * Muestra la notificación flotante y la oculta después de 3 segundos.
   * @param {string} text - El mensaje a mostrar.
   * @param {string} type - 'success' o 'error'.
   */
  function showNotification(text, type) {
    notificationText = text;
    notificationType = type;
    setTimeout(() => {
      notificationText = "";
      notificationType = "";
    }, 3000);
  }

  async function cargarTiposDoc() {
    try {
      const res = await fetch(
        "https://inntech-backend.onrender.com/tipos_documento/get_tipos_documento",
      );
      const data = await res.json();
      if (res.ok) tiposDoc = data.data;
    } catch (e) {
      console.error(e);
    }
  }

  async function cargarDocumentos() {
    try {
      const res = await fetch(
        "https://inntech-backend.onrender.com/documentos/get_documentos_completo",
      );
      const data = await res.json();
      if (res.ok) {
        // Asegúrate de que 'user' no sea null antes de filtrar
        if (user) {
          documentos = data.data.filter(
            (d) => d.id_usuario === user.id_usuario,
          );
        } else {
          documentos = [];
        }
      }
    } catch (e) {
      console.error(e);
    }
  }

  // ============================
  // Actualizar usuario
  // ============================
  async function actualizarUsuario() {
    try {
      const payload = {
        primer_nombre,
        segundo_nombre,
        primer_apellido,
        segundo_apellido,
        telefono,
      };

      const res = await fetch(
        `https://inntech-backend.onrender.com/usuarios/update_usuario/${user.id_usuario}`,
        {
          method: "PUT",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(payload),
        },
      );

      const data = await res.json();
      if (!res.ok) {
        // Muestra el error
        showNotification(data.detail || "Error al actualizar usuario", "error");
        return;
      }

      // Mostrar notificación de éxito
      showNotification("Datos actualizados correctamente", "success");

      // actualizar el localstorage
      const updatedUser = { ...user, ...payload };
      localStorage.setItem("user", JSON.stringify(updatedUser));
      user = updatedUser;
    } catch (e) {
      // Muestra el error de conexión
      showNotification("Error de conexión", "error");
    }
  }

  // ============================
  // Crear documento
  // ============================
  async function crearDocumento() {
    if (!id_tdocumento || !numero_documento || !lugar_expedicion) {
      // Muestra el error de validación
      showNotification(
        "Todos los campos del documento son obligatorios",
        "error",
      );
      return;
    }

    try {
      const payload = {
        id_tdocumento: Number(id_tdocumento),
        id_usuario: user.id_usuario,
        numero_documento,
        lugar_expedicion,
        estado: estado_doc,
      };

      const res = await fetch(
        "https://inntech-backend.onrender.com/documentos/create_documento",
        {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(payload),
        },
      );

      const data = await res.json();

      if (!res.ok) {
        // Muestra el error del servidor
        showNotification(
          data.detail || "No se pudo crear el documento",
          "error",
        );
        return;
      }

      // Mostrar notificación de éxito
      showNotification("Documento creado con éxito", "success");

      // Limpiar campos después del éxito
      numero_documento = "";
      lugar_expedicion = "";
      id_tdocumento = "";

      await cargarDocumentos();
    } catch (e) {
      // Muestra el error de conexión
      showNotification("Error de conexión", "error");
    }
  }
</script>

<Navbar />
<NavbarA />

<section class="main-container">
  <div class="main-content">
    <h2 class="title">Administrar Perfil</h2>

    <!-- INICIO DE LA NAVEGACIÓN POR PESTAÑAS -->
    <div class="category-tabs">
      <button
        class="tab-btn"
        class:active={currentView === "personales"}
        on:click={() => (currentView = "personales")}
      >
        <i class="fas fa-user-circle"></i> Datos Personales
      </button>
      <button
        class="tab-btn"
        class:active={currentView === "registro"}
        on:click={() => (currentView = "registro")}
      >
        <i class="fas fa-file-alt"></i> Registrar Nuevo Documento
      </button>
      <button
        class="tab-btn"
        class:active={currentView === "documentos"}
        on:click={() => (currentView = "documentos")}
      >
        <i class="fas fa-list-check"></i> Mis Documentos Registrados
      </button>
    </div>
    <!-- FIN DE LA NAVEGACIÓN POR PESTAÑAS -->

    <!-- INICIO DE CONTENIDO CONDICIONAL -->
    <div class="content-display">
      <!-- 1. VISTA: DATOS PERSONALES -->
      {#if currentView === "personales"}
        <div class="box user-box" transition:fly={{ y: 20, duration: 300 }}>
          <h3 class="box-title">Actualizar Datos Personales</h3>

          <div class="form-group">
            <label for="pNombre">Primer Nombre</label>
            <input id="pNombre" bind:value={primer_nombre} type="text" />
          </div>

          <div class="form-group">
            <label for="sNombre">Segundo Nombre</label>
            <input id="sNombre" bind:value={segundo_nombre} type="text" />
          </div>

          <div class="form-group">
            <label for="pApellido">Primer Apellido</label>
            <input id="pApellido" bind:value={primer_apellido} type="text" />
          </div>

          <div class="form-group">
            <label for="sApellido">Segundo Apellido</label>
            <input id="sApellido" bind:value={segundo_apellido} type="text" />
          </div>

          <div class="form-group">
            <label for="telefono">Teléfono</label>
            <input id="telefono" bind:value={telefono} type="text" />
          </div>

          <button class="btn btn-primary" on:click={actualizarUsuario}>
            <i class="fas fa-save"></i> Actualizar Datos
          </button>
        </div>
      {/if}

      <!-- 2. VISTA: REGISTRAR DOCUMENTO -->
      {#if currentView === "registro"}
        <div
          class="box document-form-box"
          transition:fly={{ y: 20, duration: 300 }}
        >
          <h3 class="box-title">Registrar Nuevo Documento</h3>

          <div class="form-group">
            <label for="tipoDoc">Tipo Documento</label>
            <select id="tipoDoc" bind:value={id_tdocumento}>
              <option value="">Seleccione...</option>
              {#each tiposDoc as t}
                <option value={t.id_tdocumento}>{t.nombre}</option>
              {/each}
            </select>
          </div>

          <div class="form-group">
            <label for="numDoc">Número de Documento</label>
            <input id="numDoc" bind:value={numero_documento} type="text" />
          </div>

          <div class="form-group">
            <label for="lugarExp">Lugar de expedición</label>
            <input id="lugarExp" bind:value={lugar_expedicion} type="text" />
          </div>

          <div class="form-group">
            <label for="estadoDoc">Estado (Activo por defecto)</label>
            <select id="estadoDoc" bind:value={estado_doc}>
              <option value={1}>Activo</option>
              <option value={0}>Inactivo</option>
            </select>
          </div>

          <button class="btn btn-secondary" on:click={crearDocumento}>
            <i class="fas fa-file-alt"></i> Guardar Documento
          </button>
        </div>
      {/if}

      <!-- 3. VISTA: DOCUMENTOS REGISTRADOS (TABLA) -->
      {#if currentView === "documentos"}
        <div
          class="box documents-table-box"
          transition:fly={{ y: 20, duration: 300 }}
        >
          <h3 class="box-title">Mis Documentos Registrados</h3>

          {#if documentos.length === 0}
            <p class="no-data">
              No hay documentos registrados para tu usuario.
            </p>
          {:else}
            <div class="table-responsive">
              <table>
                <thead>
                  <tr>
                    <th>ID</th>
                    <th>Tipo</th>
                    <th>Número</th>
                    <th>Lugar Expedición</th>
                    <th>Estado</th>
                  </tr>
                </thead>

                <tbody>
                  {#each documentos as d}
                    <tr>
                      <td class="highlight">{d.id_documento}</td>
                      <td>{d.tipo_documento}</td>
                      <td>{d.numero_documento}</td>
                      <td>{d.lugar_expedicion}</td>
                      <td>
                        <span
                          class="status-badge status-{d.estado === 1
                            ? 'active'
                            : 'inactive'}"
                        >
                          {d.estado === 1 ? "Activo" : "Inactivo"}
                        </span>
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
    <!-- FIN DE CONTENIDO CONDICIONAL -->
  </div>
</section>

<Footer />

<!-- Notificación Flotante -->
{#if notificationText}
  <div
    class="floating-notification {notificationType}"
    transition:fly={{ y: 50, duration: 300 }}
  >
    <i
      class="fas"
      class:fa-check-circle={notificationType === "success"}
      class:fa-times-circle={notificationType === "error"}
    ></i>
    {notificationText}
  </div>
{/if}

<style>
  /* ============================================== */
  /* Variables y Resets Globales */
  /* ============================================== */
  :root {
    --color-dark: #0f172a; /* Fondo principal de Nav/Footer */
    --color-body-bg: #1a202c; /* Fondo del contenido (ligeramente más claro) */
    --color-primary: #1e3a8a; /* Azul principal */
    --color-secondary: #fcd34d; /* Amarillo/Naranja de acento */
    --color-text: #e2e8f0; /* Texto claro */
    --color-success: #15803d; /* Verde oscuro para éxito */
    --color-success-light: #dcfce7; /* Fondo claro para éxito (Nuevo) */
    --color-danger: #b91c1c; /* Rojo oscuro para error */
    --color-danger-light: #fee2e2; /* Fondo claro para error (Nuevo) */
    --color-neutral: #94a3b8; /* Gris neutro */
  }

  :global(html) {
    background: var(--color-dark);
    box-sizing: border-box;
    width: 100%;
    overflow-x: hidden;
  }

  :global(body) {
    margin: 0;
    padding: 0;
    min-height: 100vh;
    width: 100%;
    font-family: "Inter", sans-serif;
    overflow-x: hidden;
  }

  /* ---------------------------------- */
  /* ESTRUCTURA PRINCIPAL */
  /* ---------------------------------- */
  .main-container {
    min-height: calc(100vh - 110px); /* Ajustado para Navbars y Footer */
    background: var(--color-body-bg);
    padding: 40px 20px;
  }

  .main-content {
    max-width: 1200px;
    margin: 0 auto;
    color: var(--color-text);
  }

  .title {
    text-align: center;
    font-size: clamp(2rem, 4vw, 2.8rem);
    color: var(--color-secondary);
    margin-bottom: 30px;
    font-weight: 800;
  }

  /* ---------------------------------- */
  /* LAYOUT Y PESTAÑAS (TABS) - NUEVO */
  /* ---------------------------------- */
  .category-tabs {
    display: flex;
    justify-content: center;
    gap: 10px;
    margin-bottom: 40px; /* Margen antes del contenido */
    flex-wrap: wrap;
  }

  .tab-btn {
    padding: 12px 20px;
    border: 2px solid var(--color-dark);
    border-radius: 8px;
    background-color: #111827; /* Fondo de botón no activo */
    color: var(--color-neutral);
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 1rem;
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

  .content-display {
    /* Mantiene el contenido centrado y responsive */
    max-width: 900px;
    margin: 0 auto;
    /* Evita el cambio de layout cuando se pasa de formulario a tabla */
    min-height: 400px;
  }

  /* ---------------------------------- */
  /* CAJAS (BOXES) */
  /* ---------------------------------- */
  /* Ahora las cajas tienen un ancho máximo en el content-display */
  .box {
    background: var(
      --color-dark
    ); /* Fondo de la caja más oscuro que el fondo del cuerpo */
    padding: 30px;
    border-radius: 12px;
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.5);
    height: fit-content;
    margin-bottom: 30px;
  }

  .box-title {
    font-size: 1.5rem;
    color: var(--color-secondary);
    margin-bottom: 25px;
    border-bottom: 2px solid var(--color-primary);
    padding-bottom: 10px;
    font-weight: 700;
  }

  /* ---------------------------------- */
  /* FORMULARIO */
  /* ---------------------------------- */
  .form-group {
    margin-bottom: 20px;
    display: flex;
    flex-direction: column;
  }

  label {
    margin-bottom: 8px;
    font-size: 0.9rem;
    color: var(--color-text);
    font-weight: 500;
  }

  input,
  select {
    padding: 12px;
    border-radius: 8px;
    border: 1px solid var(--color-primary);
    background: #111827; /* Gris muy oscuro */
    color: white;
    font-size: 1rem;
    transition:
      border-color 0.3s,
      box-shadow 0.3s;
  }

  input:focus,
  select:focus {
    outline: none;
    border-color: var(--color-secondary);
    box-shadow: 0 0 0 3px rgba(252, 211, 77, 0.3);
  }

  /* ---------------------------------- */
  /* BOTONES */
  /* ---------------------------------- */
  .btn {
    width: 100%; /* El botón se ajusta al ancho del contenedor .box */
    border: none;
    padding: 12px 15px;
    border-radius: 8px;
    cursor: pointer;
    margin-top: 15px;
    font-weight: bold;
    font-size: 1rem;
    transition:
      background 0.3s,
      transform 0.2s,
      box-shadow 0.3s;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
  }

  .btn-primary {
    background: var(--color-primary);
    color: white;
  }

  .btn-primary:hover {
    background: #102a65;
    transform: translateY(-1px);
    box-shadow: 0 4px 10px rgba(30, 58, 138, 0.4);
  }

  .btn-secondary {
    background: var(--color-secondary);
    color: var(--color-dark);
  }

  .btn-secondary:hover {
    background: #fbbd24;
    transform: translateY(-1px);
    box-shadow: 0 4px 10px rgba(252, 211, 77, 0.4);
  }

  /* ---------------------------------- */
  /* TABLA DE DOCUMENTOS */
  /* ---------------------------------- */
  .table-responsive {
    overflow-x: auto;
  }

  table {
    width: 100%;
    border-collapse: separate;
    border-spacing: 0;
    margin-top: 15px;
    background: #111827;
    border-radius: 8px;
    overflow: hidden;
  }

  th,
  td {
    padding: 12px 15px;
    text-align: left;
    border-bottom: 1px solid #333;
  }

  th {
    background: var(--color-primary);
    color: white;
    font-weight: 600;
    text-transform: uppercase;
    font-size: 0.8rem;
  }

  tbody tr:nth-child(even) {
    background: #1f2937;
  }

  tbody tr:hover {
    background: #253347;
  }

  .highlight {
    color: var(--color-secondary);
    font-weight: 600;
  }

  /* Status Badges */
  .status-badge {
    padding: 4px 8px;
    border-radius: 4px;
    font-size: 0.8rem;
    font-weight: 600;
  }

  .status-active {
    background: var(--color-success);
    color: white;
  }

  .status-inactive {
    background: var(--color-danger);
    color: white;
  }

  .no-data {
    text-align: center;
    padding: 15px;
    font-style: italic;
    color: var(--color-neutral);
  }

  /* ---------------------------------- */
  /* NOTIFICACIÓN FLOTANTE */
  /* ---------------------------------- */
  .floating-notification {
    position: fixed;
    top: 20px;
    right: 20px;
    padding: 15px 20px;
    border-radius: 8px;
    font-weight: 600;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.4);
    max-width: 300px;
    z-index: 1000;
    display: flex;
    align-items: center;
    gap: 10px;
    color: var(--color-dark);
  }

  .floating-notification.success {
    background: var(--color-success);
    color: var(--color-success-light);
    border: 1px solid var(--color-success);
  }

  .floating-notification.error {
    background: var(--color-danger);
    color: var(--color-danger-light);
    border: 1px solid var(--color-danger);
  }

  /* ---------------------------------- */
  /* RESPONSIVIDAD */
  /* ---------------------------------- */
  @media (max-width: 768px) {
    .main-container {
      padding: 20px 10px;
    }

    .category-tabs {
      flex-direction: column;
      align-items: stretch;
    }

    .tab-btn {
      width: 100%;
      font-size: 0.9rem;
    }

    .box {
      padding: 20px;
    }

    .box-title {
      font-size: 1.3rem;
    }

    /* Ajuste para notificaciones en móvil */
    .floating-notification {
      top: 10px;
      left: 10px;
      right: 10px;
      max-width: none;
      text-align: center;
      justify-content: center;
    }
  }
</style>
