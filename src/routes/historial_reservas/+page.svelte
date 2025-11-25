<script>
  import Navbar from "$lib/components/Navbar.svelte";
  import NavbarA from "$lib/components/NavbarA.svelte";
  import Footer from "$lib/components/Footer.svelte";
  import { onMount } from "svelte";

  let user = null;
  let fullName = "";
  let rol = "";
  let historial = [];
  let error = "";
  let isLoading = true;
  // Estado para controlar cuando el historial ha sido cargado y la animación debe iniciar
  let isLoaded = false;

  // ===== Cargar usuario desde localStorage =====

  onMount(async () => {
    if (typeof localStorage !== "undefined") {
      const storedUser = localStorage.getItem("user");
      if (storedUser) {
        user = JSON.parse(storedUser);
        // Aseguramos que los nombres no sean nulos para evitar errores al concatenar
        const primer_nombre = user.primer_nombre || "";
        const segundo_nombre = user.segundo_nombre
          ? ` ${user.segundo_nombre}`
          : "";
        const primer_apellido = user.primer_apellido || "";
        const segundo_apellido = user.segundo_apellido
          ? ` ${user.segundo_apellido}`
          : "";

        fullName =
          `${primer_nombre}${segundo_nombre} ${primer_apellido}${segundo_apellido}`
            .replace(/\s+/g, " ")
            .trim();
        rol = user.rol;

        await cargarHistorial();
      } else {
        error = "No se encontró información de usuario.";
      }
    }
    isLoading = false;
  });

  // ===== Obtener historial =====

  async function cargarHistorial() {
    if (!user) return;

    try {
      // Usar retardo con backoff para la API (mismo patrón que NavbarA)
      const MAX_RETRIES = 3;
      for (let i = 0; i < MAX_RETRIES; i++) {
        const res = await fetch(
          `https://inntech-backend.onrender.com/reservas/terminadas/${user.id_usuario}`,
        );
        const data = await res.json();

        if (res.ok) {
          historial = data.data;
          if (historial.length === 0) {
            error = "No tienes reservas finalizadas.";
          }
          // MARCA: Carga completa, iniciar animación
          isLoaded = true;
          return;
        }

        if (i < MAX_RETRIES - 1) {
          await new Promise((resolve) =>
            setTimeout(resolve, Math.pow(2, i) * 1000),
          );
        } else {
          error =
            data.detail ??
            "Error al cargar historial después de varios intentos.";
        }
      }
    } catch (e) {
      error = "Error de conexión al servidor.";
    }
  }
</script>

<Navbar />

<NavbarA />

