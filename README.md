# Porfidev Projects (Monorepo)

Monorepo para alojar el sitio principal y mini-proyectos como **submódulos Git**, administrados con **pnpm workspaces** y desplegados en **DigitalOcean App Platform**.

## 📦 Estructura

```
porfidev-projects/
  apps/
    porfidev-website/   # submódulo (Astro)
  package.json           # scripts del monorepo
  pnpm-workspace.yaml    # configuración de workspaces
  .gitmodules            # definición de submódulos
```

> Nota: cada proyecto dentro de `apps/*` mantiene su propio `package.json` y `node_modules`.

---

## ✅ Requisitos

- Node.js 18+ (recomendado 20+)
- pnpm `10.20.0` (o la versión marcada en `package.json` root)
- Git con acceso a los repos de submódulos (SSH recomendado)

---

## 🚀 Primer uso (clonar + preparar submódulos)

```bash
# 1) Clona el monorepo + submódulos
git clone --recurse-submodules git@github.com:porfidev/porfidev-projects.git
cd porfidev-projects

# (si olvidaste --recurse-submodules)
git submodule update --init --recursive
```

---

## 📥 Instalar dependencias

> pnpm no instala automáticamente dependencias dentro de submódulos. Instala por proyecto:

```bash
# Instalar SOLO el sitio (paquete cuyo "name" es porfidev-website)
pnpm --filter porfidev-website install

# (opcional) Instalar todas las apps si hubiera más
pnpm -r install
```

> Para ver los nombres de los paquetes (workspaces):
```bash
pnpm -r list --depth -1
```

---

## 🧑‍💻 Desarrollo local

### Porfidev Website (Astro)
```bash
# Levantar el sitio Astro (porfidev-website)
pnpm --filter porfidev-website dev
```

---

## 🛠️ Scripts útiles (root)

```bash
# Compilar TODO el monorepo
pnpm -r build

# Compilar solo el sitio
pnpm --filter porfidev-website build

# Preview del sitio (cuando aplica)
pnpm --filter porfidev-website preview
```

---

## 🔄 Flujo de trabajo con submódulos

**Actualizar el submódulo al último commit de su rama (ej. main):**
```bash
git submodule update --remote apps/porfidev-website
git add apps/porfidev-website
git commit -m "chore: bump porfidev-website submodule"
```

**Trabajar dentro del submódulo (commits propios):**
```bash
cd apps/porfidev-website
# ... cambios ...
git add .
git commit -m "feat: nueva sección"
git push origin main

# volver al root y anclar el nuevo commit del submódulo
cd ../..
git add apps/porfidev-website
git commit -m "chore: update submodule ref"
git push
```

**Seguir automáticamente una rama en el submódulo (opcional):**
```bash
git config -f .gitmodules submodule.apps/porfidev-website.branch main
git add .gitmodules
git commit -m "chore: track main branch for porfidev-website"
```

---

## ☁️ Despliegue en DigitalOcean App Platform

### Opción A: SSR (Node)

- **Activar submódulos** al conectar el repo (en la UI: _Include submodules / Recursively clone submodules_).
- Servicio con Node, usando `PORT` y `start`.

`app.yaml` (fragmento):

```yaml
name: porfidev-portfolio
services:
  - name: porfidev-website
    source_dir: apps/porfidev-website
    environment_slug: node-js
    instance_size_slug: basic-xxs
    http_port: 8080
    build_command: "pnpm i --frozen-lockfile && pnpm --filter porfidev-website build"
    run_command: "pnpm --filter porfidev-website start"
    routes:
      - path: /
    envs:
      - key: NODE_ENV
        value: "production"
      - key: PORT
        value: "8080"
```

### Dominios

- Puedes usar **subdominios por app**:
```yaml
routes:
  - path: /
    domain: porfidev.com            # sitio principal
# para otros servicios/demos:
# - path: /
#   domain: canvas.porfidev.com
```

Configura los **CNAME** en tu DNS hacia la App; DO gestiona SSL automáticamente.

---

## 🧪 Troubleshooting

**1) App Platform no ve los submódulos**
- Asegúrate de habilitar **Include submodules** al configurar el repo.
- Si es privado, configura **Deploy Keys**/SSH con acceso de lectura.

**2) `Readiness probe failed / connect: connection refused` (SSR)**
- Verifica que la app **escuche `process.env.PORT`** (el adapter `@astrojs/node` + `start` lo hace).
- Asegúrate de que `http_port` en `app.yaml` coincida (8080).

**3) Recursos estáticos con rutas rotas (subpaths)**
- En Vite/mini-apps estáticas bajo subpath, usa `base: ""` en `vite.config.ts`.

**4) `getCollection('posts')` no encuentra contenido**
- En Astro 5, usa Content Collections en `src/content` o `loader: glob()` con `base` correcta (relativa a `src/content/config.ts`).

---

## 🧱 Añadir una nueva app como submódulo

```bash
# Ejemplo: apps/demo-canvas
git submodule add git@github.com:porfidev/demo-canvas.git apps/demo-canvas
git commit -m "feat: add demo-canvas as submodule"

# Instalar y ejecutar
pnpm --filter demo-canvas install
pnpm --filter demo-canvas dev
```

> Recuerda añadir un servicio adicional en `app.yaml` si quieres desplegarlo (estático o Node) y opcionalmente asignarle un dominio propio.

---

## 🌊 Publicación en Digital Ocean - App Platform

- Elegir el repo desde Git
* Info
  * Resource type
    - Web Service
- Deployment settings
  - Build command
  - ``` pnpm --filter porfidev-website build ```
  - Run command
  - ``` pnpm --filter porfidev-website start ```
- Network
  - Public HTTP Port
  - ``` 8080 ```
  - HTTP request routes
  - ``` / ```

## 📜 Licencia

ISC — © PorfiDev
