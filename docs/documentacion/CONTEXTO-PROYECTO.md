# Rentamax — Contexto Completo del Proyecto

> Este documento contiene toda la información necesaria para que cualquier desarrollador o IA comprenda el proyecto, su lógica de negocio, sus decisiones técnicas y pueda continuar el desarrollo siguiendo las fases definidas en `docs/Fases/`.

---

## 1. Qué es Rentamax

Rentamax es una **plataforma PropTech de inteligencia de inversión inmobiliaria** de uso privado (2 usuarios). Su propósito es identificar inmuebles rentables en el mercado español mediante:

- **Scraping** de portales inmobiliarios (Idealista, Fotocasa, pisos.com)
- **Cálculos financieros avanzados** (ROI neto real, CAPEX, fiscalidad IRPF)
- **Visualización geoespacial** (mapas de calor, capas de datos, scatter plots)

No es un agregador de anuncios. Es un **auditor de inversiones** que cruza datos reales de mercado con carga fiscal y costes de reforma para obtener la **Rentabilidad Neta Real**.

### Usuarios

- 2 personas (los fundadores/inversores)
- Cada uno ejecuta la app en local (`npm run dev`)
- Comparten la misma base de datos en Supabase
- No hay autenticación ni roles
- No hay despliegue público (solo local)

---

## 2. Los 3 Problemas que Resuelve

### 2.1 Opacidad del Dato (Listing vs. Closing Price)

Los portales muestran el **precio de oferta** (lo que pide el vendedor), que está inflado entre un **7% y un 10%** respecto al **precio real de cierre** (escritura notarial). Los inversores basan sus cálculos en precios falsos.

**Solución Rentamax:** Calcular automáticamente el `estimated_closing_price` ajustando el precio de oferta con el porcentaje de negociación típico del mercado.

### 2.2 Incertidumbre en Costes de Reforma (CAPEX Oculto)

Muchos inversores descartan oportunidades rentables porque no saben estimar el coste de reforma ni el potencial de revalorización post-obra.

**Solución Rentamax:** Algoritmo de estimación de reformas paramétrico basado en superficie y partidas (cocina, baños, electricidad, fontanería, etc.) con factores de corrección (imprevistos, licencias, honorarios técnicos) y cálculo de plusvalía post-reforma (24-33%).

### 2.3 Complejidad Regulatoria y Fiscal

La fiscalidad (IVA, ITP, IBI, IRPF) y la Ley de Vivienda (zonas tensionadas, límites al alquiler) impactan drásticamente en la rentabilidad final. Ignorar el impacto fiscal puede reducir la rentabilidad neta en más de 2 puntos porcentuales.

**Solución Rentamax:** Motor de cálculo que integra impuestos de compra por CCAA, gastos operativos, y carga fiscal IRPF con las 4 reducciones vigentes (2025/2026).

---

## 3. Lógica Matemática del Motor de Cálculo

### 3.1 Fórmula Principal: ROI Neto Real

```
ROI_neto = ((Ingresos Anuales - Gastos Operativos - Impuestos) / Inversión Total Inicial) × 100
```

### 3.2 Inversión Total Inicial (Denominador)

```
Inversión Total = Precio Compra + Impuestos Compra + Gastos Cierre + CAPEX (Reforma)
```

**Impuestos de compra (varían según tipo de activo):**

| Tipo | Impuesto | Porcentaje |
|---|---|---|
| Vivienda nueva | IVA + AJD | 10% + (0,5%-1,5% según CCAA) |
| Vivienda usada | ITP | 6%-11% según CCAA |
| VPO | IVA reducido | 4% |

**Gastos de cierre (fricción):** Notaría + Registro + Gestoría. Estimación: 10%-15% extra sobre el precio.

**Tabla de ITP por Comunidad Autónoma:**

| CCAA | ITP General |
|---|---|
| Andalucía | 7% |
| Aragón | 8% |
| Asturias | 8% |
| Baleares | 8-13% |
| Canarias | 6,5% |
| Cantabria | 10% |
| Castilla-La Mancha | 9% |
| Castilla y León | 8% |
| Cataluña | 10% |
| Comunidad Valenciana | 10% |
| Extremadura | 8% |
| Galicia | 9% |
| La Rioja | 7% |
| Madrid | 6% |
| Murcia | 8% |
| Navarra | 6% |
| País Vasco | 4% |
| Ceuta y Melilla | 6% |

### 3.3 Gastos Operativos Anuales (Restar del numerador)

