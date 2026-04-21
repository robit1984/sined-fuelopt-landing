[roulette.html](https://github.com/user-attachments/files/26916506/roulette.html)
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>🎰 РУЛЕТКА SINED • Нейро-фон</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background: black;
            font-family: 'Segoe UI', Roboto, sans-serif;
        }
        #canvas-neural {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 0;
        }
        #roulette-container {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            z-index: 10;
            text-align: center;
            color: white;
            pointer-events: auto;
        }
        .wheel-wrapper {
            position: relative;
            width: min(85vw, 450px);
            height: min(85vw, 450px);
            margin: 0 auto;
        }
        #wheelCanvas {
            width: 100% !important;
            height: 100% !important;
            filter: drop-shadow(0 0 30px #00d4ff);
        }
        .pointer {
            position: absolute;
            top: -15px;
            left: 50%;
            transform: translateX(-50%);
            width: 0;
            height: 0;
            border-left: 25px solid transparent;
            border-right: 25px solid transparent;
            border-top: 50px solid #ffd700;
            filter: drop-shadow(0 0 20px gold);
            z-index: 20;
        }
        .spin-btn {
            background: linear-gradient(145deg, #00d4ff, #0099cc);
            border: none;
            color: #0a0a0f;
            font-weight: bold;
            font-size: 28px;
            padding: 18px 50px;
            border-radius: 60px;
            margin: 20px 0;
            cursor: pointer;
            box-shadow: 0 0 40px #00d4ff;
            border: 1px solid rgba(255,255,255,0.3);
            transition: 0.2s;
            text-transform: uppercase;
            letter-spacing: 2px;
        }
        .spin-btn:active { transform: scale(0.95); }
        .spin-btn:disabled { opacity: 0.5; cursor: not-allowed; }
        .result {
            font-size: 32px;
            font-weight: bold;
            min-height: 50px;
            color: #00ffaa;
            text-shadow: 0 0 20px #00ffaa;
        }
        .back-btn {
            background: transparent;
            border: 1px solid #6c5ce7;
            color: #a29bfe;
            padding: 12px 30px;
            border-radius: 40px;
            font-size: 18px;
            cursor: pointer;
            margin-top: 10px;
        }
        h1 {
            color: #00d4ff;
            text-shadow: 0 0 20px #00d4ff;
            margin-bottom: 10px;
            font-size: 2.5rem;
        }
    </style>
    <!-- Three.js -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <!-- Confetti -->
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1"></script>
</head>
<body>
    <!-- Нейро-фон -->
    <canvas id="canvas-neural"></canvas>

    <!-- Рулетка -->
    <div id="roulette-container">
        <h1>🎰 SINED ROULETTE</h1>
        <div class="wheel-wrapper">
            <div class="pointer"></div>
            <canvas id="wheelCanvas" width="600" height="600"></canvas>
        </div>
        <button class="spin-btn" id="spinButton">🎰 КРУТИТЬ</button>
        <div class="result" id="result"></div>
        <button class="back-btn" id="closeButton">« Закрыть</button>
    </div>

    <script>
        // ========== РУЛЕТКА ==========
        const canvas = document.getElementById('wheelCanvas');
        const ctx = canvas.getContext('2d');
        
        const sectors = [
            { label: '100 Л', color: '#00d4ff', prize: 100 },
            { label: '50 Л', color: '#6c5ce7', prize: 50 },
            { label: '20 Л', color: '#ff6b6b', prize: 20 },
            { label: '10 Л', color: '#ffd93d', prize: 10 },
            { label: '5 Л', color: '#6bcb77', prize: 5 },
            { label: '0 Л', color: '#4d4d4d', prize: 0 },
        ];
        
        let rotation = 0;
        let spinning = false;
        
        function drawWheel() {
            const centerX = canvas.width / 2;
            const centerY = canvas.height / 2;
            const radius = Math.min(centerX, centerY) - 15;
            const angleStep = (Math.PI * 2) / sectors.length;
            
            sectors.forEach((sector, i) => {
                const startAngle = i * angleStep + rotation;
                const endAngle = startAngle + angleStep;
                
                ctx.beginPath();
                ctx.moveTo(centerX, centerY);
                ctx.arc(centerX, centerY, radius, startAngle, endAngle);
                ctx.closePath();
                
                ctx.fillStyle = sector.color;
                ctx.fill();
                ctx.strokeStyle = '#0a0a0f';
                ctx.lineWidth = 3;
                ctx.stroke();
                
                ctx.save();
                ctx.translate(centerX, centerY);
                ctx.rotate(startAngle + angleStep / 2);
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillStyle = '#fff';
                ctx.font = 'bold 22px Segoe UI';
                ctx.shadowColor = '#000';
                ctx.shadowBlur = 8;
                ctx.fillText(sector.label, radius * 0.7, 0);
                ctx.restore();
            });
            
            ctx.beginPath();
            ctx.arc(centerX, centerY, 30, 0, Math.PI * 2);
            ctx.fillStyle = '#0a0a0f';
            ctx.fill();
            ctx.strokeStyle = '#00d4ff';
            ctx.lineWidth = 4;
            ctx.stroke();
        }
        
        function getPrize() {
            const angleStep = (Math.PI * 2) / sectors.length;
            const pointerAngle = (rotation + Math.PI * 1.5) % (Math.PI * 2);
            const index = Math.floor(sectors.length - (pointerAngle / angleStep)) % sectors.length;
            return sectors[index].prize;
        }
        
        async function spin() {
            if (spinning) return;
            spinning = true;
            
            const btn = document.getElementById('spinButton');
            btn.disabled = true;
            document.getElementById('result').textContent = '';
            
            const spins = 5 + Math.floor(Math.random() * 5);
            const targetRotation = rotation + (Math.PI * 2 * spins) + Math.random() * Math.PI * 2;
            const duration = 2000;
            const startTime = Date.now();
            const startRotation = rotation;
            
            function animate() {
                const elapsed = Date.now() - startTime;
                const progress = Math.min(elapsed / duration, 1);
                const easeOut = 1 - Math.pow(1 - progress, 3);
                rotation = startRotation + (targetRotation - startRotation) * easeOut;
                
                drawWheel();
                
                if (progress < 1) {
                    requestAnimationFrame(animate);
                } else {
                    spinning = false;
                    btn.disabled = false;
                    
                    const prize = getPrize();
                    const resultDiv = document.getElementById('result');
                    
                    if (prize > 0) {
                        resultDiv.textContent = `🎉 ВЫИГРЫШ: +${prize} ЛИТРОВ!`;
                        confetti({ particleCount: 200, spread: 80, origin: { y: 0.6 } });
                        setTimeout(() => confetti({ particleCount: 150, spread: 100, origin: { y: 0.5, x: 0.3 } }), 200);
                        setTimeout(() => confetti({ particleCount: 150, spread: 100, origin: { y: 0.5, x: 0.7 } }), 400);
                    } else {
                        resultDiv.textContent = '😢 ПУСТО';
                    }
                }
            }
            
            animate();
        }
        
        drawWheel();
        document.getElementById('spinButton').addEventListener('click', spin);
        document.getElementById('closeButton').addEventListener('click', () => window.close());
    </script>

    <script>
        // ========== НЕЙРО-ФОН (КВАНТОВАЯ НЕЙРОСЕТЬ) ==========
        (function() {
            const canvas = document.getElementById('canvas-neural');
            const renderer = new THREE.WebGLRenderer({ canvas, alpha: false, antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setClearColor(0x050508);
            
            const scene = new THREE.Scene();
            scene.fog = new THREE.FogExp2(0x050508, 0.002);
            
            const camera = new THREE.PerspectiveCamera(65, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.set(0, 6, 22);
            
            // Звёзды
            const starsGeo = new THREE.BufferGeometry();
            const starsCount = 4000;
            const posArray = new Float32Array(starsCount * 3);
            const colorArray = new Float32Array(starsCount * 3);
            for (let i = 0; i < starsCount * 3; i += 3) {
                const r = 50 + Math.random() * 150;
                const theta = Math.random() * Math.PI * 2;
                const phi = Math.acos(2 * Math.random() - 1);
                posArray[i] = r * Math.sin(phi) * Math.cos(theta);
                posArray[i+1] = r * Math.sin(phi) * Math.sin(theta);
                posArray[i+2] = r * Math.cos(phi);
                
                const color = new THREE.Color().setHSL(0.6 + Math.random() * 0.3, 0.8, 0.5 + Math.random() * 0.3);
                colorArray[i] = color.r;
                colorArray[i+1] = color.g;
                colorArray[i+2] = color.b;
            }
            starsGeo.setAttribute('position', new THREE.BufferAttribute(posArray, 3));
            starsGeo.setAttribute('color', new THREE.BufferAttribute(colorArray, 3));
            const starsMat = new THREE.PointsMaterial({ size: 0.15, vertexColors: true, transparent: true, blending: THREE.AdditiveBlending });
            const stars = new THREE.Points(starsGeo, starsMat);
            scene.add(stars);
            
            // Частицы (нейроны)
            const particleGeo = new THREE.BufferGeometry();
            const particleCount = 800;
            const particlePos = new Float32Array(particleCount * 3);
            for (let i = 0; i < particleCount * 3; i += 3) {
                particlePos[i] = (Math.random() - 0.5) * 30;
                particlePos[i+1] = (Math.random() - 0.5) * 20;
                particlePos[i+2] = (Math.random() - 0.5) * 30;
            }
            particleGeo.setAttribute('position', new THREE.BufferAttribute(particlePos, 3));
            const particleMat = new THREE.PointsMaterial({ 
                color: 0x667eea, 
                size: 0.08, 
                transparent: true, 
                blending: THREE.AdditiveBlending 
            });
            const particles = new THREE.Points(particleGeo, particleMat);
            scene.add(particles);
            
            // Связи (линии)
            const linesGeo = new THREE.BufferGeometry();
            const linePositions = [];
            const lineColors = [];
            for (let i = 0; i < 200; i++) {
                const start = new THREE.Vector3(
                    (Math.random() - 0.5) * 25,
                    (Math.random() - 0.5) * 15,
                    (Math.random() - 0.5) * 25
                );
                const end = new THREE.Vector3(
                    start.x + (Math.random() - 0.5) * 8,
                    start.y + (Math.random() - 0.5) * 8,
                    start.z + (Math.random() - 0.5) * 8
                );
                linePositions.push(start.x, start.y, start.z);
                linePositions.push(end.x, end.y, end.z);
                
                const hue = 0.6 + Math.random() * 0.3;
                const color = new THREE.Color().setHSL(hue, 0.9, 0.5);
                lineColors.push(color.r, color.g, color.b);
                lineColors.push(color.r, color.g, color.b);
            }
            linesGeo.setAttribute('position', new THREE.Float32BufferAttribute(linePositions, 3));
            linesGeo.setAttribute('color', new THREE.Float32BufferAttribute(lineColors, 3));
            const linesMat = new THREE.LineBasicMaterial({ vertexColors: true, transparent: true, opacity: 0.25 });
            const lines = new THREE.LineSegments(linesGeo, linesMat);
            scene.add(lines);
            
            // Анимация
            let time = 0;
            function animate() {
                requestAnimationFrame(animate);
                time += 0.005;
                
                stars.rotation.y += 0.0001;
                particles.rotation.y += 0.001;
                lines.rotation.y += 0.0005;
                
                // Пульсация частиц
                particleMat.size = 0.08 + Math.sin(time * 5) * 0.02;
                
                renderer.render(scene, camera);
            }
            animate();
            
            window.addEventListener('resize', () => {
                camera.aspect = window.innerWidth / window.innerHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(window.innerWidth, window.innerHeight);
            });
        })();
    </script>
</body>
</html>
