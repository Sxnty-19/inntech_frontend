<script>
    import { goto } from "$app/navigation";

    // Variables de estado del formulario
    let username = "";
    let password = "";
    let inlineError = ""; // Para errores de validación inmediata

    // ESTILO DE CARGA: Estado de carga para deshabilitar el botón y mostrar el overlay
    let isLoading = false;

    // NUEVO: Para controlar la visibilidad de la contraseña
    let passwordFieldType = "password";

    // Variables para el sistema de mensajes/modal personalizado (Reemplazo de alert)
    let message = "";
    let isSuccess = false;
    let showMessage = false;

    // NUEVA FUNCIÓN: Alterna la visibilidad de la contraseña
    function togglePasswordVisibility() {
        passwordFieldType =
            passwordFieldType === "password" ? "text" : "password";
    }

    async function login() {
        inlineError = "";
        showMessage = false;

        if (!username || !password) {
            inlineError = "Por favor, ingresa tu usuario y contraseña.";
            return;
        }

        // Bloquea el botón y establece el estado de carga
        isLoading = true;

        try {
            const formData = new FormData();
            formData.append("username", username);
            formData.append("password", password);

            // Reemplazar URL con la de Firebase si se utiliza la versión de Canvas.
            const response = await fetch(
                "https://inntech-backend.onrender.com/auth/login",
                {
                    method: "POST",
                    body: formData,
                },
            );

            const data = await response.json();
            console.log("DATA LOGIN", data);

            if (!response.ok) {
                inlineError = data.detail || "Credenciales incorrectas.";
                return;
            }

            if (data.success) {
                // Almacenamiento local (se recomienda usar Firestore para apps reales)
                localStorage.setItem("token", data.access_token);
                localStorage.setItem("user", JSON.stringify(data.user));

                const fullName = `
                    ${data.user.primer_nombre} 
                    ${data.user.segundo_nombre ?? ""} 
                    ${data.user.primer_apellido} 
                    ${data.user.segundo_apellido ?? ""}
                `
                    .replace(/\s+/g, " ")
                    .trim();

                // Mostrar mensaje de bienvenida personalizado
                message = `¡Bienvenido(a), ${fullName}!`;
                isSuccess = true;
                showMessage = true;

                // Navegar después de mostrar el mensaje por un breve tiempo
                setTimeout(() => {
                    goto("/principal");
                }, 1500);
            } else {
                inlineError = "No se pudo iniciar sesión. Verifica tus datos.";
            }
        } catch (e) {
            console.error(e);
            inlineError = "Error de conexión. Inténtalo de nuevo más tarde.";
        } finally {
            // Desbloquea el botón
            isLoading = false;
        }
    }

    function irRegistro() {
        goto("/register");
    }

    // FUNCIÓN ACTUALIZADA: Usa el modal de mensajes en lugar de alert()
    function irRecuperarContrasena() {
        message = "Funcionalidad de recuperación de contraseña en desarrollo.";
        isSuccess = false;
        showMessage = true;
    }

    function volverInicio() {
        goto("/");
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
        <!-- Overlay de carga centralizado: aparece cuando isLoading es true -->
        {#if isLoading}
            <div class="loading-overlay">
                <i class="fas fa-spinner fa-spin fa-3x text-secondary"></i>
                <p>Iniciando sesión...</p>
            </div>
        {/if}

        <h1>Iniciar Sesión</h1>
        <p class="slogan">Ingresa tus credenciales para acceder a tu cuenta.</p>

        <!-- Mensaje de error de validación inmediato -->
        {#if inlineError}
            <p class="inline-error">{inlineError}</p>
        {/if}

        <!-- El formulario se atenúa y deshabilita si está cargando -->
        <form
            on:submit|preventDefault={login}
            style={isLoading ? "opacity: 0.5; pointer-events: none;" : ""}
        >
            <div class="form-group">
                <label for="username">Usuario</label>
                <input
                    id="username"
                    type="text"
                    placeholder="Tu nombre de usuario"
                    bind:value={username}
                    required
                    disabled={isLoading}
                />
            </div>

            <div class="form-group">
                <label for="password">Contraseña</label>
                <!-- CONTENEDOR NUEVO: Para el campo y el botón de visibilidad -->
                <div class="password-wrapper">
                    <input
                        id="password"
                        type={passwordFieldType}
                        placeholder="Tu contraseña secreta"
                        bind:value={password}
                        required
                        disabled={isLoading}
                    />
                    <button
                        type="button"
                        class="toggle-password"
                        on:click={togglePasswordVisibility}
                        aria-label="Toggle password visibility"
                        disabled={isLoading}
                    >
                        <!-- Alterna entre icono de ojo abierto y cerrado -->
                        <i
                            class="fas"
                            class:fa-eye={passwordFieldType === "password"}
                            class:fa-eye-slash={passwordFieldType === "text"}
                        ></i>
                    </button>
                </div>

                <!-- Enlace de recuperación de contraseña -->
                <button
                    type="button"
                    class="link-btn password-forgot"
                    on:click={irRecuperarContrasena}
                    disabled={isLoading}
                >
                    ¿Olvidaste tu contraseña?
                </button>
            </div>

            <button type="submit" class="btn-login" disabled={isLoading}>
                Ingresar
            </button>

            <div class="links">
                <button
                    type="button"
                    class="link-btn"
                    on:click={irRegistro}
                    disabled={isLoading}
                >
                    ¿No tienes cuenta? Regístrate
                </button>
                <button
                    type="button"
                    class="link-btn link-back"
                    on:click={volverInicio}
                    disabled={isLoading}
                >
                    ← Volver al Inicio
                </button>
            </div>
        </form>
    </div>

    <!-- Notificación Personalizada (Reemplazo de alert) -->
    {#if showMessage}
        <div class="message-modal">
            <!-- Animación de rebote aplicada en message-content -->
            <div class="message-content" class:success={isSuccess}>
                <!-- Iconos dinámicos según el estado -->
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
    /* Target HTML y Body para eliminar márgenes y paddings por defecto */
    :global(html),
    :global(body) {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
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

    /* Clase auxiliar para el spinner de carga */
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
        padding: 20px;
    }

    .form-card {
        background: var(--color-card);
        padding: 40px;
        border-radius: 12px;
        width: 100%;
        max-width: 400px;
        text-align: center;
        /* Borde y sombra destacados */
        border: 2px solid var(--color-primary);
        box-shadow: 0 10px 30px rgba(0, 0, 0, 0.7);
        transition: transform 0.3s ease-in-out;
        position: relative; /* CRUCIAL para que el overlay de carga se posicione correctamente */
    }

    .form-card:hover {
        transform: translateY(-2px);
        box-shadow: 0 15px 40px rgba(0, 0, 0, 0.8);
    }

    h1 {
        margin-bottom: 5px; /* Ajustado para el slogan */
        color: var(--color-secondary);
        font-size: 2rem;
        font-weight: 800;
    }

    /* ESTILO SLOGAN */
    .slogan {
        color: var(--color-link-default);
        margin-bottom: 30px;
        font-size: 0.95rem;
    }

    /* OVERLAY DE CARGA CENTRALIZADO */
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
    /* FIN OVERLAY */

    /* Campos de formulario */
    .form-group {
        display: flex;
        flex-direction: column;
        margin-bottom: 20px;
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
        padding-right: 40px; /* Espacio para el botón de alternar */
    }

    input {
        padding: 12px;
        border-radius: 8px;
        border: 1px solid #444;
        background: #374151; /* Ligeramente más claro que la tarjeta */
        color: white;
        transition:
            border-color 0.3s,
            box-shadow 0.3s;
    }

    input:focus {
        border-color: var(--color-secondary);
        box-shadow: 0 0 0 3px rgba(252, 211, 77, 0.3); /* Sombra de foco color naranja */
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

    /* Botón Principal */
    .btn-login {
        width: 100%;
        padding: 14px 0;
        margin-top: 20px;
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

    /* Estilo para botón deshabilitado */
    .btn-login:disabled {
        background: #9ca3af; /* Gris tenue */
        cursor: not-allowed;
        transform: none;
        box-shadow: none;
    }

    .btn-login:hover:not(:disabled) {
        background: #fbdc6f; /* Tono más claro */
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

    /* Enlaces Secundarios (Registrarse / Volver) */
    .links {
        margin-top: 20px;
        display: flex;
        flex-direction: column; /* Apilados para mejor diseño en móvil/card */
        gap: 10px;
    }

    .password-forgot {
        /* Mueve el botón de "Olvidé mi contraseña" a la derecha, debajo del campo */
        align-self: flex-end;
        margin-top: 5px;
        font-size: 0.8rem;
    }

    .link-btn {
        background: none;
        border: none;
        color: var(--color-link-default);
        cursor: pointer;
        font-size: 0.9rem;
        transition: color 0.2s;
        text-decoration: none;
        padding: 5px;
        display: block;
    }

    .link-btn:hover {
        color: var(--color-secondary);
    }

    .link-back {
        color: var(--color-primary);
        font-size: 0.95rem; /* Hacemos este un poco más grande para destacarlo */
    }
    .link-back:hover {
        color: var(--color-secondary);
    }

    /* ---------------------------------- */
    /* ESTILOS DEL MENSAJE PERSONALIZADO (REEMPLAZO DE ALERT) */
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
        animation: bounce 0.5s ease-out; /* Animación de rebote al aparecer */
    }

    .message-content.success {
        background-color: var(--color-success);
    }

    .message-content i {
        font-size: 1.2rem;
    }

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
