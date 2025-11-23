<script>
    import Footer from "$lib/components/Footer.svelte";

    // Array con datos de las 5 habitaciones solicitadas
    const habitaciones = [
        {
            nombre: "Habitación Sencilla",
            descripcion:
                "Diseñada para el viajero solitario, ofrece comodidad y privacidad con una cama individual grande.",
            imagen: "img/habitaciones/sencilla.png",
        },
        {
            nombre: "Habitación Doble",
            descripcion:
                "Perfecta para parejas o dos amigos. Disponible con una cama doble grande o dos camas individuales.",
            imagen: "/img/habitaciones/doble.png",
        },
        {
            nombre: "Habitación Múltiple",
            descripcion:
                "Ideal para grupos pequeños o familias. Ofrece tres o cuatro camas individuales cómodas.",
            imagen: "/img/habitaciones/multiple.png",
        },
        {
            nombre: "Habitación Quíntuple",
            descripcion:
                "Gran espacio con cinco camas, pensada para equipos deportivos o grandes grupos de amigos.",
            imagen: "/img/habitaciones/quintuple.png",
        },
        {
            nombre: "Habitación Séxtuple",
            descripcion:
                "Nuestra opción más espaciosa, con seis camas. Máxima capacidad y excelente para delegaciones.",
            imagen: "/img/habitaciones/sextuple.png",
        },
    ];
</script>

<svelte:head>
    <style>
        /* Variables CSS */
        :root {
            --color-primary: #1e3a8a; /* Azul oscuro elegante */
            --color-secondary: #fcd34d; /* Amarillo/Naranja sutil */
            --color-dark: #0f172a; /* Casi negro para el fondo de la barra de navegación */
            --color-text: #e2e8f0; /* Gris claro para el texto de HERO */
            --color-light: #ffffff;
            --color-background-gray: #f8f8f8; /* Gris muy claro para secciones de contenido */
            --color-dark-text: #1f2937; /* Texto oscuro para secciones claras */
            --color-medium-gray: #4b5563; /* Texto secundario */
            --shadow-subtle: 0 4px 12px rgba(0, 0, 0, 0.05); /* Sombra sutil */
            --shadow-lift: 0 10px 25px rgba(0, 0, 0, 0.1);
        }

        /* CORRECCIÓN: Reset global para eliminar bordes blancos y asegurar responsive */
        * {
            box-sizing: border-box;
        }

        body {
            margin: 0;
            padding: 0;
            overflow-x: hidden;
            /* Mejora: Definición de una fuente moderna */
            font-family: "Inter", "Helvetica Neue", Helvetica, Arial, sans-serif;
        }
    </style>
</svelte:head>

<nav class="navbar">
    <div class="logo">
        <img src="/logo.png" alt="Logo de Hospedaje Santandereano del Sur" />
        <a href="/" class="site-title">Hospedaje Santandereano del Sur</a>
    </div>

    <ul class="menu">
        <li><a href="/" class="nav-link">Inicio</a></li>
        <li><a href="/login" class="btn btn-outline">Iniciar Sesión</a></li>
    </ul>
</nav>

<section class="hero">
    <div class="content">
        <img src="/logo.png" alt="Logo del Hotel" class="hero-logo" />
        <h1>Bienvenido al Hospedaje Santandereano del Sur</h1>
        <p>
            Descubre el espacio perfecto para tu estancia en Santandereano del
            Sur.
        </p>
    </div>
</section>

