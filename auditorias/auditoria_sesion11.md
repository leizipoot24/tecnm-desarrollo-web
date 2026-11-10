# Bitácora de Requerimientos y Auditoría IA — Sesión 11
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Arquitectura en Capas: Router, Servicio y Capa de Datos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Leizi Astril Poot cabrera]  
**Matrícula:** [23531123]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Separación manual de controladores y lógica  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
Una de las dificultades que encontré fue organizar el código para que cada archivo tuviera una función diferente. También tuve que identificar cómo recibir los datos de un libro, validar que el año fuera un número y guardar la información sin mezclar las rutas con la lógica del programa. Al principio parecía más sencillo colocar todo en un solo archivo, pero al separarlo pude comprender mejor cómo funciona cada capa.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Python 3 y FastAPI en Google Colab. |
| **Rol (R)** | 
Desarrollador de aplicaciones web encargado de organizar una API mediante arquitectura en capas. |
| **Especificación (E)** | Crear una API de libros separando las responsabilidades en app, router, servicio y repositorio. Implementar GET y POST, validar los datos recibidos y conservar los códigos HTTP 200, 201 y 422. |
| **Output (O)** |
Cuatro archivos Python independientes: app.py, router.py, servicio.py y repositorio.py, con código funcional y organizado.|

### 2.2 Prompt Estructurado
```text
Ayudame a realizar mi actividad guiando y explicandome paso a paso lo que estoy haciendo
```

---

## 3. Código Devuelto por el Modelo
```python
 %%writefile app.py

from fastapi import FastAPI
from router import router

app = FastAPI(title="Biblioteca por capas")

app.include_router(router)

```
%%writefile repositorio.py

libros = []

def listar_libros():
    return libros

def guardar_libro(libro):
    libros.append(libro)
    return libro
```
%%writefile repositorio.py

libros = []

def listar_libros():
    return libros

def guardar_libro(libro):
    libros.append(libro)
    return libro
```
%%writefile servicio.py

from repositorio import listar_libros, guardar_libro

def obtener_libros():
    return listar_libros()

def crear_libro(titulo, anio):
    libros_actuales = listar_libros()

    nuevo_libro = {
        "id": len(libros_actuales) + 1,
        "titulo": titulo,
        "anio": anio
    }

    return guardar_libro(nuevo_libro)
```
    %%writefile router.py

from fastapi import APIRouter
from pydantic import BaseModel
from servicio import obtener_libros, crear_libro

router = APIRouter()

class Libro(BaseModel):
    titulo: str
    anio: int

@router.get("/api/libros")
def listar():
    return {"libros": obtener_libros()}

@router.post("/api/libros", status_code=201)
def crear(libro: Libro):
    return crear_libro(libro.titulo, libro.anio)
---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de aislamiento de errores en lógica de negocio

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
Durante la revisión comprendí que era importante mantener los códigos de respuesta que solicitaba la actividad. Una parte importante fue configurar el código 201 para cuando se registra un libro nuevo. También comprobé que FastAPI rechazara los datos incorrectos, como cuando se envía el año escrito con letras en lugar de un número, devolviendo el código 422. No se agregó una respuesta 400 porque no era necesaria para el contrato solicitado.

---
**Firma digital:** [leizipoot24]
