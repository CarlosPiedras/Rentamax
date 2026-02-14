# Fase 7: Testing, Optimización y Pulido Final

---

## Objetivo

Asegurar la calidad, rendimiento y robustez de toda la aplicación mediante testing exhaustivo, optimización de rendimiento, corrección de bugs y pulido de la experiencia de usuario. Esta fase garantiza que Rentamax funciona de forma fiable para uso diario.

---

## 7.1 Testing Unitario (Vitest)

### Setup de Vitest

- [ ] Instalar y configurar Vitest:
  ```
  npm install -D vitest @testing-library/react @testing-library/jest-dom jsdom
  ```
- [ ] Crear `vitest.config.ts` con configuración para Next.js
- [ ] Crear setup file con providers necesarios (QueryClient, ThemeProvider)
- [ ] Añadir script en `package.json`: `"test": "vitest"`, `"test:coverage": "vitest --coverage"`

### Tests del Motor de Cálculo (Prioridad Alta)

- [ ] `roi-calculator.test.ts`:
  - Test: Inversión total con vivienda nueva (IVA 10% + AJD)
  - Test: Inversión total con vivienda usada (ITP según CCAA)
  - Test: Inversión total con VPO (IVA 4%)
  - Test: Gastos operativos anuales completos
  - Test: IRPF con reducción general 50%
  - Test: IRPF con reducción zona tensionada 90%
  - Test: IRPF con reducción jóvenes 70%
  - Test: IRPF con reducción rehabilitación 60%
  - Test: ROI neto final con datos completos
  - Test: ROE con hipoteca
  - Test: Cashflow mensual positivo y negativo
  - Test: Payback en años
  - Test: Edge case — precio 0
  - Test: Edge case — sin reforma (CAPEX = 0)
  - Test: Edge case — sin hipoteca (ROE = ROI)

- [ ] `reform-calculator.test.ts`:
  - Test: Reforma integral 90m² en rango 60.000-100.000€
  - Test: Reforma cosmética
  - Test: Desglose por partidas suma correctamente
  - Test: Factor de imprevistos se aplica correctamente (10-15%)
  - Test: Licencias/ICIO se calculan sobre presupuesto base
  - Test: Honorarios técnicos se suman cuando hay cambio de distribución
  - Test: Coste de oportunidad temporal (meses sin alquiler)
  - Test: Plusvalía post-reforma en rango 24-33%
  - Test: Edge case — superficie 0

- [ ] `scoring-engine.test.ts`:
  - Test: Score alto (>70) para inmueble con yield >7% y buen precio
  - Test: Score bajo (<40) para inmueble caro con baja rentabilidad
  - Test: Score medio (40-69) para casos intermedios
  - Test: Ponderaciones suman 100%
  - Test: Detección de outliers positivos
  - Test: Clasificación de riesgo correcta

- [ ] `mortgage-calculator.test.ts`:
  - Test: Cuota mensual verificada con calculadora externa
  - Test: Diferentes plazos (15, 20, 25, 30 años)
  - Test: Impacto en ROE
  - Test: Tabla de amortización coherente (suma = capital total)

### Tests de Utilidades

- [ ] Tests de formateo de números (moneda, porcentajes)
- [ ] Tests de normalización de datos del scraper
- [ ] Tests de deduplicación de propiedades
- [ ] Tests de serialización/deserialización de filtros

### Tests de Componentes

- [ ] `PropertyCard.test.tsx` — Renderiza datos correctamente
- [ ] `PropertyFilters.test.tsx` — Filtros actualizan estado
- [ ] `KPIDashboard.test.tsx` — Muestra métricas correctas
- [ ] `FavoriteActions.test.tsx` — Toggle favorito funciona
- [ ] `NotificationBadge.test.tsx` — Muestra conteo correcto

---

## 7.2 Testing E2E (Playwright)

### Setup de Playwright

- [ ] Instalar y configurar Playwright:
  ```
  npm install -D @playwright/test
  npx playwright install
  ```
- [ ] Crear `playwright.config.ts` con configuración para Next.js
- [ ] Configurar base URL: `http://localhost:3000`

### Flujos Críticos E2E

- [ ] **Flujo de Dashboard:**
  - Cargar dashboard
  - Verificar que se muestran KPIs
  - Verificar que la lista de propiedades carga
  - Verificar que el mapa renderiza
  - Verificar tema claro/oscuro funciona

- [ ] **Flujo de Búsqueda y Filtros:**
  - Aplicar filtros de precio
  - Verificar que la lista se actualiza
  - Aplicar filtros de zona
  - Verificar resultados coherentes
  - Limpiar filtros
  - Guardar búsqueda

- [ ] **Flujo de Detalle de Propiedad:**
  - Click en una propiedad
  - Verificar que carga el análisis financiero
  - Verificar desglose de inversión
  - Cambiar parámetros del simulador de hipoteca
  - Verificar que cashflow se recalcula

- [ ] **Flujo de Favoritos:**
  - Añadir propiedad a favoritos
  - Navegar a página de favoritos
  - Verificar que aparece
  - Añadir notas y tags
  - Quitar de favoritos
  - Verificar que desaparece

- [ ] **Flujo de Exportación:**
  - Exportar listado a CSV
  - Verificar que se descarga el archivo
  - Exportar detalle a PDF
  - Verificar que se genera

- [ ] **Flujo de Configuración:**
  - Cambiar CCAA por defecto
  - Verificar que ITP cambia en los cálculos
  - Cambiar reducción IRPF
  - Verificar que el yield neto se recalcula

---

