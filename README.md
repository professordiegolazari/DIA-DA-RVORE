<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Monitor de Ruído - Projaf</title>
    <style>
        body { margin: 0; overflow: hidden; background: #0f172a; font-family: 'Segoe UI', sans-serif; color: #f8fafc; }
        canvas { display: block; }
        #setup {
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            background: rgba(30, 41, 59, 0.95); padding: 30px; border-radius: 20px;
            text-align: center; border: 2px solid #22c55e; box-shadow: 0 10px 25px rgba(0,0,0,0.5);
            z-index: 100; min-width: 300px;
        }
        button {
            background: #22c55e; color: white; border: none; padding: 15px 30px;
            font-size: 1.2rem; font-weight: bold; border-radius: 12px; cursor: pointer;
            transition: all 0.2s; box-shadow: 0 4px 0 #16a34a;
        }
        button:active { transform: translateY(4px); box-shadow: none; }
        #status { margin-top: 15px; font-size: 0.9rem; color: #94a3b8; }
        #ui { position: absolute; top: 20px; left: 20px; background: rgba(15, 23, 42, 0.8); padding: 10px; border-radius: 8px; pointer-events: auto; }
    </style>
</head>
<body>

    <div id="setup">
        <h2 style="margin-top:0">🔊 Monitor de Som</h2>
        <p>Clique no botão para iniciar as bolinhas no laboratório.</p>
        <button id="btnIniciar">ATIVAR MICROFONE</button>
        <div id="status">Aguardando permissão...</div>
    </div>

    <div id="ui" style="display:none">
        <label>Sensibilidade: <input type="range" id="sens" min="1" max="100" value="50"></label>
    </div>

    <canvas id="canvas"></canvas>

    <script>
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        const btn = document.getElementById('btnIniciar');
        const status = document.getElementById('status');
        const ui = document.getElementById('ui');

        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;

        let particles = [];
        let audioCtx, analyser, dataArray;

        class Particle {
            constructor() {
                this.r = Math.random() * 20 + 10;
                this.x = Math.random() * (canvas.width - this.r * 2) + this.r;
                this.y = canvas.height - this.r;
                this.color = `hsl(${Math.random() * 360}, 80%, 60%)`;
                this.vy = 0;
                this.vx = (Math.random() - 0.5) * 4;
            }
            update(volume) {
                const s = document.getElementById('sens').value * 0.01;
                if (volume > 20) this.vy -= (volume * s);
                
                this.vy += 0.8; // Gravidade
                this.y += this.vy;
                this.x += this.vx;

                if (this.y > canvas.height - this.r) {
                    this.y = canvas.height - this.r;
                    this.vy *= -0.7; // Rebote
                }
                if (this.x < this.r || this.x > canvas.width - this.r) this.vx *= -1;

                ctx.beginPath();
                ctx.arc(this.x, this.y, this.r, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.fill();
                ctx.strokeStyle = "white";
                ctx.stroke();
            }
        }

        async function initAudio() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
                audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                const source = audioCtx.createMediaStreamSource(stream);
                analyser = audioCtx.createAnalyser();
                analyser.fftSize = 256;
                source.connect(analyser);
                dataArray = new Uint8Array(analyser.frequencyBinCount);

                document.getElementById('setup').style.display = 'none';
                ui.style.display = 'block';
                for(let i=0; i<40; i++) particles.push(new Particle());
                animate();
            } catch (err) {
                status.style.color = "#ef4444";
                status.innerText = "ERRO: O navegador bloqueou o microfone. Use HTTPS ou um link externo.";
                console.error(err);
            }
        }

        function animate() {
            ctx.fillStyle = '#0f172a';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            analyser.getByteFrequencyData(dataArray);
            let avg = dataArray.reduce((a, b) => a + b) / dataArray.length;
            particles.forEach(p => p.update(avg));
            requestAnimationFrame(animate);
        }

        btn.onclick = initAudio;
        window.onresize = () => { canvas.width = window.innerWidth; canvas.height = window.innerHeight; };
    </script>
</body>
</html>
