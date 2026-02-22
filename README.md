<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>RIC-SW | Capacitación Profesional</title>
    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>
    <style>
        :root { --rojo: #b71c1c; --verde: #2e7d32; }
        body { font-family: 'Segoe UI', sans-serif; background: #f0f2f5; margin: 0; padding: 10px; }
        .card { max-width: 500px; background: white; margin: 20px auto; padding: 25px; border-radius: 15px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
        h2 { color: var(--rojo); text-align: center; margin-top: 0; }
        .hidden { display: none; }
        .btn { width: 100%; padding: 15px; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; margin-top: 10px; font-size: 16px; }
        .btn-rojo { background: var(--rojo); color: white; }
        .btn-modulo { background: white; border: 2px solid #ddd; color: #333; text-align: left; display: flex; justify-content: space-between; align-items: center; }
        .btn-modulo.completado { border-color: var(--verde); background: #f1f8e9; }
        .pregunta { background: #f9f9f9; padding: 15px; border-radius: 10px; margin-bottom: 15px; border-left: 5px solid var(--rojo); }
        input[type="radio"] { margin-right: 10px; transform: scale(1.2); }
        .status-icon { font-size: 20px; }
    </style>
</head>
<body>

<div id="p-registro" class="card">
    <h2>RIC-SW</h2>
    <p style="text-align:center; color:#666;">Sistema de Capacitación Sinchi Wayra</p>
    <input type="text" id="nombre" placeholder="Nombre Completo" style="width:100%; padding:12px; margin:10px 0; border:1px solid #ccc; border-radius:8px;">
    <input type="text" id="ci" placeholder="CI (Ej: 1234567 OR)" style="width:100%; padding:12px; margin:10px 0; border:1px solid #ccc; border-radius:8px;">
    
    <input type="file" id="camara" accept="image/*" capture="camera" style="display:none" onchange="procesarFoto(this)">
    <button class="btn" style="background:#555; color:white;" onclick="document.getElementById('camara').click()">📸 TOMAR FOTO C.I.</button>
    <img id="preview" style="width:100%; display:none; margin-top:10px; border-radius:8px;">
    
    <button class="btn btn-rojo" id="btnIni" style="margin-top:20px;" onclick="iniciarSesion()">INGRESAR AL SISTEMA</button>
</div>

<div id="p-menu" class="card hidden">
    <h2>Mi Progreso</h2>
    <p id="userLabel" style="font-weight:bold; color:#444; text-align:center;"></p>
    <hr>
    <button class="btn btn-modulo" id="m-ind" onclick="abrirExamen('p-ind')">
        <span>1. Inducción General</span> <span class="status-icon" id="s-ind">⏳</span>
    </button>
    <button class="btn btn-modulo" id="m-alt" onclick="abrirExamen('p-alt')">
        <span>2. Trabajos en Altura</span> <span class="status-icon" id="s-alt">⏳</span>
    </button>
    <button class="btn btn-modulo" id="m-ene" onclick="abrirExamen('p-ene')">
        <span>3. Aislamiento de Energía</span> <span class="status-icon" id="s-ene">⏳</span>
    </button>
    <p style="font-size:12px; color:#888; text-align:center; margin-top:20px;">Complete todos para generar certificado.</p>
</div>

<div id="p-ind" class="card hidden">
    <h2>Inducción General</h2>
    <div class="pregunta">
        <p>¿Qué prioridad tiene la Seguridad en Sinchi Wayra?</p>
        <label><input type="radio" name="q1" value="100"> Prioridad #1 sobre la producción</label><br>
        <label><input type="radio" name="q1" value="0"> Igual que la producción</label>
    </div>
    <button class="btn btn-rojo" onclick="calificar('q', 'p-ind', 's-ind', 'notaInd')">FINALIZAR</button>
</div>

<div id="p-alt" class="card hidden">
    <h2>ERC 3: Trabajo en Altura</h2>
    <div class="pregunta">
        <p>¿A qué altura es obligatorio el uso de protección anticaídas?</p>
        <label><input type="radio" name="h1" value="100"> 1.80 metros o más</label><br>
        <label><input type="radio" name="h1" value="0"> 1.50 metros o más</label>
    </div>
    <div class="pregunta">
        <p>¿Qué significa la "C" en el sistema ABCD?</p>
        <label><input type="radio" name="h2" value="100"> Conector (Línea de vida/anclaje)</label><br>
        <label><input type="radio" name="h2" value="0"> Casco de seguridad</label>
    </div>
    <button class="btn btn-rojo" onclick="calificar('h', 'p-alt', 's-alt', 'notaAlt')">FINALIZAR</button>
</div>

<div id="p-ene" class="card hidden">
    <h2>ERC 2: Aislamiento de Energía</h2>
    <div class="pregunta">
        <p>¿Cuál es la primera Regla de Oro del aislamiento?</p>
        <label><input type="radio" name="e1" value="100"> Corte visible de todas las fuentes</label><br>
        <label><input type="radio" name="e1" value="0"> Avisar al supervisor</label>
    </div>
    <button class="btn btn-rojo" onclick="calificar('e', 'p-ene', 's-ene', 'notaEne')">FINALIZAR</button>
</div>

<script>
    // Tu Configuración Real de Firebase (obtenida de tus capturas)
    const firebaseConfig = {
        apiKey: "AIzaSyCp7opkKQzw0pf3VbQcOO7295Kz1E1zDTk",
        authDomain: "ric-sw.firebaseapp.com",
        projectId: "ric-sw",
        storageBucket: "ric-sw.firebasestorage.app",
        messagingSenderId: "883694529169",
        appId: "1:883694529169:web:acbaf9645b12bbee532d30"
    };

    firebase.initializeApp(firebaseConfig);
    const db = firebase.firestore();
    let currentDocId = "";
    let fotoBase64 = "";

    function procesarFoto(input) {
        const reader = new FileReader();
        reader.onload = e => {
            fotoBase64 = e.target.result;
            document.getElementById('preview').src = fotoBase64;
            document.getElementById('preview').style.display = 'block';
        };
        reader.readAsDataURL(input.files[0]);
    }

    async function iniciarSesion() {
        const nombre = document.getElementById('nombre').value;
        const ci = document.getElementById('ci').value;
        if(!nombre || !ci || !fotoBase64) return alert("Falta nombre, CI o Foto.");

        const res = await db.collection("trabajadores").add({
            nombre, ci, fotoCI: fotoBase64, fecha: new Date().toLocaleString(),
            notaInd: 0, notaAlt: 0, notaEne: 0
        });
        currentDocId = res.id;
        document.getElementById('userLabel').innerText = "Trabajador: " + nombre;
        document.getElementById('p-registro').classList.add('hidden');
        document.getElementById('p-menu').classList.remove('hidden');
    }

    function abrirExamen(id) {
        document.getElementById('p-menu').classList.add('hidden');
        document.getElementById(id).classList.remove('hidden');
    }

    async function calificar(prefijo, pActual, sIcon, campo) {
        const resps = document.querySelectorAll(`input[name^="${prefijo}"]:checked`);
        let suma = 0;
        resps.forEach(r => suma += parseInt(r.value));
        const final = Math.round(suma / resps.length);

        await db.collection("trabajadores").doc(currentDocId).update({ [campo]: final });
        
        document.getElementById(sIcon).innerText = "✅ (" + final + ")";
        document.getElementById(pActual).classList.add('hidden');
        document.getElementById('p-menu').classList.remove('hidden');
    }
</script>
</body>
</html>
