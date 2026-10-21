<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Juego 3D: Defensor Neón</title>
    <script src="https://code.playcanvas.com/playcanvas-stable.min.js"></script>
    <style>
        body { margin: 0; padding: 0; overflow: hidden; background-color: #000; }
        canvas { display: block; width: 100vw; height: 100vh; }
        #interfaz {
            position: absolute;
            top: 20px;
            left: 20px;
            color: white;
            font-family: system-ui, sans-serif;
            background: rgba(0, 0, 0, 0.8);
            padding: 15px;
            border-radius: 8px;
            border: 2px solid #ff47ff;
            box-shadow: 0 0 15px #ff47ff44;
        }
        #puntos { color: #ff47ff; font-size: 24px; font-weight: bold; }
    </style>
</head>
<body>

    <div id="interfaz">
        Impactos: <span id="puntos">0</span><br>
        <span style="font-size: 14px; color: #a2aec9;">Mover: Flechas | Disparar: ESPACIO</span>
    </div>

    <canvas id="mundo3D"></canvas>

    <script>
        const canvas = document.getElementById('mundo3D');
        const spanPuntos = document.getElementById('puntos');
        
        const app = new pc.Application(canvas, { keyboard: new pc.Keyboard(window) });
        app.start();
        app.setCanvasFillMode(pc.FILLMODE_FILL_WINDOW);
        app.setCanvasResolution(pc.RESOLUTION_AUTO);
        window.addEventListener('resize', () => app.resizeCanvas());

        // 1. ESCENARIO Y CÁMARA
        const camara = new pc.Entity();
        camara.addComponent('camera', { clearColor: new pc.Color(0.02, 0.02, 0.05) });
        camara.setPosition(0, 5, 10);
        camara.lookAt(0, 0, -10);
        app.root.addChild(camara);

        const luz = new pc.Entity();
        luz.addComponent('light', { type: 'directional', color: new pc.Color(1, 1, 1), intensity: 1.2 });
        luz.setEulerAngles(45, -30, 0);
        app.root.addChild(luz);

        // 2. JUGADOR
        const jugador = new pc.Entity();
        jugador.addComponent('render', { type: 'box' });
        jugador.setPosition(0, 0.5, 0);
        const matJugador = new pc.StandardMaterial();
        matJugador.diffuse.set(0.2, 1, 0.5); // Verde Neón
        matJugador.update();
        jugador.render.material = matJugador;
        app.root.addChild(jugador);

        // Materiales predefinidos para reutilizar
        const matBala = new pc.StandardMaterial();
        matBala.diffuse.set(1, 1, 0.2); // Amarillo
        matBala.emissive.set(1, 1, 0.2); // Que brille
        matBala.update();

        const matEnemigo = new pc.StandardMaterial();
        matEnemigo.diffuse.set(1, 0.2, 0.6); // Rosa/Rojo Neón
        matEnemigo.update();

        // 3. VARIABLES DEL JUEGO
        let puntos = 0;
        const balas = [];
        const enemigos = [];
        let tiempoRecarga = 0;
        let temporizadorEnemigos = 0;

        // 4. BUCLE PRINCIPAL
        app.on('update', function (dt) {
            
            // --- MOVIMIENTO DEL JUGADOR ---
            let posJugador = jugador.getPosition();
            if ((app.keyboard.isPressed(pc.KEY_LEFT) || app.keyboard.isPressed(pc.KEY_A)) && posJugador.x > -8) {
                jugador.translate(-12 * dt, 0, 0);
            }
            if ((app.keyboard.isPressed(pc.KEY_RIGHT) || app.keyboard.isPressed(pc.KEY_D)) && posJugador.x < 8) {
                jugador.translate(12 * dt, 0, 0);
            }

            // --- DISPARO ---
            tiempoRecarga -= dt;
            if (app.keyboard.isPressed(pc.KEY_SPACE) && tiempoRecarga <= 0) {
                tiempoRecarga = 0.25; // Tiempo entre balas (enfriamiento)
                
                const bala = new pc.Entity();
                bala.addComponent('render', { type: 'sphere' });
                bala.setLocalScale(0.3, 0.3, 0.3); // Hacerla pequeña
                bala.render.material = matBala;
                bala.setPosition(posJugador.x, 0.5, posJugador.z - 1);
                
                app.root.addChild(bala);
                balas.push(bala);
            }

            // --- GENERAR ENEMIGOS ---
            temporizadorEnemigos += dt;
            if (temporizadorEnemigos > 0.8) {
                temporizadorEnemigos = 0;
                const enemigo = new pc.Entity();
                enemigo.addComponent('render', { type: 'box' });
                enemigo.setLocalScale(1.5, 1.5, 1.5);
                enemigo.render.material = matEnemigo;
                
                let xAleatorio = (Math.random() * 16) - 8;
                enemigo.setPosition(xAleatorio, 0.5, -40); // Aparecen a lo lejos
                
                app.root.addChild(enemigo);
                enemigos.push(enemigo);
            }

            // --- MOVER BALAS ---
            for (let i = balas.length - 1; i >= 0; i--) {
                let b = balas[i];
                b.translate(0, 0, -30 * dt); // Mover hacia el fondo (-Z)
                if (b.getPosition().z < -50) {
                    b.destroy();
                    balas.splice(i, 1);
                }
            }

            // --- MOVER ENEMIGOS Y DETECTAR COLISIONES ---
            for (let e = enemigos.length - 1; e >= 0; e--) {
                let enemigo = enemigos[e];
                enemigo.translate(0, 0, 10 * dt); // Se acercan hacia el jugador (+Z)
                enemigo.rotate(30 * dt, 45 * dt, 0); // Rotación para efecto visual

                let posE = enemigo.getPosition();

                // Destruir si pasan de largo
                if (posE.z > 5) {
                    enemigo.destroy();
                    enemigos.splice(e, 1);
                    continue; // Saltar a la siguiente iteración
                }

                // Colisión Bala vs Enemigo (Bucle anidado)
                for (let b = balas.length - 1; b >= 0; b--) {
                    let bala = balas[b];
                    let posB = bala.getPosition();

                    let dx = posE.x - posB.x;
                    let dz = posE.z - posB.z;
                    
                    // Si están cerca, ambos se destruyen
                    if (Math.abs(dx) < 1.5 && Math.abs(dz) < 1.5) {
                        enemigo.destroy();
                        enemigos.splice(e, 1);
                        
                        bala.destroy();
                        balas.splice(b, 1);
                        
                        puntos++;
                        spanPuntos.textContent = puntos;
                        break; // Salir del bucle de balas porque el enemigo ya explotó
                    }
                }
            }
        });
    </script>
</body>
</html>