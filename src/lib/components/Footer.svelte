<svelte:head>
    <link
        rel="stylesheet"
        href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css"
        crossorigin="anonymous"
        referrerpolicy="no-referrer"
    />
    <style>
        :root {
            --color-primary: #1e3a8a;
            --color-secondary: #fcd34d;
            --color-mid-dark: #0f172a;
            --color-text: #e2e8f0;
        }

        footer {
            background: var(--color-mid-dark);
            color: var(--color-text);
            padding-top: 60px;
            padding-bottom: 20px;
            font-size: 0.95rem;
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
        }

        .footer-info h3,
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
        }

        .footer-icon {
            margin-right: 12px;
            font-size: 1.1em;
            color: var(--color-text);
            width: 18px;
            text-align: center;
        }

        .social-icons {
            margin-bottom: 20px;
            display: flex;
            gap: 12px;
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
            color: var(--color-text);
            line-height: 1;
        }

        .footer-bottom {
            text-align: center;
            opacity: 0.7;
        }

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

            .footer-info,
            .footer-cta {
                text-align: center;
            }

            .footer-info p {
                justify-content: center;
            }

            .social-icons {
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
