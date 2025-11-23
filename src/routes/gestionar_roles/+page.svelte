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
    estado: 1
  };

  // Modal asignar
  let showCreateModal = false;
  let showAssignModal = false;
  let currentRole = null; // rol seleccionado para asignar módulos
  let selectedModuloToAssign = null;
  let assigningBusy = false;

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

  // -------------------------
  // CARGAS
  // -------------------------
  async function cargarRoles() {
    cargandoRoles = true;
    errorMsg = "";
    try {
      const res = await fetch(R_GET_ROLES);
      const data = await res.json();
      roles = data.data || [];
    } catch (err) {
      console.error("Error cargando roles:", err);
      errorMsg = "Error cargando roles (ver consola).";
    } finally {
      cargandoRoles = false;
    }
  }

  async function cargarModulos() {
    cargandoModulos = true;
    try {
      const res = await fetch(R_GET_MODULOS);
      const data = await res.json();
      modulos = data.data || [];
    } catch (err) {
      console.error("Error cargando módulos:", err);
      modulos = [];
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
      assignedModules = data.data || []; // tu backend devuelve lista aunque esté vacía
    } catch (err) {
      console.error("Error cargando módulos asignados:", err);
      assignedModules = [];
    } finally {
      cargandoAsignados = false;
    }
  }

  // -------------------------
  // ACCIONES SOBRE ROLES
  // -------------------------
  async function crearRol() {
    if (!nuevoRol.nombre.trim()) {
      alert("El nombre del rol es obligatorio.");
      return;
    }

    creandoRol = true;
    try {
      const res = await fetch(R_CREATE_ROL, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(nuevoRol)
      });
      const data = await res.json();

      if (!res.ok) {
        console.error("Error crear rol:", data);
        alert(data.detail || data.message || "Error al crear rol");
      } else {
        // res contiene id_rol
        showCreateModal = false;
        limpiarNuevoRol();
        await cargarRoles();
        // abrir modal de asignación para el rol creado?
        if (data.id_rol) {
          // opcional: abrir asignación automática
          // abrirModalAsignar(data.id_rol, nuevoRol.nombre);
        }
      }
    } catch (err) {
      console.error("Error creando rol:", err);
      alert("Error creando rol (ver consola).");
    } finally {
      creandoRol = false;
    }
  }

  function limpiarNuevoRol() {
    nuevoRol = {
      nombre: "",
      descripcion: "",
      estado: 1
    };
  }

  async function toggleEstadoRol(rol) {
    const confirmMsg = rol.estado === 1
      ? `¿Desactivar rol "${rol.nombre}"?`
      : `¿Activar rol "${rol.nombre}"?`;
    if (!confirm(confirmMsg)) return;

    const url = rol.estado === 1
      ? `${R_DESACTIVAR_ROL}/${rol.id_rol}`
      : `${R_ACTIVAR_ROL}/${rol.id_rol}`;

    try {
      const res = await fetch(url, { method: "PUT" }); // tus rutas usan PUT
      const data = await res.json();
      if (!res.ok) {
        console.error("Error al cambiar estado rol:", data);
        alert(data.detail || data.message || "Error al cambiar estado");
      } else {
        // refrescar
        await cargarRoles();
      }
    } catch (err) {
      console.error("Error toggleEstadoRol:", err);
      alert("Error al cambiar estado (ver consola).");
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

  async function asignarModulo() {
    if (!currentRole) return;
    if (!selectedModuloToAssign) {
      alert("Selecciona un módulo para asignar.");
      return;
    }

    if (!confirm(`Asignar módulo al rol "${currentRole.nombre}"?`)) return;

    assigningBusy = true;
    try {
      const payload = {
        id_modulo: +selectedModuloToAssign,
        id_rol: +currentRole.id_rol,
        estado: 1
      };

      const res = await fetch(R_CREATE_MODULOROL, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(payload)
      });

      const data = await res.json();
      if (!res.ok) {
        console.error("Error asignando módulo:", data);
        alert(data.detail || data.message || "Error al asignar módulo");
      } else {
        // recargar lista asignada (tu endpoint devuelve lista vacía si no hay)
        await cargarModulosAsignados(currentRole.id_rol);
        // opcional: dejar seleccionado otro módulo (o vaciar)
        selectedModuloToAssign = null;
      }
    } catch (err) {
      console.error("Error en asignarModulo:", err);
      alert("Error al asignar módulo (ver consola).");
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

<Navbar />
<NavbarA />

<section class="main">
  <div class="welcome-box">
    <h2 class="title">Gestión de <span class="highlight">Roles</span></h2>
    <p class="role">Administra roles, activa/desactiva y asigna módulos.</p>
  </div>

  <!-- Botones -->
  <div class="top-actions">
    <button class="btn-create" on:click={() => (showCreateModal = !showCreateModal)}>
      {showCreateModal ? "Cerrar" : "Crear Rol"}
    </button>
    <button class="btn-refresh" on:click={() => cargarRoles()}>
      {cargandoRoles ? "Cargando..." : "Refrescar"}
    </button>
  </div>

  {#if errorMsg}
    <p class="error">{errorMsg}</p>
  {/if}

  <!-- Modal crear rol -->
  {#if showCreateModal}
    <div class="modal-backdrop" on:click={() => (showCreateModal = false)}></div>
    <div class="modal">
      <h3>Crear nuevo Rol</h3>

      <div class="form-grid modal-grid">
        <input placeholder="Nombre" bind:value={nuevoRol.nombre} />
        <input placeholder="Descripción" bind:value={nuevoRol.descripcion} />
        <select bind:value={nuevoRol.estado}>
          <option value="1">Activo</option>
          <option value="0">Inactivo</option>
        </select>
      </div>

      <div class="modal-actions">
        <button class="btn-save" on:click={crearRol} disabled={creandoRol}>
          {creandoRol ? "Creando..." : "Crear"}
        </button>
        <button class="btn-cancel" on:click={() => (showCreateModal = false)}>Cancelar</button>
      </div>
    </div>
  {/if}

  <!-- Tabla roles -->
  <div class="table-wrap">
    <table class="tabla">
      <thead>
        <tr>
          <th>ID</th>
          <th>Nombre</th>
          <th>Descripción</th>
          <th>Estado</th>
          <th>Acciones</th>
        </tr>
      </thead>

      <tbody>
        {#if cargandoRoles}
          <tr><td colspan="5">Cargando roles...</td></tr>
        {:else if roles.length === 0}
          <tr><td colspan="5">No hay roles registrados.</td></tr>
        {:else}
          {#each roles as r}
            <tr>
              <td>{r.id_rol}</td>
              <td>{r.nombre}</td>
              <td>{r.descripcion}</td>
              <td>
                <span class={r.estado === 1 ? "badge-active" : "badge-inactive"}>
                  {r.estado === 1 ? "Activo" : "Inactivo"}
                </span>
              </td>
              <td>
                <button class="btn-toggle" on:click={() => toggleEstadoRol(r)}>
                  {r.estado === 1 ? "Desactivar" : "Activar"}
                </button>

                <button class="btn-assign" on:click={() => abrirModalAsignar(r)}>
                  Asignar módulos
                </button>
              </td>
            </tr>
          {/each}
        {/if}
      </tbody>
    </table>
  </div>

  <!-- Modal asignar módulos -->
  {#if showAssignModal && currentRole}
    <div class="modal-backdrop" on:click={() => (showAssignModal = false)}></div>

    <div class="modal large">
      <h3>Asignar módulos a: <span class="highlight">{currentRole.nombre}</span></h3>

      <div class="assign-row">
        <div class="assign-left">
          <label>Seleccionar módulo</label>
          {#if cargandoModulos}
            <p>Cargando módulos...</p>
          {:else if modulos.length === 0}
            <p>No hay módulos disponibles.</p>
          {:else}
            <select bind:value={selectedModuloToAssign}>
              <option value={null}>-- Selecciona un módulo --</option>
              {#each modulos as m}
                <option value={m.id_modulo}>{m.nombre} — {m.ruta}</option>
              {/each}
            </select>

            <div style="margin-top:10px;">
              <button class="btn-save" on:click={asignarModulo} disabled={assigningBusy}>
                {assigningBusy ? "Asignando..." : "Asignar módulo"}
              </button>
            </div>
          {/if}
        </div>

        <div class="assign-right">
          <label>Módulos asignados</label>
          {#if cargandoAsignados}
            <p>Cargando módulos asignados...</p>
          {:else if assignedModules.length === 0}
            <p>Este rol aún no tiene módulos asignados.</p>
          {:else}
            <table class="tabla-small">
              <thead>
                <tr><th>ID</th><th>Nombre</th><th>Ruta</th></tr>
              </thead>
              <tbody>
                {#each assignedModules as am}
                  <tr>
                    <td>{am.id_modulo ?? am.id_mxr}</td>
                    <td>{am.nombre}</td>
                    <td>{am.ruta}</td>
                  </tr>
                {/each}
              </tbody>
            </table>
          {/if}
        </div>
      </div>

      <div class="modal-actions">
        <button class="btn-close" on:click={() => (showAssignModal = false)}>Cerrar</button>
      </div>
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

  .welcome-box {
    margin-bottom: 20px;
    text-align: center;
  }

  .top-actions {
    display:flex;
    gap:10px;
    margin-bottom: 14px;
  }

  .table-wrap {
    overflow-x:auto;
  }

  table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 10px;
  }

  table th, table td {
    border: 1px solid #444;
    padding: 10px;
    text-align: left;
  }

  .tabla-small th, .tabla-small td {
    font-size: 0.9rem;
    padding: 6px;
  }

  .btn-create, .btn-refresh, .btn-save, .btn-toggle, .btn-assign, .btn-close {
    background: orange;
    border: none;
    padding: 8px 12px;
    color: black;
    cursor: pointer;
    border-radius:6px;
    font-weight: 700;
    margin-right:6px;
  }

  .btn-cancel {
    background: #444;
    color: white;
    border: none;
    padding: 8px 12px;
    border-radius: 6px;
    cursor: pointer;
  }

  .badge-active {
    background: #36c937;
    padding: 6px 8px;
    border-radius: 6px;
    color: black;
    font-weight: 700;
  }

  .badge-inactive {
    background: #c94b4b;
    padding: 6px 8px;
    border-radius: 6px;
    color: black;
    font-weight: 700;
  }

  /* Modal */
  .modal-backdrop {
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,0.6);
    z-index: 40;
  }

  .modal {
    position: fixed;
    z-index: 50;
    left: 50%;
    transform: translateX(-50%);
    top: 12%;
    background: #0b0b0b;
    padding: 18px;
    border-radius: 8px;
    border: 2px solid orange;
    width: 90%;
    max-width: 720px;
    box-shadow: 0 8px 30px rgba(0,0,0,0.6);
  }

  .modal.large {
    max-width: 1000px;
    top: 8%;
  }

  .modal h3 {
    color: orange;
  }

  .form-grid {
    display: grid;
    gap: 8px;
    margin-top: 10px;
  }

  .modal-grid {
    grid-template-columns: repeat(2, 1fr);
  }

  input, select {
    padding: 8px;
    border-radius: 6px;
    border: 1px solid #444;
    background: #111;
    color: white;
    width: 100%;
  }

  .modal-actions {
    margin-top: 12px;
    display:flex;
    justify-content:flex-end;
    gap:8px;
  }

  .assign-row {
    display:flex;
    gap:20px;
    margin-top:10px;
  }

  .assign-left, .assign-right {
    flex:1;
    min-width: 260px;
  }

  .highlight { color: white; font-weight: 800; color: orange; }
  .error { color: #ff7676; margin-top: 8px; }
</style>