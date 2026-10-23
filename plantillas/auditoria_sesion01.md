# Bitácora de Requerimientos y Auditoría IA — Sesión 01
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Encuadre, Diagnóstico HTTP y Servidor en Colab  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [LEIZI ASTRIL POOT CABRERA]  
**Matrícula:** [23531123]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Procesamiento manual de JSON de petición sin IA  
Al intentar validar la entrada "a mano", me di cuenta de que debía escribir múltiples condicionales: comprobar que el cuerpo existiera, que fuera JSON válido, que contuviera el campo mensaje y que fuera de texto. Si olvidaba una sola comprobación, el servidor podía devolver un error 500 en lugar de rechazar la petición con el código correcto. También tuve que manejar por separado el caso en que el método no coincidiera con la ruta. Era fácil cometer un error y difícil cubrir todos los escenarios sin repetir mucho código.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
Elemento C-R-E-O	Definición Técnica
Contexto (C)	FastAPI 0.110+ sobre Python 3.10+; entorno de ejecución Google Colab; archivos generados en memoria (app.py); dependencias: pydantic, uvicorn, httpx; todo se valida con el cliente de prueba integrado, sin exponer puertos externos
Rol (R)	Arquitecto de software senior especialista en diseño de APIs RESTful: debe respetar códigos HTTP semánticos, separar validación de lógica de negocio, devolver estructuras predecibles y nunca colgarse por entradas mal formadas
Especificación (E)	— GET /api/estado → 200 OK, {"servidor":"ok","curso":"DAB-2201"}
— POST /api/eco con {"mensaje":"texto"} → 200 OK, {"mensaje":"texto","recibido":true}
— JSON inválido o estructura incorrecta → 422 Unprocessable Entity
— Método no permitido → 405 Method Not Allowed
— Content-Type: application/json en todas las respuestas
Output (O)	Archivo app.py modular: instancia FastAPI, modelo MensajeEntrada (Pydantic), rutas definidas con decoradores; sin capturas genéricas de excepción; validación automática; código legible y autoexplicativo
### 2.2 Prompt Estructurado
```text
[Guiame paso a paso desde cero para realizar la actividad explicando lo que se esta haciendo.]
```

---

## 3. Código Devuelto por el Modelo
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI(title="Servidor Sesión 01")

class MensajeEntrada(BaseModel:
    mensaje: str

@app.get("/api/estado", status_code=200)
def estado():
    return {"servidor": "ok", "curso": "DAB-2201"}

@app.post("/api/eco", status_code=200)
def eco(mensaje: MensajeEntrada):
    return {"mensaje": mensaje.mensaje, "recibido": True}
```
Corregí un error de sintaxis en la línea del modelo (MensajeEntrada(BaseModel: → MensajeEntrada(BaseModel)). Por lo demás, la estructura era correcta
---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de códigos de estado HTTP 200, 400 y 422

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
La IA omitió un detalle de sintaxis en la definición de la clase Pydantic (faltó cerrar el paréntesis), lo que habría impedido importar el código. Lo corregí revisando la definición de clases en Python. Por lo demás, la estructura fue correcta: aprovechó la validación automática de FastAPI/Pydantic, por lo que los códigos 422 y 405 funcionaron sin escribir manejo de errores manual. No hubo omisión de códigos de estado HTTP importantes.

---
**Firma digital:** [leizipoot24]
