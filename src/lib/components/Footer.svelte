<script>
    // Variables CSS utilizadas para la consistencia de la paleta de colores.
    const VARS = {
        // Colores principales de la marca (Dark Mode)
        "--color-primary": "#1e3a8a", // Azul Oscuro Elegante
        "--color-primary-light": "#3b82f6", // Azul claro para gradientes
        "--color-secondary": "#fcd34d", // Dorado/Amarillo (Acento)
        "--color-secondary-light": "#ffc400", // Dorado/Amarillo más claro
        "--color-text-primary": "#e2e8f0", // Texto claro
        "--color-text-secondary": "#94a3b8", // Texto sutil/grisáceo
        "--transition-base": "0.3s ease-in-out",
    };
</script>

<svelte:head>
    <link
        rel="stylesheet"
        href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css"
        crossorigin="anonymous"
        referrerpolicy="no-referrer"
    />
</svelte:head>

<footer
    style="
    --color-primary: {VARS['--color-primary']};
    --color-primary-light: {VARS['--color-primary-light']};
    --color-secondary: {VARS['--color-secondary']};
    --color-secondary-light: {VARS['--color-secondary-light']};
    --color-text-secondary: {VARS['--color-text-secondary']};
    --transition-base: {VARS['--transition-base']};
"
>
    <div class="footer-container">
        <div class="footer-main-content">
            <div class="footer-info">
                <h3>Contáctanos</h3>
                <p>
                    <i class="fa-solid fa-phone footer-icon"></i>
                    +57 312 3694858
                </p>
                <p>
                    <i class="fa-solid fa-envelope footer-icon"></i>
                    reservas@hospedajesur.com
                </p>
                <p>
                    <i class="fa-solid fa-location-dot footer-icon"></i>
                    Cra. 4#19A-09 Santa Marta
                </p>
            </div>

            <div class="footer-cta">
                <h3>Síguenos</h3>
                <div class="social-icons">
                    <a
                        href="https://facebook.com"
                        class="social-icon facebook"
                        aria-label="Facebook"
                    >
                        <i class="fa-brands fa-facebook-f"></i>
                    </a>
                    <a
                        href="https://instagram.com"
                        class="social-icon instagram"
                        aria-label="Instagram"
                    >
                        <i class="fa-brands fa-instagram"></i>
                    </a>
                    <a
                        href="https://twitter.com"
                        class="social-icon twitter"
                        aria-label="Twitter"
                    >
                        <i class="fa-brands fa-twitter"></i>
                    </a>
                    <a
                        href="https://wa.me/3123694858"
                        class="social-icon whatsapp"
                        aria-label="WhatsApp"
                    >
                        <i class="fa-brands fa-whatsapp"></i>
                    </a>
                </div>
            </div>
        </div>
    </div>

    <div class="footer-bottom">
        <p>
            © 2025 Hospedaje Santandereano del Sur. Todos los derechos
            reservados.
        </p>
    </div>
</footer>

