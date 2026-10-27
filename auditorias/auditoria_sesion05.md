# Bitácora de Requerimientos y Auditoría IA — Sesión 05
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Pydantic v2 / DTOs en PHP y Tipado Estático  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Leizi Astril Poot Cabrera]  
**Matrícula:** [23531123]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* DTO manual con validación de tipos primitivos  
La dificultad que encontré fue revisar que los datos enviados fueran correctos, porque no todos llegaban con el tipo esperado. Por ejemplo, el año podía venir como texto. También era necesario evitar que se enviaran campos que no estaban definidos. Con Pydantic pude validar estos datos de una manera más sencilla.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Trabajé con Python, FastAPI y Pydantic v2 en Google Colab. |
| **Rol (R)** | 
Crear una API que registre libros y revise que los datos recibidos sean correctos. |
| **Especificación (E)** | Crear una ruta para registrar libros y otra para consultar el listado. Validar el título, año, precio y disponibilidad. Aceptar la conversión de un año numérico escrito como texto, rechazar años no numéricos y campos adicionales. |
| **Output (O)** | Obtener un archivo app.py funcional, con un modelo de entrada y otro de salida, respuestas JSON y códigos HTTP correctos. |

### 2.2 Prompt Estructurado
```text
Contexto: Estoy trabajando con Python, FastAPI y Pydantic v2 en Google Colab.

Rol: Ayúdame a crear una API sencilla para registrar y consultar libros.

Especificación: El modelo de entrada debe tener titulo como texto, anio como entero, precio como decimal y disponible como booleano. Debe aceptar la conversión de "1967" a 1967, pero rechazar un año no numérico y los campos adicionales. La creación debe responder con HTTP 201, los errores de validación con HTTP 422 y el listado con HTTP 200.

Output: Un archivo app.py con un modelo para recibir los datos, otro para devolverlos, una ruta POST para registrar libros y una ruta GET para listarlos.
```

---

## 3. Código Devuelto por el Modelo
```python
%%writefile app.py
from fastapi import FastAPI
from pydantic import BaseModel, ConfigDict

app = FastAPI(title="API de libros")

# Modelo para recibir los datos
class LibroEntrada(BaseModel):
    model_config = ConfigDict(extra="forbid")

    titulo: str
    anio: int
    precio: float
    disponible: bool


# Modelo para devolver los datos
class LibroSalida(BaseModel):
    id: int
    titulo: str
    anio: int
    precio: float
    disponible: bool


libros = []
siguiente_id = 1


@app.post("/api/libros", response_model=LibroSalida, status_code=201)
def crear_libro(libro: LibroEntrada):
    global siguiente_id

    nuevo_libro = {
        "id": siguiente_id,
        "titulo": libro.titulo,
        "anio": libro.anio,
        "precio": libro.precio,
        "disponible": libro.disponible
    }

    libros.append(nuevo_libro)
    siguiente_id += 1

    return nuevo_libro



@app.get("/api/libros")
def listar_libros():
    return {"libros": libros}
```
Se utilizó como base el archivo app.py, que contiene los modelos LibroEntrada y LibroSalida, una ruta POST para registrar libros y una ruta GET para consultar el listado.

Durante la revisión se encontró que el listado devolvía una lista directamente, pero el verificador esperaba un objeto JSON con la clave libros. Se corrigió la respuesta para que tuviera el formato solicitado.
---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de tipos incompatibles y coerción

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
El error que no había previsto fue que el listado devolvía una lista cuando el verificador esperaba un objeto JSON con la clave libros. Lo corregí cambiando la respuesta de la ruta GET para incluir esa clave. Así entendí que no solo importa que los datos sean correctos, sino también que la respuesta tenga el formato esperado.

---
**Firma digital:** [leizipoot24]
