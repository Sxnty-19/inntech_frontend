<script>
  import { onMount } from "svelte";
  // Asumiendo que estos componentes están en $lib/components/
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";

  // Variables de estado del componente de Gestión de Usuarios
  let usuarios = [];
  let roles = [];
  let loading = true;
  let error = null;

  let nuevoUsuario = {
    id_rol: 3, // Rol por defecto (cliente)
    primer_nombre: "",
    segundo_nombre: "",
    primer_apellido: "",
    segundo_apellido: "",
    telefono: "",
    correo: "",
    username: "",
    password: "",
    estado: 1,
  };

  let isSubmitting = false; // Para evitar doble submit en la creación
  let activeView = "history"; // Controla la vista activa ('create' o 'history' - ver usuarios)

  // Estado para manejar peticiones pendientes de actualización (ej. cambio de rol/estado)
  // Solo se usará para deshabilitar los controles individuales.
  const pendingUpdates = new Map();

  // -------------------------------------
  // VARIABLES PARA EL MODAL FLOTANTE (COPIADO DE NOTIFICACIONES)
  // -------------------------------------
  let message = ""; // Contenido del mensaje
  let isSuccess = false; // Define si es éxito (verde) o error (rojo)
  let showMessage = false; // Controla si el modal está en el DOM
  let isModalActive = false; // Controla la animación de entrada/salida

  // -------------------------------------
  // FUNCIÓN: Oculta el modal con animación de salida
  // -------------------------------------
  function hideMessageWithTransition(duration = 300) {
    isModalActive = false; // Inicia la animación de fade-out
    return new Promise((resolve) =>
      setTimeout(() => {
        showMessage = false; // Oculta completamente después de la animación
        resolve();
      }, duration),
    );
  }

  // -------------------------------------
  // FUNCIÓN: Para mostrar mensajes de forma temporal
  // -------------------------------------
  function showFloatingMessage(type, text) {
    if (showMessage) {
      hideMessageWithTransition(100); // Rápida transición si ya había uno
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

  // -------------------------------------
  // CARGAR ROLES DESDE LA BD
  // -------------------------------------
  async function cargarRoles() {
    try {
      const res = await fetch(
        "https://inntech-backend.onrender.com/roles/combo",
      ); // ← GET
      if (!res.ok) throw new Error("Fallo al obtener roles");
      const data = await res.json();
      roles = data.data || [];
    } catch (err) {
      console.error("Error cargando roles:", err);
      showFloatingMessage(
        "error",
        "No se pudieron cargar los roles del sistema.",
      );
    }
  }

  // -------------------------------------
  // CARGAR USUARIOS
  // -------------------------------------
  async function cargarUsuarios() {
    loading = true;
    error = null;
    try {
      const res = await fetch(
        "https://inntech-backend.onrender.com/usuarios/get_usuarios",
      ); // ← GET
      if (!res.ok) throw new Error("Fallo al obtener usuarios");
      const data = await res.json();

      // Ordenar por id_usuario de forma descendente (más reciente primero)
      usuarios = data.data
        ? data.data.sort((a, b) => b.id_usuario - a.id_usuario)
        : [];
    } catch (err) {
      console.error("Error cargando usuarios:", err);
      error = "No se pudieron cargar los usuarios.";
      showFloatingMessage("error", "Fallo al cargar la lista de usuarios.");
    } finally {
      loading = false;
    }
  }

  // -------------------------------------
  // ACTIVAR / DESACTIVAR USUARIO
  // -------------------------------------
  async function toggleEstado(usuario) {
    // Oculta el mensaje si estaba visible antes de la nueva acción
    if (showMessage) await hideMessageWithTransition(100);

    // Marcar como pendiente (para deshabilitar el botón/select)
    pendingUpdates.set(usuario.id_usuario, true);

    try {
      const url =
        usuario.estado === 1
          ? `https://inntech-backend.onrender.com/usuarios/desactivar/${usuario.id_usuario}` // ← PUT
          : `https://inntech-backend.onrender.com/usuarios/activar/${usuario.id_usuario}`; // ← PUT

      const res = await fetch(url, { method: "PUT" });

      if (!res.ok) throw new Error("Error al cambiar el estado");

      // Refrescar datos desde el servidor
      await cargarUsuarios();

      const action = usuario.estado === 1 ? "desactivado" : "activado";
      showFloatingMessage(
        "success",
        `Usuario ${usuario.username} ha sido ${action} con éxito.`,
      );
    } catch (err) {
      console.error("Error al cambiar estado:", err);
      showFloatingMessage("error", "Fallo al cambiar el estado del usuario.");
    } finally {
      // Desmarcar como pendiente
      pendingUpdates.delete(usuario.id_usuario);
    }
  }

  // -------------------------------------
  // CAMBIAR ROL DE USUARIO
  // -------------------------------------
  async function cambiarRol(usuario, event) {
    const nuevoRolStr = event.target.value;
    // Aseguramos que el ID de Rol siempre sea un número entero
    const nuevoRolId = parseInt(nuevoRolStr);

    // Oculta el mensaje si estaba visible antes de la nueva acción
    if (showMessage) await hideMessageWithTransition(100);

    // Si el rol es el mismo o el ID no es válido, salir
    if (usuario.id_rol === nuevoRolId || isNaN(nuevoRolId)) return;

    try {
      const res = await fetch(
        `https://inntech-backend.onrender.com/usuarios/cambiar-rol/${usuario.id_usuario}/${nuevoRolId}`,
        { method: "PUT" },
      );

      if (!res.ok) throw new Error("Error al cambiar el rol");

      // Refrescar datos desde el servidor para actualizar la UI
      await cargarUsuarios();
      showFloatingMessage(
        "success",
        `Rol de ${usuario.username} actualizado correctamente.`,
      );
    } catch (err) {
      console.error("Error al cambiar rol:", err);
      showFloatingMessage("error", "Fallo al cambiar el rol del usuario.");
    } finally {
      // Desmarcar como pendiente
      pendingUpdates.delete(usuario.id_usuario);
    }
  }

  // -------------------------------------
  // CREAR NUEVO USUARIO
  // -------------------------------------
  async function crearUsuario() {
    if (
      !nuevoUsuario.primer_nombre ||
      !nuevoUsuario.correo ||
      !nuevoUsuario.username ||
      !nuevoUsuario.password ||
      !nuevoUsuario.primer_apellido
    ) {
      showFloatingMessage(
        "error",
        "Por favor, rellena los campos obligatorios (*).",
      );
      return;
    }

    if (isSubmitting) return;

    isSubmitting = true;
    // Oculta el mensaje si estaba visible antes de la nueva acción
    if (showMessage) await hideMessageWithTransition(100);

    try {
      const res = await fetch(
        "https://inntech-backend.onrender.com/auth/register",
        {
          // ← POST
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(nuevoUsuario),
        },
      );

      if (!res.ok) {
        const errorData = await res.json();
        throw new Error(
          errorData.detail || `Error ${res.status} al registrar.`,
        );
      }

      limpiarNuevo();
      await cargarUsuarios();

      isSubmitting = false;
      activeView = "history"; // Cambiar a la vista de historial después de crear
      showFloatingMessage("success", "Usuario creado y registrado con éxito.");
    } catch (err) {
      console.error("Error al crear usuario:", err);
      isSubmitting = false;
      showFloatingMessage(
        "error",
        err.message || "Error desconocido al intentar crear el usuario.",
      );
    }
  }

  function limpiarNuevo() {
    nuevoUsuario = {
      id_rol: 3,
      primer_nombre: "",
      segundo_nombre: "",
      primer_apellido: "",
      segundo_apellido: "",
      telefono: "",
      correo: "",
      username: "",
      password: "",
      estado: 1,
    };
  }

  onMount(() => {
    cargarRoles();
    cargarUsuarios();
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
    <h1 class="page-title">Gestión de Usuarios</h1>

    <!-- ---------------------------------- -->
    <!-- NAVEGACIÓN TIPO PESTAÑA -->
    <!-- ---------------------------------- -->
    <div class="category-tabs">
      <button
        class="tab-btn"
        class:active={activeView === "history"}
        on:click={() => (activeView = "history")}
      >
        <!-- Icono de usuarios -->
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
          ><path d="M16 21v-2a4 4 0 0 0-4-4H6a4 4 0 0 0-4 4v2" /><circle
            cx="9"
            cy="7"
            r="4"
          /><path d="M22 21v-2a4 4 0 0 0-3-3.87" /><path
            d="M16 3.13a4 4 0 0 1 0 7.75"
          /></svg
        >
        Ver Usuarios
      </button>
      <button
        class="tab-btn"
        class:active={activeView === "create"}
        on:click={() => (activeView = "create")}
      >
        <!-- Icono de agregar usuario -->
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
          ><path d="M16 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2" /><circle
            cx="8.5"
            cy="7"
            r="4"
          /><path d="M20 8v6M23 11h-6" /></svg
        >
        Crear Usuario
      </button>
    </div>

    <!-- ---------------------------------- -->
    <!-- VISTA 1: CREAR USUARIO -->
    <!-- ---------------------------------- -->
    {#if activeView === "create"}
      <div class="card card-create">
        <h2 class="subtitulo">Registrar Nuevo Usuario</h2>

        <div class="form form-grid-user">
          <input
            placeholder="Primer nombre *"
            bind:value={nuevoUsuario.primer_nombre}
            required
            class="input-field"
          />
          <input
            placeholder="Segundo nombre"
            bind:value={nuevoUsuario.segundo_nombre}
            class="input-field"
          />
          <input
            placeholder="Primer apellido *"
            bind:value={nuevoUsuario.primer_apellido}
            required
            class="input-field"
          />
          <input
            placeholder="Segundo apellido"
            bind:value={nuevoUsuario.segundo_apellido}
            class="input-field"
          />
          <input
            placeholder="Teléfono"
            bind:value={nuevoUsuario.telefono}
            class="input-field"
          />
          <input
            type="email"
            placeholder="Correo *"
            bind:value={nuevoUsuario.correo}
            required
            class="input-field"
          />
          <input
            placeholder="Username *"
            bind:value={nuevoUsuario.username}
            required
            class="input-field"
          />
          <input
            type="password"
            placeholder="Contraseña *"
            bind:value={nuevoUsuario.password}
            required
            class="input-field"
          />

          <div class="select-container">
            <select bind:value={nuevoUsuario.id_rol} class="input-field">
              <option value="" disabled>Seleccionar Rol</option>
              {#each roles as r}
                <option value={r.id_rol}>{r.nombre}</option>
              {/each}
            </select>
          </div>
          <div class="form-footer">
            <button
              class="btn-crear"
              on:click={crearUsuario}
              disabled={isSubmitting}
            >
              {isSubmitting ? "Registrando..." : "Guardar Nuevo Usuario"}
            </button>
          </div>
        </div>
      </div>
    {/if}

    <!-- ---------------------------------- -->
    <!-- VISTA 2: LISTA DE USUARIOS (HISTORIAL) -->
    <!-- ---------------------------------- -->
    {#if activeView === "history"}
      <div class="card card-history">
        <h2 class="subtitulo">Lista de Usuarios Registrados</h2>

        {#if loading}
          <!-- Animación de carga sutil para la tabla -->
          <div class="loading-container-bar">
            <div class="loading-bar"></div>
          </div>
        {:else if error}
          <p class="error-text">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="24"
              height="24"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
              class="icon-error"
              ><circle cx="12" cy="12" r="10" /><path d="M15 9l-6 6" /><path
                d="M9 9l6 6"
              /></svg
            >
            {error}
          </p>
        {:else if usuarios.length === 0}
          <p class="subtext">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="20"
              height="20"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
              class="icon-info"
              ><circle cx="12" cy="12" r="10" /><path d="M12 16v-4" /><path
                d="M12 8h.01"
              /></svg
            >
            No hay usuarios registrados en el sistema.
          </p>
        {:else}
          <div class="table-wrap">
            <table class="tabla">
              <thead>
                <tr>
                  <th class="col-id">ID</th>
                  <th>Nombre Completo</th>
                  <th class="col-tel">Teléfono</th>
                  <th>Correo</th>
                  <th>Username</th>
                  <th>Rol</th>
                  <th>Estado</th>
                  <th>Acciones</th>
                </tr>
              </thead>
              <tbody>
                {#each usuarios as u (u.id_usuario)}
                  <!-- Se eliminó la clase pending-row de aquí -->
                  <tr class:active-row={u.estado === 1}>
                    <td class="col-id" data-label="ID">{u.id_usuario}</td>
                    <td data-label="Nombre Completo">
                      {u.primer_nombre}
                      {u.segundo_nombre}
                      {u.primer_apellido}
                      {u.segundo_apellido}
                    </td>
                    <td class="col-tel" data-label="Teléfono"
                      >{u.telefono || "-"}</td
                    >
                    <td data-label="Correo" class="descripcion-celda"
                      >{u.correo}</td
                    >
                    <td data-label="Username">{u.username}</td>

                    <td data-label="Rol">
                      <!-- Deshabilitar solo el select mientras la actualización está en curso -->
                      <select
                        value={u.id_rol}
                        on:change={(e) => cambiarRol(u, e)}
                        disabled={pendingUpdates.has(u.id_usuario)}
                      >
                        {#each roles as r}
                          <option value={r.id_rol}>{r.nombre}</option>
                        {/each}
                      </select>
                    </td>

                    <td data-label="Estado">
                      <span
                        class="estado"
                        class:estado-activa={u.estado === 1}
                        class:estado-cerrada={u.estado !== 1}
                      >
                        {u.estado === 1 ? "Activo" : "Inactivo"}
                      </span>
                    </td>

                    <td data-label="Acciones">
                      <!-- Deshabilitar el botón mientras la actualización está en curso -->
                      <button
                        class="btn-toggle"
                        on:click={() => toggleEstado(u)}
                        class:btn-deactivate={u.estado === 1}
                        class:btn-activate={u.estado !== 1}
                        disabled={pendingUpdates.has(u.id_usuario)}
                      >
                        {u.estado === 1 ? "Desactivar" : "Activar"}
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

  <!-- ---------------------------------- -->
  <!-- NOTIFICACIÓN FLOTANTE (COMO EN EL LOGIN) -->
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
    --color-bg-primary: #0f172a; /* Fondo principal muy oscuro (slate-900) */
    --color-card-bg: #1e293b; /* Fondo de tarjeta/componente (slate-800) */
    --color-table-bg: #1a202c; /* Fondo de tabla muy oscuro (gray-900) */
    --color-accent-gold: #fcd34d; /* Amarillo principal (Dorado) */
    --color-accent-blue: #60a5fa; /* Azul para highlights (blue-400) */
    --color-text-light: #e2e8f0; /* Texto claro (slate-200) */
    --color-text-muted: #94a3b8; /* Texto sutil (slate-400) */
    --color-border: #334155; /* Borde sutil (slate-700) */
    --color-primary: #1e3a8a; /* Azul (para botón activo) */
    --color-success: #10b981; /* Verde (Activa) */
    --color-danger: #ef4444; /* Rojo (Cerrada/Error) */
    --shadow-card: 0 10px 25px rgba(0, 0, 0, 0.4);
    --shadow-btn: 0 5px 15px rgba(252, 211, 77, 0.3);

    /* COLORES PARA EL MODAL FLOTANTE */
    --color-modal-success: var(--color-success); /* Verde para éxito */
    --color-modal-error: var(--color-danger); /* Rojo para error */
    /* Variables de color para el Scrollbar */
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
    max-width: 650px;
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
    width: 100%; /* Asegura que la tarjeta use todo el ancho de su contenedor */
  }

  .card-create {
    max-width: 800px; /* Ancho más generoso para el formulario de usuario */
  }

  .card-history {
    max-width: 1200px;
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

  /* ---------------------------------- */
  /* Formulario (Creación) */
  /* ---------------------------------- */

  /* GRID de 4 columnas para el formulario de creación de usuario */
  .form-grid-user {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 20px;
  }

  .form-grid-user .select-container {
    grid-column: span 2; /* El selector de rol ocupa 2 columnas */
  }

  .form-grid-user .form-footer {
    grid-column: 1 / -1; /* El botón ocupa todo el ancho */
    text-align: right;
    padding-top: 10px;
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

  .form textarea {
    height: 100px;
    resize: vertical;
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
  /* Tabla (Historial) */
  /* ---------------------------------- */

  .table-wrap::-webkit-scrollbar {
    overflow-x: auto;
    max-height: 70vh;
    overflow-y: auto;
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
    min-width: 1000px; /* Asegura el scroll horizontal */
    border-collapse: collapse;
    background: var(--color-table-bg);
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

  .tabla th:nth-child(2) {
    width: 20%;
  } /* Nombre completo */
  .tabla th:nth-child(4) {
    width: 18%;
  } /* Correo */
  .tabla th:nth-child(6) {
    width: 12%;
  } /* Rol */
  .tabla th:nth-child(8) {
    width: 10%;
  } /* Acciones */

  .tabla td {
    padding: 12px 15px;
    border-bottom: 1px solid var(--color-border);
    font-size: 0.95rem;
    color: var(--color-text-light);
    vertical-align: middle;
  }

  /* Estilos específicos para el select de rol */
  .tabla td select {
    padding: 8px;
    border-radius: 6px;
    border: 1px solid #444;
    background: #1e293b;
    color: white;
    font-size: 0.9rem;
    width: 100%;
    min-width: 100px;
    transition: background 0.3s;
  }

  .tabla td select:disabled {
    background: #334155;
    cursor: progress; /* Cursor de progreso mientras está deshabilitado */
  }

  /* Fila activa (Estado = 1) */
  .active-row {
    background-color: rgba(96, 165, 250, 0.1);
  }

  .active-row:hover td {
    background: rgba(96, 165, 250, 0.15);
  }

  tr:hover:not(.active-row) td {
    background: rgba(255, 255, 255, 0.03);
  }

  /* Se ha eliminado la clase .pending-row */

  .resaltado {
    color: var(--color-accent-gold);
    font-weight: 700;
  }

  /* Estilos de Estado */
  .estado {
    padding: 6px 12px;
    border-radius: 20px; /* Pill shape */
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

  /* Botones de acción en tabla */
  .btn-toggle {
    border: none;
    padding: 8px 12px;
    border-radius: 6px;
    color: white;
    font-weight: 600;
    cursor: pointer;
    transition: opacity 0.2s;
    font-size: 0.9rem;
  }
  .btn-toggle:hover:not(:disabled) {
    opacity: 0.9;
  }
  .btn-toggle:disabled {
    opacity: 0.5;
    cursor: progress; /* Cursor de progreso mientras está deshabilitado */
  }

  .btn-deactivate {
    background: var(--color-danger); /* Rojo para desactivar */
  }
  .btn-activate {
    background: var(--color-success); /* Verde para activar */
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
  /* Indicador de Carga (Solo Barra de Progreso) */
  /* ---------------------------------- */
  .loading-container-bar {
    width: 100%;
    display: flex;
    justify-content: center; /* Centrar la barra */
    padding: 5px 0; /* Espacio para que no se pegue */
  }

  .loading-bar {
    width: 80%; /* Ancho de la barra de progreso */
    height: 5px;
    background: var(--color-bg-light); /* Fondo estático */
    border-radius: 5px;
    overflow: hidden;
  }

  .loading-bar::before {
    content: "";
    display: block;
    height: 100%;
    width: 30%; /* Tamaño visible de la barra animada */
    background: linear-gradient(
      90deg,
      var(--color-primary),
      var(--color-secondary)
    );
    animation: loading-move 1.5s infinite linear;
  }

  @keyframes loading-move {
    0% {
      transform: translateX(-100%);
    }
    100% {
      transform: translateX(333%); /* 100% + 2 veces el ancho para salir */
    }
  }
  /* ---------------------------------- */
  /* ESTILOS DEL MENSAJE FLOTANTE (COPIADO DE NOTIFICACIONES) */
  /* ---------------------------------- */
  .message-modal {
    position: fixed;
    top: 100px; /* Ajustado para estar debajo de la doble nav bar */
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
      top: 90px;
      right: 10px;
      left: 10px; /* Ocupa casi todo el ancho en móvil */
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
    }

    /* Formulario: 2 columnas en móvil */
    .form-grid-user {
      grid-template-columns: repeat(2, 1fr);
      gap: 15px;
    }
    .form-grid-user .select-container {
      grid-column: span 2;
    }
    .form-grid-user .form-footer {
      grid-column: 1 / -1;
    }
    .form-grid-user .btn-crear {
      width: 100%;
      text-align: center;
    }

    /* ------------------------------------------ */
    /* CAMBIOS CRÍTICOS PARA LA TABLA RESPONSIVA */
    /* ------------------------------------------ */

    .table-wrap {
      overflow-x: hidden; /* Eliminar scroll horizontal en vista de tarjeta */
      border: none; /* Quitar el borde principal para que las tarjetas se vean mejor */
      max-height: none; /* Permitir que crezca verticalmente */
      overflow-y: visible; /* Permitir que crezca verticalmente */
      padding: 0 5px; /* Pequeño padding a los lados */
    }

    .tabla {
      min-width: 100%; /* El ancho mínimo se ajusta */
      display: block; /* La tabla se comporta como un bloque */
      background: none; /* Quitar fondo de tabla, usar fondo de tarjeta */
    }

    .tabla thead {
      display: none; /* Ocultar encabezados de columna */
    }

    .tabla tbody {
      display: block; /* El cuerpo de la tabla se comporta como un bloque */
      width: 100%;
    }

    .tabla tr {
      display: block; /* Cada fila se convierte en una tarjeta */
      margin-bottom: 20px;
      padding: 15px;
      border: 1px solid var(--color-border);
      border-radius: 8px;
      background: var(--color-card-bg); /* Fondo de tarjeta */
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
      transition: all 0.3s ease;
    }

    .active-row {
      background-color: var(
        --color-card-bg
      ); /* Mantener el color base de la tarjeta */
      border-left: 5px solid var(--color-accent-blue); /* Resaltar con barra lateral */
    }

    /* Eliminar el efecto hover de filas enteras ya que ahora son tarjetas separadas */
    .tabla tr:hover td,
    .active-row:hover td {
      background: transparent;
    }

    .tabla td {
      display: flex; /* La celda se convierte en una fila flex (Etiqueta | Valor) */
      justify-content: space-between;
      text-align: right;
      padding: 10px 0;
      border-bottom: 1px dotted rgba(255, 255, 255, 0.1);
      font-size: 0.9rem;
      height: auto;
      align-items: center;
    }

    .tabla td:last-child {
      border-bottom: none;
      padding-bottom: 0;
    }

    /* Mostrar la etiqueta de la celda usando el atributo data-label */
    .tabla td::before {
      content: attr(data-label);
      font-weight: 700;
      text-transform: uppercase;
      color: var(--color-text-muted);
      text-align: left;
      flex-basis: 50%; /* Dar espacio fijo a la etiqueta */
      margin-right: 15px;
      flex-shrink: 0;
    }

    /* Ajuste para los elementos de control dentro de la celda */
    .tabla td[data-label="Rol"],
    .tabla td[data-label="Acciones"] {
      flex-direction: column; /* Apilar Etiqueta y control */
      align-items: flex-start;
      text-align: left;
    }

    .tabla td[data-label="Rol"]::before,
    .tabla td[data-label="Acciones"]::before {
      text-align: left;
      margin-bottom: 5px;
      flex-basis: auto; /* Dejar que la etiqueta tome su tamaño */
    }

    .tabla td[data-label="Rol"] select,
    .tabla td[data-label="Acciones"] .btn-toggle {
      width: 100%;
      box-sizing: border-box;
    }

    .estado {
      /* Asegurar que el pill de estado se vea bien a la derecha */
      text-align: right;
    }
  }
</style>
