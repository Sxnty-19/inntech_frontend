<script>
    import Footer from '$lib/components/Footer.svelte';
</script>

<nav class="navbar">
    <div class="logo">
        <img src="/logo.png" alt="Logo" />
        <span>Hospedaje Santandereano del Sur</span>
    </div>

    <ul class="menu">
        <li><a href="/login" class="btn">Iniciar Sesión</a></li>
    </ul>
</nav>

<section class="hero">
    <div class="content">
        <img src="/logo.png" alt="Hotel Logo" class="hero-logo" />

        <h1>Sistema de Gestión Hotelera con IA</h1>
        <p>Gestiona reservas, controla usuarios y obtén ayuda inmediata con inteligencia artificial.</p>
    </div>
</section>

<Footer />

<style>
    nav.navbar {
        background: #0b1622;
        color: white;
        padding: 15px 40px;
        display: flex;
        justify-content: space-between;
        align-items: center;
        position: fixed;
        width: 100%;
        top: 0;
        z-index: 50;
    }

    .logo {
        display: flex;
        align-items: center;
        gap: 10px;
        font-weight: bold;
        font-size: 18px;
    }

    .logo img {
        height: 35px;
    }

    .menu {
        list-style: none;
        display: flex;
        gap: 25px;
    }

    .menu a {
        color: white;
        text-decoration: none;
        font-size: 16px;
    }

    .menu .btn {
        background: #ff9d2a;
        padding: 8px 14px;
        border-radius: 6px;
    }

    section.hero {
        height: 100vh;
        background: black;
        color: white;
        display: flex;
        align-items: center;
        justify-content: center;
        text-align: center;
        padding-top: 80px;
    }

    .hero-logo {
        width: 140px;
        margin-bottom: 20px;
        opacity: 0.9;
    }

    h1 {
        font-size: 2.5rem;
        color: #ff9d2a;
    }
</style>