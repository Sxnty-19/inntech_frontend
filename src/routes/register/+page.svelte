<script>
  import { goto } from "$app/navigation";

  // VARIABLES DE ESTADO
  let formData = {
    primer_nombre: "",
    segundo_nombre: "",
    primer_apellido: "",
    segundo_apellido: "",
    telefono: "",
    correo: "",
    username: "",
    password: "",
  };
  let confirmPassword = ""; // Campo adicional para confirmar la contraseña

  let inlineError = "";
  let isLoading = false;
  let passwordFieldType = "password";
  let confirmPasswordFieldType = "password";

  // Variables para el sistema de mensajes/modal
  let message = "";
  let isSuccess = false;
  let showMessage = false;

  // DEFINICIÓN DE CAMPOS
  // Cambio: Teléfono y Correo ahora usan span: 6 para ir uno al lado del otro en escritorio.
  let campos = [
    // Fila 1: Nombres
    { label: "Primer Nombre", key: "primer_nombre", required: true, span: 6 },
    { label: "Segundo Nombre", key: "segundo_nombre", span: 6 },

    // Fila 2: Apellidos
    {
      label: "Primer Apellido",
      key: "primer_apellido",
      required: true,
      span: 6,
    },
    { label: "Segundo Apellido", key: "segundo_apellido", span: 6 },

    // Fila 3: Contacto (Antes span: 12 cada uno)
    {
      label: "Teléfono",
      key: "telefono",
      type: "tel",
      required: true,
      span: 6,
    },
    {
      label: "Correo Electrónico",
      key: "correo",
      required: true,
      type: "email",
      span: 6,
    },

    // Fila 4: Usuario y Contraseñas
    { label: "Usuario", key: "username", required: true, span: 12 },
    {
      label: "Contraseña",
      key: "password",
      required: true,
      type: "password",
      span: 6,
    },
  ];
  // Se deja el campo de confirmación fuera del array de campos para manejar su validación separadamente

  // FUNCIONES UX
  function togglePasswordVisibility(field) {
    if (field === "password") {
      passwordFieldType =
        passwordFieldType === "password" ? "text" : "password";
    } else if (field === "confirm") {
      confirmPasswordFieldType =
        confirmPasswordFieldType === "password" ? "text" : "password";
    }
  }

  async function registrar() {
    inlineError = "";
    showMessage = false;

    // 1. VALIDACIÓN EN CLIENTE: Contraseñas deben coincidir
    if (formData.password !== confirmPassword) {
      inlineError = "La contraseña y su confirmación no coinciden.";
      return;
    }

    // 2. Bloquea el botón
    isLoading = true;

    try {
      const response = await fetch(
        "https://inntech-backend.onrender.com/auth/register",
        {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            // Nota: Asegúrate de que id_rol y estado sean los correctos para tu backend
            id_rol: 3,
            estado: 1,
            ...formData,
          }),
        },
      );

      const result = await response.json();

      if (!response.ok) {
        // Muestra error usando la notificación personalizada
        message = result.detail || "Error al registrar usuario.";
        isSuccess = false;
        showMessage = true;
        return;
      }

      // Éxito
      message = "¡Usuario registrado exitosamente! Redirigiendo al login.";
      isSuccess = true;
      showMessage = true;

      // Navegar después de mostrar el mensaje por un breve tiempo
      setTimeout(() => {
        goto("/login");
      }, 1500);
    } catch (error) {
      console.error("Error de conexión:", error);
      inlineError = "Error de conexión con el servidor.";
    } finally {
      // 3. Desbloquea el botón
      isLoading = false;
    }
  }

  function irLogin() {
    goto("/login");
  }
</script>

<!-- Se necesita Font Awesome para los iconos (Loading, Ojo) -->
<svelte:head>
  <link
    rel="stylesheet"
    href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css"
    xintegrity="sha512-SnH5WK+bZxgPHs44uWIX+LLMD/CD+5N+yEStRjS1K48xSgH44n/B+xJ1A40k8c7oE9a9u5pG2Gq0wF04iP5w=="
    crossorigin="anonymous"
    referrerpolicy="no-referrer"
  />
</svelte:head>

