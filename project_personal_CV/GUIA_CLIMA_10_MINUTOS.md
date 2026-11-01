# ⏱️ Guía Exprés (10 Minutos): Cómo Añadir el Clima a tu CV con una API REST

Si estás editando tu CV en esta carpeta (`project_personal_CV`) y quieres incorporar una llamada a una API en vivo tal y como hicimos en `8. PRACTICA_CLASE 2026/index_api.html`, sigue estos **3 pasos exactos**:

---

## 🗺️ El Mapa de lo que vas a hacer
Vas a tocar únicamente 3 archivos:
1. [`index.html`](file:///Users/fjbanezares/IdeaProjects/Hands-On-HTML-and-CSS-Building-Real-World-Websites-and-Web-Apps/project_personal_CV/index.html) (Añadir la caja visual y el enlace al script).
2. [`styles-modified.css`](file:///Users/fjbanezares/IdeaProjects/Hands-On-HTML-and-CSS-Building-Real-World-Websites-and-Web-Apps/project_personal_CV/styles-modified.css) (Pegar los estilos visuales del widget).
3. `clima.js` (Crear este archivo con el código que conecta con la API).

---

## Paso 1: En tu `index.html`

### A) Añade la sección visual
Dentro de `<div id="cv">`, justo después de tu `<header>` o de tu sección de contacto, pega este bloque:

```html
        <!-- ============================================== -->
        <!-- WIDGET DEL CLIMA CON API REST (OPEN-METEO)     -->
        <!-- ============================================== -->
        <section id="widget-clima" class="seccion-clima">
            <h2>🌤️ Estado del Tiempo en Directo</h2>
            <div class="caja-clima-cv">
                <input type="text" id="ciudad-clima" value="Madrid" placeholder="Escribe tu ciudad...">
                <button type="button" id="btn-clima">Ver Clima</button>
            </div>
            <div id="resultado-clima" class="resultado-clima">
                Pulsa el botón para consultar el clima.
            </div>
        </section>
```

### B) Conecta el archivo JavaScript
Baja al final del archivo `index.html` y, **justo antes de la etiqueta `</body>`**, añade:

```html
    <!-- Script que conecta con la API -->
    <script src="clima.js"></script>
</body>
</html>
```

---

## Paso 2: En tu `styles-modified.css`

Abre `styles-modified.css` y añade al final estas reglas CSS:

```css
/* ======================================================= */
/* ESTILOS DEL WIDGET DEL CLIMA (API REST)                 */
/* ======================================================= */
.seccion-clima {
    background: #f0f7ff;
    border: 2px solid #cce3fe;
    border-radius: 12px;
    padding: 16px 20px;
    margin: 20px 0;
}

.caja-clima-cv {
    display: flex;
    gap: 10px;
    margin-bottom: 12px;
}

.caja-clima-cv input {
    flex: 1;
    padding: 8px 12px;
    border: 1px solid #94c5fd;
    border-radius: 6px;
    font-size: 15px;
}

.caja-clima-cv button {
    background-color: #2563eb;
    color: white;
    border: none;
    padding: 8px 16px;
    border-radius: 6px;
    font-weight: 600;
    cursor: pointer;
    transition: background-color 0.2s ease;
}

.caja-clima-cv button:hover {
    background-color: #1d4ed8;
}

.resultado-clima {
    font-size: 15px;
    padding: 10px 14px;
    background: #ffffff;
    border-radius: 8px;
    border-left: 4px solid #2563eb;
    color: #1e293b;
}

.resultado-clima.error {
    border-left-color: #ef4444;
    color: #991b1b;
    background: #fef2f2;
}

.resultado-clima.exito {
    border-left-color: #10b981;
    color: #065f46;
    background: #ecfdf5;
}
```

---

## Paso 3: Crea el archivo `clima.js`

Crea un archivo nuevo llamado `clima.js` en esta misma carpeta `project_personal_CV/` y pega este código:

```javascript
/**
 * clima.js - Conexión con la API meteorológica pública Open-Meteo
 */
async function obtenerClima() {
    const input = document.getElementById("ciudad-clima");
    const resultado = document.getElementById("resultado-clima");

    // 1. Tomamos la ciudad escrita por el usuario
    const ciudad = (input.value.trim() !== "") ? input.value.trim() : "Madrid";

    // 2. Mensaje de carga mientras la red responde
    resultado.className = "resultado-clima";
    resultado.innerHTML = `⏳ Consultando el clima para <strong>${ciudad}</strong>...`;

    try {
        // PASO 1: Geocodificación (Obtener Latitud y Longitud)
        const geoUrl = `https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(ciudad)}&count=1&language=es&format=json`;
        const resGeo = await fetch(geoUrl);
        
        if (!resGeo.ok) throw new Error("No se pudo conectar con el servicio de geocodificación.");

        const dataGeo = await resGeo.json();

        if (!dataGeo.results || dataGeo.results.length === 0) {
            resultado.className = "resultado-clima error";
            resultado.innerHTML = `❌ No se encontró la ciudad "<strong>${ciudad}</strong>".`;
            return;
        }

        const { latitude, longitude, name, country } = dataGeo.results[0];

        // PASO 2: Consulta del Clima actual con las coordenadas
        const weatherUrl = `https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&current=temperature_2m&timezone=auto`;
        const resWeather = await fetch(weatherUrl);

        if (!resWeather.ok) throw new Error("El servicio meteorológico no respondió correctamente.");

        const dataWeather = await resWeather.json();
        const temp = dataWeather.current.temperature_2m;
        const unidad = dataWeather.current_units.temperature_2m;

        // PASO 3: Modificar el DOM con el resultado final
        resultado.className = "resultado-clima exito";
        resultado.innerHTML = `📍 En <strong>${name} (${country || ""})</strong> hay actualmente <strong>${temp} ${unidad}</strong> 🌤️`;

    } catch (error) {
        resultado.className = "resultado-clima error";
        resultado.innerHTML = `⚠️ Fallo de conexión o red: ${error.message}`;
    }
}

// Conectar el botón al cargar el documento
document.addEventListener("DOMContentLoaded", () => {
    const boton = document.getElementById("btn-clima");
    const input = document.getElementById("ciudad-clima");

    if (boton) {
        boton.addEventListener("click", obtenerClima);
    }

    if (input) {
        input.addEventListener("keydown", (e) => {
            if (e.key === "Enter") obtenerClima();
        });
    }

    // Consulta inicial automática para Madrid
    obtenerClima();
});
```

---

## 🛠️ Comprobación rápida (¿Cómo saber si funciona?)
1. Abre tu `index.html` en el navegador (doble clic o mediante Live Server).
2. Verás cómo aparece automáticamente la temperatura de Madrid.
3. Escribe *"Barcelona"*, *"Tokyo"* o *"Sevilla"* y pulsa el botón **Ver Clima** o la tecla **Enter**.
4. Si algo falla, abre las herramientas de desarrollo (**F12** en Windows o **Cmd+Option+I** en Mac) y mira la pestaña **Console**.

