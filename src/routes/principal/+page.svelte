<script>
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";
  // Importante: La lógica de carga aquí no es reactiva, solo se ejecuta una vez.

  let user = null;
  let fullName = "Usuario"; // Valor por defecto seguro
  let rol = "";

  // Obtener info del LocalStorage
  // Usamos el check de 'typeof localStorage' para el renderizado inicial en el servidor (SSR)
  if (typeof localStorage !== "undefined") {
    const storedUser = localStorage.getItem("user");
    if (storedUser) {
      user = JSON.parse(storedUser);

      // Lógica limpia para construir el nombre completo
      fullName = `
        ${user.primer_nombre}
        ${user.segundo_nombre ?? ""}
        ${user.primer_apellido}
        ${user.segundo_apellido ?? ""}
      `
        .replace(/\s+/g, " ")
        .trim();

      rol = user.rol;
    }
  }

  // Fallback seguro: Si no se encontró el nombre completo, usamos el username
  const displayName =
    fullName && fullName !== "" ? fullName : (user?.username ?? "Invitado");
</script>

<!-- Las barras de navegación (sticky) se quedan arriba -->
<Navbar />
<NavbarA />

<section class="main-content">
  <!-- FIX CLAVE: Envuelve el contenido que usa 'user' en un bloque {#if user} -->
  {#if user}
    <div class="welcome-box">
      <h2 class="title">
        ¡Bienvenido(a)! <br /><span class="highlight">{displayName}</span>
      </h2>
      <div class="tip-box">
        <p>
          Utiliza la barra de "Módulos" (arriba) para navegar a las distintas
          secciones de gestión. Tu rol actual es: <strong>{rol}</strong>.
        </p>
      </div>
    </div>
  {:else}
    <!-- Si no hay usuario logueado (solo en caso de error o logout), muestra un mensaje alternativo -->
    <div class="no-user-message">
      <h2 class="title">Cargando...</h2>
      <p>
        Si esto tarda, por favor, <a href="/login" class="login-link"
          >inicia sesión</a
        >.
      </p>
    </div>
  {/if}
</section>

<Footer />

<style>
  /* ============================================== */
  /* Variables de color para coherencia */
  /* ============================================== */
  :root {
    --color-dark: #0f172a; /* Color para Navbars y Footer */
    --color-body-bg: #1a202c; /* Color de fondo del contenido principal */
    --color-secondary: #fcd34d;
    --color-primary: #1e3a8a;
    --color-text: #e2e8f0;
  }

  /* ============================================== */
  /* FIX GLOBAL AGRESIVO: CUBRE EL 100% DEL VIEWPORT */
  /* ============================================== */
  :global(html) {
    /* Mantiene el fondo oscuro en el nivel más alto (la ventana) */
    background: var(--color-dark);
    box-sizing: border-box;
    /* Asegura que el elemento raíz ocupe todo el espacio */
    width: 100%;
    height: 100%;
    /* FIX DE SCROLL HORIZONTAL */
    overflow-x: hidden;
  }

  :global(body) {
    /* Reset total de márgenes y paddings por defecto del navegador */
    margin: 0;
    padding: 0;
    /* Asegura que el body se estire para cubrir la altura del viewport */
    min-height: 100vh;
    width: 100%;
    /* Evita el scroll horizontal causado por elementos que exceden el ancho */
    overflow-x: hidden;
    /* FIX CLAVE: Definir la fuente principal para todo el cuerpo */
    font-family: "Inter", sans-serif;
  }

  /* Aplica el fondo ligeramente más claro para el contenido principal */
  .main-content {
    /* AJUSTE CLAVE en la altura mínima para cubrir el espacio entre Navbars y Footer */
    min-height: calc(100vh - 110px);
    flex-grow: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 50px 20px;
    background: var(
      --color-body-bg
    ); /* CAMBIO CLAVE: Fondo ligeramente más claro */
    color: white;
    text-align: center;
    /* La fuente se hereda ahora del body, pero la mantenemos por redundancia */
    /* font-family: "Inter", sans-serif; */
  }

  .welcome-box {
    margin: 30px;
    background: rgba(30, 58, 138, 0.2);
    padding: 30px 40px;
    border-radius: 12px;
    border: 2px solid var(--color-secondary);
    box-shadow:
      0 4px 20px rgba(0, 0, 0, 0.5),
      0 0 15px var(--color-secondary);
    max-width: 500px;
    width: 90%;
  }

  .tip-box {
    margin-top: 20px;
    padding: 10px;
    background: rgba(252, 211, 77, 0.1);
    border-left: 4px solid var(--color-secondary);
    color: var(--color-text);
    font-size: 0.9rem;
    text-align: left;
    border-radius: 0 4px 4px 0;
  }

  .title {
    font-size: clamp(1.8rem, 4vw, 2.5rem);
    color: var(--color-secondary);
    margin-bottom: 5px;
  }

  .highlight {
    color: white;
    font-weight: 800;
  }

  .no-user-message {
    color: var(--color-text);
  }
  .login-link {
    color: var(--color-secondary);
    text-decoration: underline;
  }
</style>
