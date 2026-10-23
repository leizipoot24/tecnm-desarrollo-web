# Bitácora de Requerimientos y Auditoría IA — Sesión 03
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Headers, Content-Type y Negociación de Contenido  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Leizi Astril Poot Cabrera]  
**Matrícula:** [23531123]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Lectura y emisión manual de encabezados HTTP  
*La dificultad que encontré fue comprender cómo leer y validar los encabezados de una petición HTTP. Tuve que distinguir entre Content-Type, que indica el formato de los datos enviados, y Accept, que indica el formato que acepta el cliente para recibir la respuesta. También fue necesario identificar qué código HTTP debía devolver la API cuando el formato no era compatible.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Python 3.13 y FastAPI en Google Colab, utilizando el verificador proporcionado por el docente. |
| **Rol (R)** |Desarrollar una API REST que valide los encabezados HTTP y gestione libros. |
| **Especificación (E)** | Implementar POST /api/libros y GET /api/libros. El POST acepta application/json y devuelve 201 Created al crear un libro; si recibe un formato no compatible, responde 415 Unsupported Media Type. El GET devuelve JSON con 200 OK y responde 406 Not Acceptable cuando el formato solicitado no es compatible. |
| **Output (O)** |Código Python en app.py, respuestas JSON y códigos HTTP correctos según el contrato de la actividad. |

### 2.2 Prompt Estructurado
```text
[Desarrolla una API REST con Python y FastAPI para registrar y consultar libros. Valida los encabezados Content-Type y Accept, utiliza respuestas JSON y devuelve los códigos HTTP 201, 415, 200 y 406 según las condiciones establecidas en la actividad.]
```

---

## 3. Código Devuelto por el Modelo
```python
%%writefile app.py
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse
from pydantic import BaseModel, ValidationError

app = FastAPI(title="API de libros con negociación de contenido")


# Modelo para validar los datos de un libro
class LibroEntrada(BaseModel):
    titulo: str
    anio: int


# Almacenamiento temporal en memoria
libros = []
siguiente_id = 1


# POST: crear un libro
@app.post("/api/libros")
async def crear_libro(request: Request):
    global siguiente_id

    # Verificar el formato de los datos recibidos
    content_type = request.headers.get("content-type", "")
    formato = content_type.split(";")[0].strip().lower()

    if formato != "application/json":
        return JSONResponse(
            status_code=415,
            content={"detail": "Solo se acepta application/json"}
        )

    # Leer y validar el cuerpo JSON
    try:
        datos = await request.json()
        libro = LibroEntrada.model_validate(datos)
    except (ValueError, ValidationError):
        return JSONResponse(
            status_code=422,
            content={"detail": "Los datos del libro no son validos"}
        )

    # Guardar el libro y asignarle un identificador
    nuevo_libro = {
        "id": siguiente_id,
        "titulo": libro.titulo,
        "anio": libro.anio
    }

    libros.append(nuevo_libro)
    siguiente_id += 1

    return JSONResponse(
        status_code=201,
        content=nuevo_libro,
        media_type="application/json"
    )


# GET: consultar los libros
@app.get("/api/libros")
async def listar_libros(request: Request):
    accept = request.headers.get("accept", "*/*")

    # Revisar los formatos que acepta el cliente
    formatos = [
        parte.split(";")[0].strip().lower()
        for parte in accept.split(",")
    ]

    acepta_json = any(
        formato in (
            "application/json",
            "*/*",
            "application/*"
        )
        for formato in formatos
    )

    if not acepta_json:
        return JSONResponse(
            status_code=406,
            content={"detail": "Solo se ofrece application/json"}
        )

    return JSONResponse(
        status_code=200,
        content=libros,
        media_type="application/json"
    )

```
El código se ejecutó en Google Colab y debía comprobarse con el verificador proporcionado por el docente.

El archivo implementa un modelo Pydantic para validar los datos de los libros, comprueba los encabezados Content-Type y Accept, y utiliza respuestas JSON con los códigos HTTP correspondientes.


---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas con tipos de contenido no soportados (415)

* [X] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
La implementación debe comprobar que los encabezados se validen antes de procesar los datos. Los códigos 415 y 406 permiten distinguir entre un formato de entrada no compatible y un formato de respuesta que el servidor no puede proporcionar. También se debe comprobar que las respuestas correctas incluyan Content-Type: application/json. 

---
**Firma digital:** [leizipoot24]
