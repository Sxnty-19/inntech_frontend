<script>
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";

  let idReserva = "";
  let mensaje = "";
  let estado = ""; 
  let habitaciones = []; 
  let menuAbierto = null; // Guarda el id_habitacion que tiene el menú abierto
 let habitacionSeleccionada = null;
 let mostrarInputDocumento = false;
let numeroDocumento = "";
let usuarioEncontrado = null;
let cargandoUsuario = false;
let errorUsuario = "";
let mostrarRegistroNuevo = false;

let nuevo = {
  primer_nombre: "",
  primer_apellido: "",
  correo: "",
  username: "",
  password: "",
  numero_documento: "",
  lugar_expedicion: ""
};

let cargandoNuevo = false;
let errorNuevo = "";



  // ---------------------------------------------------
  // Función para verificar capacidad de una habitación
  // ---------------------------------------------------
  // ---------------------------------------------------
// Verificar capacidad (mejorada)
// ---------------------------------------------------
async function verificarCapacidad(habitacion) {
  try {
    const res = await fetch("https://inntech-backend.onrender.com/usuarios_habitaciones/check_capacidad", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        id_reserva: Number(idReserva),
        id_habitacion: habitacion.id_habitacion,
        capacidad_max: habitacion.capacidad_max
      })
    });

    if (!res.ok) {
      console.error("check_capacidad status:", res.status, await res.text());
      // marcar como no disponible para no permitir acciones por seguridad
      habitacion.ocupados = 0;
      habitacion.disponible = false;
      return;
    }

    const data = await res.json();
    console.log("check_capacidad response for", habitacion.id_habitacion, data);

    // Asegúrate del formato que devuelve tu API:
    // si tu API devuelve success + disponible, usamos esos; si no, intenta inferir.
    if (data && ("disponible" in data)) {
      habitacion.ocupados = data.ocupados ?? 0;
      habitacion.disponible = !!data.disponible;
    } else if (data && data.success && data.disponible !== undefined) {
      habitacion.ocupados = data.ocupados ?? 0;
      habitacion.disponible = !!data.disponible;
    } else {
      // formato inesperado: por seguridad lo desactivamos y logueamos
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
  // Buscar reserva + cargar sus habitaciones
  // ---------------------------------------------------
  async function buscarReserva() {
    mensaje = "";
    estado = "";
    habitaciones = [];

    if (!idReserva.trim()) {
      estado = "error";
      mensaje = "⚠️ Debe ingresar un ID de reserva.";
      return;
    }

    try {
      // 1. Buscar reserva
      const res = await fetch(
        `https://inntech-backend.onrender.com/reservas/get_reserva/${idReserva}`
      );

      if (!res.ok) {
        estado = "error";
        mensaje = "❌ Reserva no encontrada.";
        return;
      }

      const data = await res.json();

      if (!data || !data.data) {
        estado = "error";
        mensaje = "❌ No se encontró la reserva.";
        return;
      }

      estado = "ok";
      mensaje = `✔️ Reserva encontrada.`;

      // 2. Buscar habitaciones asociadas
      const resHab = await fetch(
        `https://inntech-backend.onrender.com/reservas_habitaciones/get_habitaciones_by_reserva/${idReserva}`
      );

      const dataHab = await resHab.json();

      if (dataHab.success) {
        habitaciones = dataHab.habitaciones;

        // 3. Verificar capacidad de cada habitación automáticamente
        if (dataHab.success) {
          habitaciones = dataHab.habitaciones;

          // Verificar todas en paralelo (más rápido)
          await Promise.all(habitaciones.map(h => verificarCapacidad(h)));

          habitaciones = [...habitaciones];
        }

      }

    } catch (err) {
      estado = "error";
      mensaje = "❌ Error al conectar con el servidor.";
      console.error(err);
    }
  }

    function abrirMenu(h) {
        if (menuAbierto === h.id_habitacion) {
            menuAbierto = null; // cerrar si ya estaba abierto
            habitacionSeleccionada = null;
        } else {
            menuAbierto = h.id_habitacion;
            habitacionSeleccionada = h;
        }
    }


    function opcionUsuario(tipo) {
    if (tipo === "registrado") {
        mostrarInputDocumento = true;
        mostrarRegistroNuevo = false;
    } else {
        mostrarRegistroNuevo = true;
        mostrarInputDocumento = false;
    }

    menuAbierto = null; 
}

async function registrarNuevoUsuario() {
    errorNuevo = "";

    // Validación mínima
    if (!nuevo.primer_nombre ||
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
        const resUser = await fetch("https://inntech-backend.onrender.com/auth/register", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify(nuevo)
        });

        const dataUser = await resUser.json();

        if (!resUser.ok) {
            errorNuevo = dataUser.detail || "Error creando usuario.";
            cargandoNuevo = false;
            return;
        }

        const idUsuario = dataUser.id_usuario;

        // 2. Crear documento
        await fetch("https://inntech-backend.onrender.com/documentos/create_documento", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({
                id_tdocumento: 1,               // Asignas tu tipo 1 por defecto
                id_usuario: idUsuario,
                numero_documento: nuevo.numero_documento,
                lugar_expedicion: nuevo.lugar_expedicion,
                estado: 1
            })
        });

        // 3. Registrarlo en habitación
        await fetch("https://inntech-backend.onrender.com/usuarios_habitaciones/create_usuario_habitacion", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({
                id_reserva: Number(idReserva),
                id_habitacion: habitacionSeleccionada.id_habitacion,
                id_usuario: idUsuario
            })
        });

        alert("✔ Usuario creado y registrado en habitación exitosamente");

        // Reset
        mostrarRegistroNuevo = false;
        nuevo = {
            primer_nombre: "",
            primer_apellido: "",
            correo: "",
            username: "",
            password: "",
            numero_documento: "",
            lugar_expedicion: ""
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
            `https://inntech-backend.onrender.com/documentos/buscar_usuario_por_documento/${numeroDocumento}`
        );

        const data = await res.json();

        if (!res.ok) {
            errorUsuario = data.detail || "Usuario no encontrado";
            cargandoUsuario = false;
            return;
        }

        usuarioEncontrado = data.data; // contiene id_usuario
        cargandoUsuario = false;

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
        id_usuario: usuarioEncontrado.id_usuario
    };

    const res = await fetch("https://inntech-backend.onrender.com/usuarios_habitaciones/create_usuario_habitacion", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(body)
    });

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