<section class="form-section">
  <div class="form-card">
    <!-- Overlay de carga centralizado -->
    {#if isLoading}
      <div class="loading-overlay">
        <i class="fas fa-spinner fa-spin fa-3x text-secondary"></i>
        <p>Procesando solicitud...</p>
      </div>
    {/if}

    <h1>Registro de Usuario</h1>
    <!-- NUEVO: Slogan descriptivo -->
    <p class="slogan">
      Únete a nuestra plataforma en pocos pasos y comienza a disfrutar de todos
      los beneficios.
    </p>

    <!-- Mensaje de error de validación inmediato -->
    {#if inlineError}
      <p class="inline-error">{inlineError}</p>
    {/if}

    <form on:submit|preventDefault={registrar}>
      <!-- CONTENEDOR GRID PARA CAMPOS -->
      <!-- Los campos y botones están deshabilitados por la prop 'disabled={isLoading}' -->
      <div
        class="form-grid"
        style={isLoading ? "opacity: 0.5; pointer-events: none;" : ""}
      >
        {#each campos as field}
          <div class="form-group span-{field.span}">
            <label for={field.key}>{field.label}</label>

            {#if field.key === "password"}
              <!-- Contenedor con toggle de visibilidad para contraseña -->
              <div class="password-wrapper">
                <input
                  id={field.key}
                  type={passwordFieldType}
                  bind:value={formData[field.key]}
                  required={field.required}
                  placeholder="Define tu contraseña"
                  disabled={isLoading}
                />
                <button
                  type="button"
                  class="toggle-password"
                  on:click={() => togglePasswordVisibility("password")}
                  aria-label="Toggle password visibility"
                >
                  <i
                    class="fas"
                    class:fa-eye={passwordFieldType === "password"}
                    class:fa-eye-slash={passwordFieldType === "text"}
                  ></i>
                </button>
              </div>
            {:else}
              <!-- Campos normales -->
              <input
                id={field.key}
                type={field.type ?? "text"}
                bind:value={formData[field.key]}
                required={field.required}
                placeholder={`Ingresa ${field.label.toLowerCase()}`}
                disabled={isLoading}
              />
            {/if}
          </div>
        {/each}

        <!-- Campo de Confirmar Contraseña (separado) -->
        <!-- Se le aplica span-6 para que quede al lado de la Contraseña principal en escritorio -->
        <div class="form-group span-6 span-full-mobile">
          <label for="confirm_password">Confirmar Contraseña</label>
          <div class="password-wrapper">
            <input
              id="confirm_password"
              type={confirmPasswordFieldType}
              bind:value={confirmPassword}
              required
              placeholder="Repite tu contraseña"
              disabled={isLoading}
            />
            <button
              type="button"
              class="toggle-password"
              on:click={() => togglePasswordVisibility("confirm")}
              aria-label="Toggle confirm password visibility"
            >
              <i
                class="fas"
                class:fa-eye={confirmPasswordFieldType === "password"}
                class:fa-eye-slash={confirmPasswordFieldType === "text"}
              ></i>
            </button>
          </div>
        </div>
      </div>
      <!-- FIN CONTENEDOR GRID -->

      <!-- Nota: El botón de registro usa 'disabled={isLoading}' pero el contenido de 'isLoading' 
                 se maneja en el overlay centralizado. El contenido del botón se simplifica. -->
      <button type="submit" class="btn-register" disabled={isLoading}>
        Registrar Cuenta
      </button>

      <div class="links">
        <button
          type="button"
          class="link-btn"
          on:click={irLogin}
          disabled={isLoading}
        >
          ← ¿Ya tienes cuenta? Inicia sesión
        </button>
      </div>
    </form>
  </div>

  <!-- Notificación Personalizada (Reemplazo de alert) -->
  {#if showMessage}
    <div class="message-modal">
      <div class="message-content" class:success={isSuccess}>
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

<style>
  /* RESET GLOBAL PARA ELIMINAR EL BORDE BLANCO */
  :global(html),
  :global(body) {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    /* Asegura que el fondo sea continuo en toda la página */
    height: 100%;
    background: var(--color-dark);
  }

  /* VARIABLES DE COLOR */
  :root {
    --color-primary: #1e3a8a; /* Azul oscuro elegante */
    --color-secondary: #fcd34d; /* Amarillo/Naranja sutil */
    --color-dark: #0f172a; /* Fondo muy oscuro */
    --color-card: #1f2937; /* Gris oscuro para la tarjeta */
    --color-text: #e2e8f0; /* Gris claro para el texto */
    --color-error: #ef4444; /* Rojo */
    --color-success: #10b981; /* Verde */
    --color-link-default: #9ca3af;
  }

  .text-secondary {
    color: var(--color-secondary);
  }

  .form-section {
    min-height: 100vh;
    /* Fondo con gradiente elegante usando los colores dark */
    background: radial-gradient(
      circle at center,
      var(--color-dark) 0%,
      #000000 100%
    );
    display: flex;
    justify-content: center;
    align-items: center;
    color: var(--color-text);
    font-family: "Inter", sans-serif;
    /* Padding generoso para el scroll en móvil */
    padding: 40px 20px;
  }

  .form-card {
    background: var(--color-card);
    padding: 40px;
    border-radius: 12px;
    width: 100%;
    max-width: 600px; /* AJUSTE: Reducción para un look más compacto */
    text-align: center;
    border: 2px solid var(--color-primary);
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.7);
    transition: transform 0.3s ease-in-out;
    position: relative; /* Necesario para el overlay de carga */
  }

  @media (min-width: 768px) {
    .form-card:hover {
      transform: translateY(-2px);
      box-shadow: 0 15px 40px rgba(0, 0, 0, 0.8);
    }
  }

  @media (max-width: 767px) {
    .form-card {
      padding: 25px;
    }
  }

  h1 {
    margin-bottom: 5px; /* Reducido para acercar el slogan */
    color: var(--color-secondary);
    font-size: 2rem;
    font-weight: 800;
  }

  .slogan {
    color: var(--color-link-default);
    margin-bottom: 30px;
    font-size: 0.95rem;
  }

  /* Estructura de Grid para campos */
  .form-grid {
    display: grid;
    grid-template-columns: repeat(12, 1fr); /* 12 columnas base */
    gap: 20px;
    margin-bottom: 10px;
    transition: opacity 0.3s ease;
  }

  /* Definiciones de span para el grid */
  .span-12 {
    grid-column: span 12;
  } /* Ancho completo */
  .span-6 {
    grid-column: span 6;
  } /* Media columna */

  /* Regla para móvil: todo a ancho completo */
  @media (max-width: 767px) {
    .span-6,
    .span-12 {
      grid-column: span 12;
    }
    /* Ajuste para el campo de confirmación en móvil (span-full-mobile) */
    .span-full-mobile {
      grid-column: span 12;
    }
  }

  /* Campos de formulario */
  .form-group {
    display: flex;
    flex-direction: column;
    text-align: left;
  }

  .form-group label {
    font-weight: 600;
    margin-bottom: 8px;
    color: var(--color-text);
    font-size: 0.9rem;
  }

  /* Contenedor del campo de contraseña para el icono de ojo */
  .password-wrapper {
    position: relative;
    display: flex;
    align-items: center;
  }

  .password-wrapper input {
    width: 100%;
    padding-right: 40px;
  }

  input {
    padding: 12px;
    border-radius: 8px;
    border: 1px solid #444;
    background: #374151;
    color: white;
    transition:
      border-color 0.3s,
      box-shadow 0.3s;
  }

  input:focus {
    border-color: var(--color-secondary);
    box-shadow: 0 0 0 3px rgba(252, 211, 77, 0.3);
    outline: none;
  }

  input:disabled {
    cursor: not-allowed;
  }

  input::placeholder {
    color: #9ca3af;
  }

  /* Botón para mostrar/ocultar contraseña */
  .toggle-password {
    position: absolute;
    right: 0;
    top: 0;
    bottom: 0;
    background: none;
    border: none;
    padding: 0 10px;
    cursor: pointer;
    color: var(--color-link-default);
    transition: color 0.2s;
    height: 100%;
    display: flex;
    align-items: center;
    margin-right: 5px;
  }

  .toggle-password:hover {
    color: var(--color-secondary);
  }

  /* OVERLAY DE CARGA */
  .loading-overlay {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(31, 41, 55, 0.9); /* color-card con transparencia */
    border-radius: 12px;
    z-index: 10;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    transition: opacity 0.3s ease;
  }

  .loading-overlay p {
    margin-top: 15px;
    font-weight: 600;
    color: var(--color-text);
  }

  /* Botón Principal */
  .btn-register {
    grid-column: span 12; /* Ocupa todo el ancho en el grid */
    width: 100%;
    padding: 14px 0;
    margin-top: 15px;
    background: var(--color-secondary);
    border: none;
    border-radius: 8px;
    color: var(--color-dark);
    font-weight: 700;
    font-size: 1.1rem;
    cursor: pointer;
    transition:
      background 0.3s,
      transform 0.2s,
      box-shadow 0.3s;
    box-shadow: 0 4px 10px rgba(252, 211, 77, 0.4);
  }

  .btn-register:disabled {
    background: #9ca3af;
    cursor: not-allowed;
    transform: none;
    box-shadow: none;
  }

  .btn-register:hover:not(:disabled) {
    background: #fbdc6f;
    transform: translateY(-1px);
    box-shadow: 0 6px 15px rgba(252, 211, 77, 0.5);
  }

  /* Errores en línea */
  .inline-error {
    color: var(--color-error);
    background: rgba(239, 68, 68, 0.1);
    border: 1px solid var(--color-error);
    padding: 10px;
    border-radius: 6px;
    font-size: 0.9rem;
    margin-bottom: 20px;
  }

  /* Enlace a Login */
  .links {
    margin-top: 20px;
  }

  .link-btn {
    background: none;
    border: none;
    color: var(--color-link-default);
    cursor: pointer;
    font-size: 0.95rem;
    transition: color 0.2s;
    text-decoration: none;
    padding: 5px;
    display: inline-block;
  }

  .link-btn:hover {
    color: var(--color-secondary);
  }

  /* ---------------------------------- */
  /* ESTILOS DEL MENSAJE PERSONALIZADO */
  /* ---------------------------------- */
  .message-modal {
    position: fixed;
    top: 20px;
    right: 20px;
    z-index: 1000;
    animation: slideIn 0.5s forwards;
  }

  .message-content {
    padding: 15px 25px;
    border-radius: 8px;
    color: white;
    background-color: var(--color-error);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
    display: flex;
    align-items: center;
    gap: 15px;
    font-weight: 600;
    animation: bounce 0.5s ease-out; /* NUEVO: Animación de rebote al aparecer */
  }

  .message-content.success {
    background-color: var(--color-success);
  }

  .message-content i {
    font-size: 1.2rem;
  }

  /* Animación de entrada por deslizamiento */
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
</style>
