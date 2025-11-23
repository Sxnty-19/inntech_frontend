<script>
  import { goto } from '$app/navigation';

  let user = null;
  let fullName = "";

  // Obtener usuario del localStorage
  if (typeof localStorage !== "undefined") {
    const storedUser = localStorage.getItem("user");
    if (storedUser) {
      user = JSON.parse(storedUser);

      fullName = `
        ${user.primer_nombre}
        ${user.segundo_nombre ?? ""}
        ${user.primer_apellido}
        ${user.segundo_apellido ?? ""}
      `.replace(/\s+/g, " ").trim();
    }
  }

  function cerrarSesion() {
    localStorage.clear();
    goto('/login', { replaceState: true });
  }

  function editarPerfil() {
    goto('/perfil');
  }

  function irPrincipal() {
    goto('/principal');
  }

  // Manejar "Enter" para accesibilidad
  function handleKeyDown(event) {
    if (event.key === 'Enter' || event.key === ' ') {
      irPrincipal();
    }
  }
</script>

<nav class="navbar">
  <!-- CORRECCIÓN: unir las clases en un solo atributo y añadir tabindex/role para accesibilidad -->
  <div class="nav-left clickable" on:click={irPrincipal} role="button" tabindex="0" on:keydown={handleKeyDown}>
    <img src="/logo.png" alt="logo" class="logo" />
    <h1 class="brand">Hotel InnTech</h1>
  </div>

  <div class="nav-right">
    {#if fullName}
      <span class="user-name">{fullName}</span>
    {/if}

    <button class="nav-btn" on:click={editarPerfil}>Editar perfil</button>
    <button class="nav-btn logout" on:click={cerrarSesion}>Cerrar sesión</button>
  </div>
</nav>

<style>
  .navbar {
    width: 100%;
    padding: 12px 25px;
    background: #111827;
    color: white;
    display: flex;
    align-items: center;
    justify-content: space-between;
    border-bottom: 1px solid rgba(255,255,255,0.1);
  }

  .nav-left {
    display: flex;
    align-items: center;
    gap: 12px;
    cursor: pointer;
  }

  /* clase adicional opcional */
  .clickable:focus {
    outline: 2px solid #ffbf47;
    outline-offset: 2px;
  }

  .nav-left:hover .brand {
    color: #ffbf47;
  }

  .logo {
    width: 40px;
    height: 40px;
  }

  .brand {
    font-size: 1.3rem;
    font-weight: 700;
    color: orange;
    transition: 0.2s;
  }

  .nav-right {
    display: flex;
    align-items: center;
    gap: 15px;
  }

  .user-name {
    margin-right: 10px;
    font-weight: 600;
    color: orange;
    font-size: 1rem;
  }

  .nav-btn {
    background: transparent;
    border: 1px solid orange;
    padding: 6px 12px;
    border-radius: 6px;
    color: orange;
    cursor: pointer;
    transition: .2s;
  }

  .nav-btn:hover {
    background: orange;
    color: black;
  }

  .logout {
    border-color: #ff4d4d;
    color: #ff4d4d;
  }

  .logout:hover {
    background: #ff4d4d;
    color: white;
  }
</style>