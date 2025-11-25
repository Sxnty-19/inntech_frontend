<script>
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";
  import { onMount } from "svelte";
  // Importar la transición 'fly' de Svelte para la animación
  import { fly } from "svelte/transition";

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
    // Comprobar si 'fly' está disponible (depende del entorno de ejecución)
    // En un entorno Svelte completo, no es necesario, pero lo mantengo por si acaso.
    // console.log("Svelte transition fly imported.");

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
      // Solo mostrar un error si la carga es crítica
      // showNotification("Error al cargar tipos de documento", 'error');
    }
  }

  async function cargarDocumentos() {
    try {
      const res = await fetch(
        "https://inntech-backend.onrender.com/documentos/get_documentos_completo",
      );
      const data = await res.json();
      if (res.ok) {
        documentos = data.data.filter((d) => d.id_usuario === user.id_usuario);
      }
    } catch (e) {
      console.error(e);
      // showNotification("Error al cargar documentos", 'error');
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

    <div class="grid-layout">
      <!-- COLUMNA 1: FORMULARIO DE USUARIO -->
      <div class="box user-box">
        <h3 class="box-title">Datos Personales</h3>

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

      <!-- COLUMNA 2: REGISTRAR DOCUMENTO -->
      <div class="box document-form-box">
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
    </div>

    <!-- TABLA DOCUMENTOS (Ancho completo) -->
    <div class="box documents-table-box">
      <h3 class="box-title">Mis Documentos Registrados</h3>

      {#if documentos.length === 0}
        <p class="no-data">No hay documentos registrados para tu usuario.</p>
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
                  <td>{d.id_documento}</td>
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
  </div>
</section>

<Footer />

<!-- Notificación Flotante (Fuera del main-container para asegurar position: fixed) -->
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
    /* --color-success-bg: rgba(52, 211, 153, 0.15); // Eliminado */
    /* --color-danger-bg: rgba(239, 68, 68, 0.15); // Eliminado */
  }

  /* Nuevo: Reset global para eliminar el scroll horizontal no deseado */
  :global(html) {
    background: var(--color-dark);
    box-sizing: border-box;
    width: 100%;
    /* FIX CLAVE: Oculta el scroll horizontal en la raíz */
    overflow-x: hidden;
  }

  :global(body) {
    margin: 0;
    padding: 0;
    min-height: 100vh;
    width: 100%;
    /* FIX CLAVE: Fuente para herencia en el footer */
    font-family: "Inter", sans-serif;
    /* FIX CLAVE: Oculta el scroll horizontal en el cuerpo */
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
  /* NOTIFICACIÓN FLOTANTE (ESTILOS CORREGIDOS) */
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
    /* Añadido contraste de texto general */
    color: var(--color-dark);
  }

  /* Estilos corregidos para Éxito */
  .floating-notification.success {
    background: var(--color-success); /* Fondo muy claro */
    color: var(--color-success-light); /* Texto verde oscuro */
    border: 1px solid var(--color-success);
  }

  /* Estilos corregidos para Error */
  .floating-notification.error {
    background: var(--color-danger); /* Fondo muy claro */
    color: var(--color-danger-light); /* Texto rojo oscuro */
    border: 1px solid var(--color-danger);
  }

  /* ---------------------------------- */
  /* LAYOUT (GRID) */
  /* ---------------------------------- */
  .grid-layout {
    display: grid;
    /* Dos columnas en escritorio */
    grid-template-columns: 1fr 1fr;
    gap: 30px;
    margin-bottom: 35px;
  }

  /* ---------------------------------- */
  /* CAJAS (BOXES) */
  /* ---------------------------------- */
  .box {
    background: var(
      --color-dark
    ); /* Fondo de la caja más oscuro que el fondo del cuerpo */
    padding: 30px;
    border-radius: 12px;
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.5);
    height: fit-content; /* Se ajusta a su contenido */
  }

  .box-title {
    font-size: 1.5rem;
    color: var(--color-secondary);
    margin-bottom: 20px;
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
    width: 100%;
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

  /* Deshabilitar estado */
  .btn:disabled {
    opacity: 0.6;
    cursor: not-allowed;
    transform: none;
    box-shadow: none;
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
    border-spacing: 0; /* Elimina el espacio entre bordes */
    margin-top: 15px;
    background: #111827; /* Fondo de tabla ligeramente diferente */
    border-radius: 8px;
    overflow: hidden; /* Para que el radio se aplique al borde */
  }

  th,
  td {
    padding: 12px 15px;
    text-align: left;
    border-bottom: 1px solid #333;
  }

  th:first-child,
  td:first-child {
    padding-left: 20px;
  }
  th:last-child,
  td:last-child {
    padding-right: 20px;
  }

  th {
    background: var(--color-primary);
    color: white;
    font-weight: 600;
    text-transform: uppercase;
    font-size: 0.8rem;
    letter-spacing: 0.5px;
  }

  tbody tr {
    transition: background 0.3s;
  }

  tbody tr:nth-child(even) {
    background: #1f2937; /* Raya para mejor legibilidad */
  }

  tbody tr:hover {
    background: #253347;
  }

  /* Status Badges */
  .status-badge {
    padding: 4px 8px;
    border-radius: 4px;
    font-size: 0.8rem;
    font-weight: 600;
  }

  .status-active {
    /* Mantenemos el color fuerte en el texto del badge */
    background: var(--color-success);
    color: white;
  }

  .status-inactive {
    /* Mantenemos el color fuerte en el texto del badge */
    background: var(--color-danger);
    color: white;
  }

  .no-data {
    text-align: center;
    padding: 15px;
    font-style: italic;
    color: #9ca3af;
  }

  /* ---------------------------------- */
  /* RESPONSIVIDAD */
  /* ---------------------------------- */
  @media (max-width: 900px) {
    .grid-layout {
      /* Una sola columna en tablet y móvil */
      grid-template-columns: 1fr;
    }
  }

  @media (max-width: 600px) {
    .main-container {
      padding: 20px 10px;
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
      bottom: auto; /* Asegurar que no esté anclado al fondo */
      left: 10px;
      right: 10px;
      max-width: none;
    }
  }
</style>
