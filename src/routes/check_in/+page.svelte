<script>
  import { onMount } from "svelte";
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";

  // --- VARIABLES DE ESTADO ---
  let idReserva = "";
  let mensaje = "";
  let estado = ""; // 'ok' o 'error'
  let habitaciones = [];
  let menuAbierto = null; // Guarda el id_habitacion que tiene el menú abierto
  let habitacionSeleccionada = null;

  // Variables para Modal/Formulario de Usuario Registrado
  let mostrarInputDocumento = false;
  let numeroDocumento = "";
  let usuarioEncontrado = null;
  let cargandoUsuario = false;
  let errorUsuario = "";

  // Variables para Modal/Formulario de Nuevo Registro
  let mostrarRegistroNuevo = false;
  let nuevo = {
    primer_nombre: "",
    primer_apellido: "",
    correo: "",
    username: "",
    password: "",
    numero_documento: "",
    lugar_expedicion: "",
  };
  let cargandoNuevo = false;
  let errorNuevo = "";

  // ---------------------------------------------------
  // Lógica de Capacidad
  // ---------------------------------------------------
  async function verificarCapacidad(habitacion) {
    try {
      const res = await fetch(
        "https://inntech-backend.onrender.com/usuarios_habitaciones/check_capacidad",
        {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            id_reserva: Number(idReserva),
            id_habitacion: habitacion.id_habitacion,
            capacidad_max: habitacion.capacidad_max,
          }),
        },
      );

      if (!res.ok) {
        console.error("check_capacidad status:", res.status, await res.text());
        habitacion.ocupados = 0;
        habitacion.disponible = false;
        return;
      }

      const data = await res.json();

      if (data && "disponible" in data) {
        habitacion.ocupados = data.ocupados ?? 0;
        habitacion.disponible = !!data.disponible;
      } else if (data && data.success && data.disponible !== undefined) {
        habitacion.ocupados = data.ocupados ?? 0;
        habitacion.disponible = !!data.disponible;
      } else {
        console.warn("Formato inesperado de /check_capacidad:", data);
        habitacion.ocupados = data.ocupados ?? 0;
        habitacion.disponible = false;
      }
    } catch (err) {
      console.error("Error verificando capacidad:", err);
      habitacion.ocupados = 0;
      habitacion.disponible = false;
    }
  }

  // ---------------------------------------------------
  // Lógica de Búsqueda de Reserva
  // ---------------------------------------------------
  async function buscarReserva() {
    mensaje = "";
    estado = "";
    habitaciones = [];

    // ** CORRECCIÓN: Asegurar que idReserva es un string antes de usar .trim() **
    const idReservaStr = String(idReserva ?? "").trim();

    if (!idReservaStr) {
      estado = "error";
      mensaje = "⚠️ Debe ingresar un ID de reserva.";
      return;
    }

    try {
      // 1. Buscar reserva
      const res = await fetch(
        `https://inntech-backend.onrender.com/reservas/get_reserva/${idReservaStr}`,
      );

      if (!res.ok) {
        estado = "error";
        mensaje = "Reserva no encontrada.";
        return;
      }

      const data = await res.json();

      if (!data || !data.data) {
        estado = "error";
        mensaje = "No se encontró la reserva.";
        return;
      }

      estado = "ok";
      mensaje = "Reserva encontrada.";

      // 2. Buscar habitaciones asociadas
      const resHab = await fetch(
        `https://inntech-backend.onrender.com/reservas_habitaciones/get_habitaciones_by_reserva/${idReservaStr}`,
      );

      const dataHab = await resHab.json();

      if (dataHab.success) {
        habitaciones = dataHab.habitaciones;

        // 3. Verificar capacidad
        await Promise.all(habitaciones.map((h) => verificarCapacidad(h)));
        habitaciones = [...habitaciones];
      }
    } catch (err) {
      estado = "error";
      mensaje = "Error al conectar con el servidor.";
      console.error(err);
    }
  }
  function mostrarMensaje(texto, tipo = "ok") {
    mensaje = texto;
    estado = tipo;

    // Autocerrar después de 4.5s
    setTimeout(() => {
      mensaje = "";
      estado = "";
    }, 4500);
  }

  // ---------------------------------------------------
  // Lógica de Menús y Modales
  // ---------------------------------------------------

  function abrirMenu(h) {
    if (menuAbierto === h.id_habitacion) {
      menuAbierto = null; // cerrar si ya estaba abierto
      habitacionSeleccionada = null;
    } else {
      menuAbierto = h.id_habitacion;
      habitacionSeleccionada = h;
    }
    // Asegurar que los modales estén cerrados al abrir el mini-menu
    mostrarInputDocumento = false;
    mostrarRegistroNuevo = false;
  }

  function opcionUsuario(tipo) {
    // Cerrar el mini-menú
    menuAbierto = null;

    // Resetear y abrir el modal correspondiente
    if (tipo === "registrado") {
      mostrarInputDocumento = true;
      mostrarRegistroNuevo = false;
      numeroDocumento = "";
      usuarioEncontrado = null;
      errorUsuario = "";
    } else {
      mostrarRegistroNuevo = true;
      mostrarInputDocumento = false;
      // Resetear el objeto 'nuevo' para el formulario de registro
      nuevo = {
        primer_nombre: "",
        primer_apellido: "",
        correo: "",
        username: "",
        password: "",
        numero_documento: "",
        lugar_expedicion: "",
      };
      errorNuevo = "";
    }
  }

  // ---------------------------------------------------
  // Lógica de Registro de Nuevo Usuario
  // ---------------------------------------------------
  async function registrarNuevoUsuario() {
    errorNuevo = "";

    // Validación mínima
    if (
      !nuevo.primer_nombre ||
      !nuevo.primer_apellido ||
      !nuevo.correo ||
      !nuevo.username ||
      !nuevo.password ||
      !nuevo.numero_documento ||
      !nuevo.lugar_expedicion
    ) {
      errorNuevo = "Todos los campos son obligatorios.";
      return;
    }

    cargandoNuevo = true;

    try {
      // 1. Crear usuario
      const resUser = await fetch(
        "https://inntech-backend.onrender.com/auth/register",
        {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(nuevo),
        },
      );

      const dataUser = await resUser.json();

      if (!resUser.ok) {
        errorNuevo = dataUser.detail || "Error creando usuario.";
        cargandoNuevo = false;
        return;
      }

      const idUsuario = dataUser.id_usuario;

      // 2. Crear documento
      await fetch(
        "https://inntech-backend.onrender.com/documentos/create_documento",
        {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            id_tdocumento: 1, // Tipo de documento por defecto
            id_usuario: idUsuario,
            numero_documento: nuevo.numero_documento,
            lugar_expedicion: nuevo.lugar_expedicion,
            estado: 1,
          }),
        },
      );

      // 3. Registrarlo en habitación
      await fetch(
        "https://inntech-backend.onrender.com/usuarios_habitaciones/create_usuario_habitacion",
        {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            id_reserva: Number(idReserva),
            id_habitacion: habitacionSeleccionada.id_habitacion,
            id_usuario: idUsuario,
          }),
        },
      );

      alert("✔ Usuario creado y registrado en habitación exitosamente");

      // Reset
      mostrarRegistroNuevo = false;
      nuevo = {
        /* reset campos */
      };

      cargandoNuevo = false;

      // actualizar capacidad
      await verificarCapacidad(habitacionSeleccionada);
      habitaciones = [...habitaciones];
    } catch (err) {
      console.error(err);
      errorNuevo = "Error en el proceso.";
      cargandoNuevo = false;
    }
  }

  // ---------------------------------------------------
  // Lógica de Búsqueda y Registro de Usuario Existente
  // ---------------------------------------------------
  async function buscarUsuarioPorDocumento() {
    usuarioEncontrado = null;
    errorUsuario = "";

    if (!numeroDocumento.trim()) {
      errorUsuario = "Debe ingresar un número de documento.";
      return;
    }

    cargandoUsuario = true;

    try {
      const res = await fetch(
        `https://inntech-backend.onrender.com/documentos/buscar_usuario_por_documento/${numeroDocumento}`,
      );

      const data = await res.json();

      if (!res.ok) {
        errorUsuario = data.detail || "Usuario no encontrado";
        cargandoUsuario = false;
        return;
      }

      usuarioEncontrado = data.data; // contiene id_usuario y nombre_completo
      cargandoUsuario = false;
      errorUsuario = ""; // Limpiar error si se encuentra
    } catch (error) {
      console.error(error);
      errorUsuario = "Error al conectar con el servidor.";
      cargandoUsuario = false;
    }
  }

  async function registrarUsuarioEnHabitacion() {
    if (!usuarioEncontrado || !habitacionSeleccionada) return;

    const body = {
      id_reserva: Number(idReserva),
      id_habitacion: habitacionSeleccionada.id_habitacion,
      id_usuario: usuarioEncontrado.id_usuario,
    };

    const res = await fetch(
      "https://inntech-backend.onrender.com/usuarios_habitaciones/create_usuario_habitacion",
      {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(body),
      },
    );

    const data = await res.json();

    if (res.ok) {
      alert("✔ Usuario agregado correctamente");
      mostrarInputDocumento = false;
      numeroDocumento = "";
      usuarioEncontrado = null;

      // actualizar capacidad
      await verificarCapacidad(habitacionSeleccionada);
      habitaciones = [...habitaciones];
    } else {
      alert("Error: " + data.detail);
    }
  }

  // ---------------------------------------------------
  // Lógica de UX (Cerrar con Escape)
  // ---------------------------------------------------
  function handleKeydown(event) {
    if (event.key === "Escape") {
      mostrarInputDocumento = false;
      mostrarRegistroNuevo = false;
      menuAbierto = null;
    }
  }

  onMount(() => {
    document.addEventListener("keydown", handleKeydown);
    return () => {
      document.removeEventListener("keydown", handleKeydown);
    };
  });
