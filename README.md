<!DOCTYPE html>
<html lang="es">
<head>

<link rel="manifest" href="manifest.json">
<meta name="theme-color" content="#2e7d32">

  <meta charset="UTF-8">
  <title>Control de Presupuesto</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      max-width: 400px;
      margin: 40px auto;
    }
    label {
      display: block;
      margin-top: 12px;
    }
    input, select, button {
      width: 100%;
      padding: 8px;
      margin-top: 4px;
    }
    button {
      margin-top: 20px;
    }
  </style>
</head>
<body>

<h3>Resumen mensual</h3>

<select id="mesResumen"></select>

<div id="resumen">
  <p>Ingresos: $<span id="rIngresos">0</span></p>
  <p>Gastos fijos: $<span id="rFijos">0</span></p>
  <p>Gastos variables: $<span id="rVariables">0</span></p>
  <p>Deudas: $<span id="rDeudas">0</span></p>
  <p>Ahorro: $<span id="rAhorro">0</span></p>
  <hr>
  <p><strong>Saldo: $<span id="rSaldo">0</span></strong></p>
</div>


  <h2>Nuevo movimiento</h2>

  <label>Tipo</label>
  <select id="tipo">
    <option value="Ingreso">Ingreso</option>
    <option value="Gasto_Fijo">Gasto fijo</option>
    <option value="Gasto_Variable">Gasto variable</option>
    <option value="Ahorro">Ahorro</option>
    <option value="Deuda">Deuda</option>
  </select>

  <label>Categoría</label>
  <select id="categoria"></select>

  <label>Medio de pago</label>
  <select id="medio"></select>


  <label>Fecha</label>
  <input type="date" id="fecha">

  <label>Monto</label>
  <input type="number" id="monto">

  <label>Descripción</label>
  <input id="descripcion">

  <button onclick="guardar()">Guardar</button>

  <p id="estado"></p>

<script>
const URL = "https://script.google.com/macros/s/AKfycbzw5RFY4jjxoAu_lC-gnk0vkqE2zei-beYYQ8UkXs7gbEWjx18tZGS4mkU76jD5Gew_XA/exec";

let categorias = [];
let medios = [];

fetch(URL)
  .then(res => res.json())
  .then(data => {
    categorias = data.categorias;
    medios = data.medios;
    cargarCategorias();
    cargarMedios();
  });

document.getElementById("tipo").addEventListener("change", cargarCategorias);

function cargarCategorias() {
  const tipo = document.getElementById("tipo").value;
  const select = document.getElementById("categoria");

  select.innerHTML = "";

  categorias
    .filter(c => c.tipo === tipo)
    .forEach(c => {
      const option = document.createElement("option");
      option.value = c.nombre;
      option.textContent = c.nombre;
      select.appendChild(option);
    });
}


document.getElementById("fecha").valueAsDate = new Date();

function mesDesdeFecha(fecha) {
  const f = new Date(fecha);
  const meses = [
    "Enero", "Febrero", "Marzo", "Abril", "Mayo", "Junio",
    "Julio", "Agosto", "Septiembre", "Octubre", "Noviembre", "Diciembre"
  ];
  return meses[f.getMonth()] + " " + f.getFullYear();
}

function guardar() {
  const tipo = document.getElementById("tipo").value;
  const categoria = document.getElementById("categoria").value;
  const fecha = document.getElementById("fecha").value;
  const monto = document.getElementById("monto").value;
  const descripcion = document.getElementById("descripcion").value;
  const medio = document.getElementById("medio").value;


  if (!categoria || !fecha || !monto) {
    alert("Faltan datos");
    return;
  }

  let hoja = "";
  let fila = [];

  const mes = mesDesdeFecha(fecha);

  if (tipo === "Ingreso") {
    hoja = "Ingresos";
    fila = [fecha, categoria, descripcion, Number(monto), mes];
  }

  if (tipo === "Gasto_Variable") {
    hoja = "Gastos_Variables";
    fila = [fecha, categoria, descripcion, Number(monto), medio, mes];
  }

  if (tipo === "Gasto_Fijo") {
    hoja = "Gastos_Fijos";
    fila = [fecha, categoria, Number(monto), medio, mes];
  }

  if (tipo === "Ahorro") {
    hoja = "Ahorro";
    fila = [fecha, categoria, descripcion, Number(monto), mes];
  }

  if (tipo === "Deuda") {
    hoja = "Deudas";
    fila = [fecha, categoria, "", Number(monto), medio, mes];
  }

  const data = new FormData();
  data.append("hoja", hoja);
  data.append("fila", JSON.stringify(fila));

  fetch(URL, {
    method: "POST",
    body: data
  })
  .then(r => r.text())
  .then(() => {
    document.getElementById("estado").innerText = "✔ Guardado";
  })
  .catch(() => {
    document.getElementById("estado").innerText = "❌ Error";
  });
}

function cargarMedios() {
  const select = document.getElementById("medio");
  select.innerHTML = "";

  medios.forEach(m => {
    const option = document.createElement("option");
    option.value = m;
    option.textContent = m;
    select.appendChild(option);
  });
}

function cargarResumen() {
  const mes = document.getElementById("mesResumen").value;

  fetch(URL + "?mes=" + encodeURIComponent(mes))
    .then(r => r.json())
    .then(data => {
      const r = data.resumen;
      if (!r) return;

      document.getElementById("rIngresos").innerText = r.ingresos;
      document.getElementById("rFijos").innerText = r.gastosFijos;
      document.getElementById("rVariables").innerText = r.gastosVariables;
      document.getElementById("rDeudas").innerText = r.deudas;
      document.getElementById("rAhorro").innerText = r.ahorro;
      document.getElementById("rSaldo").innerText = r.saldo;
    });
}


function cargarMeses() {
  const select = document.getElementById("mesResumen");
  select.innerHTML = "";

  const hoy = new Date();
  const anio = hoy.getFullYear();

  const meses = [
    "Enero", "Febrero", "Marzo", "Abril", "Mayo", "Junio",
    "Julio", "Agosto", "Septiembre", "Octubre", "Noviembre", "Diciembre"
  ];

  meses.forEach(m => {
    const option = document.createElement("option");
    option.value = `${m} ${anio}`;
    option.textContent = `${m} ${anio}`;
    select.appendChild(option);
  });

  select.value = mesDesdeFecha(hoy);
}

document.getElementById("mesResumen")
  .addEventListener("change", cargarResumen);

cargarMeses();
cargarResumen();




</script>

<script>
if ("serviceWorker" in navigator) {
  navigator.serviceWorker.register("service-worker.js");
}
</script>


</body>
</html>

