# Cambios Realizados para Aislamiento de Workspaces

## Resumen de Modificaciones

Se han aplicado los siguientes cambios para que los managers solo vean y trabajen en los workspaces asignados:

### 1. Archivo: `server/models/workspace.js`

#### Cambio en `getWithUser` (línea 261):
```javascript
// ANTES:
if ([ROLES.admin, ROLES.manager].includes(user.role))
  return this.get(clause);

// DESPUÉS:
// Solo admins ven todo
if (user.role === ROLES.admin)
  return this.get(clause);
```

#### Cambio en `whereWithUser` (línea 389):
```javascript
// ANTES:
if ([ROLES.admin, ROLES.manager].includes(user.role))
  return await this.where(clause, limit, orderBy);

// DESPUÉS:
// Solo admins ven todo
if (user.role === ROLES.admin)
  return await this.where(clause, limit, orderBy);
```

### 2. Archivo: `server/endpoints/workspaces.js`

#### Cambio en creación de workspaces (línea 48):
```javascript
// ANTES:
[validatedRequest, flexUserRoleValid([ROLES.admin, ROLES.manager])],

// DESPUÉS:
[validatedRequest, flexUserRoleValid([ROLES.admin])],
```

#### Cambio en todos los demás endpoints (uploads, edición, etc.):
```javascript
// ANTES:
flexUserRoleValid([ROLES.admin, ROLES.manager])

// DESPUÉS:
flexUserRoleValid([ROLES.all])
```

## Comportamiento Resultante

### Administradores (admin):
- ✅ Ven todos los workspaces
- ✅ Pueden crear workspaces
- ✅ Pueden asignar usuarios a workspaces
- ✅ Control total del sistema

### Managers:
- ✅ Solo ven workspaces asignados via `workspace_users`
- ✅ Pueden subir documentos a sus workspaces
- ✅ Pueden configurar agentes en sus workspaces
- ✅ Pueden editar configuración de sus workspaces
- ❌ NO pueden ver otros workspaces
- ❌ NO pueden crear nuevos workspaces

### Usuarios Default:
- ✅ Solo ven workspaces asignados
- ✅ Pueden subir documentos a sus workspaces
- ✅ Pueden usar agentes configurados
- ❌ NO pueden crear workspaces

## Configuración Recomendada

### Proceso de Setup:

1. **Como Admin, crear workspaces:**
   ```
   Workspace: "Grupo 1"
   Workspace: "Grupo 2"
   Workspace: "Grupo 3"
   ```

2. **Crear usuarios manager para cada grupo:**
   ```
   Usuario: manager_grupo1 (rol: manager)
   Usuario: manager_grupo2 (rol: manager)
   Usuario: manager_grupo3 (rol: manager)
   ```

3. **Asignar managers a sus workspaces:**
   - manager_grupo1 → Workspace "Grupo 1"
   - manager_grupo2 → Workspace "Grupo 2"
   - manager_grupo3 → Workspace "Grupo 3"

4. **Crear usuarios normales y asignarlos:**
   ```
   Usuario: user1 (rol: default) → Workspace "Grupo 1"
   Usuario: user2 (rol: default) → Workspace "Grupo 1"
   Usuario: user3 (rol: default) → Workspace "Grupo 2"
   ```

## Listo para Coolify

Los cambios están aplicados y el proyecto está listo para desplegar en Coolify usando:

1. **docker-compose.coolify.yml** - Configuración de despliegue
2. **COOLIFY_DEPLOY.md** - Guía de despliegue
3. Código modificado para aislamiento de workspaces

### Siguientes pasos:
1. Subir cambios a tu repositorio Git
2. Configurar en Coolify usando la guía
3. Crear usuarios y workspaces según la configuración recomendada

## Variables de Entorno Importantes para Coolify

```env
# Claves de seguridad (generar únicas)
SIG_KEY=tu-clave-aleatoria-32-caracteres
SIG_SALT=tu-salt-aleatorio-32-caracteres
JWT_SECRET=tu-jwt-secret-12-caracteres

# Autenticación
AUTH_TOKEN=tu-password-admin

# LLM Configuration
LLM_PROVIDER=openai
OPEN_AI_KEY=sk-tu-api-key
OPEN_MODEL_PREF=gpt-4o-mini
```