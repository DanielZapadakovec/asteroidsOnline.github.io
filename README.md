<!DOCTYPE html>
<html lang="sk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Neon Asteroids - Multiplayer Arcade</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome for Arcade Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- PeerJS for WebRTC P2P connection across networks -->
    <script src="https://unpkg.com/peerjs@1.5.2/dist/peerjs.min.js"></script>
    <!-- QRCode.js for mobile pairing -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700;900&family=Rajdhani:wght@500;700&display=swap');

        * {
            user-select: none;
            -webkit-user-select: none;
            touch-action: manipulation;
        }

        body {
            font-family: 'Rajdhani', sans-serif;
            background-color: #030308;
            color: #fff;
            overflow: hidden;
        }

        .font-orbitron {
            font-family: 'Orbitron', sans-serif;
        }

        /* Custom Glowing & Neon Effects */
        .neon-box-cyan {
            border: 2px solid #00f3ff;
            box-shadow: 0 0 15px rgba(0, 243, 255, 0.4), inset 0 0 15px rgba(0, 243, 255, 0.2);
        }

        .neon-box-magenta {
            border: 2px solid #ff0055;
            box-shadow: 0 0 15px rgba(255, 0, 85, 0.4), inset 0 0 15px rgba(255, 0, 85, 0.2);
        }

        .neon-text-cyan {
            color: #00f3ff;
            text-shadow: 0 0 10px rgba(0, 243, 255, 0.7);
        }

        .neon-text-magenta {
            color: #ff0055;
            text-shadow: 0 0 10px rgba(255, 0, 85, 0.7);
        }

        .neon-button {
            background: rgba(0, 243, 255, 0.1);
            border: 1px solid #00f3ff;
            color: #00f3ff;
            transition: all 0.2s ease;
            box-shadow: 0 0 10px rgba(0, 243, 255, 0.2);
        }

        .neon-button:hover, .neon-button:active {
            background: rgba(0, 243, 255, 0.3);
            box-shadow: 0 0 20px rgba(0, 243, 255, 0.6);
            transform: scale(0.98);
        }

        .neon-button-start {
            background: linear-gradient(135deg, rgba(0, 243, 255, 0.3), rgba(255, 0, 85, 0.3));
            border: 2px solid #00f3ff;
            color: #ffffff;
            box-shadow: 0 0 20px rgba(0, 243, 255, 0.5);
        }

        .neon-button-start:hover {
            box-shadow: 0 0 30px rgba(0, 243, 255, 0.8), 0 0 15px rgba(255, 0, 85, 0.8);
            transform: scale(1.03);
        }

        .neon-button-fire {
            background: rgba(255, 0, 85, 0.2);
            border: 2px solid #ff0055;
            color: #ff0055;
            box-shadow: 0 0 15px rgba(255, 0, 85, 0.4);
        }

        .neon-button-fire:active {
            background: rgba(255, 0, 85, 0.5);
            box-shadow: 0 0 30px rgba(255, 0, 85, 0.9);
            transform: scale(0.95);
        }

        /* Glassmorphism Panels */
        .glass-panel {
            background: rgba(10, 10, 25, 0.88);
            backdrop-filter: blur(14px);
            border: 1px solid rgba(255, 255, 255, 0.12);
        }

        /* Virtual Joystick styling */
        #joystick-container {
            position: relative;
            width: 140px;
            height: 140px;
            border-radius: 50%;
            background: rgba(0, 243, 255, 0.05);
            border: 2px dashed rgba(0, 243, 255, 0.3);
        }

        #joystick-knob {
            position: absolute;
            width: 50px;
            height: 50px;
            border-radius: 50%;
            background: radial-gradient(circle, #00f3ff 0%, rgba(0,243,255,0.4) 100%);
            box-shadow: 0 0 15px #00f3ff;
            top: 45px;
            left: 45px;
            transform: translate(0, 0);
        }

        /* Color Picker Buttons */
        .color-dot {
            width: 36px;
            height: 36px;
            border-radius: 50%;
            cursor: pointer;
            transition: all 0.2s ease;
            border: 2px solid transparent;
        }

        .color-dot.selected {
            border-color: #ffffff;
            transform: scale(1.25);
            box-shadow: 0 0 15px currentColor;
        }
    </style>
