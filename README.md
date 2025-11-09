# Flask REST API Template

Una API REST moderna construida con Flask y Flask-RESTX que proporciona autenticación con JWT, gestión de usuarios y una estructura escalable.

## 📋 Tabla de Contenidos

- [Características](#características)
- [Requisitos Previos](#requisitos-previos)
- [Instalación](#instalación)
- [Configuración](#configuración)
- [Uso](#uso)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Endpoints](#endpoints)
- [Autenticación](#autenticación)
- [Docker](#docker)
- [Contribución](#contribución)

## ✨ Características

- ✅ API REST con Flask-RESTX
- ✅ Autenticación con JWT (JSON Web Tokens)
- ✅ Validación de roles de usuario
- ✅ Base de datos SQLite integrada
- ✅ Gestión de usuarios (crear y autenticar)
- ✅ Documentación automática con Swagger
- ✅ Docker listo para producción
- ✅ Estructura modular y escalable

## 📦 Requisitos Previos

- Python 3.8 o superior
- pip (gestor de paquetes de Python)
- Docker (opcional, para ejecutar en contenedores)

## 🚀 Instalación

### 1. Clonar el repositorio

```bash
git clone <tu-repositorio>
cd flask-rest
```

### 2. Crear un entorno virtual

```bash
python -m venv venv
```

### 3. Activar el entorno virtual

**En Windows:**
```bash
venv\Scripts\activate
```

**En macOS/Linux:**
```bash
source venv/bin/activate
```

### 4. Instalar las dependencias

```bash
pip install -r requirements.txt
```

## ⚙️ Configuración

### Variables de Entorno

Crea un archivo `.env` en la raíz del proyecto:

```env
# Base de datos
DB_NAME=api.db

# JWT
JWT_KEY=tu_clave_secreta_jwt_super_segura
```

**Importante:** En producción, utiliza claves seguras y únicas para `JWT_KEY`.

### Estructura de Configuración

La configuración se encuentra en `app/config.py` y carga las variables desde el archivo `.env`:

```python
# app/config.py
SQLALCHEMY_DATABASE_URI = 'sqlite:///database/api.db'
JWT_ACCESS_TOKEN_EXPIRES = timedelta(minutes=30)
JWT_SECRET_KEY = os.environ.get('JWT_KEY')
```

## 🏃 Uso

### Ejecutar la aplicación localmente

```bash
python main.py
```

La API estará disponible en `http://localhost:5000`

### Acceder a la documentación Swagger

Abre tu navegador e ingresa a:
```
http://localhost:5000/
```

Aquí podrás ver y probar todos los endpoints disponibles.

## 📁 Estructura del Proyecto

```
flask-rest/
├── app/
│   ├── __init__.py           # Factory y configuración de la aplicación
│   ├── config.py             # Configuración de la app
│   ├── models/               # Modelos de datos
│   │   ├── __init__.py
│   │   ├── api_models.py     # Modelos para validación de datos
│   │   ├── hello.py          # Modelo Hello
│   │   └── login.py          # Modelo de usuario
│   ├── routes/               # Rutas/Endpoints
│   │   ├── auth.py           # Endpoints de autenticación
│   │   └── hello.py          # Endpoints de prueba
│   ├── services/             # Lógica de negocio
│   │   └── __init__.py
│   └── utils/                # Utilidades
│       ├── __init__.py
│       ├── api.py            # Configuración de Flask-RESTX
│       ├── db.py             # Configuración de SQLAlchemy
│       ├── jwt.py            # Configuración de JWT
│       └── role_validation.py # Validación de roles
├── main.py                   # Punto de entrada de la aplicación
├── Dockerfile                # Configuración de Docker
├── requirements.txt          # Dependencias de Python
└── README.md                 # Este archivo
```

## 🔌 Endpoints

### Hello (Prueba)

#### GET /api/v1/hello/
Devuelve un saludo simple.

**Request:**
```bash
curl -X GET http://localhost:5000/api/v1/hello/
```

**Response:**
```json
{
  "message": "Hello from Flask!"
}
```

#### POST /api/v1/hello/
Devuelve un saludo personalizado.

**Request:**
```bash
curl -X POST http://localhost:5000/api/v1/hello/ \
  -H "Content-Type: application/json" \
  -d '{"name": "Juan"}'
```

**Response:**
```json
{
  "message": "Hello from Flask, Juan!"
}
```

### Autenticación

#### POST /api/v1/auth/token
Autentica un usuario y devuelve un token JWT.

**Request:**
```bash
curl -X POST http://localhost:5000/api/v1/auth/token \
  -H "Content-Type: application/json" \
  -d '{"username": "usuario", "password": "contraseña"}'
```

**Response (200 Success):**
```json
{
  "user_id": 1,
  "user_role": "user",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response (401 Unauthorized):**
```json
{
  "response": "Usuario o contraseña incorrectos."
}
```

#### POST /api/v1/auth/create_user
Crea un nuevo usuario.

**Request:**
```bash
curl -X POST http://localhost:5000/api/v1/auth/create_user \
  -H "Content-Type: application/json" \
  -d '{"username": "nuevo_usuario", "password": "contraseña"}'
```

**Response (200 Success):**
```json
{
  "user_id": 2,
  "user_name": "nuevo_usuario",
  "user_role": "user"
}
```

**Response (200, usuario duplicado):**
```json
{
  "message": "Usuario ya existe."
}
```

## 🔐 Autenticación

### Cómo funciona JWT

1. El usuario se autentica con su nombre de usuario y contraseña
2. El servidor devuelve un token JWT válido por 30 minutos
3. El cliente incluye el token en el header `Authorization: Bearer <token>` para futuras solicitudes

### Usar el token en requests

```bash
curl -X GET http://localhost:5000/api/v1/protected \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### Validación de Roles

Usa el decorador `@role_required` para proteger endpoints que requieren roles específicos:

```python
from app.utils.role_validation import role_required

@auth_ns.route('/admin-only')
class AdminEndpoint(Resource):
    @role_required(['admin'])
    def get(self):
        return {'message': 'Solo administradores'}, 200
```

## 🐳 Docker

### Construir la imagen

```bash
docker build -t flask-rest-api .
```

### Ejecutar el contenedor

```bash
docker run -p 5000:5000 \
  -e JWT_KEY="tu_clave_secreta" \
  -e DB_NAME="api.db" \
  flask-rest-api
```

### Docker Compose (opcional)

Si tienes un `docker-compose.yml`, ejecuta:

```bash
docker-compose up
```

## 📚 Dependencias

| Paquete | Versión | Descripción |
|---------|---------|-------------|
| Flask | 3.0.3 | Framework web |
| Flask-SQLAlchemy | 3.1.1 | ORM para bases de datos |
| Flask-JWT-Extended | 4.6.0 | Autenticación con JWT |
| flask-restx | 1.3.0 | Extensión para APIs REST |
| gunicorn | 22.0.0 | Servidor WSGI para producción |
| python-dotenv | 1.0.1 | Gestión de variables de entorno |

## 🤝 Contribución

Las contribuciones son bienvenidas. Por favor:

1. Fork el proyecto
2. Crea una rama para tu feature (`git checkout -b feature/AmazingFeature`)
3. Commit tus cambios (`git commit -m 'Add some AmazingFeature'`)
4. Push a la rama (`git push origin feature/AmazingFeature`)
5. Abre un Pull Request

## 📝 Licencia

Este proyecto está licenciado bajo la Licencia MIT - consulta el archivo LICENSE para más detalles.

## 📧 Soporte

Para preguntas o problemas, por favor abre un issue en el repositorio.

---

<p align="center">
  <b>Hecho con &#10084; por: Sebastián. </b>
</p>