- **Comunidad de propietarios:** Cuotas ordinarias y derramas
- **IBI:** 0,4% - 1,1% del valor catastral
- **Seguros:** Hogar (~200-400€/año) + Impago (~3-5% renta anual)
- **Mantenimiento:** ~1% del valor del inmueble/año
- **Tasas municipales:** Basuras ~100-300€/año
- **Suministros:** Solo si los paga el arrendador (default: 0)
- **Gestión externa:** ~8-10% de la renta si se externaliza (default: 0)

### 3.4 Carga Fiscal IRPF (Restar del numerador)

1. Calcular **Rendimiento Neto** = Ingresos alquiler - Gastos deducibles
   - Deducibles: IBI, comunidad, seguros, reparaciones, amortización (3% valor construcción), intereses hipoteca
2. Aplicar **reducción** según caso:
   - **General:** 50%
   - **Zona tensionada:** 90% (si se baja renta un 5%)
   - **Jóvenes (18-35):** 70%
   - **Rehabilitación:** 60% (obras en últimos 2 años)
3. Aplicar **tipo marginal IRPF** según tramos:

| Base imponible | Tipo |
|---|---|
| 0 - 12.450€ | 19% |
| 12.450 - 20.200€ | 24% |
| 20.200 - 35.200€ | 30% |
| 35.200 - 60.000€ | 37% |
| 60.000 - 300.000€ | 45% |
| > 300.000€ | 47% |

### 3.5 Métricas Resultado

- **Yield Neto (Net Yield):** La métrica principal visible en la UI
- **ROI:** Retorno sobre inversión total
- **ROE:** Retorno sobre capital propio (si hay hipoteca)
- **Cashflow mensual:** Flujo de caja neto mensual
- **Payback:** Años para recuperar la inversión

### 3.6 Algoritmo de Estimación de Reformas (CAPEX)

**Coste base por m²:**

| Tipo | €/m² |
|---|---|
| Cosmética | 150 - 400 |
| Parcial | 400 - 600 |
| Integral | 600 - 1.200 |

**Desglose por partidas:**

| Partida | Rango (€) |
|---|---|
| Cocina | 6.000 - 12.000 |
| Baños | 3.000 - 6.000 / unidad |
| Electricidad | 4.000 - 6.000 |
| Fontanería | 3.500 - 5.000 |
| Demoliciones | 15 - 20 €/m² |
| Albañilería | 60 - 90 €/m² |
| Suelos | 30 - 80 €/m² |
| Pintura | 8 - 15 €/m² |
| Carpintería | 2.000 - 5.000 |

**Factores de corrección obligatorios:**

1. Margen de imprevistos: +10% - 15%
2. Licencias y ICIO: +3% - 7%
3. Honorarios técnicos (si hay arquitecto): +6% - 10%

**Factor temporal:** La reforma implica vacancia (8-16 semanas en integral). El sistema resta del cashflow del primer año los meses sin alquiler + gastos fijos durante obra.

**Plusvalía post-reforma:** Revalorización estimada del 24-33% sobre precio de compra.

### 3.7 Algoritmo de Scoring (0-100)

| Criterio | Peso |
|---|---|
| Rentabilidad neta | 30% |
| Precio vs media de zona | 20% |
| Zona (demanda, absorción) | 15% |
| Cashflow mensual | 15% |
| DOM (días en mercado) | 10% |
| Estado/potencial reforma | 10% |

**Clasificación:**
- 70-100: Riesgo **bajo** — Oportunidad alta
- 40-69: Riesgo **medio** — Analizar con detalle
- 0-39: Riesgo **alto** — Precaución

### 3.8 Hipoteca (Sistema Francés)

```
Cuota = Capital × (i × (1+i)^n) / ((1+i)^n - 1)
```
- `i` = tipo de interés mensual (anual / 12)
- `n` = número de cuotas (años × 12)

---

## 4. Diseño de Interfaz (UX/UI)

### 4.1 Dashboard Híbrido Geoespacial

La interfaz principal es una **visualización dual sincronizada**:

- **Panel izquierdo:** Lista inteligente de propiedades con KPIs
  - Patrón de lectura en "F"
  - Tarjetas minimalistas: precio, yield neto (destacado), etiqueta de estado
  - Ordenadas por scoring del algoritmo
- **Panel derecho:** Mapa de calor interactivo con capas
  - Capa de rentabilidad (verde = alta, rojo = baja)
  - Capa de demanda (zonas calientes con bajo DOM)
  - Capa de servicios (transporte, colegios, supermercados)
  - Capa de zonas tensionadas
  - Zoom semántico: macro (provincias) → medio (barrios) → micro (inmuebles)

**Sincronización:** Hover en tarjeta → pin se ilumina en mapa. Click en pin → scroll a tarjeta.

