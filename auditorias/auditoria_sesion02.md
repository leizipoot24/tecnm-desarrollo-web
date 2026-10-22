# Bitácora de Requerimientos y Auditoría IA — Sesión 02
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Gestión en GitHub Web y Métodos HTTP (GET, POST, PUT, DELETE)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Leizi Astrol Poot Cabrera]  
**Matrícula:** [23531123]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Enrutamiento manual por verbo HTTP  
Al realizar la práctica, me resultó un poco complicado entender cómo funcionaba cada método HTTP y cómo se utilizaba para trabajar con los libros. Tuve que revisar el código para comprender cómo agregar, consultar, modificar y eliminar información. También aprendí que cada operación debe responder con un código de estado diferente según el resultado.
---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
Elemento C-R-E-O

Definición

Contexto (C)

Python con FastAPI, trabajando desde Google Colab.

Rol (R)

Ayudante para explicar las instrucciones, elaborar el código y resolver dudas de la práctica.

Especificación (E)

Crear una API para agregar, consultar, modificar y eliminar libros usando los métodos GET, POST, PUT y DELETE.

Output (O)

Código en app.py y pruebas para comprobar que las operaciones funcionen correctamente.

### 2.2 Prompt Estructurado
Ayúdame a realizar la práctica de la Sesión 02 usando Python y FastAPI en Google Colab. Necesito crear una API para agregar, consultar, modificar y eliminar libros utilizando los métodos GET, POST, PUT y DELETE. Explícame el código de forma sencilla y ayúdame a corregir los errores para que las pruebas de la actividad funcionen correctamente.

```

---

## 3. Código Devuelto por el Modelo
El código que utilicé fue el archivo app.py, donde se programaron las operaciones para trabajar con los libros.

from fastapi import FastAPI, HTTPException, Response
from pydantic import BaseModel

app = FastAPI(title="CRUD de libros")

class Libro(BaseModel):
    titulo: str
    anio: int

libros = []
siguiente_id = 1

@app.get("/api/libros")
def listar_libros():
    return {"libros": libros}

@app.post("/api/libros", status_code=201)
def crear_libro(libro: Libro):
    global siguiente_id
    nuevo_libro = {
        "id": siguiente_id,
        "titulo": libro.titulo,
        "anio": libro.anio
    }
    libros.append(nuevo_libro)
    siguiente_id += 1
    return nuevo_libro

@app.get("/api/libros/{id}")
def obtener_libro(id: int):
    for libro in libros:
        if libro["id"] == id:
            return libro
    raise HTTPException(
        status_code=404,
        detail="Libro no encontrado"
    )

@app.put("/api/libros/{id}")
def actualizar_libro(id: int, libro: Libro):
    for i, existente in enumerate(libros):
        if existente["id"] == id:
            libro_actualizado = {
                "id": id,
                "titulo": libro.titulo,
                "anio": libro.anio
            }
            libros[i] = libro_actualizado
            return libro_actualizado
    raise HTTPException(
        status_code=404,
        detail="Libro no encontrado"
    )

@app.delete("/api/libros/{id}", status_code=204)
def eliminar_libro(id: int):
    for i, libro in enumerate(libros):
        if libro["id"] == id:
            libros.pop(i)
            return Response(status_code=204)
    raise HTTPException(
        status_code=404,
        detail="Libro no encontrado"
    )

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de métodos no permitidos (HTTP 405)
Pruebas ejecutadas: Verificación de las operaciones de la API y de los métodos HTTP no permitidos.

[X] Las respuestas de los métodos implementados utilizan los códigos esperados, como 200, 201 y 204; los casos de error revisados responden con 404 o 405.

[X] Las ocho pruebas de la actividad terminaron correctamente, según el resultado obtenido.

[] Se comprobó que no existen respuestas HTTP 500 en todos los casos posibles.

[] Se revisaron por separado los encabezados Content-Type: application/json.

Las dos últimas casillas quedan sin marcar porque no comprobamos esos aspectos por separado durante la práctica.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
Durante la actividad tuve un problema porque no se encontraba el archivo verificar.py, necesario para ejecutar las pruebas. Conseguí nuevamente el archivo y pude continuar hasta que las ocho pruebas salieron correctamente.

También aprendí que cada operación utiliza un código de respuesta distinto. Por ejemplo, al agregar un libro se utiliza el código 201 y, cuando se elimina correctamente, se utiliza el 204. Si se intenta buscar un libro que no existe, el programa responde con el código 404.
**Firma digital:** [leizipoot24] 
