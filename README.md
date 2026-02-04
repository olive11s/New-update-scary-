# New-update-scary-
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Hotel Escape - Giselle ❤️</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; touch-action: manipulation; }
        body, html { width: 100%; height: 100%; overflow: hidden; background: #000; font-family: 'Courier New', Courier, monospace; color: white; }

        /* Perspective Hallway */
        #game-view { position: relative; width: 100%; height: 100%; display: flex; flex-direction: column; align-items: center; overflow: hidden; }
        
        .running-bob { animation: bob 0.25s infinite linear; }
        @keyframes bob { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-15px); } }

        #hallway {
            position: absolute; inset: 0; z-index: -1;
            background: linear-gradient(90deg, #0a0a0a 5%, transparent 20%, transparent 80%, #0a0a0a 95%),
                        repeating-linear-gradient(0deg, #000 0px, #000 50px, #1a1a1a 50px, #1a1a1a 100px);
            background-size: 100% 100%, 100% 200%;
        }

        /* Fixed HUD - Now at the very top with padding for iPhone notch */
        #hud { 
            position: absolute; top: 0; width: 100%; 
            display: flex; justify-content: space-between; 
            padding: 50px 25px 20px 25px; /* Added top padding */
            z-index: 100; text-shadow: 0 0 10px red;
            background: linear-gradient(to bottom, rgba(0,0,0,0.8), transparent);
        }

        #door-zone {
            display: flex; gap: 15px; margin-top: 30vh; z-index: 50;
        }
        .door {
            width: 85px; height: 150px; background: #3d2b1f;
            border: 4px solid #2a1d15; border-radius: 5px;
            display: flex; justify-content: center; align-items: center;
            font-weight: bold; font-size: 1.8rem; box-shadow: 0 0 20px #000;
        }

        /* Controls */
        #controls { position: absolute; bottom: 40px; width: 95%; display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 8px; z-index: 200; }
        .btn { background: rgba(30, 30, 30, 0.95); color: white; border: 1px solid #444; padding: 18px; border-radius: 10px; font-weight: bold; }
        .btn-run { grid-column: span 3; background: #900; padding: 25px; font-size: 1.2rem; border: none; box-shadow: 0 0 20px #500; }

        /* Scary Flash */
        #scare-flash { 
            display: none; position: fixed; inset: 0; 
            background: url('https://media.giphy.com/media/v1.Y2lkPTc5MGI3NjExM3VxdXo5M2Z4bmN4bmN4bmN4bmN4bmN4bmN4bmN4bmN4bmN4JmVwPXYxX2ludGVybmFsX2dpZl9ieV9pZCZjdD1n/3o7TKsWZyGKY1cKG1W/giphy.gif') center/cover;
            z-index: 500; 
        }

        .full-screen { display: none; position: fixed; inset: 0; z-index: 1000; flex-direction: column; justify-content: center; align-items: center; text-align: center; background: #000; padding: 20px; }
        .vignette { position: fixed; inset: 0; pointer-events: none; z-index: 60; box-shadow: inset 0 0 100px rgba(0,0,0,1); transition: box-shadow 0.3s; }
        .meteor { position: absolute; top: -200px; width: 150px; height: 150px; background: radial-gradient(circle, #fff, #ff0, #f00); border-radius: 50%; box-shadow: 0 0 80px #f40; }
    </style>
</head>
<body>

<div class="vignette" id="vignette"></div>
<div id="scare-flash"></div>

<div id="game-view">
    <div id="hallway"></div>
    
    <div id="hud">
        <div style="color: #ff4d4d;">HP: <span id="hp-ui">❤️❤️❤️</span></div>
        <div style="color: #ffeb3b;">FLOOR: <span id="floor-ui">1</span>/7</div>
    </div>

    <div id="door-zone">
        <div class="door" onclick="chooseDoor(0)">I</div>
        <div class="door" onclick="chooseDoor(1)">II</div>
        <div class="door" onclick="chooseDoor(2)">III</div>
    </div>

    <div id="controls">
        <button class="btn" onclick="look('left')">LEFT</button>
        <button class="btn" onclick="lookBack()">LOOK BACK</button>
        <button class="btn" onclick="look('right')">RIGHT</button>
        <button class="btn btn-run" onmousedown="startRun()" onmouseup="stopRun()" ontouchstart="startRun()" ontouchend="stopRun()">HOLD TO RUN FORWARD</button>
    </div>
</div>

<div id="chest-screen" class="full-screen">
    <h1>YOU FOUND THE CHEST.</h1>
    <div style="font-size: 100px; margin: 30px; cursor: pointer;" onclick="openChest()">🧳</div>
    <p>TAP TO OPEN</p>
</div>

<div id="shoot-screen" class="full-screen">
    <h1 style="color:red; margin-bottom:20px;">TARGET IN SIGHT!</h1>
    <div style="font-size: 120px; margin-bottom:30px;">👹</div>
    <button class="btn-run" style="width:200px;" onclick="killMonster()">SHOOT!</button>
</div>

<div id="ending" class="full-screen">
    <div id="meteor" class="meteor"></div>
    <h1 id="final-text">CONGRATULATIONS!<br>You survived the apocalypse.</h1>
</div>

<audio id="sfx-steps" loop><source src="https://assets.mixkit.co/active_storage/sfx/28/28-preview.mp3" type="audio/mpeg"></audio>

<script>
    let floor = 1, hp = 3, monsterDist = 0, runActive = false, offset = 0;
    let correct = Math.floor(Math.random() * 3);

    function startRun() {
        runActive = true;
        document.getElementById('game-view').classList.add('running-bob');
        document.getElementById('sfx-steps').play();
        animate();
    }
    function stopRun() {
        runActive = false;
        document.getElementById('game-view').classList.remove('running-bob');
        document.getElementById('sfx-steps').pause();
    }
    function animate() {
        if (!runActive) return;
        offset += (14 + monsterDist * 2);
        document.getElementById('hallway').style.backgroundPosition = `0px ${offset}px`;
        requestAnimationFrame(animate);
    }

    function chooseDoor(choice) {
        if (choice === correct) {
            floor++;
            if (floor > 7) {
                document.getElementById('chest-screen').style.display = 'flex';
            } else {
                document.getElementById('floor-ui').innerText = floor;
                correct = Math.floor(Math.random() * 3);
            }
        } else {
            hp--;
            monsterDist += 1.5;
            triggerFlash();
            updateStatus();
            if (hp <= 0) caught();
        }
    }

    function lookBack() {
        monsterDist += 1.2;
        triggerFlash();
        updateStatus();
        if (monsterDist >= 5.5) caught();
    }

    function triggerFlash() {
        const flash = document.getElementById('scare-flash');
        flash.style.display = 'block';
        if (navigator.vibrate) navigator.vibrate(200);
        setTimeout(() => flash.style.display = 'none', 150);
    }

    function updateStatus() {
        document.getElementById('hp-ui').innerText = "❤️".repeat(Math.max(0, hp));
        document.getElementById('vignette').style.boxShadow = `inset 0 0 ${monsterDist * 60}px rgba(255,0,0,0.85)`;
    }

    function openChest() {
        document.getElementById('chest-screen').style.display = 'none';
        document.getElementById('shoot-screen').style.display = 'flex';
    }

    function killMonster() {
        document.getElementById('shoot-screen').style.display = 'none';
        document.getElementById('ending').style.display = 'flex';
        setTimeout(() => {
            const m = document.getElementById('meteor');
            m.style.transition = "top 1.5s ease-in";
            m.style.top = "110%";
            setTimeout(() => {
                document.body.style.background = "white";
                document.getElementById('ending').style.background = "white";
                document.getElementById('final-text').style.color = "black";
                document.getElementById('final-text').innerHTML = "TO BE CONTINUED...";
            }, 1200);
        }, 2000);
    }

    function caught() { 
        document.getElementById('scare-flash').style.display = 'block';
        alert("HE GOT YOU."); 
        location.reload(); 
    }
    
    function look(dir) {
        const dz = document.getElementById('door-zone');
        dz.style.transition = "transform 0.2s";
        dz.style.transform = dir === 'left' ? "translateX(40px)" : "translateX(-40px)";
        setTimeout(() => dz.style.transform = "none", 200);
    }
</script>
</body>
</html>
