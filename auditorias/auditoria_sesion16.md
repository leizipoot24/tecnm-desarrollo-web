
# Bitácora de Requerimientos y Auditoría IA — Sesión 16
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** OAuth2 Password Flow y Header Authorization Bearer  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Leizi Astril Poot Cabrera]  
**Matrícula:** [23531123]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Extracción manual del encabezado Authorization  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
La dificultad que encontré fue comprender cómo obtener y validar el token que llega en el encabezado Authorization. También se me complicó diferenciar entre los datos que se envían en el formulario para iniciar sesión y el token que se utiliza después para acceder a una ruta protegida. Con FastAPI pude facilitar este proceso utilizando sus herramientas de OAuth2.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | FastAPI con Python 3.10 o superior, ejecutado en Google Colab. |
| **Rol (R)** | Arquitecto de software especializado en APIs REST y seguridad web. |
| **Especificación (E)** | Implementar OAuth2 Password Flow mediante OAuth2PasswordRequestForm para recibir las credenciales y OAuth2PasswordBearer para validar el encabezado Authorization.|
| **Output (O)** | Archivo app.py funcional con las rutas /token y /api/perfil, validación de usuario y contraseña y respuestas HTTP correctas.s |

### 2.2 Prompt Estructurado
```text
[ayudame a realizar las actividad guiandome paso a paso y explicame]
```

---

## 3. Código Devuelto por el Modelo
```python
# 
%%writefile app.py

from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm

app = FastAPI(title="OAuth2 Password Flow")

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

USUARIOS = {
    "ana": "secreta"
}

TOKENS = {
    "token-ana": "ana"
}


@app.post("/token")
def login(form_data: OAuth2PasswordRequestForm = Depends()):

    usuario = form_data.username
    password = form_data.password

    if usuario not in USUARIOS or USUARIOS[usuario] != password:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Usuario o contraseña incorrectos",
            headers={"WWW-Authenticate": "Bearer"}
        )

    return {
        "access_token": "token-ana",
        "token_type": "bearer"
    }


@app.get("/api/perfil")
def perfil(token: str = Depends(oauth2_scheme)):

    if token not in TOKENS:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Token inválido",
            headers={"WWW-Authenticate": "Bearer"}
        )

    return {
        "usuario": TOKENS[token]
    }

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de acceso no autorizado sin token (HTTP 401)

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
Durante la revisión comprendí que era importante comprobar que las solicitudes sin token recibieran el código HTTP 401 y el encabezado WWW-Authenticate: Bearer. También verifiqué que no se permitiera utilizar otro esquema de autenticación. Para resolverlo utilicé OAuth2PasswordBearer, que facilita la validación del encabezado y el rechazo de solicitudes sin el formato requerido. 

---
**Firma digital:** [leizipoot24]
