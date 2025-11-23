<script>
  import Navbar from "$lib/components/Navbar.svelte";
  import Footer from "$lib/components/Footer.svelte";
    import NavbarA from "$lib/components/NavbarA.svelte";

  let user = null;
  let fullName = "";
  let rol = "";

  // Obtener info del LocalStorage
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

      rol = user.rol;
    }
  }
</script>

<Navbar />
<NavbarA />

<section class="main">
  <div class="welcome-box">
    <h2 class="title">Bienvenido <span class="highlight">{fullName}</span></h2>
    <p class="role">Rol: <strong>{rol}</strong></p>
  </div>

</section>

<Footer />

<style>
  .main {
    min-height: calc(100vh - 160px);
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 50px 20px;
    background: #000;
    color: white;
    text-align: center;
  }

  .welcome-box {
    margin-bottom: 30px;
    background: rgba(255, 165, 0, 0.1);
    padding: 25px 35px;
    border-radius: 12px;
    border: 2px solid orange;
    box-shadow: 0 0 12px rgba(255, 165, 0, 0.3);
  }

  .title {
    font-size: 2.2rem;
    color: orange;
    margin-bottom: 10px;
  }

  .highlight {
    color: white;
    font-weight: 700;
  }

  .role {
    font-size: 1.2rem;
    margin-top: 5px;
    color: #ccc;
  }

  .hero-img {
    max-width: 750px;
    width: 90%;
    border-radius: 10px;
    border: 2px solid orange;
  }
</style>