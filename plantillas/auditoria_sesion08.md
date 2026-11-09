# Bitácora de Requerimientos y Auditoría IA — Sesión 08
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Respuestas Semánticas y Excepciones HTTP Personalizadas  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Leizi Astril Poot Cabrera]  
**Matrícula:** [23531123]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Lanzamiento manual de excepciones estructuradas  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
La dificultad que encontré fue hacer que la API identificara correctamente los errores cuando se consultaba un libro que no existía o cuando se enviaban datos incorrectos. También me costó conseguir que los mensajes de error tuvieran el mismo formato y mostraran el código HTTP correspondiente.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Estoy trabajando con Python y FastAPI en Google Colab para crear una API de libros. |
| **Rol (R)** | Quiero que la IA me ayude a desarrollar el código y explicar cómo manejar los errores de la API. |
| **Especificación (E)** | Crear una API que permita consultar y registrar libros, identificar cuando un libro no existe y validar que los datos enviados sean correctos. También debe utilizar los códigos HTTP correspondientes. |
| **Output (O)** | Obtener un archivo app.py que funcione correctamente, muestre mensajes de error claros y cumpla con las pruebas solicitadas. |

### 2.2 Prompt Estructurado
```text
Ayudame a realizar la actividad guiandome paso a paso y explicando a detalle lo que estoy haciendo y si lo estoy haciendo bien y si llega a salir algo mal guiame paso a paso para encontrar la solucion.
```

---

## 3. Código Devuelto por el Modelo
```python
%%writefile app.py

from fastapi import FastAPI, HTTPException, Request
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
from pydantic import BaseModel, Field, field_validator

app = FastAPI(
    title="Biblioteca API - Manejo de errores",
    description="API con respuestas semánticas y excepciones personalizadas",
    version="1.0.0"
)


# Modelo de entrada para los libros
class LibroEntrada(BaseModel):
    titulo: str = Field(min_length=1)
    anio: int = Field(ge=1500)

    @field_validator("titulo")
    @classmethod
    def validar_titulo(cls, valor):
        if not valor.strip():
            raise ValueError("El título no puede estar vacío")
        return valor.strip()


# Excepción personalizada
class LibroNoEncontrado(Exception):
    def __init__(self, libro_id: int):
        self.libro_id = libro_id


# Base de datos inicial
libros = {
    1: {
        "id": 1,
        "titulo": "Rayuela",
        "anio": 1963
    }
}

siguiente_id = 2


# Manejador de libros inexistentes
@app.exception_handler(LibroNoEncontrado)
async def manejar_libro_no_encontrado(
    request: Request,
    exc: LibroNoEncontrado
):
    return JSONResponse(
        status_code=404,
        content={
            "error": {
                "codigo": "NO_ENCONTRADO",
                "mensaje": f"El libro {exc.libro_id} no existe"
            }
        }
    )


# Manejador global de errores de validación
@app.exception_handler(RequestValidationError)
async def manejar_error_validacion(
    request: Request,
    exc: RequestValidationError
):
    return JSONResponse(
        status_code=422,
        content={
            "error": {
                "codigo": "VALIDACION",
                "mensaje": "Revisa los datos enviados"
            }
        }
    )


# Consultar un libro
@app.get("/api/libros/{libro_id}")
async def obtener_libro(libro_id: int):
    if libro_id not in libros:
        raise LibroNoEncontrado(libro_id)

    return libros[libro_id]


# Registrar un libro
@app.post("/api/libros", status_code=201)
async def crear_libro(libro: LibroEntrada):
    global siguiente_id

    nuevo_libro = {
        "id": siguiente_id,
        "titulo": libro.titulo,
        "anio": libro.anio
    }

    libros[siguiente_id] = nuevo_libro
    siguiente_id += 1

    return nuevo_libro
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de captura de errores sin crashing (HTTP 500)

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
Durante la revisión me di cuenta de que era necesario controlar los errores de validación y los libros que no existen para evitar que la API mostrara mensajes incorrectos. Lo corregí agregando una excepción personalizada para los libros inexistentes y un manejador para los errores de validación. También cuidé que los mensajes no mostraran información interna del programa.

---
**Firma digital:** [leizipoot24]
