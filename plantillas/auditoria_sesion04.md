# Bitácora de Requerimientos y Auditoría IA — Sesión 04
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Anatomía RESTful, URIs Nombres y Recursos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Leizi Astril Poot Cabrera]  
**Matrícula:** [23531123]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Diseño de URIs jerárquicas sin verbos en la ruta  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La dificultad que encontré fue organizar las rutas para consultar los libros de cada autor y comprobar que los datos fueran correctos. También tuve que considerar cómo filtrar los libros por género y limitar los resultados. Aprendí que la ruta debe representar el recurso y que el método HTTP indica la operación que se realiza.
---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Python con FastAPI, ejecutado en Google Colab |
| **Rol (R)** | Asistente de programación especializado en el diseño de APIs RESTful. |
| **Especificación (E)** | Crear las rutas para consultar los libros de un autor, filtrar por género, limitar los resultados, comprobar si existe el autor y consultar la colección completa. |
| **Output (O)** | Archivo app.py con las rutas solicitadas, respuestas JSON y códigos HTTP adecuados. |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy trabajando en Google Colab con Python y FastAPI
para una actividad de Desarrollo de Aplicaciones Web.

Rol:
Actúa como asistente de programación y ayúdame a crear
una API RESTful de autores y libros.

Especificación:
Crea GET /api/autores/{autor_id}/libros para consultar
los libros de un autor.
Permite los parámetros opcionales genero y limite.
Si el autor no existe, responde con HTTP 404 y el mensaje
"Autor no encontrado".
Crea GET /api/libros para consultar todos los libros.
Registra a Gabriel García Márquez con los libros
"Cien años de soledad" (novela) y "Doce cuentos
peregrinos" (cuento), y a Julio Cortázar con "Rayuela"
(novela).
Las rutas deben representar recursos y no acciones.

Output:
Entrega el código completo de app.py, con respuestas
JSON y códigos HTTP adecuados. Explica cómo comprobar
las rutas y sus resultados.
```

---

## 3. Código Devuelto por el Modelo
```python
%%writefile app.py
from fastapi import FastAPI, HTTPException
from typing import Optional

app = FastAPI(title="API de libros y autores")

autores = {
    1: {"id": 1, "nombre": "Gabriel García Márquez"},
    2: {"id": 2, "nombre": "Julio Cortázar"}
}

libros = [
    {"id": 1, "autor_id": 1, "titulo": "Cien años de soledad", "genero": "novela"},
    {"id": 2, "autor_id": 1, "titulo": "Doce cuentos peregrinos", "genero": "cuento"},
    {"id": 3, "autor_id": 2, "titulo": "Rayuela", "genero": "novela"}
]

@app.get("/api/libros")
def mostrar_libros():
    return {"libros": libros}

@app.get("/api/autores/{autor_id}/libros")
def buscar_libros(autor_id: int, genero: Optional[str] = None,
                  limite: Optional[int] = None):

    if autor_id not in autores:
        raise HTTPException(status_code=404, detail="Autor no encontrado")

    resultado = []

    for libro in libros:
        if libro["autor_id"] == autor_id:
            resultado.append(libro)

    if genero is not None:
        resultado = [
            libro for libro in resultado
            if libro["genero"].lower() == genero.lower()
        ]

    if limite is not None:
        if limite < 0:
            raise HTTPException(
                status_code=422,
                detail="El limite no puede ser negativo"
            )
        resultado = resultado[:limite]

    return {
        "autor_id": autor_id,
        "libros": resultado
    }
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de peticiones a recursos inexistentes (404)

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
>La respuesta HTTP 404 para un autor inexistente se incluyó mediante una validación que comprueba si el autor está registrado antes de consultar sus libros. También revisé que las respuestas de consulta fueran JSON.

Las pruebas de esta actividad se enfocan principalmente en los códigos HTTP 200 y 404. Los códigos 201 y 400 no forman parte de las cuatro pruebas principales descritas en el contrato, por lo que no afirmo haberlos comprobado. El código 422 se utiliza para validar entradas no aceptadas, como un límite negativo en esta implementación. 

---
**Firma digital:** [Leizi Astril Poot Cabrera]