</script>

<Navbar />
<NavbarA />

<section class="main">
  <div class="welcome-box">
    <h2 class="title">
      Gestión de <span class="highlight">Check-In</span>
    </h2>
    <p class="role">Buscar una reserva por ID.</p>
  </div>

  <!-- FORMULARIO -->
  <div class="form-box">
    <h3>Buscar Reserva</h3>

    <div class="form-grid">
      <input placeholder="ID de la reserva" bind:value={idReserva} />
    </div>

    <button class="btn-save" on:click={buscarReserva}>Buscar</button>
  </div>

  {#if mensaje}
    <div class="msg {estado}">
      {mensaje}
    </div>
  {/if}

  <!-- TABLA DE HABITACIONES -->
  {#if estado === "ok" && habitaciones.length > 0}
    <div class="tabla-box">
      <h3>Habitaciones asociadas</h3>

      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Número</th>
            <th>Capacidad</th>
            <th>Ocupados</th>
            <th>Acción</th>
          </tr>
        </thead>

        <tbody>
          {#each habitaciones as h}
            <tr>
              <td>{h.id_habitacion}</td>
              <td>{h.numero}</td>
              <td>{h.capacidad_max}</td>
              <td>{h.ocupados}</td>
              
              <td>
                {#if h.disponible}
                  <button class="btn-accion" on:click={() => abrirMenu(h)}>
                ➕ Agregar usuario
                </button>

                {#if menuAbierto === h.id_habitacion}
                <div class="mini-menu">
                    <button on:click={() => opcionUsuario('registrado')}>
                    👤 Usuario registrado
                    </button>

                    <button on:click={() => opcionUsuario('nuevo')}>
                    🆕 Usuario no registrado
                    </button>
                </div>
                {/if}

                {:else}
                  <span style="color: red; font-weight: bold;">
                    Hab. llena
                  </span>
                {/if}
              </td>

            </tr>
          {/each}
        </tbody>
      </table>
    </div>
  {/if}

  {#if mostrarInputDocumento}
<div class="registro-box">
    <h3>Agregar Usuario Registrado</h3>

    <input
        placeholder="Número de documento"
        bind:value={numeroDocumento}
    />

    <button class="btn-save" on:click={buscarUsuarioPorDocumento}>
        Buscar usuario
    </button>

    {#if cargandoUsuario}
        <p>Buscando...</p>
    {/if}

    {#if errorUsuario}
        <p style="color: red">{errorUsuario}</p>
    {/if}

    {#if usuarioEncontrado}
        <p style="color: lightgreen">
            Usuario encontrado: {usuarioEncontrado.nombre_completo}
        </p>

        <button class="btn-save" on:click={registrarUsuarioEnHabitacion}>
            Registrar en habitación
        </button>
    {/if}
</div>
{/if}

{#if mostrarRegistroNuevo}
<div class="registro-box">
    <h3>Registrar Usuario Nuevo</h3>

    <input placeholder="Primer nombre" bind:value={nuevo.primer_nombre} />
    <input placeholder="Primer apellido" bind:value={nuevo.primer_apellido} />
    <input placeholder="Correo" type="email" bind:value={nuevo.correo} />
    <input placeholder="Username" bind:value={nuevo.username} />
    <input placeholder="Password" type="password" bind:value={nuevo.password} />

    <input placeholder="Número de documento" bind:value={nuevo.numero_documento} />
    <input placeholder="Lugar de expedición" bind:value={nuevo.lugar_expedicion} />

    {#if errorNuevo}
        <p style="color:red">{errorNuevo}</p>
    {/if}

    {#if cargandoNuevo}
        <p>Registrando...</p>
    {:else}
        <button class="btn-save" on:click={registrarNuevoUsuario}>
            Crear usuario y registrar
        </button>
    {/if}
</div>
{/if}

</section>

<Footer />

<style>
  .main {
    padding: 40px;
    background: #000;
    min-height: calc(100vh - 160px);
    color: white;
  }

  .form-box {
    background: rgba(255, 165, 0, 0.1);
    padding: 20px;
    border: 2px solid orange;
    margin-bottom: 25px;
    border-radius: 10px;
    max-width: 500px;
    margin: auto;
  }

  input {
    padding: 8px;
    border-radius: 6px;
    background: #111;
    border: 1px solid #444;
    color: white;
  }

  .btn-save {
    background: orange;
    border: none;
    padding: 10px;
    border-radius: 6px;
    font-weight: bold;
    width: 100%;
  }

  .tabla-box {
    margin-top: 25px;
    background: rgba(255, 165, 0, 0.1);
    padding: 20px;
    border: 2px solid orange;
    border-radius: 10px;
    max-width: 800px;
    margin: auto;
  }

  table {
    width: 100%;
    border-collapse: collapse;
    color: white;
  }

  th, td {
    padding: 10px;
    border-bottom: 1px solid #444;
  }

  th {
    background: #222;
  }

  .btn-accion {
    background: #ff9800;
    padding: 6px 12px;
    border-radius: 6px;
    border: none;
    cursor: pointer;
    font-weight: bold;
  }
</style>