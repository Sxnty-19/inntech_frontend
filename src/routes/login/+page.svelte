<script>
    import { goto } from "$app/navigation";

    let username = "";
    let password = "";
    let error = "";

    async function login() {
        error = "";

        if (!username || !password) {
            error = "Por favor ingresa usuario y contraseña";
            return;
        }

        try {
            const formData = new FormData();
            formData.append("username", username);
            formData.append("password", password);

            const response = await fetch("https://inntech-backend.onrender.com/auth/login", {
                method: "POST",
                body: formData
            });

            const data = await response.json();
            console.log("DATA LOGIN", data);

            if (!response.ok) {
                error = data.detail || "Error al iniciar sesión";
                return;
            }

           if (data.success) {
                localStorage.setItem("token", data.access_token);
                localStorage.setItem("user", JSON.stringify(data.user));

                const fullName = `
                    ${data.user.primer_nombre} 
                    ${data.user.segundo_nombre ?? ""} 
                    ${data.user.primer_apellido} 
                    ${data.user.segundo_apellido ?? ""}
                `.replace(/\s+/g, " ").trim();

                alert(`Bienvenido ${fullName}`);
                goto("/principal");
                
            } else {
                error = "No se pudo iniciar sesión.";
            }

        } catch (e) {
            console.error(e);
            error = "No se puede conectar con el servidor";
        }
    }

    function irRegistro() {
        goto("/register");
    }

    function volverInicio() {
        goto("/");
    }
</script>

<section class="form-section">
    <div class="form-card">
        <h1>Iniciar Sesión</h1>

        {#if error}
            <p class="error">{error}</p>
        {/if}

        <form on:submit|preventDefault={login}>
            <div class="form-group">
                <label for="username">Usuario</label>
                <input
                    id="username"
                    type="text"
                    placeholder="Ingresa tu usuario"
                    bind:value={username}
                />
            </div>

            <div class="form-group">
                <label for="password">Contraseña</label>
                <input
                    id="password"
                    type="password"
                    placeholder="Ingresa tu contraseña"
                    bind:value={password}
                />
            </div>

            <button type="submit" class="btn-login">Ingresar</button>

            <div class="links">
                <button type="button" class="link-btn" on:click={irRegistro}>
                    Registrarse
                </button>
                <button type="button" class="link-btn" on:click={volverInicio}>
                    Volver
                </button>
            </div>
        </form>
    </div>
</section>

<style>
    .form-section {
        min-height: 100vh;
        background: linear-gradient(180deg, #000000, #111827);
        display: flex;
        justify-content: center;
        align-items: center;
        color: white;
        font-family: "Segoe UI", sans-serif;
        padding: 20px;
    }

    .form-card {
        background: rgba(17, 24, 39, 0.9);
        padding: 35px 30px;
        border-radius: 10px;
        width: 100%;
        max-width: 420px;
        text-align: center;
        border: 1px solid rgba(255, 165, 0, 0.3);
        box-shadow: 0 4px 15px rgba(0, 0, 0, 0.6);
    }

    h1 {
        margin-bottom: 25px;
        color: orange;
        font-size: 1.8rem;
        font-weight: 700;
    }

    .form-group {
        display: flex;
        flex-direction: column;
        margin-bottom: 16px;
        text-align: left;
    }

    .form-group label {
        font-weight: 600;
        margin-bottom: 5px;
        color: orange;
    }

    input {
        padding: 10px 12px;
        border-radius: 6px;
        border: 1px solid #444;
        background: #1f2937;
        color: white;
        transition: all 0.2s;
    }

    input::placeholder {
        color: #aaa;
    }

    .btn-login {
        width: 100%;
        padding: 12px 0;
        margin-top: 10px;
        background: orange;
        border: none;
        border-radius: 8px;
        color: black;
        font-weight: 600;
        cursor: pointer;
    }

    .btn-login:hover {
        background: #fbbf24;
    }

    .error {
        color: #ff6b6b;
        font-size: 0.9rem;
        margin-bottom: 12px;
    }

    .links {
        margin-top: 16px;
        display: flex;
        justify-content: space-between;
    }

    .link-btn {
        background: none;
        border: none;
        color: #ccc;
        cursor: pointer;
        font-size: 0.9rem;
        text-decoration: underline;
    }

    .link-btn:hover {
        color: white;
    }
</style>