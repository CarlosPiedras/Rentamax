# Fase 1: Fundación y Setup del Proyecto

---

## Objetivo

Crear la base técnica completa del proyecto: scaffolding de Next.js, configuración de herramientas de calidad de código, conexión con Supabase y estructura de carpetas por features. Al finalizar esta fase, el proyecto debe compilar, tener linting funcional y estar conectado a la base de datos.

---

## 1.1 Scaffolding del Proyecto

### Inicialización de Next.js 15

- [ ] Crear proyecto con `npx create-next-app@latest` con las siguientes opciones:
  - App Router (`/app`)
  - TypeScript activado
  - Tailwind CSS 4.x
  - ESLint activado
  - `src/` directory activado
  - Import alias `@/*`
- [ ] Verificar que `npm run dev` funciona correctamente
- [ ] Limpiar archivos de ejemplo (page.tsx, globals.css por defecto)

### Estructura de Carpetas (Arquitectura por Features)

- [ ] Crear la siguiente estructura base:

```
src/
├── app/                          # App Router (páginas y layouts)
│   ├── layout.tsx                # Layout raíz
│   ├── page.tsx                  # Página principal (redirige a dashboard)
│   ├── dashboard/
│   │   ├── layout.tsx            # Layout del dashboard
│   │   └── page.tsx              # Dashboard principal
│   └── globals.css               # Estilos globales Tailwind
│
├── features/                     # Módulos funcionales del negocio
│   ├── properties/               # Búsqueda y listado de inmuebles
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── lib/
│   │   ├── types/
│   │   └── index.ts
│   ├── analysis/                 # Cálculos financieros (ROI, CAPEX, fiscal)
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── lib/
│   │   ├── types/
│   │   └── index.ts
│   ├── map/                      # Visualización geoespacial (heatmap, capas)
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── lib/
│   │   ├── types/
│   │   └── index.ts
│   ├── favorites/                # Sistema de favoritos/guardados
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── lib/
│   │   ├── types/
│   │   └── index.ts
│   ├── notifications/            # Sistema de alertas
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── lib/
│   │   ├── types/
│   │   └── index.ts
│   └── export/                   # Exportación PDF/Excel/CSV
│       ├── components/
│       ├── hooks/
│       ├── lib/
│       ├── types/
│       └── index.ts
│
├── components/                   # Componentes compartidos (UI genérica)
│   ├── ui/                       # Componentes shadcn/ui
│   ├── layout/                   # Header, Sidebar, Footer
│   └── shared/                   # Componentes reutilizables del proyecto
│
├── lib/                          # Utilidades globales
│   ├── supabase/                 # Cliente y helpers de Supabase
│   │   ├── client.ts
│   │   ├── server.ts
│   │   └── types.ts
│   ├── utils.ts                  # Utilidades generales (cn, formatters)
│   └── constants.ts              # Constantes globales
│
├── hooks/                        # Hooks globales compartidos
│
└── types/                        # Tipos globales TypeScript
    └── index.ts
```

- [ ] Crear los directorios vacíos con archivos `index.ts` de barrel export
- [ ] Verificar que la estructura compila sin errores

---

## 1.2 Configuración de Calidad de Código

### ESLint + Prettier

- [ ] Instalar Prettier: `npm install -D prettier eslint-config-prettier`
- [ ] Crear `.prettierrc` con configuración del proyecto:
  - Semi: false o true (definir convención)
  - Single quotes
  - Tab width: 2
  - Trailing comma: "es5"
- [ ] Crear `.prettierignore`
- [ ] Configurar ESLint para que no entre en conflicto con Prettier
- [ ] Añadir scripts en `package.json`:
  - `"lint": "next lint"`
  - `"format": "prettier --write ."`
  - `"format:check": "prettier --check ."`

### Husky + lint-staged

- [ ] Instalar Husky: `npx husky init`
- [ ] Instalar lint-staged: `npm install -D lint-staged`
- [ ] Configurar lint-staged en `package.json`:
  ```json
  "lint-staged": {
    "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{json,md,css}": ["prettier --write"]
  }
  ```
- [ ] Configurar hook pre-commit de Husky para ejecutar lint-staged
- [ ] Verificar que el hook funciona creando un commit de prueba

### Commits Convencionales