## 7.3 Optimización de Rendimiento

### Rendimiento de Carga

- [ ] Implementar lazy loading de componentes pesados:
  - Mapa: `dynamic(() => import('./PropertyMap'), { ssr: false })`
  - Gráfico scatter: dynamic import
  - Exportadores: dynamic import (solo se cargan al exportar)
- [ ] Code splitting por ruta (automático con App Router)
- [ ] Optimizar imágenes:
  - Usar `next/image` para todas las imágenes de propiedades
  - Configurar dominios permitidos en `next.config.js`
  - Formatos: WebP/AVIF automáticos
  - Tamaños: srcset responsive
- [ ] Prefetch de rutas probables (detalle de propiedad al hover)

### Rendimiento de Datos

- [ ] Implementar paginación en queries de Supabase (no cargar todo):
  - Página de 20-50 inmuebles
  - Scroll infinito con `useInfiniteQuery`
- [ ] Cachear datos con React Query:
  - `staleTime`: 5 minutos para listados
  - `staleTime`: 30 minutos para market_data
  - `gcTime`: 1 hora
- [ ] Debounce en filtros (300ms) para evitar queries excesivas
- [ ] Memoización de cálculos financieros pesados (`useMemo`)

### Rendimiento del Mapa

- [ ] Clustering de markers para evitar render de cientos de pins
- [ ] Lazy load de capas (solo cargar la capa activa)
- [ ] Viewport-based loading (solo cargar inmuebles visibles en el mapa)
- [ ] Debouncear eventos de movimiento del mapa (200ms)

### Análisis de Bundle

- [ ] Ejecutar `next build` y analizar output:
  - Verificar tamaños de chunks
  - Identificar dependencias grandes
- [ ] Instalar `@next/bundle-analyzer` si es necesario para inspeccionar
- [ ] Objetivo: First Load JS < 200KB en la ruta principal

---

## 7.4 Accesibilidad (a11y)

- [ ] Verificar navegación por teclado en toda la app
- [ ] Verificar contraste de colores (WCAG AA mínimo):
  - Especialmente en los badges de scoring (verde/amarillo/rojo)
  - Verificar en modo claro y oscuro
- [ ] Añadir `aria-labels` a elementos interactivos del mapa
- [ ] Verificar que los tooltips son accesibles
- [ ] Verificar que los formularios tienen labels correctos
- [ ] Verificar skip links y estructura de headings

---

## 7.5 Manejo de Errores Global

- [ ] Implementar Error Boundaries en secciones críticas:
  - Mapa (si falla, no tumbar todo el dashboard)
  - Gráficos
  - Cálculos financieros
- [ ] Crear página `src/app/error.tsx` (error global)
- [ ] Crear página `src/app/not-found.tsx` (404)
- [ ] Verificar que ningún error no controlado llega al usuario

---

## 7.6 Revisión y Pulido Final

### UX Review

- [ ] Revisar todos los estados de carga (skeletons correctos)
- [ ] Revisar todos los estados vacíos (mensajes útiles, alternativas)
- [ ] Revisar todos los mensajes de error (claros, accionables)
- [ ] Revisar consistencia de estilos (espaciados, tipografía, colores)
- [ ] Revisar tooltips explicativos en métricas financieras
- [ ] Verificar que no hay "dead ends" (siempre hay opción de navegar)
- [ ] Verificar que los datos están etiquetados ("Estimación IA" vs "Dato Registral")

### Revisión de Código

- [ ] Ejecutar `npm run lint` — 0 errores, 0 warnings
- [ ] Ejecutar `npm run format:check` — todo formateado
- [ ] Ejecutar `npx tsc --noEmit` — 0 errores de TypeScript
- [ ] Revisar que no hay console.log en código de producción
- [ ] Revisar que no hay credenciales o keys hardcodeadas
- [ ] Revisar que `.env.local` no está commiteado

### Documentación Mínima

- [ ] README.md actualizado con:
  - Descripción del proyecto
  - Requisitos (Node.js, npm, Supabase)
  - Instrucciones de instalación y setup
  - Variables de entorno necesarias
  - Cómo ejecutar en desarrollo
  - Cómo ejecutar tests
  - Cómo ejecutar el scraping
- [ ] Verificar que tu socio puede clonar y levantar el proyecto siguiendo el README

---

## 7.7 Checklist Final Pre-Release

- [ ] Todos los tests unitarios pasan (Vitest)
- [ ] Todos los tests E2E pasan (Playwright)
- [ ] `npm run build` sin errores ni warnings
- [ ] Rendimiento aceptable (dashboard carga en < 3 segundos)
- [ ] Tema claro/oscuro funcional en todas las vistas
- [ ] Responsive funcional en móvil, tablet y desktop
- [ ] Favoritos, notificaciones y exportación funcionan end-to-end
- [ ] Motor de cálculo produce resultados verificados y coherentes
- [ ] Scraping extrae datos correctamente de al menos 1 portal
- [ ] Configuración persiste y se aplica correctamente
- [ ] El proyecto puede ser clonado y ejecutado por tu socio desde cero

---

## Criterios de Aceptación (Definition of Done)

- [ ] Cobertura de tests unitarios > 80% en features/analysis
- [ ] Tests E2E pasan para los 6 flujos críticos
- [ ] 0 errores de lint, format y TypeScript
- [ ] Bundle size First Load < 200KB
- [ ] Dashboard carga en < 3 segundos en local
- [ ] README completo y funcional para onboarding de tu socio
- [ ] Todas las funcionalidades documentadas funcionan correctamente
