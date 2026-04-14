# Contribuir a SYNK-IA

Gracias por tu interés en contribuir. Esta guía explica cómo hacerlo de forma ordenada.

---

## Tabla de contenidos

- [Primeros pasos](#primeros-pasos)
- [Reportar bugs](#reportar-bugs)
- [Proponer mejoras](#proponer-mejoras)
- [Flujo de trabajo con Pull Requests](#flujo-de-trabajo-con-pull-requests)
- [Convenciones de commits](#convenciones-de-commits)
- [Estilo de código](#estilo-de-código)
- [Estructura de ramas](#estructura-de-ramas)

---

## Primeros pasos

1. Haz **fork** del repositorio
2. Clona tu fork:
   ```bash
   git clone https://github.com/TU_USUARIO/synk-ia-public.git
   cd synk-ia-public
   ```
3. Instala dependencias:
   ```bash
   npm install
   cd server && npm install && cd ..
   ```
4. Crea una rama para tu cambio:
   ```bash
   git checkout -b feat/mi-mejora
   ```

---

## Reportar bugs

Abre un [issue nuevo](https://github.com/neo44hd/synk-ia-public/issues/new) con esta información:

### Plantilla de bug report

```markdown
**Descripción**
Explicación breve del problema.

**Pasos para reproducir**
1. Ir a '...'
2. Hacer clic en '...'
3. Ver error

**Comportamiento esperado**
Qué debería pasar.

**Comportamiento actual**
Qué pasa realmente.

**Entorno**
- SO: macOS / Linux / Windows
- Node.js: (salida de `node -v`)
- Ollama: (salida de `ollama --version`)
- Navegador: Chrome / Firefox / Safari

**Logs relevantes**
Pega aquí la salida de `pm2 logs sinkia-api --lines 20` o la consola del navegador.

**Capturas de pantalla**
Si aplica.
```

### Antes de reportar

- Busca en [issues existentes](https://github.com/neo44hd/synk-ia-public/issues) para evitar duplicados
- Comprueba que estás en la última versión: `git pull origin main`
- Verifica que Ollama está corriendo: `curl http://localhost:11434/api/tags`

---

## Proponer mejoras

Para proponer una nueva funcionalidad:

1. Abre un issue con el título `[Feature] Descripción breve`
2. Describe el problema que resuelve
3. Propón una solución (opcional)
4. Si la mejora es grande, espera feedback antes de implementar

---

## Flujo de trabajo con Pull Requests

### 1. Sincroniza tu fork

```bash
git remote add upstream https://github.com/neo44hd/synk-ia-public.git
git fetch upstream
git checkout main
git merge upstream/main
```

### 2. Crea tu rama

```bash
git checkout -b tipo/descripcion-corta
# Ejemplos:
#   feat/context-bar-drag-drop
#   fix/ollama-timeout-error
#   docs/api-endpoint-examples
```

### 3. Haz tus cambios

- Mantén los commits pequeños y enfocados
- Prueba que el servidor arranca sin errores: `node server/index.js`
- Comprueba que el frontend compila: `npm run build`

### 4. Commit y push

```bash
git add -A
git commit -m "feat: añadir drag & drop al context bar"
git push origin feat/context-bar-drag-drop
```

### 5. Abre el Pull Request

- Título claro siguiendo las [convenciones de commits](#convenciones-de-commits)
- Describe qué cambia y por qué
- Referencia issues relacionados: `Closes #42`
- Incluye capturas si hay cambios visuales

### Checklist del PR

- [ ] El servidor arranca sin errores
- [ ] `npm run build` compila sin errores
- [ ] No hay secretos, tokens ni datos personales en el código
- [ ] Los commits siguen las convenciones
- [ ] He actualizado la documentación si es necesario

---

## Convenciones de commits

Usamos [Conventional Commits](https://www.conventionalcommits.org/es/). Cada mensaje de commit sigue este formato:

```
tipo(ámbito opcional): descripción breve
```

### Tipos permitidos

| Tipo | Cuándo usarlo |
|------|---------------|
| `feat` | Nueva funcionalidad |
| `fix` | Corrección de bug |
| `docs` | Solo documentación |
| `style` | Formato, punto y coma, etc. (sin cambio de lógica) |
| `refactor` | Refactorización sin cambiar funcionalidad |
| `perf` | Mejora de rendimiento |
| `test` | Añadir o corregir tests |
| `chore` | Tareas de mantenimiento (deps, CI, configs) |
| `ci` | Cambios en CI/CD (GitHub Actions) |

### Ejemplos

```
feat: añadir auto-context RAG a todos los paneles de chat
feat(chat): soporte para streaming de respuestas
fix: corregir timeout en proxy Claude → Ollama
fix(auth): JWT expira después de 24h en lugar de 1h
docs: documentar endpoints de trabajadores
chore: actualizar dependencias de Radix UI
refactor(brain): extraer lógica de contexto a fileContext.js
ci: añadir check de lint a GitHub Actions
```

### Reglas

- Primera línea: máximo 72 caracteres
- Usa imperativo presente: "añadir", no "añadido" ni "añade"
- No termines con punto
- Si el commit cierra un issue: añade `Closes #N` en el cuerpo

```
fix(ollama): usar URL de env en lugar de hardcoded

El proxy de Claude Code usaba localhost:12345 (LM Studio) hardcoded
en vez de leer OLLAMA_URL del .env. Ahora todas las rutas leen
la URL de Ollama desde process.env.

Closes #13
```

---

## Estilo de código

### JavaScript (Backend)

- ES Modules (`import`/`export`, no `require`)
- Comillas simples
- Punto y coma al final
- `const` por defecto, `let` solo cuando sea necesario
- Nombres en inglés para código, comentarios en español si aclaran lógica de negocio
- Manejo de errores con try/catch en cada ruta

```javascript
// Bien
const PORT = process.env.PORT || 3001;
const ADMIN_TOKEN = process.env.ADMIN_TOKEN;

// Mal
var port = 3001;
let token = "hardcoded";
```

### React (Frontend)

- Componentes funcionales con hooks
- Tailwind CSS para estilos (no CSS modules)
- Radix UI para componentes de interfaz accesibles
- Estructura: una página = un archivo en `src/pages/`

### Archivos de configuración

- Variables sensibles van en `.env` (nunca en código)
- Los fallbacks en código deben usar placeholders genéricos:
  ```javascript
  // Bien
  const TOKEN = process.env.ADMIN_TOKEN;
  
  // Aceptable (con placeholder obvio)
  const TOKEN = process.env.ADMIN_TOKEN || 'CHANGE_ME_ADMIN_TOKEN';
  
  // Mal
  const TOKEN = process.env.ADMIN_TOKEN || 'mi-token-real-123';
  ```

---

## Estructura de ramas

| Rama | Propósito |
|------|-----------|
| `main` | Producción estable |
| `feat/*` | Nuevas funcionalidades |
| `fix/*` | Correcciones de bugs |
| `docs/*` | Documentación |
| `chore/*` | Mantenimiento |

---

## Preguntas

Si tienes dudas, abre un [issue con la etiqueta `question`](https://github.com/neo44hd/synk-ia-public/issues/new) o contacta al equipo.

---

Gracias por contribuir a SYNK-IA.
