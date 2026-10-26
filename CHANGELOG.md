# Changelog

Todos los cambios relevantes de este flujo se documentan aquí.
Formato basado en [Keep a Changelog](https://keepachangelog.com/es-ES/1.1.0/) y versionado [SemVer](https://semver.org/lang/es/).

## [1.0.1] - 2026-10-25

### Corregido
- Modelo de Groq: `llama-3.3-70b-versatile` respondía "model does not exist or you do not have access". Se cambió a `openai/gpt-oss-20b` y el nodo pasó a llamarse **Groq - Modelo GPT-OSS 20B**.
- Límite de tokens subido de 250 a 1500: gpt-oss razona antes de responder y con 250 terminaba con `finish_reason: "length"` y texto vacío, así que siempre salía el párrafo de respaldo.

## [1.0.0] - 2026-10-25

### Añadido
- Webhook `POST /webhook/snackcheck` que acepta `barcode` en el cuerpo JSON o en la URL (`?barcode=`).
- Guía de uso (200) cuando no llega código de barras.
- Validación de formato: solo dígitos, entre 8 y 14; si no, 400 con ejemplo.
- Consulta a Open Food Facts (sin clave) con *Never Error* para leer el cuerpo del 404, y salida de error con respuesta 503 si la API se cae o tarda más de 10 s.
- Respuesta 404 para productos que no están en la base de datos.
- Respuesta de *datos insuficientes* cuando falta azúcar, sal o grasa: esos productos nunca se clasifican.
- Semáforo por nutriente según los umbrales del fundador.
- Nodo Code de un solo ítem con la puntuación de preocupación (0–3) y la lectura de semáforo.
- Lectura de Nutri-Score con ruta alternativa cuando la letra es `unknown`, unidas con Merge (Append).
- Veredicto final = la peor de las dos lecturas, con registro de cuál decidió.
- Párrafo con IA (Groq, `llama-3.3-70b-versatile`) con tono según el nivel; párrafo de respaldo si Groq falla.
- Respuesta final en texto plano: titular, números 🔴/🟢 y párrafo; cabeceras `X-SnackCheck-Nivel` y `X-SnackCheck-Preocupacion`.
- Notas en cada nodo, dos notas adhesivas con el resumen y las reglas, diagrama Excalidraw, README y registro de pruebas.

### Seguridad
- La exportación no contiene claves: la credencial de Groq se referencia solo por nombre y se elige al importar.
