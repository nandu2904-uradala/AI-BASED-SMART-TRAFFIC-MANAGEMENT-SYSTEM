<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>STLS.OS - Production Ready Traffic Sim</title>
    <style>
        /* Modern CSS Reset & Variables */
        :root {
            --slate-950: #020617;
            --slate-900: #0f172a;
            --slate-800: #1e293b;
            --slate-700: #334155;
            --slate-500: #64748b;
            --slate-400: #94a3b8;
            --emerald-400: #34d399;
            --emerald-500: #10b981;
            --blue-400: #60a5fa;
            --blue-500: #3b82f6;
            --pink-500: #ec4899;
            --amber-400: #fbbf24;
            --white: #ffffff;
            
            --road-color: #1e293b;
            --grass-color: #064e3b;
            --center-line: #eab308;
            --sky-filter: rgba(0, 0, 0, 0);
        }
        
        body.night {
            --road-color: #0f172a;
            --grass-color: #022c22;
            --sky-filter: rgba(15, 23, 42, 0.6);
        }

        * { box-sizing: border-box; margin: 0; padding: 0; }
        body { 
            background: var(--slate-950); 
            color: var(--white); 
            font-family: system-ui, -apple-system, sans-serif; 
            overflow: hidden; 
            height: 100vh;
        }

        /* Layout */
        .app-container { display: flex; height: 100vh; }
        
        .sidebar {
            width: 320px;
            background: rgba(15, 23, 42, 0.8);
            backdrop-filter: blur(20px);
            border-right: 1px solid var(--slate-800);
            padding: 24px;
            display: flex;
            flex-direction: column;
            gap: 20px;
            z-index: 100;
        }

        .main-view {
            flex: 1;
            position: relative;
            background-color: var(--grass-color);
            background-image: radial-gradient(#14532d 1.5px, transparent 1.5px);
            background-size: 30px 30px;
            transition: all 1s ease;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .main-view::after {
            content: '';
            position: absolute;
            inset: 0;
            background: var(--sky-filter);
            pointer-events: none;
            transition: background 1s ease;
            z-index: 60;
        }

        /* UI Components */
        .title-group h1 { font-size: 1.25rem; font-weight: 900; letter-spacing: -0.025em; }
        .title-group p { font-size: 10px; color: var(--slate-500); text-transform: uppercase; letter-spacing: 0.1em; font-weight: 900; }
        
        .card {
            background: rgba(30, 41, 59, 0.4);
            border-radius: 12px;
            border-left: 4px solid var(--white);
            padding: 16px;
        }

        .telemetry-row { display: flex; justify-content: space-between; align-items: center; margin-bottom: 8px; }
        .telemetry-label { font-size: 12px; color: var(--slate-400); }
        .telemetry-value { font-size: 12px; font-weight: bold; font-family: monospace; }

        .btn {
            width: 100%;
            padding: 12px;
            border-radius: 8px;
            font-weight: 900;
            text-transform: uppercase;
            cursor: pointer;
            transition: all 0.2s;
            border: 1px solid transparent;
            font-size: 11px;
        }

        .btn-primary {
            background: rgba(255, 255, 255, 0.1);
            border-color: var(--white);
            color: var(--white);
        }
        .btn-primary:hover { background: rgba(255, 255, 255, 0.2); }

        .btn-secondary {
            background: var(--slate-800);
            color: var(--slate-400);
        }
        .btn-secondary:hover { background: var(--slate-700); }

        .terminal {
            background: #000;
            border: 1px solid var(--slate-800);
            font-family: 'Courier New', Courier, monospace;
            font-size: 10px;
            color: var(--emerald-500);
            padding: 10px;
            height: 160px;
            overflow-y: auto;
            border-radius: 6px;
            margin-top: auto;
        }

        /* Simulation Elements */
        #viewport { position: relative; width: 800px; height: 800px; }
        
        .road { 
            background: var(--road-color); 
            position: absolute;
            box-shadow: inset 0 0 60px rgba(0,0,0,0.6);
            transition: background 1s ease;
        }
        .road.v { left: 50%; transform: translateX(-50%); width: 192px; height: 100%; }
        .road.h { top: 50%; transform: translateY(-50%); width: 100%; height: 192px; }

        .center-line-v { position: absolute; width: 4px; height: 100%; left: 50%; transform: translateX(-50%); background: var(--center-line); opacity: 0.6; }
        .center-line-h { position: absolute; height: 4px; width: 100%; top: 50%; transform: translateY(-50%); background: var(--center-line); opacity: 0.6; }

        .crosswalk { position: absolute; display: justify-content: space-between; pointer-events: none; z-index: 2; }
        .crosswalk.h { width: 192px; height: 48px; flex-direction: row; display: flex;}
        .crosswalk.v { width: 48px; height: 192px; flex-direction: column; display: flex;}
        .stripe { background: rgba(255,255,255,0.4); }
        .crosswalk.h .stripe { width: 10px; height: 100%; }
        .crosswalk.v .stripe { width: 100%; height: 10px; }

        /* Traffic Lights */
        .traffic-light { width: 20px; height: 54px; background: #111; border-radius: 4px; display: flex; flex-direction: column; align-items: center; justify-content: space-around; padding: 3px; border: 1px solid var(--slate-700); z-index: 100; }
        .bulb { width: 12px; height: 12px; border-radius: 50%; background: #1a1a1a; transition: all 0.2s; }
        .red.active { background: #ef4444; box-shadow: 0 0 15px #ef4444; }
        .yellow.active { background: #eab308; box-shadow: 0 0 15px #eab308; }
        .green.active { background: #22c55e; box-shadow: 0 0 15px #22c55e; }

        .ped-signal { width: 14px; height: 28px; background: #111; border: 1px solid #444; display: flex; flex-direction: column; padding: 2px; border-radius: 2px; z-index: 101; }
        .ped-bulb { height: 10px; width: 10px; margin-bottom: 2px; border-radius: 1px; background: #222; }
        .ped-walk.active { background: #fff; box-shadow: 0 0 8px #fff; }
        .ped-stop.active { background: #f97316; box-shadow: 0 0 8px #f97316; }

        /* Entities */
        .car { position: absolute; z-index: 70; transition: transform 0.05s linear; }
        .headlight { position: absolute; right: -60px; top: -10px; width: 100px; height: 40px; background: radial-gradient(ellipse at left, rgba(255,255,230,0.5) 0%, transparent 70%); pointer-events: none; opacity: 0; transition: opacity 1s; }
        body.night .headlight { opacity: 1; }

        .emergency-strobe {
            position: absolute;
            top: 50%; left: 50%;
            width: 16px; height: 16px;
            border-radius: 50%;
            transform: translate(-50%, -50%);
            animation: siren-strobe 0.15s infinite alternate;
        }

        @keyframes siren-strobe {
            0% { background: #ef4444; box-shadow: 0 0 25px #ef4444; }
            50% { background: #3b82f6; box-shadow: 0 0 25px #3b82f6; }
            100% { background: #ffffff; box-shadow: 0 0 25px #ffffff; }
        }

        .pedestrian { position: absolute; width: 8px; height: 8px; background: #fde68a; border-radius: 50%; z-index: 65; box-shadow: 0 0 5px rgba(253, 230, 138, 0.5); }
    </style>
</head>
<body>
    <div class="app-container">
        <aside class="sidebar">
            <div class="title-group">
                <h1>STLS<span style="color: var(--blue-500)">.OS</span></h1>
                <p>Interceptor Core v4.0</p>
            </div>

            <div class="card">
                <p style="font-size: 10px; color: var(--slate-500); font-weight: bold; text-transform: uppercase; margin-bottom: 12px;">Live Telemetry</p>
                <div class="telemetry-row">
                    <span class="telemetry-label">Status:</span>
                    <span id="target-axis" class="telemetry-value" style="color: var(--blue-400)">H-FLOW</span>
                </div>
                <div class="telemetry-row">
                    <span class="telemetry-label">Phase:</span>
                    <span id="green-timer" class="telemetry-value" style="color: var(--emerald-400)">00:00</span>
                </div>
                <div class="telemetry-row">
                    <span class="telemetry-label">Priority:</span>
                    <span id="priority-status" class="telemetry-value" style="color: var(--slate-400)">STANDBY</span>
                </div>
            </div>

            <button onclick="tripleDispatch()" class="btn btn-primary">
                🚨 HIGH-SPEED DISPATCH 🚨
            </button>
            <button onclick="spawnCrowd()" class="btn btn-secondary">
                Spawn Pedestrian Crowd
            </button>

            <div id="log" class="terminal"></div>

            <button onclick="toggleNight()" class="btn btn-secondary" style="margin-top: 12px;">
                <span id="mode-icon">☀️</span> <span id="mode-text">Daytime Ops</span>
            </button>
        </aside>

        <main class="main-view">
            <div id="viewport">
                <!-- Roads -->
                <div class="road v"><div class="center-line-v"></div></div>
                <div class="road h"><div class="center-line-h"></div></div>

                <!-- Crosswalks -->
                <div class="crosswalk h" style="left: 304px; top: 228px;"><div class="stripe"></div><div class="stripe"></div><div class="stripe"></div><div class="stripe"></div><div class="stripe"></div></div>
                <div class="crosswalk h" style="left: 304px; bottom: 228px;"><div class="stripe"></div><div class="stripe"></div><div class="stripe"></div><div class="stripe"></div><div class="stripe"></div></div>
                <div class="crosswalk v" style="top: 304px; left: 228px;"><div class="stripe"></div><div class="stripe"></div><div class="stripe"></div><div class="stripe"></div><div class="stripe"></div></div>
                <div class="crosswalk v" style="top: 304px; right: 228px;"><div class="stripe"></div><div class="stripe"></div><div class="stripe"></div><div class="stripe"></div><div class="stripe"></div></div>

                <!-- Lights -->
                <div id="light-N" class="traffic-light" style="position: absolute; top: 210px; right: 300px;"><div class="bulb red active"></div><div class="bulb yellow"></div><div class="bulb green"></div></div>
                <div id="light-S" class="traffic-light" style="position: absolute; bottom: 210px; left: 300px;"><div class="bulb red active"></div><div class="bulb yellow"></div><div class="bulb green"></div></div>
                <div id="light-W" class="traffic-light" style="position: absolute; top: 300px; left: 210px; transform: rotate(90deg);"><div class="bulb red"></div><div class="bulb yellow"></div><div class="bulb green active"></div></div>
                <div id="light-E" class="traffic-light" style="position: absolute; bottom: 300px; right: 210px; transform: rotate(-90deg);"><div class="bulb red"></div><div class="bulb yellow"></div><div class="bulb green active"></div></div>

                <!-- Pedestrian Signals -->
                <div id="ped-EW" class="ped-signal" style="position: absolute; top: 285px; left: 220px;"><div class="ped-bulb ped-stop active"></div><div class="ped-bulb ped-walk"></div></div>
                <div id="ped-NS" class="ped-signal" style="position: absolute; top: 220px; right: 285px; transform: rotate(90deg);"><div class="ped-bulb ped-stop active"></div><div class="ped-bulb ped-walk"></div></div>

                <div id="entity-layer" style="position: absolute; inset: 0; pointer-events: none;"></div>
            </div>
        </main>
    </div>

    <script>
        // Audio Engine
        let audioCtx = null;
        function initAudio() {
            if (!audioCtx) {
                audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            }
        }

        function createSiren() {
            if (!audioCtx) return null;
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            osc.type = 'triangle';
            osc.frequency.setValueAtTime(400, audioCtx.currentTime);
            gain.gain.setValueAtTime(0, audioCtx.currentTime);
            gain.gain.linearRampToValueAtTime(0.04, audioCtx.currentTime + 0.1);
            osc.connect(gain);
            gain.connect(audioCtx.destination);
            osc.start();
            return { osc, gain };
        }

        const entityLayer = document.getElementById('entity-layer');
        const logEl = document.getElementById('log');
        const timerEl = document.getElementById('green-timer');
        const axisEl = document.getElementById('target-axis');
        const priorityEl = document.getElementById('priority-status');
        
        let cars = [];
        let pedestrians = [];
        let state = { 
            hGreen: true, 
            timer: 0, 
            isNight: false, 
            yellowPhase: false,
            phaseEnd: 400,
            priorityOverride: false,
            priorityAxis: null
        };

        const config = {
            stopLines: { N: 240, S: 240, E: 240, W: 240 },
            pedWaitLines: { E: 280, W: 520, N: 520, S: 280 }
        };

        function addLog(msg, color = 'var(--emerald-500)') {
            const div = document.createElement('div');
            div.style.marginBottom = '4px';
            div.style.color = color;
            div.innerText = `> ${msg}`;
            logEl.prepend(div);
            if(logEl.childNodes.length > 25) logEl.lastChild.remove();
        }

        function tripleDispatch() {
            initAudio();
            const directions = ['N', 'S', 'E', 'W'];
            const targetDir = directions[Math.floor(Math.random() * 4)];
            addLog(`URGENT: DISPATCHING INTERCEPTORS (${targetDir})`, 'var(--white)');
            
            for(let i = 0; i < 3; i++) {
                // Higher speeds and tighter offsets for the dispatch
                setTimeout(() => spawnCar(true, targetDir, i * -120), i * 150);
            }
        }

        function spawnCar(isEmergency = false, fixedDir = null, startOffset = -100) {
            const directions = ['N', 'S', 'E', 'W'];
            const dir = fixedDir || directions[Math.floor(Math.random() * 4)];
            const shouldBeEmergency = isEmergency || Math.random() < 0.10;

            const car = {
                id: Math.random(), 
                dir, 
                pos: startOffset,
                // SIGNIFICANTLY INCREASED SPEED FOR EMERGENCY VEHICLES (from ~7 to ~12)
                maxSpeed: shouldBeEmergency ? (11 + Math.random() * 3) : (1.8 + Math.random() * 1.5),
                speed: 0, 
                color: shouldBeEmergency ? '#ffffff' : `hsl(${Math.random() * 360}, 30%, 35%)`,
                isEmergency: shouldBeEmergency, 
                length: shouldBeEmergency ? 60 : (Math.random() > 0.8 ? 70 : 45), 
                width: 24,
                safetyBuffer: shouldBeEmergency ? 200 : 80,
                audio: (shouldBeEmergency && audioCtx) ? createSiren() : null
            };

            const el = document.createElement('div');
            el.className = 'car';
            el.style.width = `${car.length}px`;
            el.style.height = `${car.width}px`;
            el.innerHTML = `
                <div class="headlight"></div>
                ${car.isEmergency ? '<div class="emergency-strobe"></div>' : ''}
                <svg viewBox="0 0 ${car.length} ${car.width}">
                    <rect width="100%" height="100%" rx="6" fill="${car.color}"/>
                    <rect x="${car.length - 18}" y="4" width="12" height="${car.width-8}" rx="2" fill="rgba(0,0,0,0.5)"/>
                    <circle cx="5" cy="5" r="3" fill="#fff" opacity="0.9"/>
                    <circle cx="5" cy="${car.width-5}" r="3" fill="#fff" opacity="0.9"/>
                </svg>`;
            
            entityLayer.appendChild(el);
            car.el = el;
            cars.push(car);
        }

        function spawnPedestrian() {
            const directions = ['E', 'W', 'N', 'S'];
            const target = directions[Math.floor(Math.random() * 4)];
            let x, y, vx = 0, vy = 0;
            const jitter = (Math.random() - 0.5) * 40;

            switch(target) {
                case 'E': x = 0; y = 250 + jitter; vx = 1; break;
                case 'W': x = 800; y = 550 + jitter; vx = -1; break;
                case 'N': x = 550 + jitter; y = 800; vy = -1; break;
                case 'S': x = 250 + jitter; y = 0; vy = 1; break;
            }

            const ped = {
                id: Math.random(), x, y, vx, vy, target,
                speed: 0.9 + Math.random() * 0.6,
                el: document.createElement('div')
            };
            ped.el.className = 'pedestrian';
            entityLayer.appendChild(ped.el);
            pedestrians.push(ped);
        }

        function spawnCrowd() {
            for(let i=0; i<12; i++) setTimeout(spawnPedestrian, i * 120);
            addLog("EVENT: Pedestrian surge detected", "var(--amber-400)");
        }

        function updateSignals() {
            const set = (ids, color) => ids.forEach(id => {
                const el = document.getElementById(id);
                el.querySelectorAll('.bulb').forEach(b => b.classList.remove('active'));
                el.querySelector('.' + color).classList.add('active');
            });

            if (state.yellowPhase) {
                set(['light-E', 'light-W', 'light-N', 'light-S'], 'yellow');
            } else {
                set(['light-E', 'light-W'], state.hGreen ? 'green' : 'red');
                set(['light-N', 'light-S'], state.hGreen ? 'red' : 'green');
            }

            const setPed = (el, walk) => {
                el.querySelector('.ped-walk').classList.toggle('active', walk);
                el.querySelector('.ped-stop').classList.toggle('active', !walk);
            };
            setPed(document.getElementById('ped-EW'), state.hGreen && !state.yellowPhase);
            setPed(document.getElementById('ped-NS'), !state.hGreen && !state.yellowPhase);
        }

        function checkPriority() {
            const emergencyCars = cars.filter(c => c.isEmergency && c.pos < 300);
            if (emergencyCars.length > 0) {
                const eCar = emergencyCars[0];
                const isEH = (eCar.dir === 'E' || eCar.dir === 'W');
                
                if (state.hGreen !== isEH || state.yellowPhase) {
                    if (!state.priorityOverride) {
                        state.priorityOverride = true;
                        state.priorityAxis = isEH ? 'H' : 'V';
                        addLog(`AI: EMERGENCY VEHICLE DETECTED (${eCar.dir}). PRE-EMPTING PHASE.`, 'var(--pink-500)');
                        priorityEl.innerText = "ACTIVE-EO";
                        priorityEl.style.color = "var(--pink-500)";
                        
                        if (!state.yellowPhase) {
                            state.yellowPhase = true;
                            state.timer = 0;
                            state.phaseEnd = 60;
                        } else {
                            state.yellowPhase = false;
                            state.hGreen = isEH;
                            state.timer = -300; 
                        }
                        updateSignals();
                    }
                } else {
                    if (state.timer > state.phaseEnd - 100) {
                        state.timer = state.phaseEnd - 101; 
                        priorityEl.innerText = "HOLDING";
                    }
                }
            } else if (state.priorityOverride) {
                const clearing = cars.filter(c => c.isEmergency && c.pos < 600);
                if (clearing.length === 0) {
                    state.priorityOverride = false;
                    state.priorityAxis = null;
                    priorityEl.innerText = "STANDBY";
                    priorityEl.style.color = "var(--slate-400)";
                    addLog("AI: Clear path confirmed. Resuming standard flow.", "var(--blue-400)");
                }
            }
        }

        function gameLoop() {
            state.timer++;
            checkPriority();

            if (state.timer > state.phaseEnd) {
                state.yellowPhase = !state.yellowPhase;
                state.timer = 0;
                if (!state.yellowPhase) state.hGreen = !state.hGreen;
                state.phaseEnd = state.yellowPhase ? 120 : 480;
                updateSignals();
            }

            timerEl.innerText = state.timer < 0 ? "EXP" : `${Math.floor(Math.max(0, state.timer) / 60)}s`;
            axisEl.innerText = state.yellowPhase ? "CAUTION" : (state.hGreen ? "H-AXIS" : "V-AXIS");

            for (let i = pedestrians.length - 1; i >= 0; i--) {
                const ped = pedestrians[i];
                const isHorizontal = (ped.target === 'E' || ped.target === 'W');
                const canWalk = isHorizontal ? (state.hGreen && !state.yellowPhase) : (!state.hGreen && !state.yellowPhase);
                const waitPos = config.pedWaitLines[ped.target];
                let isWaiting = false;

                if (!canWalk) {
                    if (ped.target === 'E' && ped.x >= waitPos - 5 && ped.x < waitPos) isWaiting = true;
                    if (ped.target === 'W' && ped.x <= waitPos + 5 && ped.x > waitPos) isWaiting = true;
                    if (ped.target === 'S' && ped.y >= waitPos - 5 && ped.y < waitPos) isWaiting = true;
                    if (ped.target === 'N' && ped.y <= waitPos + 5 && ped.y > waitPos) isWaiting = true;
                }

                if (!isWaiting) {
                    ped.x += ped.vx * ped.speed;
                    ped.y += ped.vy * ped.speed;
                }
                ped.el.style.transform = `translate(${ped.x}px, ${ped.y}px)`;
                if (ped.x < -100 || ped.x > 900 || ped.y < -100 || ped.y > 900) {
                    ped.el.remove();
                    pedestrians.splice(i, 1);
                }
            }

            for (let i = cars.length - 1; i >= 0; i--) {
                const car = cars[i];
                const isH = (car.dir === 'E' || car.dir === 'W');
                const isRed = isH ? !state.hGreen : state.hGreen;
                
                let targetSpeed = car.maxSpeed;
                const stopLine = config.stopLines[car.dir];
                
                if (!car.isEmergency) {
                    if ((isRed || state.yellowPhase) && car.pos > stopLine - 100 && car.pos < stopLine) {
                        targetSpeed = 0;
                    }
                } else {
                    // Emergency vehicles slow down less at red lights during pre-emption
                    if (isRed && car.pos > stopLine - 50 && car.pos < stopLine && !state.priorityOverride) {
                        targetSpeed = car.maxSpeed * 0.5; 
                    }
                }

                const carAhead = cars.find(other => other.id !== car.id && other.dir === car.dir && other.pos > car.pos && other.pos < car.pos + car.safetyBuffer);
                if (carAhead) {
                    // Emergency vehicles are more aggressive and follow closer
                    const followMult = car.isEmergency ? 0.95 : 0.85;
                    targetSpeed = Math.min(targetSpeed, carAhead.speed * followMult);
                }

                // Higher acceleration factor for emergency vehicles (0.15 vs 0.08)
                const accelFactor = car.isEmergency ? 0.15 : 0.08;
                car.speed += (targetSpeed - car.speed) * accelFactor;
                car.pos += car.speed;

                if (car.audio && audioCtx) {
                    const time = audioCtx.currentTime;
                    // Faster siren modulation for faster cars
                    const sirenFreq = car.isEmergency ? 8 : 6;
                    const freq = 450 + Math.sin(time * sirenFreq) * 200 + (car.speed * 20);
                    car.audio.osc.frequency.setTargetAtTime(freq, time, 0.05);
                    if (car.pos > 750) {
                        const falloff = Math.max(0, 1 - (car.pos - 750) / 350);
                        car.audio.gain.gain.setTargetAtTime(0.04 * falloff, time, 0.1);
                    }
                }

                const center = 400, offset = 48;
                let x, y, rot;
                switch(car.dir) {
                    case 'W': x = 800 - car.pos; y = center - offset - 12; rot = 180; break;
                    case 'E': x = car.pos - car.length; y = center + offset - 12; rot = 0; break;
                    case 'N': x = center + offset - (car.length/2); y = car.pos; rot = 90; break;
                    case 'S': x = center - offset - (car.length/2); y = 800 - car.pos; rot = -90; break;
                }
                car.el.style.transform = `translate(${x}px, ${y}px) rotate(${rot}deg)`;
                
                if (car.pos > 1400) {
                    if (car.audio) {
                        car.audio.osc.stop();
                        car.audio.osc.disconnect();
                    }
                    car.el.remove();
                    cars.splice(i, 1);
                }
            }
            requestAnimationFrame(gameLoop);
        }

        function toggleNight() { 
            state.isNight = !state.isNight; 
            document.body.classList.toggle('night'); 
            document.getElementById('mode-icon').innerText = state.isNight ? '🌙' : '☀️';
            document.getElementById('mode-text').innerText = state.isNight ? 'Night Ops' : 'Daytime Ops';
        }
        
        window.onload = () => {
            setInterval(() => { if(cars.length < 24) spawnCar(); }, 1000);
            setInterval(() => { if(pedestrians.length < 18) spawnPedestrian(); }, 2500);
            updateSignals();
            gameLoop();
        };
    </script>
</body>
</html>