</script>

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

<section class="main">
  <div class="content-container">
    <h1 class="page-title">Gestión de Check-In</h1>

    <div class="card card-search">
      <h2 class="card-subtitle">
        <i class="fas fa-search"></i> Buscar Reserva
      </h2>

      <div class="form-grid-search">
        <input
          placeholder="ID de la reserva"
          bind:value={idReserva}
          class="input-field"
          type="number"
        />
        <button class="btn-primary" on:click={buscarReserva}>
          <i class="fas fa-search icon-left"></i>
          Buscar Reserva
        </button>
      </div>
    </div>

    {#if mensaje}
      <div
        class="msg-inline"
        class:msg-success={estado === "ok"}
        class:msg-error={estado === "error"}
      >
        <i
          class="fas"
          class:fa-check-circle={estado === "ok"}
          class:fa-times-circle={estado === "error"}
        ></i>
        {mensaje}
      </div>
    {/if}

    {#if estado === "ok" && habitaciones.length > 0}
      <div class="card card-table">
        <h2 class="card-subtitle">
          <i class="fas fa-bed"></i> Habitaciones Asociadas
        </h2>

        <div class="table-wrap">
          <table class="tabla">
            <thead>
              <tr>
                <th>ID</th>
                <th>Número</th>
                <th>Capacidad Máx.</th>
                <th>Ocupados</th>
                <th class="col-actions">Acción</th>
              </tr>
            </thead>

            <tbody>
              {#each habitaciones as h}
                <tr>
                  <td>{h.id_habitacion}</td>
                  <td class="resaltado">{h.numero}</td>
                  <td>{h.capacidad_max}</td>
                  <td>{h.ocupados}</td>

                  <td class="col-actions menu-parent">
                    {#if h.disponible}
                      <button class="btn-accion" on:click={() => abrirMenu(h)}>
                        <i class="fas fa-plus icon-left"></i> Agregar usuario
                      </button>

                      {#if menuAbierto === h.id_habitacion}
                        <div class="mini-menu">
                          <button
                            class="btn-mini"
                            on:click={() => opcionUsuario("registrado")}
                          >
                            👤 Usuario registrado
                          </button>
                          <button
                            class="btn-mini"
                            on:click={() => opcionUsuario("nuevo")}
                          >
                            🆕 Usuario nuevo
                          </button>
                        </div>
                      {/if}
                    {:else}
                      <span class="badge badge-danger">
                        <i class="fas fa-ban icon-left"></i> Hab. llena
                      </span>
                    {/if}
                  </td>
                </tr>
              {/each}
            </tbody>
          </table>
        </div>
      </div>
    {/if}

    {#if mostrarInputDocumento}
      <div
        class="modal-backdrop"
        on:click={() => (mostrarInputDocumento = false)}
      ></div>
      <div class="registro-box">
        <h3 class="modal-title">
          <i class="fas fa-user-check"></i> Agregar Usuario Registrado a Hab. {habitacionSeleccionada
            ? habitacionSeleccionada.numero
            : "N/A"}
        </h3>

        <input
          placeholder="Número de documento"
          bind:value={numeroDocumento}
          class="input-field"
        />

        <div
          class="modal-actions-footer"
          style="border-top: none; padding-top: 15px; margin-top: 0;"
        >
          <button
            class="btn-primary"
            on:click={buscarUsuarioPorDocumento}
            disabled={cargandoUsuario}
          >
            <i class="fas fa-search icon-left" class:fa-spin={cargandoUsuario}
            ></i>
            {cargandoUsuario ? "Buscando..." : "Buscar Usuario"}
          </button>
        </div>

        {#if errorUsuario}
          <p class="error-inline">
            <i class="fas fa-times-circle"></i>
            {errorUsuario}
          </p>
        {/if}

        {#if usuarioEncontrado}
          <div class="found-user-box">
            <p class="success-inline">
              <i class="fas fa-check-circle"></i> Usuario encontrado: **{usuarioEncontrado.nombre_completo}**
            </p>
          </div>
        {/if}

        <div class="modal-actions-footer">
          <button
            class="btn-cancel"
            on:click={() => (mostrarInputDocumento = false)}>Cancelar</button
          >
          {#if usuarioEncontrado}
            <button class="btn-save" on:click={registrarUsuarioEnHabitacion}>
              <i class="fas fa-link icon-left"></i> Registrar en Habitación
            </button>
          {/if}
        </div>
      </div>
    {/if}

    {#if mostrarRegistroNuevo}
      <div
        class="modal-backdrop"
        on:click={() => (mostrarRegistroNuevo = false)}
      ></div>
      <div class="registro-box large-modal">
        <h3 class="modal-title">
          <i class="fas fa-user-plus"></i> Registrar Usuario Nuevo y Check-In en
          Hab. {habitacionSeleccionada ? habitacionSeleccionada.numero : "N/A"}
        </h3>

        <div class="form-grid wide-form-grid">
          <input
            placeholder="Primer nombre"
            bind:value={nuevo.primer_nombre}
            class="input-field"
          />
          <input
            placeholder="Primer apellido"
            bind:value={nuevo.primer_apellido}
            class="input-field"
          />
          <input
            placeholder="Correo"
            type="email"
            bind:value={nuevo.correo}
            class="input-field"
          />
          <input
            placeholder="Username"
            bind:value={nuevo.username}
            class="input-field"
          />
          <input
            placeholder="Password"
            type="password"
            bind:value={nuevo.password}
            class="input-field"
          />

          <input
            placeholder="Número de documento"
            bind:value={nuevo.numero_documento}
            class="input-field"
          />
          <input
            placeholder="Lugar de expedición"
            bind:value={nuevo.lugar_expedicion}
            class="input-field"
          />
        </div>

        {#if errorNuevo}
          <p class="error-inline">
            <i class="fas fa-times-circle"></i>
            {errorNuevo}
          </p>
        {/if}

        <div class="modal-actions-footer">
          <button
            class="btn-cancel"
            on:click={() => (mostrarRegistroNuevo = false)}>Cancelar</button
          >
          <button
            class="btn-save"
            on:click={registrarNuevoUsuario}
            disabled={cargandoNuevo}
          >
            <i class="fas fa-check icon-left" class:fa-spin={cargandoNuevo}></i>
            {cargandoNuevo
              ? "Registrando..."
              : "Crear y Registrar en Habitación"}
          </button>
        </div>
      </div>
    {/if}
  </div>
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
    --color-primary-dark: #1e3a8a; /* Azul oscuro (para encabezados/tabla) */
    --color-success: #10b981; /* Verde (Activa) */
    --color-danger: #ef4444; /* Rojo (Desactivada) */
    --color-warning: #f59e0b; /* Naranja (Warning/Confirmación) */
    --shadow-card: 0 10px 25px rgba(0, 0, 0, 0.4);
    --shadow-btn-primary: 0 5px 15px rgba(252, 211, 77, 0.3);
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

  .card-search {
    max-width: 650px;
    align-self: center;
    padding: 30px;
  }

  .card-table {
    overflow: hidden;
    justify-content: center;
  }

  .card-subtitle {
    color: var(--color-accent-blue);
    font-size: clamp(1.4rem, 2.5vw, 1.8rem);
    padding: 0 0 15px 0;
    font-weight: 700;
    display: flex;
    align-items: center;
    gap: 10px;
    border-bottom: 1px solid var(--color-border);
    margin-bottom: 20px;
  }

  /* ---------------------------------- */
  /* Inputs y Formularios */
  /* ---------------------------------- */

  /* Inputs generales dentro del componente, incluyendo los enlazados en el script */
  input:not(.btn-accion, .btn-primary) {
    box-sizing: border-box;
    width: 100%;
    padding: 12px 16px;
    background: var(--color-table-bg);
    color: var(--color-text-light);
    border: 1px solid var(--color-border);
    border-radius: 8px;
    transition: all 0.3s;
  }

  input::placeholder {
    color: var(--color-text-muted);
  }

  input:focus {
    border-color: var(--color-accent-gold);
    box-shadow: 0 0 0 3px rgba(252, 211, 77, 0.4);
    outline: none;
  }

  .form-grid-search {
    display: grid;
    gap: 15px;
    grid-template-columns: 2fr 1fr;
  }

  .form-grid {
    display: grid;
    gap: 20px;
    margin-top: 10px;
    grid-template-columns: 1fr;
  }

  .wide-form-grid {
    display: grid;
    gap: 15px;
    grid-template-columns: repeat(2, 1fr);
  }

  /* ---------------------------------- */
  /* Botones de Acción Global */
  /* ---------------------------------- */

  .btn-primary,
  .btn-save,
  .btn-cancel,
  .btn-accion {
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
    white-space: nowrap;
  }

  /* Sobrescribe .btn-save del código original */
  .btn-save {
    background: var(--color-success);
    color: #111;
    font-weight: 800;
    padding: 12px 20px;
    width: auto !important;
  }

  .form-box .btn-save {
    width: 100% !important;
    background: var(--color-accent-gold);
    color: #111;
    font-weight: 800;
    padding: 12px 20px;
  }

  .form-box .btn-save:hover {
    background: #ffaa2b;
    transform: translateY(-1px);
    box-shadow: 0 8px 20px rgba(252, 211, 77, 0.5);
  }

  .btn-primary {
    background: var(--color-accent-gold);
    color: #111;
    box-shadow: var(--shadow-btn-primary);
    padding: 12px 25px;
  }

  .btn-primary:hover:not(:disabled) {
    background: #ffaa2b;
    transform: translateY(-1px);
    box-shadow: 0 8px 20px rgba(252, 211, 77, 0.5);
  }

  .btn-primary:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }

  .btn-save:hover {
    background: #10e092;
    transform: translateY(-1px);
  }

  .btn-cancel {
    background: var(--color-card-bg);
    color: var(--color-text-light);
    border: 1px solid var(--color-border);
    padding: 12px 20px;
  }

  .btn-cancel:hover {
    background: #2a3547;
    border-color: var(--color-accent-blue);
  }

  /* Botones de Acción de Fila */
  .col-actions {
    position: relative;
    padding-top: 10px;
    padding-bottom: 10px;
  }

  .btn-accion {
    background: var(--color-accent-blue) !important;
    color: var(--color-bg-primary) !important;
    padding: 8px 12px !important;
    border-radius: 6px !important;
    font-weight: 600 !important;
    font-size: 0.85rem !important;
    gap: 6px;
  }

  .btn-accion:hover {
    background: #4a91ff !important;
  }

  /* ---------------------------------- */
  /* TABLA */
  /* ---------------------------------- */
  .table-wrap {
    overflow-x: auto;
    max-height: 70vh;
    overflow-y: auto;
    border: 1px solid var(--color-border);
    border-radius: 8px;
  }

  .tabla {
    width: 100%;
    min-width: 600px;
    border-collapse: collapse;
    background: var(--color-table-bg);
  }

  .tabla th {
    background: var(--color-primary-dark) !important;
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
    border-bottom: 1px solid var(--color-border) !important;
  }

  .tabla td {
    padding: 12px 15px;
    border-bottom: 1px solid var(--color-border) !important;
    font-size: 0.95rem;
    color: var(--color-text-light);
    vertical-align: middle;
  }

  .tabla tbody tr:hover td {
    background: rgba(255, 255, 255, 0.03);
  }

  .resaltado {
    color: var(--color-accent-gold);
    font-weight: 600;
  }

  /* Badges de Estado */
  .badge {
    padding: 6px 12px;
    border-radius: 4px;
    font-weight: 700;
    font-size: 0.85rem;
    text-transform: uppercase;
    display: inline-flex;
    align-items: center;
    gap: 5px;
  }

  .badge-danger {
    background: rgba(239, 68, 68, 0.2);
    color: var(--color-danger);
  }

  /* MINI MENÚ */
  .mini-menu {
    position: absolute;
    right: 0;
    top: 55px;
    background: var(--color-card-bg);
    border: 1px solid var(--color-border);
    border-radius: 6px;
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.5);
    z-index: 10;
    display: flex;
    flex-direction: column;
    width: 200px;
    min-width: max-content;
  }

  .mini-menu button {
    background: transparent !important;
    color: var(--color-text-light) !important;
    padding: 10px 15px !important;
    border: none !important;
    text-align: left;
    cursor: pointer;
    font-size: 0.9rem !important;
    transition: background 0.2s;
    display: flex;
    align-items: center;
    gap: 8px;
    width: 100% !important;
    font-weight: normal !important;
    border-radius: 0 !important;
  }

  .mini-menu button:hover {
    background: var(--color-table-bg) !important;
    color: var(--color-accent-gold) !important;
  }

  /* ---------------------------------- */
  /* MODALES (registro-box) */
  /* ---------------------------------- */

  .modal-backdrop {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.8);
    z-index: 100;
  }

  .registro-box {
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
    max-width: 650px;
    box-shadow: 0 15px 40px rgba(0, 0, 0, 0.9);
    animation: fadeInScale 0.3s ease-out forwards;
    display: flex; /* Añadido para mejor disposición interna */
    flex-direction: column; /* Añadido para mejor disposición interna */
    gap: 15px; /* Añadido para espaciado consistente */
  }

  .large-modal {
    max-width: 800px;
  }

  .modal-title {
    color: var(--color-accent-gold);
    font-size: 1.8rem;
    margin-bottom: 5px; /* Reducido para usar el gap */
    border-bottom: 1px solid var(--color-border);
    padding-bottom: 10px;
    display: flex;
    align-items: center;
    gap: 10px;
  }

  .modal-actions-footer {
    margin-top: 5px;
    padding-top: 15px;
    border-top: 1px solid var(--color-border);
    display: flex;
    justify-content: flex-end;
    gap: 10px;
  }

  .found-user-box {
    padding: 15px;
    border: 1px solid var(--color-success);
    border-radius: 8px;
    background: rgba(16, 185, 129, 0.1);
  }

  /* ---------------------------------- */
  /* Mensajes Inline y de Error */
  /* ---------------------------------- */

  .msg-inline {
    position: fixed;
    top: 20px;
    right: 20px;
    z-index: 1000;

    max-width: 650px;
    padding: 15px 25px;
    margin: -15px auto 15px auto;

    display: flex;
    align-items: center;
    gap: 15px;

    font-weight: 600;
    color: white;
    background-color: var(--color-error);
    border-radius: 8px;

    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
    transition: all 0.3s;

    animation: bounce 0.5s ease-out; /* Animación de aparición */
  }

  .msg-success {
    color: var(--color-text-light);
    background: var(--color-success);
    border: 1px solid var(--color-success);
  }

  .msg-error {
    color: var(--color-text-light);
    background: var(--color-danger);
    border: 1px solid var(--color-danger);
  }

  /* Inline simples */
  .error-inline,
  .success-inline {
    margin-top: 5px;
    padding: 10px 15px;

    display: flex;
    align-items: center;
    gap: 8px;

    font-weight: 600;
    border-radius: 6px;
  }

  .error-inline {
    color: var(--color-text-light);
    background: var(--color-danger);
    border: 1px solid var(--color-danger);
  }

  .success-inline {
    color: var(--color-text-light);
    background: var(--color-success);
    border: 1px solid var(--color-success);
  }

  /* Animación Modal */
  @keyframes fadeInScale {
    from {
      opacity: 0;
      transform: translate(-50%, -50%) scale(0.9);
    }
    to {
      opacity: 1;
      transform: translate(-50%, -50%) scale(1);
    }
  }

  /* Media Queries */
  @media (max-width: 768px) {
    .form-grid-search {
      grid-template-columns: 1fr;
    }

    .wide-form-grid {
      grid-template-columns: 1fr;
    }

    .registro-box {
      width: 90%;
      padding: 20px;
    }

    .tabla th,
    .tabla td {
      padding: 10px;
      font-size: 0.85rem;
    }

    .col-actions {
      display: block;
    }

    .btn-accion {
      width: 100%;
      justify-content: center;
    }

    .mini-menu {
      left: 50%;
      transform: translateX(-50%);
      top: 50px;
      width: 90%;
    }
  }

  /* --- SOBRESCRITURAS DEL CÓDIGO ORIGINAL (Clean up) --- */

  /* Ocultar elementos originales reemplazados */
  .welcome-box {
    display: none;
  }
  .role {
    display: none;
  }
  .msg {
    display: none;
  }

  /* Sobrescribir estilos originales obsoletos para usar los nuevos */
  .form-box {
    background: var(--color-bg-primary) !important;
    padding: 30px !important;
    border: 1px solid var(--color-border) !important;
    border-radius: 12px !important;
    max-width: 650px !important;
    margin: 0 auto 30px auto !important;
    box-shadow: var(--shadow-card);
  }

  .form-box h3 {
    display: none; /* Reemplazado por .card-subtitle */
  }

  .tabla-box {
    /* Mantenido para que funcione si se usa, pero es redundante con .card-table */
    margin-top: 25px;
    background: var(--color-bg-primary);
    padding: 30px;
    border: 1px solid var(--color-border);
    border-radius: 12px;
    max-width: 1000px;
    margin: 30px auto;
    box-shadow: var(--shadow-card);
  }
</style>
