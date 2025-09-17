# Estrategia de Actualización y Mantenimiento

## ⚠️ Impacto de las Modificaciones en Actualizaciones Futuras

### Lo que estás haciendo ahora:
Al hacer un fork y modificar el repositorio, especialmente los permisos y roles, tendrás estos **desafíos con actualizaciones**:

### 🔴 Problemas Potenciales:

1. **Conflictos de Merge**
   - Cada vez que AnythingLLM actualice los archivos que modificaste, tendrás conflictos
   - Archivos críticos: `user.js`, `multiUserProtected.js`, `admin.js`

2. **Pérdida de Features Nuevas**
   - Si no actualizas, perderás nuevas funcionalidades, parches de seguridad, y mejoras

3. **Incompatibilidad de Base de Datos**
   - Cambios en el schema de usuarios podría romper con actualizaciones

## ✅ Estrategias Recomendadas

### Opción 1: Mínima Modificación (RECOMENDADA)
**No modifiques el core, usa la configuración existente:**

```bash
# Usa los roles existentes (admin, manager, default)
# Configura permisos mediante la UI de AnythingLLM
# Personaliza solo mediante variables de entorno
```

**Ventajas:**
- Actualizaciones directas sin conflictos
- Mantiene compatibilidad total
- Soporte oficial

### Opción 2: Plugin/Extension Pattern
**Crea una capa de abstracción:**

```javascript
// custom-permissions.js - Tu archivo separado
const customPermissions = {
  checkCustomRole: (user, resource) => {
    // Tu lógica personalizada
  }
}

// Inyecta sin modificar el core
module.exports = customPermissions;
```

### Opción 3: Fork con Estrategia de Git (Tu caso actual)

#### Setup Inicial:
```bash
# 1. Agrega el repositorio original como upstream
git remote add upstream https://github.com/Mintplex-Labs/anything-llm.git

# 2. Crea una rama para tus cambios custom
git checkout -b custom-permissions

# 3. Mantén main sincronizado con upstream
git checkout main
git pull upstream main
```

#### Proceso de Actualización:
```bash
# 1. Actualiza main desde upstream
git checkout main
git fetch upstream
git merge upstream/main

# 2. Rebase tus cambios custom
git checkout custom-permissions
git rebase main

# 3. Resuelve conflictos manteniendo tus modificaciones
# 4. Deploy desde custom-permissions
```

## 📋 Plan de Mantenimiento Recomendado

### Si vas a modificar el core:

1. **Documenta TODOS tus cambios:**
```markdown
## Cambios Custom
- [ ] multiUserProtected.js - Líneas 3-8: Nuevo rol 'viewer'
- [ ] user.js - Línea 40: Validación de rol viewer
- [ ] admin.js - Líneas 36-38: Endpoints para viewer
```

2. **Crea tests para tus modificaciones:**
```javascript
// tests/custom-roles.test.js
describe('Custom Viewer Role', () => {
  test('viewer can only read', () => {
    // Tu test
  });
});
```

3. **Script de migración automática:**
```bash
#!/bin/bash
# update-with-custom.sh

# Backup actual
cp -r server/utils server/utils.backup

# Pull últimos cambios
git fetch upstream main

# Intenta merge automático
git merge upstream/main --no-commit

# Si hay conflictos, aplica tus patches
if [ $? -ne 0 ]; then
  echo "Aplicando patches custom..."
  git checkout --ours server/utils/middleware/multiUserProtected.js
  # etc...
fi
```

## 🎯 Mejor Práctica: Configuración Modular

### En lugar de modificar el core, crea un archivo de configuración:

```javascript
// config/custom-roles.json
{
  "roles": {
    "viewer": {
      "inherits": "default",
      "permissions": {
        "read": true,
        "write": false,
        "delete": false
      }
    }
  }
}
```

### Y un middleware wrapper:
```javascript
// middleware/custom-auth.js
const originalAuth = require('../server/utils/middleware/multiUserProtected');

function customRoleValid(allowedRoles) {
  // Tu lógica custom aquí
  return originalAuth.flexUserRoleValid(allowedRoles);
}

module.exports = { customRoleValid };
```

## 🚨 Decisión Crítica

### Pregúntate:
1. **¿Realmente necesitas roles custom?**
   - Los 3 roles existentes cubren la mayoría de casos

2. **¿Puedes lograr lo mismo con workspaces?**
   - AnythingLLM permite permisos granulares por workspace

3. **¿Vale la pena el mantenimiento extra?**
   - Cada actualización = 2-4 horas de trabajo

## 📊 Matriz de Decisión

| Estrategia | Facilidad Update | Flexibilidad | Mantenimiento | Recomendado para |
|------------|-----------------|--------------|---------------|------------------|
| Sin modificar | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ | 90% de casos |
| Variables ENV | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | Configuración básica |
| Fork + Rebase | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐ | Cambios profundos |
| Plugin Pattern | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | Extensibilidad |

## 🔄 Automatización con GitHub Actions

```yaml
# .github/workflows/sync-upstream.yml
name: Sync with Upstream

on:
  schedule:
    - cron: '0 0 * * 0' # Semanal
  workflow_dispatch:

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Sync upstream
        run: |
          git config --global user.email "bot@example.com"
          git config --global user.name "Update Bot"
          git remote add upstream https://github.com/Mintplex-Labs/anything-llm.git
          git fetch upstream
          git checkout main
          git merge upstream/main --no-edit
          git push origin main
      - name: Create PR if conflicts
        if: failure()
        run: |
          echo "Conflictos detectados - revisar manualmente"
          # Crear issue o PR para revisión manual
```

## Conclusión

**Para el 90% de casos:** No modifiques el core. Usa la configuración existente.

**Si DEBES modificar:**
1. Mantén cambios mínimos y documentados
2. Usa git rebase strategy
3. Automatiza lo que puedas
4. Prepárate para 2-4 horas de trabajo por actualización mayor