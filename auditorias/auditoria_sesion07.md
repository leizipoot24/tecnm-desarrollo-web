# Bitácora de Requerimientos y Auditoría IA — Sesión 07
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Especificación OpenAPI (Swagger) y ReDoc  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Leizi Astril Poot Cabrera]  
**Matrícula:** [23531123]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Anotación manual de metadata de endpoints  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
La dificultad que encontré fue entender cómo se podía mostrar la documentación de mi API y cómo organizar la información de los libros. Al principio mi código mostraba los libros, pero no tenía la respuesta en el formato que pedía el profesor. Tuve que revisar y corregir algunas partes hasta que funcionara como se solicitaba.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | 

Estoy trabajando con Python y FastAPI en Google Colab. Utilizo Pydantic para crear el modelo de los libros. |
| **Rol (R)** | Necesito que la IA me ayude como un programador para crear y organizar una API correctamente. |
| **Especificación (E)** | Crear una API llamada Biblioteca API, con versión 1.0.0. Debe tener las rutas /api/libros, /openapi.json, /docs y /redoc. También debe mostrar el modelo Libro y responder correctamente al consultar los libros. |
| **Output (O)** | Obtener el código completo de app.py, que funcione correctamente y tenga la documentación automática de Swagger y ReDoc. |

### 2.2 Prompt Estructurado
```text
[Ayudame a realizar esta actividad guiandome y explicando cada parte sin excepciones ]
```

---

## 3. Código Devuelto por el Modelo
Tuve que hacer algunas correcciones porque al principio mi programa mostraba una lista de libros, pero el verificador esperaba una lista vacía dentro de la propiedad libros.
Despues la IA me corrigio el error y me proporcionó un código en Python utilizando FastAPI y Pydantic. Con ese código pude crear la aplicación, agregar el modelo Libro y configurar las rutas para mostrar la documentación.
```python
# %%writefile app.py

from fastapi import FastAPI
from pydantic import BaseModel

# Modelo de los libros
class Libro(BaseModel):
    id: int
    titulo: str
    anio: int

# Modelo de respuesta
class ListaLibros(BaseModel):
    libros: list[Libro]

# Crear la aplicación
app = FastAPI(
    title="Biblioteca API",
    version="1.0.0",
    description="API de libros del curso DAB-2201"
)

# Lista de libros vacía
libros = []

# Endpoint para listar libros
@app.get(
    "/api/libros",
    response_model=ListaLibros,
    summary="Listar libros"
)
def listar_libros():
    return {"libros": libros}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Verificación de renderizado de interfaz Swagger UI

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
El error que encontré fue que la respuesta de mi API no coincidía con lo que pedía el verificador. Al principio tenía tres libros registrados, pero la actividad esperaba una lista vacía. Lo corregí cambiando la lista y dejando la estructura solicitada. Después de eso pude cumplir con los requisitos de la actividad.

---
**Firma digital:** [leizipoot24]