**Scatter plot:** Gráfico de dispersión (Precio vs Rentabilidad) para detectar outliers visualmente.

### 4.2 Principios UX Críticos

1. **Anti parálisis por análisis:** Solo mostrar Yield Neto en grande. Detalles técnicos en segundo nivel ("Ver desglose").
2. **Transparencia radical:** Etiquetar datos como "Estimación IA" o "Dato Registral". Tooltips que expliquen métricas en lenguaje llano.
3. **Sin dead ends:** Nunca pantalla vacía. Siempre ofrecer alternativas ("En el barrio contiguo hay 3 oportunidades similares").
4. **Feedback instantáneo:** Skeletons de carga al cambiar filtros. Nunca dejar al usuario esperando sin feedback.
5. **Mobile-first:** 80% navegará desde móvil. Mapas gestuales. CTAs en zona del pulgar.
6. **Datos como historia:** En vez de "Zona Tensionada: Sí", decir "Alquiler limitado por ley, pero bonificación fiscal del 90% disponible".

### 4.3 Vista de Detalle de Propiedad

- Galería de imágenes
- KPIs principales (yield, ROI, ROE, cashflow, scoring)
- Desglose completo: inversión, OPEX, fiscalidad
- Estimador de reforma interactivo (activar/desactivar partidas)
- Simulador de hipoteca (sliders)
- Contexto de zona (mini-mapa + market data)

---

## 5. Stack Técnico

| Tecnología | Uso |
|---|---|
| **Next.js 15** | Framework principal (App Router) |
| **TypeScript** | Tipado estricto en todo el proyecto |
| **Tailwind CSS 4** | Estilos utility-first |
| **shadcn/ui** | Componentes UI base |
| **next-themes** | Modo claro/oscuro |
| **React Query** | Fetching, cache y estado de servidor |
| **React Hook Form + Zod** | Formularios y validación |
| **Supabase** | PostgreSQL + Storage + Edge Functions |
| **Mapbox GL / Google Maps** | Mapas interactivos y heatmaps |
| **Recharts** | Gráficos (scatter plot, etc.) |
| **Puppeteer/Playwright** | Scraping de portales |
| **Cheerio** | Parsing HTML |
| **jspdf / react-pdf** | Exportación a PDF |
| **xlsx / exceljs** | Exportación a Excel |
| **Vitest** | Testing unitario |
| **Playwright** | Testing E2E |
| **ESLint + Prettier** | Linting y formateo |
| **Husky + lint-staged** | Pre-commit hooks |
| **commitlint** | Commits convencionales |

---

## 6. Arquitectura del Código

Arquitectura **por Features** con App Router:

```
src/
├── app/                          # Páginas (App Router)
│   ├── layout.tsx
│   ├── page.tsx
│   └── dashboard/
│       ├── layout.tsx
│       ├── page.tsx              # Dashboard principal
│       ├── property/[id]/page.tsx
│       ├── favorites/page.tsx
│       ├── notifications/page.tsx
│       └── settings/page.tsx
│
├── features/                     # Módulos de negocio
│   ├── properties/               # Búsqueda, listado, scraping
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── lib/
│   │   │   └── scrapers/         # Scrapers por portal
│   │   └── types/
│   ├── analysis/                 # Motor de cálculo financiero
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── lib/
│   │   │   ├── roi-calculator.ts
│   │   │   ├── reform-calculator.ts
│   │   │   ├── scoring-engine.ts
│   │   │   └── mortgage-calculator.ts
│   │   └── types/
│   ├── map/                      # Mapa y capas geoespaciales
│   ├── favorites/                # Sistema de favoritos
│   ├── notifications/            # Alertas inteligentes
│   └── export/                   # Exportación PDF/Excel/CSV
│
├── components/                   # Componentes compartidos
│   ├── ui/                       # shadcn/ui
│   ├── layout/                   # Header, Sidebar, MobileNav
│   └── shared/
│
├── lib/                          # Utilidades globales
│   ├── supabase/                 # Clientes Supabase
│   ├── utils.ts
│   └── constants.ts
│
├── hooks/                        # Hooks globales
└── types/                        # Tipos globales
```

---

## 7. Base de Datos (Supabase PostgreSQL)

### Tablas principales

| Tabla | Propósito |
|---|---|
| `properties` | Inmuebles scrapeados de portales |
| `market_data` | Datos agregados por zona (heatmap) |
| `property_analysis` | Análisis financiero calculado por inmueble |
| `reform_estimates` | Desglose de CAPEX por partidas |
| `favorites` | Inmuebles guardados con notas y tags |
| `saved_searches` | Búsquedas guardadas con filtros |
| `notifications` | Alertas (nuevos matches, bajadas precio, retirados) |
| `scraping_logs` | Registro de ejecuciones del scraper |

