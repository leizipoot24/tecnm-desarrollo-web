# Bitácora de Requerimientos y Auditoría IA — Sesión 13
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Manejo de Sesiones Stateful vs Stateless Tokens  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Leizi Astril Poot Cabrera]  
**Matrícula:** [23531123]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Simulador de tabla de sesiones en memoria  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
La dificultad que encontré fue mantener el control de las sesiones de los usuarios y conseguir que el servidor reconociera cuándo una persona había iniciado sesión. También tuve que comprender cómo enviar y recuperar la cookie en las diferentes peticiones HTTP, ya que no solamente era validar el usuario y la contraseña, sino comprobar que la sesión siguiera activa. 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Python 3.10+ y FastAPI 0.110+ ejecutado en Google Colab. Se utilizan Pydantic y almacenamiento de sesiones en memoria. |
| **Rol (R)** | 
Arquitecto de software especializado en APIs RESTful, autenticación y seguridad web.|
| **Especificación (E)** | Implementar login, perfil y logout mediante sesiones Stateful. Validar las credenciales, generar un identificador aleatorio, utilizar una cookie HttpOnly y eliminar la sesión al cerrar sesión. |
| **Output (O)** |Código completo en app.py, con modelos Pydantic, rutas funcionales y respuestas HTTP 200, 401 y 204, cumpliendo el contrato del verificador. |

### 2.2 Prompt Estructurado
```text
[ayudame a realizar mi actividad guiandome paso a paso y explicandome a detalle]
```

---

## 3. Código Devuelto por el Modelo
```python

%%writefile app.py

from fastapi import FastAPI, HTTPException, Response, Cookie, status
from pydantic import BaseModel
import secrets

app = FastAPI(title="Sistema de Sesiones Stateful")

USUARIOS = {
    "ana": "secreta"
}

sesiones = {}


class LoginEntrada(BaseModel):
    usuario: str
    password: str


@app.post("/api/login")
def login(datos: LoginEntrada, response: Response):

    if USUARIOS.get(datos.usuario) != datos.password:
        raise HTTPException(
            status_code=401,
            detail="Credenciales incorrectas"
        )

    session_id = secrets.token_urlsafe(32)

    sesiones[session_id] = datos.usuario

    response.set_cookie(
        key="session_id",
        value=session_id,
        httponly=True,
        samesite="lax",
        path="/"
    )

    return {"mensaje": "sesión iniciada"}


@app.get("/api/perfil")
def perfil(session_id: str = Cookie(default=None)):

    usuario = sesiones.get(session_id)

    if usuario is None:
        raise HTTPException(
            status_code=401,
            detail="Sesión no válida"
        )

    return {"usuario": usuario}


@app.post("/api/logout", status_code=204)
def logout(
    session_id: str = Cookie(default=None)
):

    if session_id:
        sesiones.pop(session_id, None)

    return Response(
        status_code=204,
        headers={
            "Set-Cookie": "session_id=; HttpOnly; Path=/; SameSite=Lax; Max-Age=0"
        }
    )

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de expiración y revocación de sesión

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
*  x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
Durante la primera verificación encontré un problema en la ruta de logout, porque no estaba devolviendo correctamente el código HTTP 204. El verificador mostraba HTTP None, aunque las demás pruebas estaban aprobadas. Para solucionarlo modifiqué la respuesta de logout para devolver directamente un Response con estado 204 y agregué la eliminación de la cookie. Después de realizar el cambio volví a ejecutar las pruebas y conseguí que los cuatro casos fueran aprobados. 

---
**Firma digital:** [leizipoot24]
