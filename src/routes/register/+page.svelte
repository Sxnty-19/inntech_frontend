<script>
  import { goto } from "$app/navigation";

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

  let campos = [
    { label: "Primer Nombre", key: "primer_nombre", required: true },
    { label: "Segundo Nombre", key: "segundo_nombre" },
    { label: "Primer Apellido", key: "primer_apellido", required: true },
    { label: "Segundo Apellido", key: "segundo_apellido" },
    { label: "Teléfono", key: "telefono", type: "tel" },
    { label: "Correo", key: "correo", required: true, type: "email" },
    { label: "Usuario", key: "username", required: true },
    { label: "Contraseña", key: "password", required: true, type: "password" }
  ];

  async function registrar() {
    try {
      const response = await fetch("https://inntech-backend.onrender.com/auth/register", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          id_rol: 3,
          estado: 1,
          ...formData
        }),
      });

      const result = await response.json();

      if (!response.ok) {
        alert(result.detail || "Error al registrar usuario");
        return;
      }

      alert("Usuario registrado exitosamente");
      goto("/login");

    } catch (error) {
      console.error("Error de conexión:", error);
      alert("Error de conexión con el servidor");
    }
  }

  function irLogin() {
    goto("/login");
  }
</script>

<section class="form-section">
  <div class="form-card">
    <h1>Registro de Usuario</h1>

    <form on:submit|preventDefault={registrar}>
      {#each campos as field}
        <div class="form-group">
          <label for={field.key}>{field.label}</label>
          <input
            id={field.key}
            type={field.type ?? "text"}
            bind:value={formData[field.key]}
            required={field.required}
            placeholder={`Ingresa ${field.label.toLowerCase()}`}
          />
        </div>
      {/each}

      <button type="submit" class="btn-login">Registrar</button>

      <div class="links">
        <button type="button" class="link-btn" on:click={irLogin}>
          ¿Ya tienes cuenta? Inicia sesión
        </button>
      </div>
    </form>
  </div>
</section>

<style>
  .form-section {
    min-height: 100%;
    background: linear-gradient(180deg, #000000, #111827);
    display: flex;
    justify-content: center;
    align-items: center;
    color: white;
    font-family: "Segoe UI", sans-serif;
    padding: 40px 20px;
  }

  :global(html, body) {
    height: 100%;
    margin: 0;
    background: linear-gradient(180deg, #000000, #111827);
  }

  .form-card {
    background: rgba(17, 24, 39, 0.95);
    padding: 30px 25px;
    border-radius: 10px;
    width: 100%;
    max-width: 370px;
    text-align: center;
    border: 1px solid rgba(255, 165, 0, 0.3);
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.6);
  }

  h1 {
    margin-bottom: 20px;
    color: orange;
    font-size: 1.6rem;
    font-weight: 700;
  }

  form {
    display: flex;
    flex-direction: column;
    gap: 12px;
  }

  .form-group {
    display: flex;
    flex-direction: column;
    text-align: left;
  }

  label {
    font-weight: 600;
    margin-bottom: 4px;
    color: orange;
    font-size: 0.9rem;
  }

  input {
    padding: 8px 10px;
    border-radius: 6px;
    border: 1px solid #444;
    background: #1f2937;
    color: white;
    transition: all 0.2s;
    font-size: 0.9rem;
  }

  input::placeholder {
    color: #aaa;
  }

  .btn-login {
    width: 100%;
    padding: 10px 0;
    margin-top: 10px;
    background: orange;
    border: none;
    border-radius: 8px;
    color: black;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s;
    font-size: 0.95rem;
  }

  .btn-login:hover {
    background: #fbbf24;
  }

  .links {
    margin-top: 14px;
  }

  .link-btn {
    background: none;
    border: none;
    color: #ccc;
    cursor: pointer;
    font-size: 0.85rem;
    text-decoration: underline;
  }

  .link-btn:hover {
    color: white;
  }

  /* responsive */
  @media (max-width: 480px) {
    .form-card {
      padding: 25px 18px;
      max-width: 92%;
    }

    h1 {
      font-size: 1.4rem;
    }

    label {
      font-size: 0.85rem;
    }

    input {
      font-size: 0.85rem;
      padding: 7px 9px;
    }

    .btn-login {
      font-size: 0.9rem;
      padding: 9px 0;
    }

    .link-btn {
      font-size: 0.8rem;
    }
  }

  @media (max-width: 768px) and (min-width: 481px) {
    .form-card {
      max-width: 360px;
      padding: 28px;
    }

    h1 {
      font-size: 1.5rem;
    }
  }
</style>