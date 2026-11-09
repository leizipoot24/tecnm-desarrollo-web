# Bitácora de Requerimientos y Auditoría IA — Sesión 09
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Middleware Web, Intercepción y Logging de Transacciones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Leizi Astril Poot Cabrera]  
**Matrícula:** [23531123]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Middleware manual de tiempo de respuesta (X-Process-Time)  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
La dificultad que encontré fue entender cómo medir el tiempo que tarda cada petición y cómo agregar los encabezados a las respuestas. También tuve que revisar cómo conservar el identificador que envía el cliente y cómo generar uno nuevo cuando no existe. Se me complicó un poco comprobar que el middleware también funcionara cuando la ruta no existía y devolvía un error 404.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Estoy trabajando con Python y FastAPI en Google Colab para desarrollar una API web. |
| **Rol (R)** | Quiero que la IA me ayude como alguien que conoce de programación y que me explique de manera sencilla cómo realizar la actividad. |
| **Especificación (E)** | Crear un middleware que mida el tiempo de cada petición, agregue los encabezados X-Process-Time y X-Request-ID, conserve el identificador enviado por el cliente y también funcione en respuestas 404. |
| **Output (O)** | Obtener el código completo de app.py, con explicaciones sencillas y pruebas para comprobar que los tres criterios de la actividad funcionen correctamente. |

### 2.2 Prompt Estructurado
```text
Ayudame a realizar esta actividad guiandome y explicandome paso por paso
```

---

## 3. Código Devuelto por el Modelo
```python

%%writefile app.py

from fastapi import FastAPI, Request
import time
import uuid

app = FastAPI(
    title="Todas tus respuestas llevarán estas cabeceras"
)

# Middleware para medir el tiempo y agregar identificadores
@app.middleware("http")
async def middleware_trazabilidad(request: Request, call_next):

    # Registrar el tiempo inicial
    inicio = time.perf_counter()

    # Conservar el ID enviado por el cliente o generar uno nuevo
    request_id = request.headers.get("X-Request-ID")

    if not request_id:
        request_id = uuid.uuid4().hex

    # Procesar la petición
    response = await call_next(request)

    # Calcular el tiempo de procesamiento
    tiempo_proceso = time.perf_counter() - inicio

    # Agregar encabezados a la respuesta
    response.headers["X-Process-Time"] = f"{tiempo_proceso:.4f}"
    response.headers["X-Request-ID"] = request_id

    return response


# Endpoint para consultar los libros
@app.get("/api/libros")
async def obtener_libros():
    return {"libros": []}

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de inyección de encabezados de rastreo

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
Al revisar el funcionamiento me di cuenta de que no solamente debía comprobar las respuestas correctas, sino también las respuestas de error. Por eso agregué y probé una ruta que no existe para verificar que respondiera con código 404 y que también incluyera los encabezados del middleware. Con esto pude comprobar que el seguimiento de las peticiones funcionara en ambos casos. 

---
**Firma digital:** [leizipoot24]
