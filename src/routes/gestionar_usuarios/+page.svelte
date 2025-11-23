<script>
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";

  let usuarios = [];
  let roles = [];

  let nuevoUsuario = {
    id_rol: 3,
    primer_nombre: "",
    segundo_nombre: "",
    primer_apellido: "",
    segundo_apellido: "",
    telefono: "",
    correo: "",
    username: "",
    password: "",
    estado: 1
  };

  let creando = false;
 
  // -------------------------------------
  // CARGAR ROLES DESDE LA BD
  // -------------------------------------
  async function cargarRoles() {
    try {
      const res = await fetch(
        "https://inntech-backend.onrender.com/roles/combo"
      ); // ← GET
      const data = await res.json();
      roles = data.data || [];
    } catch (err) {
      console.error("Error cargando roles:", err);
    }
  }

  // -------------------------------------
  // CARGAR USUARIOS
  // -------------------------------------
  async function cargarUsuarios() {
    try {
      const res = await fetch(
        "https://inntech-backend.onrender.com/usuarios/get_usuarios"
      ); // ← GET
      const data = await res.json();
      usuarios = data.data || [];
    } catch (err) {
      console.error("Error cargando usuarios:", err);
    }
  }

  // -------------------------------------
  // ACTIVAR / DESACTIVAR USUARIO
  // -------------------------------------
  async function toggleEstado(usuario) {
    try {
      const url =
        usuario.estado === 1
          ? `https://inntech-backend.onrender.com/usuarios/desactivar/${usuario.id_usuario}` // ← PATCH
          : `https://inntech-backend.onrender.com/usuarios/activar/${usuario.id_usuario}`; // ← PATCH

      await fetch(url, {
        method: "PUT"
      });

      cargarUsuarios();
    } catch (err) {
      console.error("Error al cambiar estado:", err);
    }
  }

  // -------------------------------------
  // CAMBIAR ROL DE USUARIO
  // -------------------------------------
  async function cambiarRol(usuario, nuevoRol) {
    try {
      await fetch(
        `https://inntech-backend.onrender.com/usuarios/cambiar-rol/${usuario.id_usuario}/${nuevoRol}`,
        {
          method: "PUT"
        }
      );

      cargarUsuarios();
    } catch (err) {
      console.error("Error al cambiar rol:", err);
    }
  }

  // -------------------------------------
  // CREAR NUEVO USUARIO
  // -------------------------------------
  async function crearUsuario() {
    try {
      await fetch("https://inntech-backend.onrender.com/auth/register", {
        // ← POST
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(nuevoUsuario)
      });

      creando = false;
      limpiarNuevo();
      cargarUsuarios();
    } catch (err) {
      console.error("Error al crear usuario:", err);
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
      estado: 1
    };
  }

  cargarRoles();
  cargarUsuarios();
</script>

<Navbar />
<NavbarA />

<section class="main">
  <div class="welcome-box">
    <h2 class="title">
      Gestión de <span class="highlight">Usuarios</span>
    </h2>
    <p class="role">Administra cuentas, roles y estados.</p>
  </div>

  <!-- BOTÓN CREAR -->
  <button class="btn-create" on:click={() => (creando = !creando)}>
    {creando ? "Cerrar" : "Crear Usuario"}
  </button>

  {#if creando}
    <div class="form-box">
      <h3>Nuevo Usuario</h3>

      <div class="form-grid">
        <input placeholder="Primer nombre" bind:value={nuevoUsuario.primer_nombre} />
        <input placeholder="Segundo nombre" bind:value={nuevoUsuario.segundo_nombre} />
        <input placeholder="Primer apellido" bind:value={nuevoUsuario.primer_apellido} />
        <input placeholder="Segundo apellido" bind:value={nuevoUsuario.segundo_apellido} />
        <input placeholder="Teléfono" bind:value={nuevoUsuario.telefono} />
        <input placeholder="Correo" bind:value={nuevoUsuario.correo} />
        <input placeholder="Username" bind:value={nuevoUsuario.username} />
        <input type="password" placeholder="Contraseña" bind:value={nuevoUsuario.password} />

        <select bind:value={nuevoUsuario.id_rol}>
          {#each roles as r}
            <option value={r.id_rol}>{r.nombre}</option>
          {/each}
        </select>
      </div>

      <button class="btn-save" on:click={crearUsuario}>Guardar</button>
    </div>
  {/if}

  <!-- TABLA DE USUARIOS -->
  <table class="tabla">
    <thead>
      <tr>
        <th>ID</th>
        <th>Nombre completo</th>
        <th>Correo</th>
        <th>Username</th>
        <th>Rol</th>
        <th>Estado</th>
        <th>Acciones</th>
      </tr>
    </thead>

    <tbody>
      {#each usuarios as u}
        <tr>
          <td>{u.id_usuario}</td>
          <td>
            {u.primer_nombre} {u.segundo_nombre} {u.primer_apellido} {u.segundo_apellido}
          </td>
          <td>{u.correo}</td>
          <td>{u.username}</td>

          <td>
            <select bind:value={u.id_rol} on:change={(e) => cambiarRol(u, e.target.value)}>
              {#each roles as r}
                <option value={r.id_rol}>{r.nombre}</option>
              {/each}
            </select>
          </td>

          <td>{u.estado === 1 ? "Activo" : "Inactivo"}</td>

          <td>
            <button class="btn-toggle" on:click={() => toggleEstado(u)}>
              {u.estado === 1 ? "Desactivar" : "Activar"}
            </button>
          </td>
        </tr>
      {/each}
    </tbody>
  </table>
</section>

<Footer />

<style>
  .main {
    padding: 40px;
    background: #000;
    min-height: calc(100vh - 160px);
    color: white;
  }

  .welcome-box {
    margin-bottom: 20px;
    text-align: center;
  }

  table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 25px;
  }

  table th,
  table td {
    border: 1px solid #444;
    padding: 10px;
    text-align: center;
  }

  .btn-create,
  .btn-save,
  .btn-toggle {
    background: orange;
    border: none;
    padding: 10px 15px;
    color: black;
    cursor: pointer;
    margin-bottom: 10px;
    border-radius: 6px;
    font-weight: bold;
  }

  .form-box {
    background: rgba(255, 165, 0, 0.1);
    padding: 20px;
    border: 2px solid orange;
    margin-bottom: 25px;
    border-radius: 10px;
  }

  .form-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 10px;
  }

  input,
  select {
    padding: 8px;
    border-radius: 6px;
    border: 1px solid #444;
    background: #111;
    color: white;
  }
</style>