</head>
<body class="w-screen h-screen flex flex-col justify-center items-center relative select-none">

    <!-- MAIN HOST CONTAINER (PC Screen) -->
    <div id="host-screen" class="w-full h-full relative flex flex-col justify-center items-center">
        <!-- Canvas Background Game Frame -->
        <canvas id="gameCanvas" class="absolute inset-0 w-full h-full z-0"></canvas>

        <!-- LOBBY overlay (Before Game Starts) -->
        <div id="lobby-screen" class="absolute inset-0 z-20 glass-panel flex flex-col items-center justify-between p-6 overflow-y-auto">
            <!-- Header -->
            <div class="text-center mt-2">
                <h1 class="font-orbitron text-3xl md:text-5xl font-black neon-text-cyan flex items-center justify-center gap-3">
                    <i class="fa-solid fa-meteor text-cyan-400"></i> NEON ASTEROIDS
                </h1>
                <p class="text-sm text-purple-300 font-bold tracking-widest mt-1">MULTIPLAYER ARCADE PVP / PVE LOBBY</p>
            </div>

            <!-- Main Connection Hub Grid -->
            <div class="grid grid-cols-1 md:grid-cols-2 gap-8 w-full max-w-4xl my-auto items-center">
                <!-- Left Box: Connection Details & QR Code -->
                <div class="glass-panel p-6 rounded-2xl neon-box-cyan flex flex-col items-center text-center">
                    <div id="qrcode" class="bg-white p-2 rounded-xl shadow-lg mb-4 border-2 border-cyan-400"></div>
                    <div class="text-xs text-cyan-300 font-semibold uppercase tracking-widest">Kód Miestnosti (Room ID)</div>
                    <div id="room-code-display" class="font-orbitron text-2xl font-black text-cyan-400 tracking-wider my-1">
                        PRIPÁJANIE...
                    </div>
                    <p class="text-xs text-gray-300 font-mono mt-1 break-all px-2">
                        Pripoj mobil skenom alebo cez URL: <br>
                        <span id="room-url" class="text-yellow-300 font-bold underline"></span>
                    </p>
                    <div class="mt-4 bg-cyan-950/60 p-2.5 rounded-lg border border-cyan-500/30 text-[11px] text-cyan-200 text-left flex items-start gap-2">
                        <i class="fa-solid fa-globe text-cyan-400 text-base mt-0.5"></i>
                        <span><strong>Siete:</strong> Hráči sa môžu pripojiť z <strong>akejkoľvek siete</strong> (Wi-Fi, 4G/5G dáta alebo hotspot).</span>
                    </div>
                </div>

                <!-- Right Box: Connected Players List & Host Controls -->
                <div class="glass-panel p-6 rounded-2xl neon-box-magenta flex flex-col h-full justify-between">
                    <div>
                        <div class="flex justify-between items-center border-b border-gray-700 pb-2 mb-3">
                            <h3 class="font-orbitron text-sm text-purple-300 font-bold tracking-wider">
                                PRIHLÁSENÍ HRÁČI (<span id="connected-count">0</span>/8)
                            </h3>
                            <span class="text-xs text-green-400 font-bold animate-pulse">● Čaká sa na štart</span>
                        </div>
                        <div id="player-cards-container" class="grid grid-cols-1 gap-2 max-h-48 overflow-y-auto pr-1">
                            <div class="text-xs text-gray-500 italic text-center py-4">Žiadni pripojení hráči. Naskenujte QR kód na mobile!</div>
                        </div>
                    </div>

                    <div class="mt-6 flex flex-col gap-2">
                        <button id="btn-start-game" onclick="startGameFromLobby()" class="w-full py-4 rounded-xl font-orbitron font-black text-lg neon-button-start transition-all">
                            <i class="fa-solid fa-play mr-2"></i> SPUSTIŤ HRU (PLAY)
                        </button>
                        <div class="text-[11px] text-center text-gray-400">
                            * Môžete spustiť hru aj pre 1 hráča (PC klávesnica alebo 1 mobil).
                        </div>
                    </div>
                </div>
            </div>

            <!-- Footer Hints -->
            <div class="text-xs text-gray-400 font-mono text-center">
                Ovládanie na PC (test): <span class="text-cyan-300">WASD / Šípky</span> = Pohyb, <span class="text-cyan-300">Medzerník</span> = Streľba
            </div>
        </div>

        <!-- IN-GAME HUD Overlay -->
        <div id="game-hud" class="hidden absolute top-4 left-4 right-4 z-10 flex justify-between items-start pointer-events-none">
            <!-- Game Title HUD -->
            <div class="glass-panel p-2.5 px-4 rounded-xl border border-cyan-500/30 flex items-center gap-3">
                <div class="font-orbitron text-lg font-black neon-text-cyan flex items-center gap-2">
                    <i class="fa-solid fa-meteor text-cyan-400"></i> NEON ASTEROIDS
                </div>
                <button onclick="returnToLobby()" class="pointer-events-auto text-xs neon-button px-2.5 py-1 rounded-md">
                    <i class="fa-solid fa-bars"></i> Lobby
                </button>
            </div>

            <!-- Live Score Leaderboard -->
            <div class="glass-panel rounded-xl p-3 border border-cyan-500/20 w-60">
                <h3 class="font-orbitron text-xs text-gray-300 border-b border-gray-700 pb-1 mb-2 tracking-wider flex justify-between">
                    <span>REBRÍČEK (LEADERBOARD)</span>
                    <span class="text-cyan-400"><i class="fa-solid fa-trophy"></i></span>
                </h3>
                <ul id="leaderboard-list" class="space-y-1 text-sm font-semibold">
                    <li class="text-xs text-gray-500 italic">Žiadni aktívni hráči...</li>
                </ul>
            </div>
        </div>
    </div>

    <!-- CONTROLLER SCREEN (Mobile Screen) -->
    <div id="controller-screen" class="hidden absolute inset-0 w-full h-full z-50 glass-panel flex flex-col justify-between p-4 bg-slate-950">
        
        <!-- STEP 1: MOBILE PROFILE SETUP SCREEN -->
        <div id="ctrl-setup-step" class="w-full h-full flex flex-col justify-center items-center max-w-sm mx-auto my-auto">
            <div class="glass-panel p-6 rounded-2xl neon-box-cyan w-full text-center">
                <h2 class="font-orbitron text-xl font-bold neon-text-cyan mb-1">PROFIL HRÁČA</h2>
                <p class="text-xs text-gray-400 mb-6">Nastav si meno a farbu lode do hry</p>

                <!-- Name Input -->
                <div class="text-left mb-5">
                    <label class="text-xs text-cyan-300 font-bold uppercase tracking-wider block mb-2">Tvoje Meno / Nick:</label>
                    <input type="text" id="player-name-input" maxlength="12" placeholder="PILOT 1" value="" class="w-full bg-black/60 border border-cyan-500/50 rounded-xl px-4 py-3 text-cyan-200 font-orbitron text-center focus:outline-none focus:border-cyan-400">
                </div>

                <!-- Color Palette -->
                <div class="text-left mb-6">
                    <label class="text-xs text-cyan-300 font-bold uppercase tracking-wider block mb-2">Farba Lode:</label>
                    <div id="color-palette" class="flex flex-wrap justify-center gap-3">
                        <!-- Color buttons dynamically generated in JS -->
                    </div>
                </div>

                <!-- Submit Button -->
                <button onclick="confirmMobileProfile()" class="w-full py-3.5 rounded-xl font-orbitron font-bold neon-button-start text-sm">
                    PRIPOJIŤ SA DO HRY <i class="fa-solid fa-arrow-right ml-1"></i>
                </button>
            </div>
        </div>

        <!-- STEP 2: ACTIVE CONTROLLER INTERFACE -->
        <div id="ctrl-active-step" class="hidden w-full h-full flex flex-col justify-between">
            <!-- Controller Top Status Bar -->
            <div class="flex justify-between items-center bg-black/50 p-2.5 px-3 rounded-xl border border-cyan-500/30">
                <div class="flex items-center gap-2">
                    <div id="player-color-badge" class="w-4 h-4 rounded-full bg-cyan-400 shadow-lg"></div>
                    <div>
                        <div id="ctrl-player-name" class="font-orbitron font-bold text-xs text-cyan-300">PILOT</div>
                        <div id="ctrl-player-stats" class="text-[11px] text-gray-400 font-mono">Body: 0 | Kills: 0</div>
                    </div>
                </div>
                <div class="text-right">
                    <div id="ctrl-game-status" class="text-xs font-bold text-yellow-400 flex items-center gap-1 justify-end">
                        <span class="w-2 h-2 rounded-full bg-yellow-400 animate-ping"></span> V LOBBY
                    </div>
                </div>
            </div>

            <!-- Controller Main Inputs Area -->
            <div class="flex-1 flex justify-between items-center my-3 gap-2">
                <!-- Left Side: Joystick -->
                <div class="flex-1 flex justify-center items-center">
                    <div id="joystick-container" class="flex justify-center items-center">
                        <div id="joystick-knob"></div>
                    </div>
                </div>

                <!-- Right Side: Action Buttons -->
                <div class="flex-1 flex flex-col justify-center items-center gap-3">
                    <button id="btn-fire" class="w-24 h-24 rounded-full neon-button-fire font-orbitron font-black text-lg flex flex-col items-center justify-center gap-1 shadow-2xl active:scale-95 transition-transform">
                        <i class="fa-solid fa-crosshairs text-xl"></i>
                        <span>PAL</span>
                    </button>

                    <div class="flex gap-2 w-full justify-center">
                        <button id="btn-thrust" class="neon-button px-4 py-2.5 rounded-xl font-orbitron text-xs font-bold flex items-center gap-1">
                            <i class="fa-solid fa-rocket"></i> PLYN
                        </button>
                        <button id="btn-brake" class="neon-button px-4 py-2.5 rounded-xl font-orbitron text-xs font-bold flex items-center gap-1 border-purple-500 text-purple-300">
                            <i class="fa-solid fa-hand-paper"></i> BRZDA
                        </button>
                    </div>
                </div>
            </div>

            <!-- Controller Bottom Info -->
            <div class="text-center text-[10px] text-gray-500 font-mono uppercase tracking-widest">
                NEON CONTROLLER v2.0 • HAPTIC FEEDBACK READY
            </div>
        </div>
    </div>

    <script>
        // Synthesized Web Audio Engine
        class SoundEngine {
            constructor() { this.ctx = null; }

            init() {
                if (!this.ctx) {
                    this.ctx = new (window.AudioContext || window.webkitAudioContext)();
                }
            }

            playShoot() {
                if (!this.ctx) return;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(880, this.ctx.currentTime);
                osc.frequency.exponentialRampToValueAtTime(110, this.ctx.currentTime + 0.15);
                
                gain.gain.setValueAtTime(0.2, this.ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + 0.15);

                osc.connect(gain);
                gain.connect(this.ctx.destination);

                osc.start();
                osc.stop(this.ctx.currentTime + 0.15);
            }

            playExplosion(isPlayer = false) {
                if (!this.ctx) return;
                const dur = isPlayer ? 0.6 : 0.3;
                const bufferSize = this.ctx.sampleRate * dur;
                const buffer = this.ctx.createBuffer(1, bufferSize, this.ctx.sampleRate);
                const data = buffer.getChannelData(0);
                for (let i = 0; i < bufferSize; i++) {
                    data[i] = Math.random() * 2 - 1;
                }

                const noise = this.ctx.createBufferSource();
                noise.buffer = buffer;

                const filter = this.ctx.createBiquadFilter();
                filter.type = 'lowpass';
                filter.frequency.setValueAtTime(isPlayer ? 400 : 800, this.ctx.currentTime);
                filter.frequency.linearRampToValueAtTime(50, this.ctx.currentTime + dur);

                const gain = this.ctx.createGain();
                gain.gain.setValueAtTime(0.4, this.ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + dur);

                noise.connect(filter);
                filter.connect(gain);
                gain.connect(this.ctx.destination);

                noise.start();
            }

            playThrust() {
                if (!this.ctx) return;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'triangle';
                osc.frequency.setValueAtTime(60, this.ctx.currentTime);
                gain.gain.setValueAtTime(0.05, this.ctx.currentTime);
                gain.gain.linearRampToValueAtTime(0, this.ctx.currentTime + 0.08);

                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start();
                osc.stop(this.ctx.currentTime + 0.08);
            }
        }

        const audioFX = new SoundEngine();

        const NEON_COLORS = [
            '#00f3ff', // Cyan
            '#ff0055', // Magenta
            '#00ff66', // Green
            '#ffdd00', // Yellow
            '#ff6600', // Orange
            '#b000ff', // Purple
            '#ff00aa', // Hot Pink
            '#00ffff'  // Electric Blue
        ];

        // Global State Variables
        let isController = false;
        let gameStarted = false;
        let selectedColor = NEON_COLORS[0];
        let peer = null;
        let peerConnections = {};
        let canvas, ctx;
        let gameWidth, gameHeight;

        // Entities
        let players = {};
        let asteroids = [];
        let bullets = [];
        let particles = [];

        // Local Keyboard Controls
        const keys = { left: false, right: false, thrust: false, brake: false, fire: false };

        class Ship {
            constructor(id, name, color, isLocal = false) {
                this.id = id;
                this.name = name || 'PILOT';
                this.color = color || '#00f3ff';
                this.isLocal = isLocal;
                this.radius = 16;
                this.score = 0;
                this.kills = 0;
                this.asteroidsDestroyed = 0;
                this.respawnTimer = 0;
                this.shieldTimer = 180; // 3 sec shield
                this.alive = true;
                this.angle = -Math.PI / 2;
                this.rotation = 0;
                this.thrusting = false;
                this.braking = false;
                this.lastFired = 0;
                this.resetPosition();
            }

            resetPosition(allShips = [], allAsteroids = []) {
                let safe = false;
                let attempts = 0;
                let rx, ry;

                while (!safe && attempts < 100) {
                    attempts++;
                    rx = Math.random() * (gameWidth || 800);
                    ry = Math.random() * (gameHeight || 600);
                    safe = true;

                    // Avoid Asteroids
                    for (let ast of allAsteroids) {
                        let dist = Math.hypot(rx - ast.x, ry - ast.y);
                        if (dist < ast.radius + 120) { safe = false; break; }
                    }

                    // Avoid Other Players
                    if (safe) {
                        for (let pid in allShips) {
                            let other = allShips[pid];
                            if (other.id !== this.id && other.alive) {
                                let dist = Math.hypot(rx - other.x, ry - other.y);
                                if (dist < 150) { safe = false; break; }
                            }
                        }
                    }
                }

                this.x = rx || (gameWidth / 2);
                this.y = ry || (gameHeight / 2);
                this.vx = 0;
                this.vy = 0;
                this.angle = -Math.PI / 2;
                this.alive = true;
                this.shieldTimer = 180;
            }

            update() {
                if (!this.alive) {
                    if (this.respawnTimer > 0) {
                        this.respawnTimer--;
                        if (this.respawnTimer === 0) {
                            this.resetPosition(players, asteroids);
                        }
                    }
                    return;
                }

                if (this.shieldTimer > 0) this.shieldTimer--;

                // Steering
                this.angle += this.rotation;

                // Thrust
                if (this.thrusting) {
                    this.vx += Math.cos(this.angle) * 0.16;
                    this.vy += Math.sin(this.angle) * 0.16;

                    // Exhaust Trail Particles
                    if (Math.random() < 0.6) {
                        particles.push(new Particle(
                            this.x - Math.cos(this.angle) * 14,
                            this.y - Math.sin(this.angle) * 14,
                            -Math.cos(this.angle) * 2 + (Math.random() - 0.5),
                            -Math.sin(this.angle) * 2 + (Math.random() - 0.5),
                            this.color,
                            20,
                            Math.random() * 3 + 1
                        ));
                    }
                }

                // Dampening
                const friction = this.braking ? 0.91 : 0.985;
                this.vx *= friction;
                this.vy *= friction;

                // Cap max velocity
                const speed = Math.hypot(this.vx, this.vy);
                const maxSpeed = 8.5;
                if (speed > maxSpeed) {
                    this.vx = (this.vx / speed) * maxSpeed;
                    this.vy = (this.vy / speed) * maxSpeed;
                }

                this.x += this.vx;
                this.y += this.vy;

                // Screen Wrap
                if (this.x < 0) this.x = gameWidth;
                if (this.x > gameWidth) this.x = 0;
                if (this.y < 0) this.y = gameHeight;
                if (this.y > gameHeight) this.y = 0;
            }

            draw(ctx) {
                if (!this.alive) return;

                ctx.save();
                ctx.translate(this.x, this.y);
                ctx.rotate(this.angle);

                // Shield Effect
                if (this.shieldTimer > 0) {
                    ctx.beginPath();
                    ctx.arc(0, 0, this.radius + 8, 0, Math.PI * 2);
                    ctx.strokeStyle = '#00f3ff';
                    ctx.lineWidth = 2;
                    ctx.shadowBlur = 15;
                    ctx.shadowColor = '#00f3ff';
                    ctx.setLineDash([6, 6]);
                    ctx.stroke();
                    ctx.setLineDash([]);
                }

                // Ship Body
                ctx.beginPath();
                ctx.moveTo(18, 0);
                ctx.lineTo(-14, -12);
                ctx.lineTo(-8, 0);
                ctx.lineTo(-14, 12);
                ctx.closePath();

                ctx.strokeStyle = this.color;
                ctx.lineWidth = 2.5;
                ctx.fillStyle = '#030308';
                ctx.shadowBlur = 14;
                ctx.shadowColor = this.color;
                ctx.fill();
                ctx.stroke();

                // Thrust Flame
                if (this.thrusting) {
                    ctx.beginPath();
                    ctx.moveTo(-10, -6);
                    ctx.lineTo(-24 - Math.random() * 8, 0);
                    ctx.lineTo(-10, 6);
                    ctx.strokeStyle = '#ff9900';
                    ctx.shadowColor = '#ff9900';
                    ctx.shadowBlur = 10;
                    ctx.stroke();
                }

                ctx.restore();

                // Name Tag
                ctx.save();
                ctx.font = '700 11px Orbitron';
                ctx.fillStyle = this.color;
                ctx.textAlign = 'center';
                ctx.shadowBlur = 8;
                ctx.shadowColor = this.color;
                ctx.fillText(this.name, this.x, this.y - this.radius - 8);
                ctx.restore();
            }

            destroy() {
                if (!this.alive) return;
                this.alive = false;
                this.respawnTimer = 180;
                audioFX.playExplosion(true);

                // Shockwave Particles
                for (let i = 0; i < 35; i++) {
                    let pAngle = Math.random() * Math.PI * 2;
                    let pSpeed = Math.random() * 5 + 1;
                    particles.push(new Particle(
                        this.x, this.y,
                        Math.cos(pAngle) * pSpeed,
                        Math.sin(pAngle) * pSpeed,
                        this.color,
                        40,
                        Math.random() * 4 + 2
                    ));
                }
            }
        }

        class Asteroid {
            constructor(x, y, radius) {
                this.x = x || Math.random() * gameWidth;
                this.y = y || Math.random() * gameHeight;
                this.radius = radius || 40;
                
                let speed = Math.random() * 1.5 + 0.5;
                let angle = Math.random() * Math.PI * 2;
                this.vx = Math.cos(angle) * speed;
                this.vy = Math.sin(angle) * speed;

                this.vertexCount = Math.floor(Math.random() * 4) + 7;
                this.offsets = [];
                for (let i = 0; i < this.vertexCount; i++) {
                    this.offsets.push((Math.random() * 0.4 + 0.8));
                }
                this.angle = 0;
                this.rotSpeed = (Math.random() - 0.5) * 0.02;
            }

            update() {
                this.x += this.vx;
                this.y += this.vy;
                this.angle += this.rotSpeed;

                if (this.x < -this.radius) this.x = gameWidth + this.radius;
                if (this.x > gameWidth + this.radius) this.x = -this.radius;
                if (this.y < -this.radius) this.y = gameHeight + this.radius;
                if (this.y > gameHeight + this.radius) this.y = -this.radius;
            }

            draw(ctx) {
                ctx.save();
                ctx.translate(this.x, this.y);
                ctx.rotate(this.angle);

                ctx.beginPath();
                for (let i = 0; i < this.vertexCount; i++) {
                    let a = (i / this.vertexCount) * Math.PI * 2;
                    let r = this.radius * this.offsets[i];
                    let px = Math.cos(a) * r;
                    let py = Math.sin(a) * r;
                    if (i === 0) ctx.moveTo(px, py);
                    else ctx.lineTo(px, py);
                }
                ctx.closePath();

                ctx.strokeStyle = '#a0a0c0';
                ctx.lineWidth = 2;
                ctx.shadowBlur = 8;
                ctx.shadowColor = '#8080a0';
                ctx.fillStyle = 'rgba(20, 20, 35, 0.7)';
                ctx.fill();
                ctx.stroke();

                ctx.restore();
            }
        }

        class Bullet {
            constructor(x, y, angle, ownerId, color) {
                this.x = x;
                this.y = y;
                this.vx = Math.cos(angle) * 12.5;
                this.vy = Math.sin(angle) * 12.5;
                this.ownerId = ownerId;
                this.color = color;
                this.life = 60;
                this.trail = [];
            }

            update() {
                this.trail.push({ x: this.x, y: this.y });
                if (this.trail.length > 6) this.trail.shift();

                this.x += this.vx;
                this.y += this.vy;
                this.life--;

                if (this.x < 0) this.x = gameWidth;
                if (this.x > gameWidth) this.x = 0;
                if (this.y < 0) this.y = gameHeight;
                if (this.y > gameHeight) this.y = 0;
            }

            draw(ctx) {
                ctx.save();

                // Motion Trail
                ctx.beginPath();
                if (this.trail.length > 0) {
                    ctx.moveTo(this.trail[0].x, this.trail[0].y);
                    for (let pt of this.trail) {
                        ctx.lineTo(pt.x, pt.y);
                    }
                }
                ctx.strokeStyle = this.color;
                ctx.lineWidth = 2;
                ctx.globalAlpha = 0.6;
                ctx.stroke();

                // Lightning Under-Glow Auspect
                ctx.beginPath();
                let perpX = -this.vy * 0.3;
                let perpY = this.vx * 0.3;
                ctx.moveTo(this.x - perpX, this.y - perpY);
                ctx.lineTo(this.x + perpX, this.y + perpY);
                ctx.strokeStyle = '#ffffff';
                ctx.lineWidth = 3;
                ctx.shadowBlur = 12;
                ctx.shadowColor = this.color;
                ctx.stroke();

                // Core Bullet Dot
                ctx.beginPath();
                ctx.arc(this.x, this.y, 3.5, 0, Math.PI * 2);
                ctx.fillStyle = '#ffffff';
                ctx.shadowBlur = 15;
                ctx.shadowColor = this.color;
                ctx.fill();

                ctx.restore();
            }
        }

        class Particle {
            constructor(x, y, vx, vy, color, life, size) {
                this.x = x;
                this.y = y;
                this.vx = vx;
                this.vy = vy;
                this.color = color;
                this.life = life;
                this.maxLife = life;
                this.size = size || 2;
            }

            update() {
                this.x += this.vx;
                this.y += this.vy;
                this.life--;
            }

            draw(ctx) {
                ctx.save();
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.shadowBlur = 6;
                ctx.shadowColor = this.color;
                ctx.globalAlpha = this.life / this.maxLife;
                ctx.fill();
                ctx.restore();
            }
        }

        window.onload = function () {
            canvas = document.getElementById('gameCanvas');
            ctx = canvas.getContext('2d');

            resizeCanvas();
            window.addEventListener('resize', resizeCanvas);

            // Check if Mobile Controller Join URL
            const urlParams = new URLSearchParams(window.location.search);
            const joinCode = urlParams.get('join');

            if (joinCode) {
                initControllerMode(joinCode);
            } else {
                initHostMode();
            }

            // Keyboard listeners for PC testing
            window.addEventListener('keydown', e => handleKey(e, true));
            window.addEventListener('keyup', e => handleKey(e, false));

            requestAnimationFrame(gameLoop);
        };

        function resizeCanvas() {
            gameWidth = window.innerWidth;
            gameHeight = window.innerHeight;
            canvas.width = gameWidth;
            canvas.height = gameHeight;
        }

        function handleKey(e, isDown) {
            if (isController) return;
            if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') keys.left = isDown;
            if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') keys.right = isDown;
            if (e.key === 'ArrowUp' || e.key === 'w' || e.key === 'W') keys.thrust = isDown;
            if (e.key === 'ArrowDown' || e.key === 's' || e.key === 'S') keys.brake = isDown;
            if (e.key === ' ' || e.key === 'Spacebar') keys.fire = isDown;

            audioFX.init();
        }

        function initHostMode() {
            const roomCode = 'NA-' + Math.floor(1000 + Math.random() * 9000);
            document.getElementById('room-code-display').innerText = roomCode;

            // Initialize WebRTC Peer with cloud STUN/TURN support
            peer = new Peer(roomCode, {
                config: {
                    iceServers: [
                        { urls: 'stun:stun.l.google.com:19302' },
                        { urls: 'stun:stun1.l.google.com:19302' }
                    ]
                }
            });

            peer.on('open', (id) => {
                const joinUrl = window.location.origin + window.location.pathname + '?join=' + id;
                document.getElementById('room-url').innerText = joinUrl;

                new QRCode(document.getElementById("qrcode"), {
                    text: joinUrl,
                    width: 110,
                    height: 110,
                    colorDark: "#00f3ff",
                    colorLight: "#030308",
                    correctLevel: QRCode.CorrectLevel.L
                });
            });

            peer.on('connection', (conn) => {
                conn.on('open', () => {
                    const playerId = conn.peer;
                    peerConnections[playerId] = conn;
                });

                conn.on('data', (data) => {
                    if (data.type === 'REGISTER_PROFILE') {
                        const playerId = data.id;
                        const newShip = new Ship(playerId, data.name, data.color);
                        players[playerId] = newShip;

                        updateLobbyUI();

                        // ACK back to mobile
                        conn.send({
                            type: 'PROFILE_ACK',
                            gameStarted: gameStarted
                        });
                    }

                    if (data.type === 'INPUT' && players[data.id]) {
                        const ship = players[data.id];
                        ship.rotation = data.steer * 0.07;
                        ship.thrusting = data.thrust;
                        ship.braking = data.brake;

                        if (data.thrust && gameStarted) audioFX.playThrust();

                        if (data.fire && gameStarted && Date.now() - ship.lastFired > 180 && ship.alive) {
                            ship.lastFired = Date.now();
                            bullets.push(new Bullet(
                                ship.x + Math.cos(ship.angle) * 20,
                                ship.y + Math.sin(ship.angle) * 20,
                                ship.angle,
                                ship.id,
                                ship.color
                            ));
                            audioFX.playShoot();
                        }
                    }
                });

                conn.on('close', () => {
                    delete players[conn.peer];
                    delete peerConnections[conn.peer];
                    updateLobbyUI();
                });
            });

            // Local Host Ship for testing
            const localShip = new Ship('local_host', 'HOST (PC)', '#00f3ff', true);
            players['local_host'] = localShip;
            updateLobbyUI();

            spawnAsteroidWave(6);
        }

        function updateLobbyUI() {
            const container = document.getElementById('player-cards-container');
            const pKeys = Object.keys(players);
            document.getElementById('connected-count').innerText = pKeys.length;

            if (pKeys.length === 0) {
                container.innerHTML = `<div class="text-xs text-gray-500 italic text-center py-4">Žiadni pripojení hráči. Naskenujte QR kód na mobile!</div>`;
                return;
            }

            container.innerHTML = pKeys.map(pid => {
                const p = players[pid];
                return `
                    <div class="flex items-center justify-between bg-black/40 p-2.5 rounded-xl border-l-4" style="border-color: ${p.color}">
                        <div class="flex items-center gap-2">
                            <div class="w-3.5 h-3.5 rounded-full" style="background-color: ${p.color}; box-shadow: 0 0 10px ${p.color}"></div>
                            <span class="font-orbitron font-bold text-xs text-cyan-200">${p.name} ${p.isLocal ? '(PC)' : ''}</span>
                        </div>
                        <span class="text-[10px] font-bold px-2 py-0.5 rounded ${gameStarted ? 'bg-green-500/20 text-green-400' : 'bg-yellow-500/20 text-yellow-400'}">
                            ${gameStarted ? 'V HRE' : 'PRIPRAVENÝ'}
                        </span>
                    </div>
                `;
            }).join('');
        }

        function startGameFromLobby() {
            gameStarted = true;
            document.getElementById('lobby-screen').classList.add('hidden');
            document.getElementById('game-hud').classList.remove('hidden');

            // Notify all mobile controllers
            for (let pid in peerConnections) {
                peerConnections[pid].send({ type: 'GAME_STARTED' });
            }
        }

        function returnToLobby() {
            gameStarted = false;
            document.getElementById('lobby-screen').classList.remove('hidden');
            document.getElementById('game-hud').classList.add('hidden');
            updateLobbyUI();
        }

        function spawnAsteroidWave(count) {
            for (let i = 0; i < count; i++) {
                asteroids.push(new Asteroid());
            }
        }

        function initControllerMode(hostCode) {
            isController = true;
            document.getElementById('host-screen').classList.add('hidden');
            document.getElementById('controller-screen').classList.remove('hidden');

            // Build Color Picker UI
            const palette = document.getElementById('color-palette');
            NEON_COLORS.forEach((color, idx) => {
                const btn = document.createElement('div');
                btn.className = `color-dot ${idx === 0 ? 'selected' : ''}`;
                btn.style.backgroundColor = color;
                btn.onclick = () => {
                    document.querySelectorAll('.color-dot').forEach(d => d.classList.remove('selected'));
                    btn.classList.add('selected');
                    selectedColor = color;
                };
                palette.appendChild(btn);
            });

            // Random default player name
            document.getElementById('player-name-input').value = 'PILOT ' + Math.floor(10 + Math.random() * 90);

            peer = new Peer(null, {
                config: {
                    iceServers: [
                        { urls: 'stun:stun.l.google.com:19302' },
                        { urls: 'stun:stun1.l.google.com:19302' }
                    ]
                }
            });

            peer.on('open', (myId) => {
                window.myPeerId = myId;
                window.hostConnection = peer.connect(hostCode);

                window.hostConnection.on('open', () => {
                    // Connected to host lobby
                });

                window.hostConnection.on('data', (data) => {
                    if (data.type === 'PROFILE_ACK') {
                        document.getElementById('ctrl-setup-step').classList.add('hidden');
                        document.getElementById('ctrl-active-step').classList.remove('hidden');
                    } else if (data.type === 'GAME_STARTED') {
                        const statusEl = document.getElementById('ctrl-game-status');
                        statusEl.className = 'text-xs font-bold text-green-400 flex items-center gap-1 justify-end';
                        statusEl.innerHTML = `<span class="w-2 h-2 rounded-full bg-green-400 animate-pulse"></span> HRÁŠ!`;
                    } else if (data.type === 'STATS') {
                        document.getElementById('ctrl-player-stats').innerText = `Body: ${data.score} | Kills: ${data.kills}`;
                    }
                });
            });
        }

        function confirmMobileProfile() {
            const name = document.getElementById('player-name-input').value.trim() || 'PILOT';
            
            document.getElementById('ctrl-player-name').innerText = name.toUpperCase();
            document.getElementById('player-color-badge').style.backgroundColor = selectedColor;
            document.getElementById('player-color-badge').style.boxShadow = `0 0 15px ${selectedColor}`;

            if (window.hostConnection && window.hostConnection.open) {
                window.hostConnection.send({
                    type: 'REGISTER_PROFILE',
                    id: window.myPeerId,
                    name: name,
                    color: selectedColor
                });

                setupControllerTouchListeners();
            }
        }

        function setupControllerTouchListeners() {
            let inputState = {
                type: 'INPUT',
                id: window.myPeerId,
                steer: 0,
                thrust: false,
                brake: false,
                fire: false
            };

            // Loop enviar vstupov na host
            setInterval(() => {
                if (window.hostConnection && window.hostConnection.open) {
                    window.hostConnection.send(inputState);
                }
            }, 1000 / 60);

            // Virtual Joystick
            const joystick = document.getElementById('joystick-container');
            const knob = document.getElementById('joystick-knob');
            let dragging = false;

            function processTouch(clientX, clientY) {
                const rect = joystick.getBoundingClientRect();
                const centerX = rect.left + rect.width / 2;
                const centerY = rect.top + rect.height / 2;

                let deltaX = clientX - centerX;
                let deltaY = clientY - centerY;
                let dist = Math.hypot(deltaX, deltaY);
                let maxDist = rect.width / 2;

                if (dist > maxDist) {
                    deltaX = (deltaX / dist) * maxDist;
                    deltaY = (deltaY / dist) * maxDist;
                }

                knob.style.transform = `translate(${deltaX}px, ${deltaY}px)`;
                inputState.steer = deltaX / maxDist;
            }

            joystick.addEventListener('touchstart', (e) => {
                dragging = true;
                processTouch(e.touches[0].clientX, e.touches[0].clientY);
            });

            window.addEventListener('touchmove', (e) => {
                if (dragging) processTouch(e.touches[0].clientX, e.touches[0].clientY);
            });

            window.addEventListener('touchend', () => {
                dragging = false;
                knob.style.transform = 'translate(0px, 0px)';
                inputState.steer = 0;
            });

            // Buttons
            const bindBtn = (element, key) => {
                element.addEventListener('touchstart', (e) => {
                    e.preventDefault();
                    inputState[key] = true;
                    if (navigator.vibrate) navigator.vibrate(18);
                });
                element.addEventListener('touchend', (e) => {
                    e.preventDefault();
                    inputState[key] = false;
                });
            };

            bindBtn(document.getElementById('btn-fire'), 'fire');
            bindBtn(document.getElementById('btn-thrust'), 'thrust');
            bindBtn(document.getElementById('btn-brake'), 'brake');
        }

        function gameLoop() {
            if (!isController) {
                updateGameLogic();
                renderGame();
            }
            requestAnimationFrame(gameLoop);
        }

        function updateGameLogic() {
            // Local PC Player Updates
            const localPlayer = players['local_host'];
            if (localPlayer) {
                localPlayer.rotation = (keys.left ? -1 : 0) + (keys.right ? 1 : 0);
                localPlayer.rotation *= 0.07;
                localPlayer.thrusting = keys.thrust;
                localPlayer.braking = keys.brake;

                if (keys.thrust && gameStarted) audioFX.playThrust();

                if (keys.fire && gameStarted && Date.now() - localPlayer.lastFired > 180 && localPlayer.alive) {
                    localPlayer.lastFired = Date.now();
                    bullets.push(new Bullet(
                        localPlayer.x + Math.cos(localPlayer.angle) * 20,
                        localPlayer.y + Math.sin(localPlayer.angle) * 20,
                        localPlayer.angle,
                        localPlayer.id,
                        localPlayer.color
                    ));
                    audioFX.playShoot();
                }
            }

            // Update All Ships
            for (let id in players) {
                players[id].update();

                // Send Stats back to mobile
                if (peerConnections[id]) {
                    peerConnections[id].send({
                        type: 'STATS',
                        score: players[id].score,
                        kills: players[id].kills
                    });
                }
            }

            // Asteroid wave regeneration
            if (asteroids.length < 4) spawnAsteroidWave(3);

            for (let ast of asteroids) ast.update();

            // Bullets Update
            for (let i = bullets.length - 1; i >= 0; i--) {
                let b = bullets[i];
                b.update();
                if (b.life <= 0) bullets.splice(i, 1);
            }

            // Particles Update
            for (let i = particles.length - 1; i >= 0; i--) {
                let p = particles[i];
                p.update();
                if (p.life <= 0) particles.splice(i, 1);
            }

            // Only run destructive collisions when game is active
            if (gameStarted) {
                // Bullets vs Asteroids
                for (let bi = bullets.length - 1; bi >= 0; bi--) {
                    let b = bullets[bi];
                    for (let ai = asteroids.length - 1; ai >= 0; ai--) {
                        let ast = asteroids[ai];
                        let dist = Math.hypot(b.x - ast.x, b.y - ast.y);

                        if (dist < ast.radius) {
                            for (let k = 0; k < 15; k++) {
                                particles.push(new Particle(
                                    ast.x, ast.y,
                                    (Math.random() - 0.5) * 6,
                                    (Math.random() - 0.5) * 6,
                                    '#a0a0c0',
                                    25,
                                    Math.random() * 3 + 1
                                ));
                            }

                            if (ast.radius > 25) {
                                asteroids.push(new Asteroid(ast.x, ast.y, ast.radius / 2));
                                asteroids.push(new Asteroid(ast.x, ast.y, ast.radius / 2));
                            }

                            if (players[b.ownerId]) {
                                players[b.ownerId].score += 100;
                                players[b.ownerId].asteroidsDestroyed++;
                            }

                            audioFX.playExplosion(false);
                            asteroids.splice(ai, 1);
                            bullets.splice(bi, 1);
                            break;
                        }
                    }
                }

                // Bullets vs Players (PvP)
                for (let bi = bullets.length - 1; bi >= 0; bi--) {
                    let b = bullets[bi];
                    for (let pid in players) {
                        let ship = players[pid];
                        if (ship.alive && ship.id !== b.ownerId && ship.shieldTimer <= 0) {
                            let dist = Math.hypot(b.x - ship.x, b.y - ship.y);
                            if (dist < ship.radius) {
                                ship.destroy();
                                if (players[b.ownerId]) {
                                    players[b.ownerId].score += 500;
                                    players[b.ownerId].kills++;
                                }
                                bullets.splice(bi, 1);
                                break;
                            }
                        }
                    }
                }

                // Ships vs Asteroids (PvE)
                for (let pid in players) {
                    let ship = players[pid];
                    if (ship.alive && ship.shieldTimer <= 0) {
                        for (let ast of asteroids) {
                            let dist = Math.hypot(ship.x - ast.x, ship.y - ast.y);
                            if (dist < ship.radius + ast.radius) {
                                ship.destroy();
                                break;
                            }
                        }
                    }
                }
            }

            updateLeaderboardUI();
        }

        function renderGame() {
            ctx.fillStyle = 'rgba(3, 3, 8, 0.38)';
            ctx.fillRect(0, 0, gameWidth, gameHeight);

            // Neon Background Grid
            ctx.strokeStyle = 'rgba(0, 243, 255, 0.03)';
            ctx.lineWidth = 1;
            let gridSize = 60;
            for (let x = 0; x < gameWidth; x += gridSize) {
                ctx.beginPath();
                ctx.moveTo(x, 0);
                ctx.lineTo(x, gameHeight);
                ctx.stroke();
            }
            for (let y = 0; y < gameHeight; y += gridSize) {
                ctx.beginPath();
                ctx.moveTo(0, y);
                ctx.lineTo(gameWidth, y);
                ctx.stroke();
            }

            // Draw Entities
            for (let ast of asteroids) ast.draw(ctx);
            for (let p of particles) p.draw(ctx);
            for (let b of bullets) b.draw(ctx);
            for (let id in players) players[id].draw(ctx);
        }

        function updateLeaderboardUI() {
            const list = document.getElementById('leaderboard-list');
            const sortedPlayers = Object.values(players).sort((a, b) => b.score - a.score);

            if (sortedPlayers.length === 0) {
                list.innerHTML = `<li class="text-xs text-gray-500 italic">Žiadni aktívni hráči...</li>`;
                return;
            }

            list.innerHTML = sortedPlayers.map((p, idx) => `
                <li class="flex justify-between items-center bg-black/40 p-1.5 rounded border-l-2" style="border-color: ${p.color}">
                    <span class="truncate max-w-[110px]" style="color: ${p.color}">#${idx + 1} ${p.name}</span>
                    <span class="font-mono text-xs text-cyan-300 font-bold">${p.score} pts</span>
                </li>
            `).join('');
        }
    </script>
</body>
</html>
