# Bitácora de Requerimientos y Auditoría IA — Sesión 18
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Rate Limiting, Throttling y Prevención de Abuso  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Leizi Astril Poot Cabrera]  
**Matrícula:** [23531123]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Contador manual de peticiones por IP en memoria  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*La dificultad que encontré fue organizar el contador de peticiones para que cada dirección IP tuviera su propio límite. También tuve que considerar el tiempo de cada solicitud para identificar cuándo habían pasado los 60 segundos. Al principio podía parecer sencillo contar las peticiones, pero era importante evitar que el límite de un cliente afectara a los demás.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Python con FastAPI, utilizando Google Colab y las herramientas del curso |
| **Rol (R)** | 	Actuar como desarrollador backend especializado en protección de API |
| **Especificación (E)** | Crear una ruta GET que permita tres peticiones por cliente en 60 segundos y responda 429 en la cuarta, incluyendo Retry-After. |
| **Output (O)** |Archivo app.py funcional, respuestas JSON y verificación de los códigos HTTP. |

### 2.2 Prompt Estructurado
```text
Ayudame a realizar esta actividad guiandome y explicando 
```

---

## 3. Código Devuelto por el Modelo
```python

%%writefile app.py
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse
from collections import defaultdict, deque
from threading import Lock
import time
import math

app = FastAPI(title="API con Rate Limiting - Sesión 18")

# Configuración del límite
LIMITE_PETICIONES = 3
VENTANA_SEGUNDOS = 60

# Contador independiente para cada cliente
peticiones_por_cliente = defaultdict(deque)

# Protección para evitar conflictos entre solicitudes simultáneas
bloqueo = Lock()


@app.get("/api/recurso")
def obtener_recurso(request: Request):

    # Identificar al cliente mediante su dirección IP
    cliente = request.client.host if request.client else "desconocido"

    ahora = time.monotonic()

    with bloqueo:

        historial = peticiones_por_cliente[cliente]

        # Eliminar peticiones que ya salieron de la ventana
        while historial and ahora - historial[0] >= VENTANA_SEGUNDOS:
            historial.popleft()

        # Comprobar si el cliente superó el límite
        if len(historial) >= LIMITE_PETICIONES:

            tiempo_restante = VENTANA_SEGUNDOS - (ahora - historial[0])
            retry_after = math.ceil(tiempo_restante)

            return JSONResponse(
                status_code=429,
                content={
                    "error": "límite de peticiones alcanzado"
                },
                headers={
                    "Retry-After": str(retry_after)
                }
            )

        # Registrar la petición permitida
        historial.append(ahora)

    return {
        "mensaje": "Petición permitida",
        "estado": "OK"
    }

```
## VERIFICACIÓN DEL RESULTADO · 4 casos · app.py

==============================================================================

✅ 01. Primera petición permitida

✅ 02. Segunda petición permitida

✅ 03. Tercera petición permitida

✅ 04. La cuarta petición se limita

------------------------------------------------------------------------------

RESULTADO: 4/4 casos correctos. ¡Resultado logrado!

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de ráfaga masiva y bloqueo temporal (HTTP 429)

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
La IA inicialmente podía pasar por alto la importancia de calcular correctamente el tiempo restante para volver a permitir las peticiones. Para solucionarlo, se agregó el encabezado Retry-After en la respuesta HTTP 429, indicando cuántos segundos debe esperar el cliente. También se organizó el contador por dirección IP para evitar que un usuario afectara las solicitudes de los demás. Finalmente, el verificador confirmó que las cuatro pruebas solicitadas funcionaron correctamente.
---
**Firma digital:** [leizipoot24]