<style>
    /* ================================== */
    /* ANIMACIÓN */
    /* ================================== */
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

    /* ================================== */
    /* MODERN FOOTER STYLES */
    /* ================================== */

    footer {
        background: #0f172a;
        color: var(--color-text-secondary);
        padding-top: 60px;
        padding-bottom: 20px;
        font-size: 0.95rem;
        position: relative;
        overflow: hidden;
        border-top: 5px solid var(--color-secondary);
    }

    footer::before {
        content: "";
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        height: 2px;
        opacity: 0.5;
    }

    .footer-container {
        max-width: 1200px;
        margin: 0 auto;
        padding: 0 20px 40px;
        display: flex;
        justify-content: center;
        border-bottom: 1px solid rgba(255, 255, 255, 0.1);
        margin-bottom: 20px;
    }

    .footer-main-content {
        display: grid;
        grid-template-columns: 1fr 1fr;
        gap: 120px;
        max-width: 700px;
        width: 100%;
        animation: fadeIn 0.6s ease-out;
    }

    .footer-info h3,
    .footer-cta h3 {
        font-size: 1.3rem;
        font-weight: 700;
        background: linear-gradient(
            135deg,
            var(--color-secondary) 0%,
            var(--color-secondary-light) 100%
        );
        -webkit-background-clip: text;
        -webkit-text-fill-color: transparent;
        background-clip: text;
        margin-bottom: 25px;
        border-left: 3px solid var(--color-primary);
        padding-left: 10px;
        position: relative;
    }

    /* Animación decorativa de línea bajo el título */
    .footer-info h3::after,
    .footer-cta h3::after {
        content: "";
        position: absolute;
        left: 0;
        bottom: -5px;
        width: 0;
        height: 2px;
        background: var(--color-secondary);
        transition: width var(--transition-base);
    }

    .footer-info:hover h3::after,
    .footer-cta:hover h3::after {
        width: 50%;
    }

    .footer-info p {
        margin: 10px 0;
        display: flex;
        align-items: center;
        line-height: 1.6;
        transition: all var(--transition-base);
    }

    /* Animación de desplazamiento al pasar el ratón */
    .footer-info p:hover {
        color: var(--color-secondary);
        transform: translateX(5px);
    }

    .footer-icon {
        margin-right: 12px;
        font-size: 1.1em;
        color: var(--color-text-secondary);
        width: 18px;
        text-align: center;
        transition: all var(--transition-base);
    }

    /* Animación de ícono al pasar el ratón */
    .footer-info p:hover .footer-icon {
        color: var(--color-secondary);
        transform: scale(1.2);
    }

    .social-icons {
        margin-bottom: 20px;
        display: flex;
        gap: 12px;
    }

    .social-icon {
        background: linear-gradient(
            135deg,
            var(--color-primary) 0%,
            var(--color-primary-light) 100%
        );
        width: 40px;
        height: 40px;
        border-radius: 50%;
        display: inline-flex;
        align-items: center;
        justify-content: center;
        text-decoration: none;
        font-size: 1.2rem;
        transition: all var(--transition-base);
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
        position: relative;
        overflow: hidden;
    }

    /* Efecto hover: cambio de fondo a dorado */
    .social-icon::before {
        content: "";
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background: linear-gradient(
            135deg,
            var(--color-secondary) 0%,
            var(--color-secondary-light) 100%
        );
        opacity: 0;
        transition: opacity var(--transition-base);
        border-radius: 50%;
    }

    .social-icon:hover::before {
        opacity: 1;
    }

    /* Efecto hover: movimiento y rotación */
    .social-icon:hover {
        transform: translateY(-5px) rotate(360deg) scale(1.1);
        box-shadow:
            0 8px 16px rgba(0, 0, 0, 0.4),
            0 0 20px rgba(252, 211, 77, 0.3);
    }

    .social-icon i {
        color: white;
        line-height: 1;
        position: relative;
        z-index: 1;
    }

    .footer-bottom {
        text-align: center;
        opacity: 0.7;
        transition: opacity var(--transition-base);
        animation: fadeIn 0.8s ease-out 0.3s both;
    }

    .footer-bottom:hover {
        opacity: 1;
    }

    /* ================================== */
    /* RESPONSIVE */
    /* ================================== */
    @media (max-width: 768px) {
        .footer-container {
            display: block;
            padding-bottom: 20px;
        }

        .footer-main-content {
            grid-template-columns: 1fr;
            gap: 30px;
            max-width: none;
        }

        .footer-info h3,
        .footer-cta h3 {
            margin-bottom: 15px;
            text-align: center;
            border-left: none;
            padding-left: 0;
            border-bottom: 2px solid var(--color-primary);
            padding-bottom: 5px;
        }

        .footer-info h3::after,
        .footer-cta h3::after {
            display: none;
        }

        .footer-info,
        .footer-cta {
            text-align: center;
        }

        .footer-info p {
            justify-content: center;
        }

        .footer-info p:hover {
            transform: translateX(0);
        }

        .social-icons {
            justify-content: center;
        }
    }
</style>
