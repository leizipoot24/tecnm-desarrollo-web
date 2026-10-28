# Bitácora de Requerimientos y Auditoría IA — Sesión 06
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Validación Estricta, Constraints y Params Query/Body  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Leizi Astril Poot Cabrera]  
**Matrícula:** [23531123]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Filtro manual de rangos y longitudes de texto  
La dificultad que encontré fue hacer que el programa revisara correctamente los datos de los libros. Tenía que asegurarme de que el título tuviera la cantidad de letras permitida, que el año estuviera dentro del rango y que el precio fuera mayor que cero. También debía evitar que se pudieran consultar los libros usando un límite incorrecto.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** |Trabajé con Python, FastAPI y Pydantic en Google Colab.|
| **Rol (R)** | Necesitaba ayuda para crear una API que registrara libros y revisara que los datos fueran correctos. |
| **Especificación (E)** | El título debía tener entre 3 y 80 caracteres, el año debía estar entre 1450 y 2100 y el precio tenía que ser mayor que cero. El límite para consultar libros debía estar entre 1 y 100. |
| **Output (O)** | Obtener un archivo app.py que funcionara correctamente y permitiera registrar y consultar libros. |

### 2.2 Prompt Estructurado
```text
[Guiame paso a paso explicando cada parte de la actividad para hacerlo juntos y entenderle mejor  y pueda cumplir con las especificaciones, no me lo des ya resuelto se mi asistente.]
```

---

## 3. Código Devuelto por el Modelo
```python
%%writefile app.py
from fastapi import FastAPI, Query
from pydantic import BaseModel, Field

# Creamos la aplicación de FastAPI
app = FastAPI(title="API de libros")

# Lista donde se guardan los libros
libros = []

# Contador para asignar un ID a cada libro
siguiente_id = 1


# Modelo que define los datos y sus restricciones
class Libro(BaseModel):
    titulo: str = Field(min_length=3, max_length=80)
    anio: int = Field(ge=1450, le=2100)
    precio: float = Field(gt=0)


# Ruta para registrar un libro
@app.post("/api/libros", status_code=201)
def crear_libro(libro: Libro):
    global siguiente_id

    # Creamos el registro con su identificador
    nuevo_libro = {
        "id": siguiente_id,
        "titulo": libro.titulo,
        "anio": libro.anio,
        "precio": libro.precio
    }

    # Guardamos el libro y aumentamos el contador
    libros.append(nuevo_libro)
    siguiente_id += 1

    # Devolvemos el libro creado
    return nuevo_libro


# Ruta para consultar los libros con un límite de resultados
@app.get("/api/libros")
def listar_libros(limite: int = Query(default=10, ge=1, le=100)):
    return libros[:limite]
```
La IA me proporcionó un código para crear la API de libros. El código utiliza FastAPI para crear las rutas y Pydantic para validar los datos. Lo revisé y realicé las correcciones necesarias para que cumpliera con lo que pedía la actividad. corrigio los errores en el codigo que yo tenia.
---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de desbordamiento de cadenas y números negativos
Durante esta parte revisé que el programa aceptara los datos correctos y rechazara los que no cumplían con las reglas. También se debía comprobar que los códigos de respuesta fueran los adecuados y que no ocurrieran errores inesperados.

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
Lo que más tuve que revisar fue que las validaciones estuvieran bien configuradas. Para solucionarlo, utilicé las restricciones del modelo para el título, el año y el precio, además de establecer un límite para las consultas. Aprendí que es importante validar los datos desde el servidor porque no siempre se puede confiar en la información que envía el usuario.

---
**Firma digital:** [leizipoot24]