- [ ] Instalar commitlint: `npm install -D @commitlint/cli @commitlint/config-conventional`
- [ ] Crear `commitlint.config.js` con convención `conventional`
- [ ] Añadir hook commit-msg en Husky para validar mensajes
- [ ] Formato de commits: `tipo(scope): descripción`
  - Tipos: `feat`, `fix`, `refactor`, `style`, `docs`, `test`, `chore`
  - Scopes: `properties`, `analysis`, `map`, `favorites`, `notifications`, `export`, `ui`, `db`

---

## 1.3 Tailwind CSS 4.x + shadcn/ui

### Configuración de Tailwind

- [ ] Verificar que Tailwind 4.x está correctamente configurado
- [ ] Definir tema personalizado en `globals.css`:
  - Paleta de colores del proyecto (tonos para rentabilidad: verdes, rojos, amarillos)
  - Variables CSS para modo claro/oscuro
  - Tipografía base (fuente sans-serif legible para datos financieros)
- [ ] Definir breakpoints responsive (mobile-first):
  - `sm`: 640px
  - `md`: 768px
  - `lg`: 1024px
  - `xl`: 1280px
  - `2xl`: 1536px

### shadcn/ui

- [ ] Inicializar shadcn/ui: `npx shadcn@latest init`
- [ ] Instalar componentes base necesarios:
  - `button`, `card`, `badge`, `input`, `select`
  - `dialog`, `sheet`, `tooltip`, `popover`
  - `table`, `tabs`, `separator`
  - `skeleton` (para estados de carga)
  - `toast` / `sonner` (para notificaciones)
- [ ] Verificar que los componentes renderizan correctamente

### next-themes (Modo Claro/Oscuro)

- [ ] Instalar next-themes: `npm install next-themes`
- [ ] Crear `ThemeProvider` y envolverlo en el layout raíz
- [ ] Crear componente `ThemeToggle` con botón de cambio de tema
- [ ] Verificar que el cambio de tema funciona en los componentes shadcn/ui

---

## 1.4 Conexión con Supabase

### Configuración del Cliente

- [ ] Instalar dependencias: `npm install @supabase/supabase-js @supabase/ssr`
- [ ] Crear archivo `.env.local` con las variables de entorno:
  ```
  NEXT_PUBLIC_SUPABASE_URL=tu_url
  NEXT_PUBLIC_SUPABASE_ANON_KEY=tu_anon_key
  ```
- [ ] Crear `.env.example` (sin valores reales) para referencia
- [ ] Añadir `.env.local` al `.gitignore` (verificar que ya está)
- [ ] Crear cliente de Supabase para el navegador (`lib/supabase/client.ts`)
- [ ] Crear cliente de Supabase para el servidor (`lib/supabase/server.ts`)
- [ ] Verificar la conexión con un query de prueba

### Estructura Inicial de la BD

- [ ] Acceder al panel de Supabase y crear el proyecto
- [ ] Verificar que la conexión desde la app funciona
- [ ] (La definición de tablas se hará en la Fase 2)

---

## 1.5 Configuración de Git

### Branches

- [ ] Verificar que el branch `master`/`main` existe
- [ ] Crear branch `dev` desde `main`
- [ ] Establecer `dev` como branch de trabajo principal
- [ ] Flujo: trabajar en `dev` → merge a `main` cuando haya versión estable

### Archivos de Configuración

- [ ] Crear/verificar `.gitignore` completo:
  - `node_modules/`, `.next/`, `.env.local`, `.env*.local`
  - `.vercel/`, `*.tsbuildinfo`
- [ ] Crear `.nvmrc` con la versión de Node.js utilizada

---

## 1.6 Dependencias Adicionales Base

- [ ] Instalar React Query: `npm install @tanstack/react-query`
- [ ] Crear `QueryProvider` y envolverlo en el layout raíz
- [ ] Instalar React Hook Form + Zod: `npm install react-hook-form zod @hookform/resolvers`
- [ ] Verificar que todas las dependencias están correctamente instaladas

---

## Criterios de Aceptación (Definition of Done)

- [ ] `npm run dev` arranca sin errores
- [ ] `npm run build` compila sin errores
- [ ] `npm run lint` pasa sin warnings
- [ ] Prettier formatea correctamente
- [ ] Husky ejecuta lint-staged en cada commit
- [ ] Los commits siguen la convención establecida
- [ ] Supabase se conecta correctamente desde la app
- [ ] El tema claro/oscuro funciona
- [ ] La estructura de carpetas está creada y compila
- [ ] shadcn/ui renderiza componentes correctamente
