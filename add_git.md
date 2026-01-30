# Guía de Actualización del Repositorio - Buenas Prácticas y Gestión Óptima

## Índice
1. [Introducción](#introducción)
2. [Flujo de Trabajo Básico](#flujo-de-trabajo-básico)
3. [Buenas Prácticas de Git](#buenas-prácticas-de-git)
4. [Gestión de Ramas](#gestión-de-ramas)
5. [Convención de Mensajes de Commit](#convención-de-mensajes-de-commit)
6. [Manejo de Conflictos](#manejo-de-conflictos)
7. [Flujos de Trabajo Avanzados](#flujos-de-trabajo-avanzados)
8. [Herramientas y Comandos Útiles](#herramientas-y-comandos-útiles)
9. [Políticas de Seguridad](#políticas-de-seguridad)

## Introducción

Esta guía proporciona las mejores prácticas y procedimientos para actualizar el repositorio de forma eficiente, segura y organizada. Sigue los principios de desarrollo ágil y las prácticas recomendadas para proyectos de software, especialmente para sistemas críticos como plataformas de trading algorítmico.

## Flujo de Trabajo Básico

### 1. Antes de comenzar a trabajar
```bash
# Actualizar la rama principal localmente
git checkout main
git pull origin main

# O si usamos master como rama principal
git checkout master
git pull origin master
```

### 2. Crear una nueva rama para tu trabajo
```bash
# Crear y cambiar a una nueva rama
git checkout -b feature/nombre-de-la-funcionalidad
# o
git checkout -b bugfix/nombre-del-fix
# o
git checkout -b hotfix/nombre-del-hotfix
```

### 3. Durante el desarrollo
```bash
# Verificar el estado actual
git status

# Añadir archivos específicos
git add nombre_del_archivo

# O añadir todos los archivos modificados
git add .

# Hacer commit con mensaje descriptivo
git commit -m "feat: descripción clara de los cambios"

# Subir cambios a la rama remota
git push origin nombre-de-la-rama
```

### 4. Finalizar el trabajo
```bash
# Actualizar tu rama con los últimos cambios de main
git checkout main
git pull origin main
git checkout nombre-de-la-rama
git rebase main

# Opcional: si hay conflictos, resolverlos y continuar
git rebase --continue

# Subir la rama actualizada
git push origin nombre-de-la-rama

# Crear un Pull Request en GitHub
# Una vez aprobado, fusionar en main
```

## Buenas Prácticas de Git

### 1. Commits pequeños y cohesivos
- Divide tus cambios en commits lógicos y pequeños
- Cada commit debe representar una unidad de cambio completa
- Evita commits demasiado grandes que sean difíciles de revisar

### 2. Mensajes de commit claros
- Usa el presente imperativo: "Add feature" no "Added feature"
- La primera línea debe ser descriptiva y tener menos de 50 caracteres
- Si es necesario, agrega cuerpo explicativo separado por línea vacía

### 3. Verificación frecuente de estado
- Usa `git status` regularmente para saber el estado de tu trabajo
- Usa `git diff` para revisar cambios antes de hacer commit

### 4. No hacer commit de código roto
- Asegúrate de que tu código compila y pasa las pruebas antes de hacer commit
- Si necesitas guardar trabajo intermedio, usa `git stash`

## Gestión de Ramas

### Estrategia de Ramas (Git Flow Simplificado)
```
main (producción estable)
├── feature/nueva-funcionalidad
├── bugfix/correccion-importante
└── hotfix/urgente-produccion
```

### Tipos de ramas
- **main/master**: Código listo para producción
- **feature/**: Nuevas funcionalidades (origen: main)
- **bugfix/**: Correcciones menores (origen: main)
- **hotfix/**: Correcciones urgentes en producción (origen: main)

### Comandos útiles de ramas
```bash
# Listar ramas locales y remotas
git branch -a

# Cambiar de rama
git checkout nombre-de-la-rama

# Crear y cambiar a nueva rama
git checkout -b nueva-rama

# Eliminar rama local
git branch -d nombre-rama

# Eliminar rama remota
git push origin --delete nombre-rama
```

## Convención de Mensajes de Commit

Usamos la convención de Angular para mensajes de commit:

### Formato
```
<tipo>(<alcance opcional>): <descripción>

<cuerpo opcional>

<pie de página opcional>
```

### Tipos permitidos
- **feat**: Nueva característica
- **fix**: Corrección de bug
- **docs**: Cambios en documentación
- **style**: Cambios que no afectan el significado del código
- **refactor**: Cambios que no corrigen un bug ni agregan una función
- **perf**: Cambios que mejoran el rendimiento
- **test**: Añadir o corregir tests
- **chore**: Otros cambios que no modifican src o test

### Ejemplos
```
feat(auth): add JWT token expiration check

fix(api): resolve timeout issue in MT5 adapter

docs(readme): update installation instructions

refactor(engine): optimize order processing algorithm
```

## Manejo de Conflictos

### 1. Prevenir conflictos
- Actualiza frecuentemente tu rama con main
- Mantén commits pequeños y enfocados
- Coordina con otros desarrolladores sobre áreas de código compartidas

### 2. Resolver conflictos
```bash
# Si hay conflictos al hacer merge/rebase
git status  # Ver archivos en conflicto

# Editar los archivos conflictivos (buscar marcas de conflicto: <<<<<<<, =======, >>>>>>>)

# Después de resolver:
git add archivo_resuelto
git rebase --continue  # si estabas en rebase
# o
git commit  # si estabas en merge
```

### 3. Herramientas de merge
```bash
# Usar herramienta visual para resolver conflictos
git mergetool
```

## Flujos de Trabajo Avanzados

### 1. Rebase interactivo
```bash
# Reorganizar commits antes de hacer PR
git rebase -i HEAD~n  # donde n es número de commits a revisar
```

### 2. Stash para cambios temporales
```bash
# Guardar cambios temporales
git stash

# Recuperar cambios guardados
git stash pop

# Listar stashes
git stash list
```

### 3. Cherry-pick
```bash
# Aplicar un commit específico de otra rama
git cherry-pick <commit-hash>
```

## Herramientas y Comandos Útiles

### Alias útiles de Git
Agrega estos alias a tu `~/.gitconfig`:
```bash
# Visualizar el log de forma compacta
git config --global alias.lg "log --oneline --graph --decorate --all"

# Estado corto
git config --global alias.s "status -s"

# Diferencia con estadísticas
git config --global alias.d "diff --stat"

# Ver cambios recientes
git config --global alias.r "log --oneline -10"
```

### Comandos de diagnóstico
```bash
# Ver cambios que no han sido staggeados
git diff

# Ver cambios que están staggeados
git diff --staged

# Ver historial de commits
git log --oneline --graph

# Ver cambios por autor
git shortlog -sn
```

## Políticas de Seguridad

### 1. No commitear credenciales
- Nunca subas contraseñas, claves API u otra información sensible
- Usa archivos `.env` o variables de entorno
- Verifica que `.gitignore` esté correctamente configurado

### 2. Validación de commits
- Asegúrate de que el código pasa todas las pruebas antes de hacer commit
- Verifica que no introduces vulnerabilidades de seguridad
- Revisa que cumples con las políticas de privacidad

### 3. Revisión de código
- Todos los cambios deben pasar por revisión de pares (Pull Request)
- Se requiere aprobación antes de mergear a main
- Se recomienda al menos una revisión para cambios pequeños y dos para grandes cambios

### 4. Política de .gitignore
Asegúrate de tener un archivo `.gitignore` completo:
```
# Environment
.env
.env.local
.env.production
.env.development

# Dependencies
node_modules/
__pycache__/
*.pyc
*.pyo
*.pyd
.Python
venv/
env/

# Logs
logs/
*.log

# OS generated files
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db

# IDE
.vscode/
.idea/
*.swp
*.swo
*~
```

## Checklist para Actualizaciones

Antes de hacer push a una rama:
- [ ] El código compila correctamente
- [ ] Pasan todas las pruebas unitarias
- [ ] Se ha probado manualmente la funcionalidad
- [ ] Se ha actualizado la documentación correspondiente
- [ ] Se ha verificado que no hay credenciales en el código
- [ ] Los mensajes de commit siguen la convención
- [ ] Se ha usado un linter si aplica
- [ ] Se ha revisado el código con otros compañeros

Antes de mergear a main:
- [ ] Se ha hecho rebase con main
- [ ] No hay conflictos pendientes
- [ ] Hay al menos una aprobación de PR
- [ ] Se han pasado todas las pruebas de CI
- [ ] Se ha verificado que no rompe funcionalidades existentes
- [ ] Se ha actualizado el CHANGELOG si aplica

## Recomendaciones Finales

1. **Comunica tus intenciones**: Si vas a trabajar en una funcionalidad grande, comunícalo al equipo
2. **Haz commits frecuentes**: Guarda tu progreso regularmente
3. **Usa ramas descriptivas**: Que indiquen claramente qué estás haciendo
4. **Mantén main limpio**: Solo código probado y aprobado debe ir a main
5. **Respeta los flujos de trabajo**: Sigue los procesos establecidos por el equipo
6. **Documenta tus cambios**: Asegúrate de que otros puedan entender tu código
7. **Sé paciente con los conflictos**: Resolver conflictos es parte normal del desarrollo colaborativo

---

**Nota**: Esta guía debe adaptarse según las necesidades específicas del proyecto y las políticas del equipo de desarrollo.