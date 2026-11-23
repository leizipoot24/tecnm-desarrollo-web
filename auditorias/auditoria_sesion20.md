# Bitácora de Requerimientos y Auditoría IA — Sesión 20
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Hito 2: API con Inyección de Dependencias, Auth JWT y RBAC  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Javier Alexander Aviles Darza, Leizi Astril Poot Cabrera, Yahir Santoyo Sanchez]  
**Matrícula:** [23531054, 23531123]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Integración de autenticación y autorización en API  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
Al realizar esta actividad, una de las dificultades fue entender cómo controlar el acceso de los usuarios a las diferentes rutas de la API. También fue necesario comprender cómo guardar las contraseñas de forma segura y cómo comprobar que los datos ingresados fueran correctos. Otra dificultad fue diferenciar los permisos de un usuario normal y los de un administrador. 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Se utilizó Python con FastAPI en Google Colab para desarrollar y probar una API. |
| **Rol (R)** | Desarrollador especializado en Python, creación de API y seguridad web. |
| **Especificación (E)** | 
Crear una API que permita registrar usuarios, iniciar sesión, generar tokens JWT, consultar perfiles y controlar el acceso al panel de administración. |
| **Output (O)** | Archivo app.py con las rutas necesarias, contraseñas protegidas, autenticación y respuestas HTTP correctas. |

### 2.2 Prompt Estructurado
```text
[Guianos paso a paso para realizar esta actividad y explicanos a detalle para que yo pueda enteder]
```

---

## 3. Código Devuelto por el Modelo
```python
# Código devuelto por el LLM
%%writefile app.py
from fastapi import FastAPI, HTTPException, Depends
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from pydantic import BaseModel
from passlib.context import CryptContext
from jose import jwt, JWTError
from datetime import datetime, timedelta

app = FastAPI(title="API de Libros - Hito 2")

# Configuracion de seguridad
SECRET_KEY = "clave_secreta_hito2"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
security = HTTPBearer(auto_error=False)

# Base de datos temporal de usuarios
usuarios = {
    "admin": {
        "usuario": "admin",
        "password": pwd_context.hash("admin123"),
        "rol": "admin"
    }
}

# Modelo para recibir los datos
class UsuarioRegistro(BaseModel):
    usuario: str
    password: str

class UsuarioLogin(BaseModel):
    usuario: str
    password: str

# Funcion para proteger las contrasenas
def verificar_password(password, password_hash):
    return pwd_context.verify(password, password_hash)

# Funcion para crear el token JWT
def crear_token(usuario, rol):
    datos = {
        "sub": usuario,
        "rol": rol,
        "exp": datetime.utcnow() + timedelta(
            minutes=ACCESS_TOKEN_EXPIRE_MINUTES
        )
    }
    return jwt.encode(datos, SECRET_KEY, algorithm=ALGORITHM)

# Funcion para obtener al usuario autenticado
def obtener_usuario(
    credentials: HTTPAuthorizationCredentials = Depends(security)
):
    if credentials is None:
        raise HTTPException(status_code=401, detail="Token requerido")

    try:
        payload = jwt.decode(
            credentials.credentials,
            SECRET_KEY,
            algorithms=[ALGORITHM]
        )
        nombre = payload.get("sub")
        rol = payload.get("rol")

        if nombre is None or nombre not in usuarios:
            raise HTTPException(status_code=401, detail="Token invalido")

        return {"usuario": nombre, "rol": rol}

    except JWTError:
        raise HTTPException(status_code=401, detail="Token invalido")

# Ruta de registro
@app.post("/api/registro", status_code=201)
def registrar_usuario(datos: UsuarioRegistro):
    if datos.usuario in usuarios:
        raise HTTPException(status_code=400, detail="El usuario ya existe")

    usuarios[datos.usuario] = {
        "usuario": datos.usuario,
        "password": pwd_context.hash(datos.password),
        "rol": "user"
    }

    return {"mensaje": "Usuario registrado correctamente",
            "usuario": datos.usuario,
            "rol": "user"}

# Ruta de inicio de sesion
@app.post("/api/token")
def iniciar_sesion(datos: UsuarioLogin):
    usuario = usuarios.get(datos.usuario)

    if not usuario or not verificar_password(
        datos.password, usuario["password"]
    ):
        raise HTTPException(status_code=401, detail="Credenciales incorrectas")

    token = crear_token(usuario["usuario"], usuario["rol"])

    return {"access_token": token, "token_type": "bearer"}

# Ruta del perfil
@app.get("/api/perfil")
def consultar_perfil(usuario=Depends(obtener_usuario)):
    return {
        "usuario": usuario["usuario"],
        "rol": usuario["rol"]
    }

# Ruta exclusiva para administradores
@app.get("/api/admin/panel")
def panel_administrador(usuario=Depends(obtener_usuario)):
    if usuario["rol"] != "admin":
        raise HTTPException(
            status_code=403,
            detail="No tienes permisos de administrador"
        )

    return {
        "mensaje": "Bienvenido al panel administrativo",
        "usuario": usuario["usuario"],
        "rol": usuario["rol"]
    }
```
## VERIFICACIÓN DEL RESULTADO · 7 casos · app.py

==============================================================================
✅ 01. Registro sin filtrar contraseñas
✅ 02. Login del usuario registrado
✅ 03. El usuario accede a su perfil
✅ 04. El usuario no entra al panel admin
✅ 05. El administrador obtiene su token
✅ 06. El administrador entra al panel
✅ 07. Sin token no hay acceso
------------------------------------------------------------------------------
RESULTADO: 7/7 casos correctos. ¡Resultado logrado!
---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Suite de pruebas con usuarios legítimos e intrusos

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
Durante la revisión se debe comprobar que las contraseñas no aparezcan en las respuestas y que los usuarios normales no puedan entrar al panel administrativo. También se debe verificar que las rutas protegidas rechacen las solicitudes sin un token válido.

La verificación final permitirá identificar si existe algún error que necesite corregirse antes de entregar el proyecto.



---
**Firma digital:** [Javier Alexander2001, leizipoot24, ]