<section class="rooms">
    <div class="container">
        <h2 class="section-title">Nuestras Habitaciones</h2>
        <p class="section-subtitle">
            Descubre el espacio perfecto para tu estancia en Santandereano del
            Sur.
        </p>

        <div class="room-grid">
            {#each habitaciones as room}
                <div class="room-card">
                    <div class="room-image-placeholder">
                        <img
                            src={room.imagen}
                            alt={`Habitación ${room.nombre}`}
                            class="room-image"
                        />
                    </div>
                    <div class="card-content">
                        <h3>{room.nombre}</h3>
                        <p>{room.descripcion}</p>
                    </div>
                </div>
            {/each}
        </div>
    </div>
</section>

<section class="about">
    <div class="container about-grid">
        <div class="about-content">
            <h2 class="section-title about-title">Nuestra Historia y Misión</h2>
            <p class="lead-text">
                En Hospedaje Santandereano del Sur, nuestra misión es ofrecer un
                refugio de paz y comodidad, combinado con la eficiencia de la
                tecnología moderna.
            </p>
            <p>
                Fundado en el corazón de Santander, hemos crecido manteniendo la
                calidez de la hospitalidad tradicional colombiana. Cada rincón
                de nuestro hotel está diseñado pensando en su descanso y
                productividad, asegurando una experiencia única para cada
                huésped.
            </p>
            <p>
                Usamos la Inteligencia Artificial no solo para gestionar
                reservas eficientemente, sino para personalizar su estancia,
                haciéndola memorable desde el momento en que llega hasta que se
                va. ¡Su bienestar es nuestra prioridad!
            </p>
        </div>
        <div class="about-image">
            <img
                src="/img/portada/sobreNosotros.png"
                alt="Vista frontal del Hospedaje Santandereano del Sur"
                class="hotel-img"
            />
        </div>
    </div>
</section>
<Footer />

<style>
    /* ---------------------------------- */
    /* ESTILOS GLOBALES Y TIPOGRAFÍA */
    /* ---------------------------------- */
    .container {
        max-width: 1200px;
        margin: 0 auto;
        padding: 0 20px;
    }

    .section-title {
        font-size: 2.5rem; /* Ligeramente más grande */
        font-weight: 800; /* Más peso */
        color: var(--color-primary);
        text-align: center;
        margin-bottom: 15px;
    }

    .section-subtitle {
        font-size: 1.15rem; /* Ligeramente más grande */
        color: var(--color-medium-gray);
        text-align: center;
        margin-bottom: 50px;
    }

    /* ---------------------------------- */
    /* NAVBAR */
    /* ---------------------------------- */
    nav.navbar {
        background: var(--color-dark);
        color: var(--color-text);
        padding: 12px 60px;
        display: flex;
        justify-content: space-between;
        align-items: center;
        position: fixed;
        width: 100%;
        top: 0;
        z-index: 100;
        box-shadow: 0 4px 15px rgba(0, 0, 0, 0.4); /* Sombra más fuerte para fixed nav */
    }

    .logo {
        display: flex;
        align-items: center;
        gap: 12px;
        cursor: pointer;
    }

    .logo img {
        height: 30px;
        transition: transform 0.3s ease;
    }

    .logo:hover img {
        transform: scale(1.05) rotate(-2deg); /* Efecto más dinámico */
    }

    .site-title {
        font-weight: 700;
        font-size: 1.2rem; /* Ligeramente más grande */
        color: var(--color-light); /* Cambié a light para mejor contraste */
        text-decoration: none;
        letter-spacing: 0.8px;
        transition: color 0.3s;
    }

    .logo:hover .site-title {
        color: var(--color-secondary);
    }

    .menu {
        list-style: none;
        display: flex;
        align-items: center;
        gap: 30px;
        margin: 0;
        padding: 0;
    }

    .nav-link {
        color: var(--color-text);
        text-decoration: none;
        font-size: 15px;
        transition:
            color 0.3s,
            border-bottom 0.3s;
    }

    .nav-link:hover {
        color: var(--color-secondary);
        border-bottom: 2px solid var(--color-secondary);
        padding-bottom: 3px;
    }

    /* ---------------------------------- */
    /* BOTONES */
    /* ---------------------------------- */
    .btn {
        padding: 9px 18px;
        border-radius: 9999px; /* Botones pill/redondeados */
        font-weight: 600;
        text-decoration: none;
        transition: all 0.3s ease;
        font-size: 15px;
        cursor: pointer;
        display: inline-block;
        text-align: center;
        border: 2px solid transparent;
        letter-spacing: 0.5px;
    }

    .btn-outline {
        background: transparent;
        color: var(--color-secondary);
        border-color: var(--color-secondary);
    }

    .btn-outline:hover {
        background: var(--color-secondary);
        color: var(--color-dark);
        box-shadow: 0 4px 10px rgba(252, 211, 77, 0.5); /* Sombra al pasar el ratón */
    }

    /* ---------------------------------- */
    /* HERO SECTION */
    /* ---------------------------------- */
    /* (Se mantienen los estilos de HERO) */
    section.hero {
        min-height: 100vh;
        background: var(--color-dark);
        background-image: linear-gradient(
            135deg,
            rgba(0, 0, 0, 0.8) 0%,
            var(--color-dark) 80%
        );
        color: white;
        display: flex;
        align-items: center;
        justify-content: center;
        text-align: center;
        padding: 120px 20px 60px;
    }

    .content {
        max-width: 800px;
        opacity: 0;
        animation: fadeIn 1.2s ease-out 0.2s forwards;
    }

    @keyframes fadeIn {
        from {
            opacity: 0;
            transform: translateY(20px);
        }
        to {
            opacity: 1;
            transform: translateY(0);
        }
    }

    .hero-logo {
        width: 100px;
        margin-bottom: 15px;
        opacity: 0.8;
    }

    h1 {
        font-size: 3.5rem; /* Más impacto */
        font-weight: 900;
        color: var(--color-secondary);
        line-height: 1.1;
        margin-bottom: 20px;
        text-shadow: 0 4px 6px rgba(0, 0, 0, 0.4);
    }

    .hero p {
        font-size: 1.3rem;
        color: var(--color-text);
        margin-bottom: 30px;
    }

    /* ---------------------------------- */
    /* ROOMS SECTION */
    /* ---------------------------------- */
    .rooms {
        padding: 100px 0; /* Más padding vertical */
        background-color: var(--color-background-gray);
    }

    .room-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
        gap: 40px; /* Más espacio entre tarjetas */
        margin-top: 50px;
    }

    .room-card {
        background: var(--color-light);
        border-radius: 16px; /* Más redondeado */
        overflow: hidden;
        box-shadow: var(--shadow-subtle); /* Sombra más suave */
        transition:
            transform 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275),
            /* Efecto spring */ box-shadow 0.4s ease;
        padding-bottom: 0;
    }

    .room-card:hover {
        transform: translateY(-8px);
        box-shadow: var(--shadow-lift); /* Más levantado al pasar el ratón */
    }

    .room-image-placeholder {
        width: 100%;
        height: 220px;
        background-color: #ddd;
        overflow: hidden;
    }

    .room-image {
        width: 100%;
        height: 100%;
        object-fit: cover;
        transition:
            transform 0.6s ease,
            filter 0.6s ease;
        filter: grayscale(80%); /* Efecto Blanco y Negro */
    }

    .room-card:hover .room-image {
        transform: scale(1.1);
        filter: grayscale(0%); /* Color al pasar el ratón */
    }

    .card-content {
        padding: 30px; /* Más padding */
    }

    .card-content h3 {
        font-size: 1.5rem;
        color: var(--color-primary);
        margin-bottom: 10px;
        font-weight: 700;
    }

    .card-content p {
        color: var(--color-medium-gray);
        margin-bottom: 0;
        line-height: 1.6;
        font-size: 1rem;
    }

    /* ---------------------------------- */
    /* ABOUT SECTION */
    /* ---------------------------------- */
    .about {
        padding: 100px 0;
        background-color: var(--color-light);
        border-top: 1px solid #f0f0f0; /* Borde más sutil */
    }

    .about-title {
        text-align: left;
        margin-bottom: 30px;
        font-size: 2.5rem;
    }

    .about-grid {
        display: grid;
        grid-template-columns: 1.2fr 0.8fr; /* Le doy más espacio al texto */
        gap: 80px; /* Más separación */
        align-items: center;
    }

    .about-content p {
        color: var(--color-dark-text);
        line-height: 1.8;
        margin-bottom: 20px;
    }

    .lead-text {
        font-size: 1.35rem; /* Más grande */
        font-weight: 700;
        color: var(--color-primary);
        margin-bottom: 25px !important;
    }

    .about-image {
        border-radius: 16px;
        overflow: hidden;
        box-shadow: var(--shadow-lift);
        transform: rotate(-1.5deg); /* Ligera rotación */
        transition: transform 0.5s cubic-bezier(0.25, 0.46, 0.45, 0.94);
    }

    .about-image:hover {
        transform: rotate(0deg) scale(1.02);
        box-shadow: 0 20px 40px rgba(0, 0, 0, 0.2);
    }

    .hotel-img {
        width: 100%;
        height: auto;
        display: block;
    }

    /* ---------------------------------- */
    /* MEDIA QUERIES (Ajustes Responsive Finos) */
    /* ---------------------------------- */
    @media (max-width: 1024px) {
        .about-grid {
            grid-template-columns: 1fr 1fr;
            gap: 40px;
        }
    }

    @media (max-width: 900px) {
        nav.navbar {
            padding: 12px 20px;
        }

        .about-grid {
            grid-template-columns: 1fr;
            gap: 40px;
        }

        .about-image {
            order: -1;
            transform: rotate(0deg);
        }

        .about-title {
            text-align: center;
        }
    }

    @media (max-width: 768px) {
        .rooms,
        .about {
            padding: 60px 0;
        }

        .room-grid {
            grid-template-columns: 1fr;
        }

        .site-title {
            font-size: 1rem;
        }

        h1 {
            font-size: 2.5rem;
        }

        .hero p {
            font-size: 1.1rem;
        }

        .section-title {
            font-size: 2rem;
        }
    }

    @media (max-width: 480px) {
        h1 {
            font-size: 2rem;
        }
        .section-title {
            font-size: 1.6rem;
        }
    }
</style>
