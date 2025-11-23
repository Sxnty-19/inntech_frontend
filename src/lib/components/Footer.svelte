<svelte:head>
    <link
        rel="stylesheet"
        href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css"
        integrity="sha512-SnH5WK+bZxgPHs44uWIX+LLMD/CD+5N+yEStRjS1K48xSgH44n/B+xJ1A40k8c7oE9a9u5pG2Gq0wF04iP5w=="
        crossorigin="anonymous"
        referrerpolicy="no-referrer"
    />
    <style>
        /* Variables y estilos globales */
        :root {
            --color-primary: #1e3a8a;
            --color-secondary: #fcd34d;
            --color-dark: #0f172a;
            --color-text: #e2e8f0;
        }

        footer {
            background: var(--color-dark);
            color: var(--color-text);
            padding-top: 60px;
            padding-bottom: 20px;
            font-size: 0.95rem;
        }

        .footer-container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 0 20px 40px;
            /* Ya no usamos grid aquí. Usamos flexbox para centrar el contenido principal */
            display: flex;
            justify-content: center; /* Centra el contenido principal (footer-main-content) */
            border-bottom: 1px solid rgba(255, 255, 255, 0.1);
            margin-bottom: 20px;
        }

        /* NUEVO: Contenedor para las dos columnas, limitando su ancho */
        .footer-main-content {
            display: grid;
            grid-template-columns: 1fr 1fr; /* 2 columnas de igual ancho */
            gap: 120px; /* Espacio más amplio entre Contacto y Síguenos */
            max-width: 700px; /* Limitamos el ancho para que parezca centrado */
            width: 100%; /* Aseguramos que ocupe el espacio disponible */
        }

        /* Títulos */
        .footer-info h3,
        .footer-links h3,
        .footer-cta h3 {
            font-size: 1.3rem;
            font-weight: 700;
            color: var(--color-secondary);
            margin-bottom: 25px;
            border-left: 3px solid var(--color-primary);
            padding-left: 10px;
        }

        .footer-info p {
            margin: 10px 0;
            display: flex;
            align-items: center;
            line-height: 1.6;
            justify-content: flex-start;
        }

        .footer-icon {
            margin-right: 12px;
            font-size: 1.1em;
            color: var(--color-secondary);
            width: 18px;
            text-align: center;
        }

        /* Redes Sociales */
        .social-icons {
            margin-bottom: 20px;
            display: flex;
            gap: 12px;
            justify-content: flex-start;
        }

        .social-icon {
            background: var(--color-primary);
            width: 40px;
            height: 40px;
            border-radius: 50%;
            display: inline-flex;
            align-items: center;
            justify-content: center;
            text-decoration: none;
            font-size: 1.2rem;
            transition:
                background 0.3s,
                transform 0.3s;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.3);
        }

        .social-icon:hover {
            background: var(--color-secondary);
            transform: translateY(-3px);
        }

        .social-icon i {
            font-style: normal;
            line-height: 1;
            color: var(--color-light);
        }

        .footer-bottom {
            text-align: center;
            opacity: 0.7;
        }

        /* MEDIA QUERY para Móvil (Centrado total) */
        @media (max-width: 768px) {
            .footer-container {
                /* Vuelve a ser un contenedor flexible normal en móvil */
                display: block;
                padding-bottom: 20px;
            }

            .footer-main-content {
                /* Una sola columna en móvil */
                grid-template-columns: 1fr;
                gap: 30px;
                max-width: none; /* Quitamos el límite de ancho */
            }

            .footer-info h3,
            .footer-cta h3 {
                margin-bottom: 15px;
                text-align: center;
                /* Centrado del título en móvil */
                border-left: none;
                padding-left: 0;
                border-bottom: 2px solid var(--color-primary);
                padding-bottom: 5px;
            }

            .footer-info,
            .footer-cta {
                text-align: center;
            }

            .footer-info p {
                /* Centrar la línea de contacto */
                justify-content: center;
            }

            .social-icons {
                /* Centrar los iconos de redes sociales */
                justify-content: center;
            }
        }
    </style>
</svelte:head>

<footer>
    <div class="footer-container">
        <div class="footer-main-content">
            <div class="footer-info">
                <h3>Contáctanos</h3>
                <p>
                    <i class="fas fa-phone-alt footer-icon"></i>
                    +57 312 3694858
                </p>
                <p>
                    <i class="fas fa-envelope footer-icon"></i>
                    reservas@hospedajesur.com
                </p>
                <p>
                    <i class="fas fa-map-marker-alt footer-icon"></i>
                    Cra. 4#19A-09 Santa Marta
                </p>
            </div>

            <div class="footer-cta">
                <h3>Síguenos</h3>
                <div class="social-icons">
                    <a
                        href="https://facebook.com"
                        aria-label="Facebook"
                        class="social-icon facebook"
                    >
                        <i class="fab fa-facebook-f"></i>
                    </a>
                    <a
                        href="https://instagram.com"
                        aria-label="Instagram"
                        class="social-icon instagram"
                    >
                        <i class="fab fa-instagram"></i>
                    </a>
                    <a
                        href="https://twitter.com"
                        aria-label="Twitter"
                        class="social-icon twitter"
                    >
                        <i class="fab fa-twitter"></i>
                    </a>
                    <a
                        href="https://wa.me/numerodetelefono"
                        aria-label="WhatsApp"
                        class="social-icon whatsapp"
                    >
                        <i class="fab fa-whatsapp"></i>
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
