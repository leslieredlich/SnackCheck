# Registro de pruebas — SnackCheck v1.0.0

**Fecha:** 2026-10-25 · **Datos:** Open Food Facts en vivo · **Endpoint:** `POST /webhook/snackcheck`

Columna **Estado**: ✅ pasó · ⏳ pendiente de correr en tu n8n con la credencial de Groq.

## Funcionales (clasificación)

| ID | Caso | Entrada | Esperado | Obtenido | Estado |
|---|---|---|---|---|---|
| TC-001 | Producto no saludable (Nutella) | `3017620422003` | `no saludable`, azúcar 🔴 56.3, grasa 🔴 30.9, Nutri-Score E, preocupación 2/3, tono cauteloso | `no saludable` · 2/3 · decidió: ambas lecturas coinciden | ✅ |
| TC-002 | Gaseosa: semáforo verde pero Nutri-Score E (Coca-Cola) | `5449000000996` | Semáforo `saludable` (azúcar 10.6 = medio), Nutri-Score gana → `no saludable`, preocupación 0/3 | `no saludable` · 0/3 · decidió: Nutri-Score | ✅ |
| TC-003 | Nutri-Score A, ruta alentadora (muesli Bjorg) | `3229820129488` | `saludable`, todo 🟢, tono alentador | `saludable` · 0/3 · ambas coinciden | ✅ |
| TC-004 | Nutri-Score A pero 1 nutriente alto (almendras Alesto) | `20724696` | Semáforo `moderado` (grasa 53.3) le gana a la A → `moderado` | `moderado` · 1/3 · decidió: semáforo | ✅ |
| TC-005 | Sin Nutri-Score (`unknown`): solo semáforo | `0049000006346` | `moderado` (sal 31.69 alto, dato erróneo de la base), Nutri-Score "sin dato" | `moderado` · 1/3 · semáforo (no hay Nutri-Score) | ✅ |
| TC-006 | Código por la URL en vez del cuerpo | `?barcode=20724696` | Mismo resultado que TC-004 | Igual a TC-004 | ✅ |
| TC-007 | Código con espacios alrededor | `" 0049000006346 "` | Se limpia y se procesa normal | Procesado normal | ✅ |

## Rutas de error

| ID | Caso | Entrada | Esperado | Obtenido | Estado |
|---|---|---|---|---|---|
| TC-008 | Sin código | `{}` | 200, guía JSON con qué hace y cómo usarla | 200 + guía | ✅ |
| TC-009 | Código no numérico | `{"barcode":"abc123"}` | 400 `codigo_invalido` + ejemplo | 400 `codigo_invalido` | ✅ |
| TC-010 | Producto inexistente (OFF responde 404) | `5000108034208` | 404 `producto_no_encontrado`, el flujo no se cae | 404 `producto_no_encontrado` | ✅ |
| TC-011 | Producto sin nutrientes completos (agua, Nutri-Score A) | `3274080005003` | 200 `datos_insuficientes`, faltan azúcar y grasa; **no** "saludable" | 200 `datos_insuficientes` · faltan ["azúcar","grasa"] | ✅ |
| TC-012 | Groq sin credencial o caído | Nutella sin credencial | 200 con veredicto completo y párrafo de respaldo | 200 + párrafo de respaldo cauteloso | ✅ |
| TC-015 | Open Food Facts caído / timeout | Cortar internet y enviar `3017620422003` | 503 `servicio_no_disponible` | — | ⏳ |

## Integración (IA)

| ID | Caso | Esperado | Estado |
|---|---|---|---|
| TC-013a | Tono con Groq real: Nutella (`no saludable`) | Cauteloso y delicado, sin contradecir el nivel | ✅ "…Puedes disfrutarla de vez en cuando y, si buscas algo más equilibrado, prueba una crema de frutos secos con un toque de cacao." (n8n online, `openai/gpt-oss-20b`, 940 tokens) |
| TC-013b | Tono con Groq real: muesli (`saludable`) | Alentador | ✅ "¡Qué buena elección! El muesli con pasas, dátiles y higos tiene un Nutri-Score A… Disfrútalo como parte de una dieta equilibrada y con alegría." |
| TC-013c | Tono con Groq real: almendras (`moderado`) | Alentador, con medida | ✅ Revisado en n8n online (2026-10-25) |

## Rendimiento

| ID | Caso | Esperado | Estado |
|---|---|---|---|
| TC-014 | Tiempo de respuesta de punta a punta | < 5 s con Groq | ⚠️ Muesli: HTTP Open Food Facts 0.31 s + Groq 0.83 s ≈ 1.1 s de trabajo real. n8n online reporta 7.7 s en total en modo prueba (Nutella 7.84 s); la diferencia es tiempo de la plataforma, no de los nodos. Confirmar con `curl -w "%{time_total}"` sobre la URL de producción |

Tiempos medidos sin Groq (curl `time_total`): guía 0.58 s · 400 0.48 s · 404 1.39 s · datos insuficientes 1.54 s · veredictos 1.86–2.37 s.

## Entorno y método

- `snackcheck-workflow.json` se importó **sin cambios** en n8n 1.85.4 local (`n8n import:workflow`), se activó y se probó con `curl` contra `POST /webhook/snackcheck` usando Open Food Facts en vivo. Todos los casos ✅ corresponden a esa ejecución.
- La prueba se corrió sin credencial de Groq, así que el párrafo final fue el de respaldo. Eso valida TC-012: el nodo de IA continúa ante el error.
- Para completar TC-013 y TC-014: selecciona tu credencial de Groq, repite TC-001, TC-003 y TC-004, pega el párrafo obtenido y el tiempo, y cambia ⏳ por ✅. Para TC-015, desconecta la red o cambia temporalmente la URL del nodo HTTP a un host inexistente.