El esquema completo con todos los campos está documentado en `docs/Fases/Fase-2_Base-de-Datos-y-Modelos.md`.

---

## 8. Flujo de Datos

```
[Portales inmobiliarios] → [Scrapers] → [Normalización] → [Deduplicación]
                                                               ↓
                                                        [Supabase DB]
                                                               ↓
                                              ┌────────────────┼────────────────┐
                                              ↓                ↓                ↓
                                    [Motor de Cálculo]   [Market Data]   [Notificaciones]
                                              ↓                ↓                ↓
                                    [property_analysis]  [Heatmap/Capas]  [Alertas UI]
                                              ↓                ↓
                                              └───────┬────────┘
                                                      ↓
                                              [Dashboard UI]
                                          (Lista + Mapa + Scatter)
                                                      ↓
                                          [Detalle + Simuladores]
                                                      ↓
                                          [Exportar PDF/Excel/CSV]
```

---

## 9. Funcionalidades Clave

1. **Scraping automatizado** de portales inmobiliarios con normalización y deduplicación
2. **Cálculo de Rentabilidad Neta Real** con fiscalidad IRPF y reducciones vigentes
3. **Estimación de reformas** paramétrica con desglose por partidas
4. **Scoring de oportunidad** (0-100) con clasificación de riesgo
5. **Dashboard dual** lista inteligente + mapa de calor interactivo
6. **Filtros avanzados** por presupuesto, rentabilidad, zona, tipo, estado...
7. **Scatter plot** precio vs rentabilidad para detectar outliers
8. **Simulador de hipoteca** con impacto en ROE y cashflow
9. **Favoritos** con notas, tags, prioridad y comparador
10. **Notificaciones inteligentes** (nuevos matches, bajadas de precio, retirados)
11. **Búsquedas guardadas** con alertas automáticas
12. **Exportación** a PDF (informe completo), Excel (múltiples hojas) y CSV
13. **Configuración** de parámetros de cálculo por defecto (CCAA, IRPF, hipoteca)

---

## 10. Decisiones Técnicas Importantes

- **Sin autenticación:** Es app privada, 2 usuarios en local. No hay login.
- **Sin SEO, analytics, legal, i18n:** No es pública. Solo español.
- **Sin pagos:** Es herramienta propia, no se cobra.
- **Sin CI/CD ni hosting:** Se ejecuta en local con `npm run dev`.
- **Supabase como BD compartida:** Ambos usuarios acceden a los mismos datos en la nube.
- **Mobile-first:** La documentación UX indica que el 80% del uso será móvil.
- **Renderizado híbrido:** RSC para datos estáticos + client components para mapas e interacciones.
- **Configuración en localStorage:** Al no haber auth, los ajustes personales se guardan localmente.

---

## 11. Fases de Desarrollo

El proyecto se desarrolla en 7 fases secuenciales. Cada fase tiene sus tareas detalladas con checkboxes y criterios de aceptación en `docs/Fases/`:

| Fase | Archivo | Resumen |
|---|---|---|
| 1 | `Fase-1_Fundacion-y-Setup.md` | Scaffolding, tooling, Supabase, estructura de carpetas |
| 2 | `Fase-2_Base-de-Datos-y-Modelos.md` | Esquema BD completo, tipos TypeScript, seeds |
| 3 | `Fase-3_Scraping-y-Obtencion-de-Datos.md` | Scrapers, normalización, deduplicación, datos complementarios |
| 4 | `Fase-4_Motor-de-Calculo-Financiero.md` | ROI neto, CAPEX, IRPF, scoring, hipoteca |
| 5 | `Fase-5_Dashboard-y-Visualizacion.md` | UI completa: lista, mapa, scatter, filtros, detalle |
| 6 | `Fase-6_Funcionalidades-Avanzadas.md` | Favoritos, notificaciones, exportación, configuración |
| 7 | `Fase-7_Testing-y-Optimizacion.md` | Tests unitarios y E2E, rendimiento, pulido final |

---

## 12. Cómo Continuar el Desarrollo

1. Leer este documento completo para entender el contexto
2. Leer la fase correspondiente en `docs/Fases/`
3. Seguir las tareas en orden (checkboxes)
4. Respetar la arquitectura por features definida
5. Usar commits convencionales: `tipo(scope): descripción`
6. Verificar los criterios de aceptación al final de cada fase
7. No saltarse fases — cada una depende de la anterior