<section class="main">
  <h1 class="page-title">Historial de Reservas</h1>

  {#if isLoading}
    <!-- Indicador de Carga Sutil -->
    <div class="loading-container-bar">
      <div class="loading-bar"></div>
    </div>
  {:else if error && historial.length === 0}
    <p class="error">{error}</p>
  {:else if historial.length > 0}
    <!-- Clase animada solo si isLoaded es true -->
    <div class="table-wrap">
      <table class="tabla" class:fade-in={isLoaded}>
        <thead>
          <tr>
            <th>ID Reserva</th>
            <th>Fecha Inicio</th>
            <th>Fecha Fin</th>
            <th>Estado</th>
          </tr>
        </thead>

        <tbody>
          <!-- USO: Usamos index (i) para el delay de la animación -->
          {#each historial as item, i}
            <tr style="animation-delay: {i * 0.05}s;">
              <td class="highlight">{item.id_reserva}</td>
              <td>{item.date_start.substring(0, 10)}</td>
              <td>{item.date_end.substring(0, 10)}</td>
              <td>
                <!-- Estilo dinámico para el estado -->
                <span
                  class:estado-terminado={item.estado === "terminada"}
                  class:estado-cancelado={item.estado === "cancelada"}
                >
                  {item.estado}
                </span>
              </td>
            </tr>
          {/each}
        </tbody>
      </table>
    </div>
  {/if}
</section>

<Footer />

<style>
  /* >>> CRITICAL FIX: GLOBAL RESET <<< */
  html,
  body {
    margin: 0;
    padding: 0;
    box-sizing: border-box; /* Asegura que padding/border no cause overflow */
    width: 100%;
    /* Previene el scroll horizontal de la página entera por desbordamiento */
    overflow-x: hidden;
  }

  *,
  *::before,
  *::after {
    box-sizing: inherit;
  }

  /* ---------------------------------- */
  /* Paleta de Colores (Diseño Dark/Moderno) */
  /* ---------------------------------- */

  :root {
    --color-bg-primary: #1a202c; /* Azul oscuro casi negro */
    --color-card-bg: #1e293b; /* Gris oscuro para la tarjeta */
    --color-table-bg: #111827; /* Fondo de tabla muy oscuro */
    --color-accent: #fcd34d; /* Amarillo primario (mantenido) */
    --color-text: #e5e7eb; /* Texto blanco suave */
    --color-border: #374151; /* Borde gris sutil */
    --color-highlight: #60a5fa; /* Azul suave para IDs y destacados */
    --color-success: #10b981; /* Verde para estado Terminado */
    --color-danger: #ef4444; /* Rojo para estado Cancelado */
    --color-bg-light: #1e293b; /* Fondo para la barra de carga */

    /* Variables de color para el Scrollbar */
    --scrollbar-track: #2d3748;
    --scrollbar-thumb: #4a5568;
    --scrollbar-thumb-hover: #6b7280;
  }

  /* ---------------------------------- */
  /* DISEÑO DEL SCROLLBAR (solo WebKit) */
  /* ---------------------------------- */
  .table-wrap::-webkit-scrollbar {
    width: 8px;
    height: 8px;
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

  /* ---------------------------------- */
  /* Animaciones */
  /* ---------------------------------- */

  /* Animación de entrada de la tabla completa */
  @keyframes fadeIn {
    from {
      opacity: 0;
      transform: translateY(10px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  /* Animación de entrada escalonada para cada fila */
  @keyframes slideIn {
    from {
      opacity: 0;
      transform: translateY(10px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  /* Aplica la animación a la tabla contenedora */
  .tabla.fade-in {
    animation: fadeIn 0.5s ease-out forwards;
  }

  /* Aplica la animación a cada fila con delay */
  .tabla tbody tr {
    opacity: 0; /* Inicia invisible */
    animation: slideIn 0.3s ease-out forwards;
    animation-fill-mode: forwards; /* Mantiene el estado final (opacity: 1) */
  }

  /* ---------------------------------- */
  /* Indicador de Carga (Similar a NavbarA) */
  /* ---------------------------------- */
  .loading-container-bar {
    width: 80%;
    max-width: 600px;
    margin: 20px auto;
    display: flex;
    justify-content: center;
    padding: 5px 0;
  }

  .loading-bar {
    width: 100%;
    height: 4px;
    background: var(--color-bg-light);
    border-radius: 5px;
    overflow: hidden;
  }

  .loading-bar::before {
    content: "";
    display: block;
    height: 100%;
    width: 30%;
    background: linear-gradient(
      90deg,
      var(--color-primary),
      var(--color-accent)
    );
    animation: loading-move 1.5s infinite linear;
  }

  @keyframes loading-move {
    0% {
      transform: translateX(-100%);
    }
    100% {
      transform: translateX(333%);
    }
  }

  /* ---------------------------------- */
  /* Estilos principales de la sección */
  /* ---------------------------------- */

  .main {
    /* El 100vh ahora es seguro gracias al reset */
    min-height: calc(100vh - 160px);
    padding: 60px 20px;
    background: var(--color-bg-primary);
    color: var(--color-text);
    font-family: "Inter", sans-serif;
  }

  .page-title {
    text-align: center;
    font-size: clamp(2rem, 4vw, 2.8rem);
    color: var(--color-accent);
    margin-bottom: 30px;
    font-weight: 800;
  }

  .error {
    color: var(--color-danger);
    text-align: center;
    font-weight: 600;
    margin-top: 20px;
  }

  /* Tabla de Historial (Diseño Responsivo y Elegante) */
  .table-wrap {
    overflow-x: auto;
    /* Scroll vertical es para esta caja, que está contenida */
    max-height: 60vh;
    overflow-y: auto;
    border: 1px solid #334155; /* Este es el borde explícito alrededor del área de scroll */
    border-radius: 8px;
    margin: 0 auto; /* Centrar el contenedor de scroll */
    max-width: 1000px; /* Limita el ancho del contenedor */
  }

  .tabla {
    /* Eliminamos overflow-x y overflow-y redundantes */
    /* width: 100% será 100% del table-wrap, lo cual está bien */
    width: 100%;
    /* Mantenemos el min-width solo para asegurar que la tabla tenga ancho suficiente y FUERZE el scroll-x en table-wrap cuando la pantalla es pequeña. */
    min-width: 700px;
    border-collapse: separate;
    border-spacing: 0;
    background: var(--color-table-bg);
    border-radius: 12px;
    /* Mantenemos overflow: hidden para que los bordes redondeados se vean bien */
    overflow: hidden;

    /* ELIMINADO: position: sticky, top: 0, z-index: 10, ya que esto debe ir en TH */
  }

  th {
    padding: 18px 20px;
    background: var(--color-primary); /* Fondo del encabezado */
    color: var(--color-text);
    font-size: 1rem;
    font-weight: 700;
    text-align: left;
    /* Esto es lo que hace que la cabecera se quede fija dentro de .table-wrap */
    position: sticky;
    top: 0;
    z-index: 10;
    text-transform: uppercase;
  }

  td {
    padding: 15px 20px;
    color: var(--color-text);
    border-bottom: 1px solid var(--color-border);
    font-size: 0.95rem;
  }

  tbody tr {
    transition: background 0.3s ease;
  }

  tbody tr:last-child td {
    border-bottom: none;
  }

  tbody tr:hover {
    background: var(--color-card-bg); /* Fondo en hover sutil */
  }

  /* Estilos de Contenido Adicional */
  .highlight {
    color: var(--color-accent);
    font-weight: 600;
  }

  .estado-terminado {
    background-color: var(--color-success);
    color: var(--color-table-bg);
    padding: 4px 10px;
    border-radius: 4px;
    font-weight: 700;
    text-transform: uppercase;
    font-size: 0.8rem;
  }

  .estado-cancelado {
    background-color: var(--color-danger);
    color: white;
    padding: 4px 10px;
    border-radius: 4px;
    font-weight: 700;
    text-transform: uppercase;
    font-size: 0.8rem;
  }
</style>
