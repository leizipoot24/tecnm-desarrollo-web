# Bitácora de Requerimientos y Auditoría IA — Sesión 10
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Hito 1: Mock API RESTful con OpenAPI y Validation Harness  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Equipo:** [Aviles Darza Javier Alexander, Poot Cabrera Leizi Astril, Yahir Ohany Santoyo Sánchez]  
**Matrícula:** [23531054,  23531123,  23531082    ]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Integración de Router y esquemas de validación  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
La dificultad que encontramos fue organizar las rutas de la API y hacer que cada una respondiera correctamente. También tuve que considerar la validación de los campos del libro, porque el título no debía estar vacío y el año debía cumplir con el valor permitido. Al principio puede ser complicado controlar manualmente todos los errores y códigos HTTP.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | 
Python con FastAPI y Pydantic, ejecutado en Google Colab.|
| **Rol (R)** |Desarrollador especializado en APIs RESTful, validación de datos y códigos HTTP. |
| **Especificación (E)** | Crear una API de libros con operaciones CRUD, identificadores automáticos, validación de título y año, manejo de errores y documentación OpenAPI. |
| **Output (O)** | Archivo app.py con código funcional, rutas GET, POST, PUT y DELETE, modelos de validación y respuestas HTTP correctas. |

### 2.2 Prompt Estructurado
```text
[PegAyudame guiandome paso a paso par poder realizar la actividad explicando y corrigiendo si tengo algun error.]
```

---

## 3. Código Devuelto por el Modelo
```python
%%writefile app.py

from fastapi import FastAPI, HTTPException, Response
from pydantic import BaseModel, Field

app = FastAPI(
    title="Biblioteca API",
    description="Mock API RESTful de libros - Hito 1",
    version="0.1.0"
)


class Libro(BaseModel):
    titulo: str = Field(min_length=1)
    anio: int = Field(ge=1500)


libros_db = {}
siguiente_id = 1


@app.get("/api/libros")
def listar_libros():
    return {"libros": list(libros_db.values())}


@app.post("/api/libros", status_code=201)
def crear_libro(libro: Libro):
    global siguiente_id

    nuevo_libro = {
        "id": siguiente_id,
        "titulo": libro.titulo,
        "anio": libro.anio
    }

    libros_db[siguiente_id] = nuevo_libro
    siguiente_id += 1

    return nuevo_libro

@app.get("/api/libros/{libro_id}")
def consultar_libro(libro_id: int):
    if libro_id not in libros_db:
        raise HTTPException(
            status_code=404,
            detail="Libro no encontrado"
        )

    return libros_db[libro_id]


@app.put("/api/libros/{libro_id}")
def actualizar_libro(libro_id: int, libro: Libro):
    if libro_id not in libros_db:
        raise HTTPException(
            status_code=404,
            detail="Libro no encontrado"
        )

    libros_db[libro_id] = {
        "id": libro_id,
        "titulo": libro.titulo,
        "anio": libro.anio
    }

    return libros_db[libro_id]


@app.delete("/api/libros/{libro_id}", status_code=204)
def eliminar_libro(libro_id: int):
    if libro_id not in libros_db:
        raise HTTPException(
            status_code=404,
            detail="Libro no encontrado"
        )

    del libros_db[libro_id]
    return Response(status_code=204)
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Suite integral de 10 pruebas HTTP destructivas

* [x  Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> 
Durante la revisión comprobé que era importante manejar correctamente los códigos HTTP cuando se intentaba consultar un libro que no existía o utilizar información inválida. También revisé que los métodos no permitidos devolvieran el código 405. Con las pruebas pude comprobar que FastAPI manejaba estas situaciones correctamente y que no se presentaban errores internos 500.
**Firma digital:** [leizipoot24, Javier Alexander2001,sancheznaranjoyairchayane-desing]
