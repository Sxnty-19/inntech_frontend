<script>
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";

  // Tipos de documento
  let tipos = [];
  let creando = false;

  let nuevoTipo = {
    nombre: "",
    descripcion: ""
  };

  // Documentos
  let documentos = [];

  // ---------------------------------------------------
  // Cargar tipos de documento
  // ---------------------------------------------------
  async function cargarTipos() {
    try {
      const res = await fetch(
        "https://inntech-backend.onrender.com/tipos_documento/get_tipos_documento"
      );
      const data = await res.json();
      tipos = data.data || [];
    } catch (err) {
      console.error("Error cargando tipos:", err);
    }
  }

  // ---------------------------------------------------
  // Cargar documentos completos
  // ---------------------------------------------------
  async function cargarDocumentos() {
    try {
      const res = await fetch(
        "https://inntech-backend.onrender.com/documentos/get_documentos_completo"
      );
      const data = await res.json();
      documentos = data.data || [];
    } catch (err) {
      console.error("Error cargando documentos:", err);
    }
  }

  // ---------------------------------------------------
  // Crear tipo de documento
  // ---------------------------------------------------
  async function crearTipo() {
    try {
      await fetch(
        "https://inntech-backend.onrender.com/tipos_documento/create_tipo_documento",
        {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(nuevoTipo)
        }
      );

      limpiarNuevo();
      creando = false;
      cargarTipos();
    } catch (err) {
      console.error("Error al crear tipo:", err);
    }
  }

  function limpiarNuevo() {
    nuevoTipo = {
      nombre: "",
      descripcion: ""
    };
  }

  cargarTipos();
  cargarDocumentos();
</script>

<Navbar />
<NavbarA />

<section class="main">
  <div class="welcome-box">
    <h2 class="title">
      Gestión de <span class="highlight">Documentos</span>
    </h2>
    <p class="role">Administración de documentos y tipos.</p>
  </div>

  <!-- BOTÓN CREAR -->
  <button class="btn-create" on:click={() => (creando = !creando)}>
    {creando ? "Cerrar" : "Crear Tipo de Documento"}
  </button>

  <!-- FORMULARIO CREAR -->
  {#if creando}
    <div class="form-box">
      <h3>Nuevo Tipo de Documento</h3>

      <div class="form-grid">
        <input placeholder="Nombre" bind:value={nuevoTipo.nombre} />
        <input placeholder="Descripción" bind:value={nuevoTipo.descripcion} />
      </div>

      <button class="btn-save" on:click={crearTipo}>Guardar</button>
    </div>
  {/if}

  <!-- TABLA TIPOS -->
  <h3 style="margin-top: 40px;">Tipos de Documento</h3>

  <table class="tabla">
    <thead>
      <tr>
        <th>ID</th>
        <th>Nombre</th>
        <th>Descripción</th>
      </tr>
    </thead>

    <tbody>
      {#each tipos as t}
        <tr>
          <td>{t.id_tdocumento}</td>
          <td>{t.nombre}</td>
          <td>{t.descripcion}</td>
        </tr>
      {/each}
    </tbody>
  </table>

  <!-- TABLA DOCUMENTOS -->
  <h3 style="margin-top: 40px;">Documentos Registrados</h3>

  <table class="tabla">
    <thead>
      <tr>
        <th>ID</th>
        <th>Tipo</th>
        <th>Número</th>
        <th>Lugar de Expedición</th>
        <th>Usuario</th>
        <th>Estado</th>
        <th>Fecha Creado</th>
      </tr>
    </thead>

    <tbody>
      {#each documentos as d}
        <tr>
          <td>{d.id_documento}</td>
          <td>{d.tipo_documento}</td>
          <td>{d.numero_documento}</td>
          <td>{d.lugar_expedicion}</td>
          <td>{d.nombre_completo}</td>
          <td>{d.estado === 1 ? "Activo" : "Inactivo"}</td>
          <td>{d.date_created}</td>
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
  .btn-save {
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
    grid-template-columns: repeat(2, 1fr);
    gap: 10px;
  }

  input {
    padding: 8px;
    border-radius: 6px;
    border: 1px solid #444;
    background: #111;
    color: white;
  }
</style>