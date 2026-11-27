# Bitácora de Requerimientos y Auditoría IA — Sesión 19
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Sanitización de Entradas, Prevención de XSS e Inyecciones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Leizi Astril Poot Cabrera]  
**Matrícula:** [23531123]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Limpieza manual de HTML y caracteres especiales  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> Al principio se me dificultó identificar cuáles caracteres del HTML podían representar un riesgo y cómo convertirlos manualmente en texto seguro. También tenía que cuidar que los comentarios normales no cambiaran al procesarlos. Comprendí que era importante proteger la información antes de guardarla y devolverla.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** |Se utiliza Python con FastAPI para desarrollar una API REST. El entorno de trabajo es Google Colab y se utiliza Pydantic para recibir y validar los datos. |
| **Rol (R)** |Actuar como desarrollador backend especializado en seguridad de aplicaciones web, implementando mecanismos de sanitización para prevenir ataques XSS. |
| **Especificación (E)** | rear dos endpoints: POST /api/comentarios para registrar comentarios y GET /api/comentarios para consultarlos. Los caracteres HTML especiales deben escaparse antes de guardar la información. El registro debe responder con HTTP 201 y la consulta con HTTP 200. |
| **Output (O)** | Un archivo app.py funcional, organizado y comentado, que utilice FastAPI y permita registrar y consultar comentarios de forma segura. |

### 2.2 Prompt Estructurado
```text
Ayudame guiandome y explicando cada paso en esta actividad
```

---

## 3. Código Devuelto por el Modelo
```python

%%writefile app.py
from fastapi import FastAPI
from pydantic import BaseModel
from html import escape

app = FastAPI(title="API de Comentarios Seguros")

# Modelo para recibir los comentarios
class ComentarioEntrada(BaseModel):
    texto: str

# Almacenamiento temporal
comentarios = []
contador_id = 1


# Registrar un comentario
@app.post("/api/comentarios", status_code=201)
def crear_comentario(datos: ComentarioEntrada):
    global contador_id

    # Escapar los caracteres HTML antes de guardar
    texto_seguro = escape(datos.texto, quote=True)

    comentario = {
        "id": contador_id,
        "texto": texto_seguro
    }

    comentarios.append(comentario)
    contador_id += 1

    return comentario


# Consultar todos los comentarios
@app.get("/api/comentarios")
def listar_comentarios():
    return {
        "comentarios": comentarios
    }

```

==============================================================================

## VERIFICACIÓN DEL RESULTADO · 3 casos · app.py

==============================================================================

✅ 01. Un comentario normal se guarda

✅ 02. El script se neutraliza

✅ 03. El listado tampoco devuelve scripts

------------------------------------------------------------------------------

RESULTADO: 3/3 casos correctos. ¡Resultado logrado!


---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de inyección de scripts en payloads

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> 
> La IA no presentó fallas en los tres casos principales de seguridad establecidos en la actividad. Sin embargo, fue necesario comprobar que los comentarios normales no se modificaran y que las etiquetas HTML peligrosas se escaparan correctamente. La verificación confirmó que los tres casos funcionaron correctamente, por lo que no fue necesario realizar correcciones adicionales.
---
**Firma digital:** [leizipoot24]
