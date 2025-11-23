<script>
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";
  import { onMount } from "svelte";

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

  let mensaje = "";
  let error = "";

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

  async function cargarTiposDoc() {
    try {
      const res = await fetch("https://inntech-backend.onrender.com/tipos_documento/get_tipos_documento");
      const data = await res.json();
      if (res.ok) tiposDoc = data.data;
    } catch (e) {
      console.error(e);
    }
  }

  async function cargarDocumentos() {
    try {
      const res = await fetch("https://inntech-backend.onrender.com/documentos/get_documentos_completo");
      const data = await res.json();
      if (res.ok) {
        documentos = data.data.filter(d => d.id_usuario === user.id_usuario);
      }
    } catch (e) {
      console.error(e);
    }
  }

  // ============================
  // Actualizar usuario
  // ============================
  async function actualizarUsuario() {
    error = "";
    mensaje = "";
    try {
      const payload = {
        primer_nombre,
        segundo_nombre,
        primer_apellido,
        segundo_apellido,
        telefono
      };

      const res = await fetch(
        `https://inntech-backend.onrender.com/usuarios/update_usuario/${user.id_usuario}`,
        {
          method: "PUT",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(payload)
        }
      );

      const data = await res.json();
      if (!res.ok) {
        error = data.detail || "Error al actualizar usuario";
        return;
      }

      mensaje = "Datos actualizados correctamente";

      // actualizar el localstorage
      const updatedUser = { ...user, ...payload };
      localStorage.setItem("user", JSON.stringify(updatedUser));
      user = updatedUser;

    } catch (e) {
      error = "Error de conexión";
    }
  }

  // ============================
  // Crear documento
  // ============================
  async function crearDocumento() {
    error = "";
    mensaje = "";

    if (!id_tdocumento || !numero_documento || !lugar_expedicion) {
      error = "Todos los campos del documento son obligatorios";
      return;
    }

    try {
      const payload = {
        id_tdocumento: Number(id_tdocumento),
        id_usuario: user.id_usuario,
        numero_documento,
        lugar_expedicion,
        estado: estado_doc
      };

      const res = await fetch("https://inntech-backend.onrender.com/documentos/create_documento", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(payload)
      });

      const data = await res.json();

      if (!res.ok) {
        error = data.detail || "No se pudo crear el documento";
        return;
      }

      mensaje = "Documento creado con éxito";
      numero_documento = "";
      lugar_expedicion = "";
      id_tdocumento = "";

      await cargarDocumentos();

    } catch (e) {
      error = "Error de conexión";
    }
  }
</script>

<Navbar />
<NavbarA />

<section class="main">
  <h2 class="title">Editar Perfil</h2>

  {#if error}
    <p class="error">{error}</p>
  {/if}

  {#if mensaje}
    <p class="mensaje">{mensaje}</p>
  {/if}

  <!-- FORMULARIO DE USUARIO -->
  <div class="box">
    <h3 class="box-title">Datos del Usuario</h3>

    <div class="form-group">
      <label>Primer Nombre</label>
      <input bind:value={primer_nombre} type="text" />
    </div>

    <div class="form-group">
      <label>Segundo Nombre</label>
      <input bind:value={segundo_nombre} type="text" />
    </div>

    <div class="form-group">
      <label>Primer Apellido</label>
      <input bind:value={primer_apellido} type="text" />
    </div>

    <div class="form-group">
      <label>Segundo Apellido</label>
      <input bind:value={segundo_apellido} type="text" />
    </div>

    <div class="form-group">
      <label>Teléfono</label>
      <input bind:value={telefono} type="text" />
    </div>

    <button class="btn" on:click={actualizarUsuario}>Actualizar Datos</button>
  </div>

  <!-- FORMULARIO DOCUMENTO -->
  <div class="box">
    <h3 class="box-title">Registrar Documento</h3>

    <div class="form-group">
      <label>Tipo Documento</label>
      <select bind:value={id_tdocumento}>
        <option value="">Seleccione...</option>
        {#each tiposDoc as t}
          <option value={t.id_tdocumento}>{t.nombre}</option>
        {/each}
      </select>
    </div>

    <div class="form-group">
      <label>Número de Documento</label>
      <input bind:value={numero_documento} type="text" />
    </div>

    <div class="form-group">
      <label>Lugar de expedición</label>
      <input bind:value={lugar_expedicion} type="text" />
    </div>

    <button class="btn" on:click={crearDocumento}>Guardar Documento</button>
  </div>

  <!-- TABLA DOCUMENTOS -->
  <div class="box">
    <h3 class="box-title">Mis Documentos</h3>

    {#if documentos.length === 0}
      <p>No hay documentos registrados.</p>
    {:else}
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
              <td>{d.estado === 1 ? "Activo" : "Inactivo"}</td>
            </tr>
          {/each}
        </tbody>
      </table>
    {/if}
  </div>
</section>

<Footer />

<style>
  .main {
    min-height: calc(100vh - 160px);
    padding: 40px;
    color: white;
    background: #000;
  }

  .title {
    text-align: center;
    font-size: 2.3rem;
    color: orange;
    margin-bottom: 30px;
  }

  .box {
    background: rgba(255, 165, 0, 0.1);
    border: 2px solid orange;
    padding: 25px;
    margin-bottom: 35px;
    border-radius: 12px;
    box-shadow: 0 0 10px rgba(255,165,0,0.3);
  }

  .box-title {
    font-size: 1.4rem;
    color: orange;
    margin-bottom: 15px;
  }

  .form-group {
    margin-bottom: 15px;
    display: flex;
    flex-direction: column;
  }

  label {
        margin-bottom: 6px;
  }

  input, select {
    padding: 8px;
    border-radius: 6px;
    border: 1px solid #444;
    background: #111;
    color: white;
  }

  table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 15px;
  }

  th, td {
    border: 1px solid #555;
    padding: 10px;
    text-align: center;
  }

  th {
    background: orange;
    color: black;
    font-weight: bold;
  }

  .btn {
    background: orange;
    border: none;
    padding: 10px 15px;
    border-radius: 8px;
    cursor: pointer;
    margin-top: 10px;
    color: black;
    font-weight: bold;
  }

  .btn:hover {
    opacity: 0.8;
  }

  .error {
    color: red;
    text-align: center;
    margin-bottom: 10px;
  }

  .mensaje {
    color: lightgreen;
    text-align: center;
    margin-bottom: 10px;
  }
</style>
