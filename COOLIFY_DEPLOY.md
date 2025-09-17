# Guía de Despliegue de AnythingLLM en Coolify

## Pasos para Desplegar desde tu Repositorio

### 1. Fork o Clonar el Repositorio

Primero, haz un fork del repositorio de AnythingLLM o clona este repositorio a tu cuenta de GitHub:

```bash
git clone https://github.com/Mintplex-Labs/anything-llm.git
cd anything-llm
git remote set-url origin https://github.com/TU_USUARIO/anythingllm.git
git push -u origin main
```

### 2. Configuración en Coolify

#### Opción A: Usando Docker Compose

1. En Coolify, crea un nuevo recurso tipo **Docker Compose**
2. Conecta tu repositorio de GitHub
3. En "Docker Compose Location", usa: `/.coolify/docker-compose.yml`
4. Configura las siguientes variables de entorno:

**IMPORTANTE**: Coolify usa Nixpacks por defecto para Node.js. Para forzar Docker Compose:
- Asegúrate de que esté configurado como "Docker Compose" (no "Application")
- Usa la ruta específica: `/.coolify/docker-compose.yml`

#### Variables de Entorno Requeridas

```env
# SEGURIDAD - CRÍTICAS (Genera strings aleatorios de 32+ caracteres)
SIG_KEY=genera-un-string-aleatorio-de-32-caracteres-minimo
SIG_SALT=genera-otro-string-aleatorio-de-32-caracteres-minimo
JWT_SECRET=genera-un-tercer-string-aleatorio-de-12-caracteres-minimo

# Autenticación para acceso remoto (IMPORTANTE)
AUTH_TOKEN=tu-password-de-acceso-a-la-aplicacion

# Puerto
HOST_PORT=3001

# Configuración del LLM (ajusta según tu proveedor)
LLM_PROVIDER=openai
OPEN_AI_KEY=sk-tu-api-key-de-openai
OPEN_MODEL_PREF=gpt-4o-mini

# Motor de embeddings
EMBEDDING_ENGINE=native
EMBEDDING_MODEL_PREF=Xenova/all-MiniLM-L6-v2

# Base de datos vectorial
VECTOR_DB=lancedb

# Otros
DISABLE_TELEMETRY=true
```

#### Opción B: Usando Dockerfile Directamente

1. En Coolify, crea un nuevo recurso tipo **Application**
2. Selecciona **Docker**
3. Conecta tu repositorio
4. En "Build Pack", especifica:
   - Dockerfile Location: `docker/Dockerfile`
   - Build Context: `.`
5. Configura las mismas variables de entorno listadas arriba

### 3. Configuración de Volúmenes Persistentes

Es importante configurar los volúmenes para mantener los datos persistentes:

- `/app/server/storage` - Almacenamiento principal
- `/app/collector/hotdir` - Directorio para documentos
- `/app/collector/outputs` - Salidas procesadas

### 4. Configuración de Red

- Puerto expuesto: `3001`
- Si usas un proxy reverso (recomendado), configura SSL/TLS en Coolify

### 5. Post-Instalación

Una vez desplegado:

1. Accede a la aplicación usando `AUTH_TOKEN` como contraseña
2. Configura el modo multi-usuario si lo necesitas
3. Crea usuarios con diferentes roles (admin, manager, default)

## Personalización de Permisos

Para modificar los permisos y roles de usuarios, edita estos archivos:

1. **`server/utils/middleware/multiUserProtected.js`** - Define los roles y middleware de autenticación
2. **`server/models/user.js`** - Modelo de usuario y validaciones
3. **`server/utils/helpers/admin/index.js`** - Lógica de permisos jerárquicos
4. **`server/endpoints/admin.js`** - Endpoints de administración

### Ejemplo de Modificación de Roles

Para agregar un nuevo rol personalizado:

1. En `multiUserProtected.js`, agrega el nuevo rol:
```javascript
const ROLES = {
  all: "<all>",
  admin: "admin",
  manager: "manager",
  viewer: "viewer",  // Nuevo rol
  default: "default",
};
```

2. En `user.js`, actualiza la validación:
```javascript
const VALID_ROLES = ["default", "admin", "manager", "viewer"];
```

3. Ajusta los permisos en los endpoints según necesites.

## Troubleshooting

### Error: Permisos de archivo
Si encuentras errores de permisos, asegúrate de que los valores de `UID` y `GID` coincidan con los del sistema:
```env
UID=1000
GID=1000
```

### Error: No se puede conectar al LLM
Verifica que tu API key esté correctamente configurada y que el proveedor sea el correcto.

### Error: Auth Token no funciona
Asegúrate de que `AUTH_TOKEN` esté configurado en las variables de entorno.

## Seguridad Importante

1. **SIEMPRE** genera claves aleatorias únicas para `SIG_KEY`, `SIG_SALT` y `JWT_SECRET`
2. **NUNCA** uses los valores de ejemplo en producción
3. **SIEMPRE** configura `AUTH_TOKEN` para proteger el acceso
4. Considera usar HTTPS (configurable en Coolify)

## Recursos Adicionales

- [Documentación oficial de AnythingLLM](https://docs.anythingllm.com)
- [Guía de configuración multi-usuario](https://docs.anythingllm.com/configuration/multi-user-mode)
- [API Documentation](https://docs.anythingllm.com/api)