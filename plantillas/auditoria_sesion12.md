# Bitácora de Requerimientos y Auditoría IA — Sesión 12
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Inyección de Dependencias (FastAPI Depends / PHP Container)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Leizi Astril Poot Cabrera]  
**Matrícula:** [23531123]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Inyector de dependencias manual por funciones  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
La dificultad que encontré fue comprender cómo pasar el repositorio a los endpoints sin tener que crearlo directamente dentro de cada función. También tuve que cuidar que los datos de los libros se guardaran correctamente y que las validaciones funcionaran cuando se enviaban datos incorrectos.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** |
Trabajar con Python y FastAPI en Google Colab, utilizando las herramientas instaladas para desarrollar y probar la API. |
| **Rol (R)** | 
Actuar como un estudiante de Desarrollo de Aplicaciones Web que busca implementar una API utilizando inyección de dependencias. |
| **Especificación (E)** | Crear un repositorio en memoria, implementar get_repositorio() y utilizar Depends() en los endpoints para registrar y consultar libros. Validar los datos recibidos y respetar los códigos HTTP. |
| **Output (O)** | Obtener un archivo app.py funcional, con los endpoints solicitados, las validaciones correspondientes y las pruebas del verificador en verde. |

### 2.2 Prompt Estructurado
```text
[Ayudame a realizar esta actividad guiandome paso a paso y explicando a detalles.]
```

---

## 3. Código Devuelto por el Modelo
```python

%%writefile app.py

from fastapi import FastAPI, Depends, HTTPException, status
from pydantic import BaseModel, Field, ConfigDict


# Modelo de entrada y validación
class LibroEntrada(BaseModel):
    titulo: str = Field(min_length=1)
    anio: int = Field(ge=1500)

    model_config = ConfigDict(str_strip_whitespace=True)


# Modelo de salida
class LibroSalida(BaseModel):
    id: int
    titulo: str
    anio: int


# Repositorio en memoria
class RepositorioMemoria:

    def __init__(self):
        self.libros = {}
        self.contador_id = 1

    def crear(self, datos: LibroEntrada):
        libro = LibroSalida(
            id=self.contador_id,
            titulo=datos.titulo,
            anio=datos.anio
        )

        self.libros[self.contador_id] = libro
        self.contador_id += 1

        return libro

    def listar(self):
        return list(self.libros.values())

    def obtener(self, libro_id: int):
        return self.libros.get(libro_id)


# Instancia compartida del repositorio
repositorio = RepositorioMemoria()


# Función de inyección de dependencias
def get_repositorio():
    return repositorio


# Aplicación FastAPI
app = FastAPI(title="Biblioteca API - Sesión 12")


# Diagnóstico de la dependencia
@app.get("/api/diagnostico")
def diagnostico(repo: RepositorioMemoria = Depends(get_repositorio)):
    return {
        "repositorio": type(repo).__name__
    }


# Registrar un libro
@app.post(
    "/api/libros",
    response_model=LibroSalida,
    status_code=status.HTTP_201_CREATED
)
def crear_libro(
    datos: LibroEntrada,
    repo: RepositorioMemoria = Depends(get_repositorio)
):
    return repo.crear(datos)


# Consultar todos los libros
@app.get("/api/libros")
def listar_libros(
    repo: RepositorioMemoria = Depends(get_repositorio)
):
    return {
        "libros": repo.listar()
    }


# Consultar un libro por ID
@app.get("/api/libros/{libro_id}", response_model=LibroSalida)
def obtener_libro(
    libro_id: int,
    repo: RepositorioMemoria = Depends(get_repositorio)
):
    libro = repo.obtener(libro_id)

    if libro is None:
        raise HTTPException(
            status_code=404,
            detail="Libro no encontrado"
        )

    return libro

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de ciclo de vida de dependencias por request

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
Durante la revisión presté atención a que los datos incorrectos fueran rechazados con el código HTTP 422 y no provocaran errores internos del servidor. También revisé que los endpoints utilizaran la dependencia correctamente para trabajar con la misma instancia del repositorio. De esta manera, los libros registrados podían consultarse posteriormente.

---
**Firma digital:** [leizipoot24]
