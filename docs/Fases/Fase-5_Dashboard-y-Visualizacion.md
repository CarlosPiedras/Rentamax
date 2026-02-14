# Fase 5: Dashboard y Visualización

---

## Objetivo

Construir la interfaz principal de Rentamax: el "Dashboard Híbrido Geoespacial" descrito en la documentación de UX/UI. Incluye el panel de lista inteligente con KPIs, el mapa de calor interactivo con capas, el gráfico de dispersión, las tarjetas de propiedad y la vista de detalle con análisis financiero completo.

---

## 5.1 Layout Principal del Dashboard

### Estructura Base

- [ ] Implementar layout del dashboard (`src/app/dashboard/layout.tsx`):
  - **Header:** Logo Rentamax + barra de búsqueda rápida + botón de notificaciones + toggle de tema
  - **Sidebar:** Navegación principal (colapsable en móvil)
    - Dashboard (vista principal)
    - Búsqueda avanzada
    - Favoritos
    - Alertas/Notificaciones
    - Configuración (parámetros de cálculo)
  - **Área principal:** Contenido dinámico según la ruta

- [ ] Implementar componentes de layout compartidos:
  - `src/components/layout/Header.tsx`
  - `src/components/layout/Sidebar.tsx`
  - `src/components/layout/MobileNav.tsx` (navegación bottom-bar para móvil)

### Responsive Design (Mobile-First)

- [ ] Diseño mobile:
  - Sidebar se convierte en bottom navigation bar
  - El mapa y la lista se muestran en tabs (no side-by-side)
  - CTAs ocupan ancho completo en zona inferior (zona del pulgar)
  - Botones mínimo 44x44px (fat finger friendly)
- [ ] Diseño tablet (md): Split view vertical (lista arriba, mapa abajo)
- [ ] Diseño desktop (lg+): Split view horizontal (lista izquierda, mapa derecha)

---

## 5.2 Panel Izquierdo: Lista Inteligente y KPIs

### KPIs Resumen (Header del Panel)

- [ ] Crear componente `src/features/properties/components/KPIDashboard.tsx`:
  - **Inmuebles totales** en BD
  - **Nuevos hoy** (últimas 24h)
  - **Rentabilidad media** de la zona seleccionada
  - **Oportunidades destacadas** (score > 70)
- [ ] Usar componentes `Card` de shadcn/ui con iconos y tendencias
- [ ] Implementar skeleton loading durante carga de datos

### Lista de Propiedades

- [ ] Crear componente `src/features/properties/components/PropertyList.tsx`:
  - Listado scrollable de tarjetas de propiedad
  - Ordenación por: scoring, precio, rentabilidad neta, DOM, fecha
  - Patrón de lectura en "F" (info crítica a la izquierda)
  - Lazy loading / scroll infinito para grandes listados

### Tarjeta de Propiedad (Property Card)

- [ ] Crear componente `src/features/properties/components/PropertyCard.tsx`:
  - **Información mínima visible** (evitar parálisis por análisis):
    - Imagen principal (thumbnail)
    - Precio de oferta + precio estimado de cierre
    - Yield Neto (en grande, dato principal)
    - Superficie (m²) + habitaciones + baños
    - Etiqueta de estado: `Badge` con color
      - Verde: "Oportunidad Alta" (score > 70)
      - Amarillo: "Analizar" (score 40-69)
      - Rojo: "Riesgo Alto" (score < 40)
    - Etiqueta de condición: "Listo para alquilar" / "Reforma necesaria"
    - Barrio + Ciudad
    - DOM (días en mercado)
  - **Microinteracción:** Al hacer hover, el pin correspondiente en el mapa se ilumina/resalta
  - **Acción rápida:** Botón de favorito (corazón) en esquina
  - **Click:** Navega a vista de detalle

### Etiquetas de Origen de Datos

- [ ] Implementar sistema de etiquetas según documentación UX:
  - Si el precio es estimado por IA → Badge "Estimación IA"
  - Si el precio es de cierre real → Badge "Dato Registral"
  - Tooltips explicativos al pasar el ratón sobre métricas complejas
  - Lenguaje llano (nivel lectura 8º grado) en todas las explicaciones

---

## 5.3 Panel Derecho: Mapa de Calor Interactivo

### Setup del Mapa

- [ ] Instalar librería de mapas:
  - **Opción A (Recomendada):** `react-map-gl` + Mapbox GL JS
  - **Opción B:** `@vis.gl/react-google-maps` + Google Maps
- [ ] Crear componente `src/features/map/components/PropertyMap.tsx`
- [ ] Configurar API key del servicio de mapas en `.env.local`
- [ ] Implementar mapa base centrado en España

### Capas de Datos (Layers)

- [ ] Implementar selector de capas con toggle on/off:
  - `src/features/map/components/LayerSelector.tsx`

- [ ] **Capa de Rentabilidad:**
  - Heatmap de zonas: verde (alta rentabilidad) → rojo (baja rentabilidad)
  - Datos de `market_data.avg_net_yield`
  - Polígonos GeoJSON por barrio/zona

- [ ] **Capa de Demanda:**
  - Zonas "calientes" con bajo DOM (< 45 días)
  - Datos de `market_data.avg_dom` y `market_data.absorption_rate`
  - Gradiente de color: caliente (alta demanda) → frío (baja demanda)

- [ ] **Capa de Servicios:**
  - Iconos simples para:
    - Transporte público (metro, bus)
    - Colegios/universidades
    - Supermercados
    - Hospitales/centros de salud
  - Se activa al hacer zoom a nivel de barrio

- [ ] **Capa de Zonas Tensionadas:**
  - Polígonos destacados para zonas con limitación de alquiler
  - Info en tooltip: "Alquiler limitado por ley, bonificación fiscal del 90% disponible"

### Pins de Propiedades

- [ ] Implementar markers/pins de propiedades en el mapa:
  - Color según scoring (verde/amarillo/rojo)
  - Cluster de pins cuando hay muchos cercanos (desagrupar al zoom)
  - Popup al click con resumen rápido (precio, yield, superficie)
  - Conexión con la lista: al seleccionar un pin, scroll automático a su tarjeta

### Zoom Semántico

- [ ] Implementar transición de información según nivel de zoom:
  - **Zoom lejano (país/región):** Heatmap de rentabilidad por provincias
  - **Zoom medio (ciudad):** Heatmap por barrios + indicadores de zona
  - **Zoom cercano (barrio):** Pins individuales de propiedades + capa de servicios

### Interacción Lista ↔ Mapa (Sincronización Dual)

- [ ] Implementar sincronización bidireccional:
  - Hover en tarjeta → resalta pin en mapa
  - Click en pin → scroll a tarjeta correspondiente + highlight
  - Mover el mapa → actualizar lista con propiedades visibles en el viewport
  - Filtrar lista → actualizar pins visibles en el mapa

---

## 5.4 Gráfico de Dispersión (Scatter Plot)

- [ ] Instalar librería de gráficos: `recharts` o `@nivo/scatterplot`
- [ ] Crear componente `src/features/properties/components/ScatterPlot.tsx`:
  - **Eje X:** Precio
  - **Eje Y:** Rentabilidad neta (%)
  - **Color del punto:** Scoring (verde/amarillo/rojo)
  - **Tamaño del punto:** Superficie (m²)
  - **Tooltip al hover:** Resumen de la propiedad
  - **Click:** Navega a vista de detalle
- [ ] Destacar outliers (oportunidades que se salen de la norma)
- [ ] Ubicar debajo de los filtros o como vista alternativa a la lista
- [ ] Responsive: en móvil, mostrar como pestaña alternativa

---

## 5.5 Sistema de Filtros

- [ ] Crear componente `src/features/properties/components/PropertyFilters.tsx`

### Filtros Principales (Visibles por defecto)

- [ ] **Presupuesto:** Range slider min-max (€)
- [ ] **Rentabilidad esperada:** Range slider min-max (%)
- [ ] **Ciudad / Zona:** Selector con búsqueda
- [ ] **Tipo de inmueble:** Chips seleccionables (piso, casa, local...)
- [ ] **Estado:** Chips (listo, reformar, cualquiera)

### Filtros Avanzados (Panel expandible "Más filtros")

- [ ] Habitaciones: min-max
- [ ] Superficie: min-max (m²)
- [ ] Año de construcción: min-max
- [ ] Planta: selector
- [ ] Con/sin ascensor, terraza, garaje, trastero
- [ ] Certificación energética
- [ ] Zona tensionada: sí/no/indiferente
- [ ] DOM: min-max (días)
- [ ] Scoring mínimo: slider
- [ ] Portal de origen: checkboxes

### Funcionalidades de Filtros

- [ ] Filtros aplicados como chips removibles (mostrar filtros activos)
- [ ] Botón "Limpiar filtros"
- [ ] Los filtros actualizan lista + mapa + scatter plot en tiempo real
- [ ] Guardar búsqueda (conectar con `saved_searches`)
- [ ] Estado de filtros en la URL (query params) para compartir/bookmarking

---

## 5.6 Vista de Detalle de Propiedad

- [ ] Crear página `src/app/dashboard/property/[id]/page.tsx`
- [ ] Crear componentes de detalle:

### Sección Superior: Galería + Info Básica

- [ ] `PropertyGallery.tsx` — Carrusel de imágenes con lightbox
- [ ] `PropertyHeader.tsx` — Título, precio, ubicación, badges
- [ ] Botones de acción: Favorito, Ver en portal original, Exportar análisis

### Sección Financiera: Análisis Completo

- [ ] `FinancialSummary.tsx` — KPIs principales en cards:
  - Yield Neto (grande, destacado)
  - ROI
  - ROE (si hay hipoteca)
  - Cashflow mensual
  - Payback (años)
  - Scoring

- [ ] `InvestmentBreakdown.tsx` — Desglose de inversión total:
  - Precio de compra
  - Impuestos (ITP/IVA + AJD)
  - Gastos de cierre
  - CAPEX (reforma)
  - **Total inversión**
  - Cada línea con tooltip explicativo

- [ ] `OpexBreakdown.tsx` — Desglose de gastos operativos anuales:
  - Comunidad, IBI, seguros, mantenimiento, tasas, gestión
  - **Total OPEX anual**

- [ ] `TaxBreakdown.tsx` — Desglose fiscal:
  - Rendimiento neto
  - Reducción aplicada (con explicación)
  - Base imponible
  - IRPF estimado
  - Tooltip: "Alquiler limitado por ley, pero bonificación fiscal del 90% disponible" (si zona tensionada)

### Sección de Reforma (CAPEX)

- [ ] `ReformEstimate.tsx` — Estimador de reforma:
  - Selector de tipo de reforma (cosmética, parcial, integral)
  - Desglose por partidas (activables/desactivables)
  - Factores de corrección
  - Coste de oportunidad temporal
  - **Total CAPEX**
  - Plusvalía estimada post-reforma

### Sección de Simulador

- [ ] `MortgageSimulator.tsx` — Simulador de hipoteca interactivo:
  - Sliders: importe, tipo de interés, plazo
  - Cuota mensual resultante
  - Impacto en ROE y cashflow
  - Tabla de amortización (colapsable)

### Sección de Contexto de Zona

- [ ] `ZoneContext.tsx` — Información de la zona:
  - Mini-mapa con ubicación
  - Datos de `market_data` de la zona
  - Precio medio zona vs precio del inmueble
  - Indicadores: DOM medio, tasa de absorción, demanda
  - Zona tensionada: sí/no con explicación

---

## 5.7 Estados de UI

### Loading States

- [ ] Implementar skeletons de carga para:
  - Lista de propiedades (tarjetas skeleton)
  - Mapa (placeholder con spinner)
  - KPIs (cards skeleton)
  - Detalle de propiedad (skeleton completo)
- [ ] Feedback visual inmediato al cambiar filtros

### Empty States

- [ ] Implementar "estado vacío" cuando no hay resultados:
  - Nunca pantalla en blanco
  - Mensaje: "No hay inmuebles con estos filtros en esta zona"
  - Sugerencias: "En el barrio contiguo hay X oportunidades similares"
  - Botón: "Ampliar búsqueda" o "Limpiar filtros"

### Error States

- [ ] Implementar manejo de errores con mensajes claros:
  - Error de conexión a Supabase
  - Error al cargar el mapa
  - Error en el cálculo financiero
  - Botón de reintentar en cada caso

---

## 5.8 Toasts y Feedback

- [ ] Configurar sistema de toast/notificaciones (Sonner):
  - Éxito: "Inmueble guardado en favoritos"
  - Info: "Datos actualizados"
  - Error: "Error al guardar"
  - Loading: "Calculando rentabilidad..." (con spinner)

---

## Criterios de Aceptación (Definition of Done)

- [ ] Dashboard renderiza con layout dual (lista + mapa) en desktop
- [ ] Diseño responsive funcional en móvil, tablet y desktop
- [ ] Lista de propiedades muestra datos reales de Supabase
- [ ] Tarjetas muestran Yield Neto como dato principal
- [ ] Mapa renderiza con al menos 1 capa de datos (rentabilidad)
- [ ] Sincronización lista ↔ mapa funcional
- [ ] Scatter plot renderiza y es interactivo
- [ ] Filtros actualizan lista + mapa + scatter en tiempo real
- [ ] Vista de detalle muestra análisis financiero completo
- [ ] Estimador de reforma interactivo
- [ ] Simulador de hipoteca funcional
- [ ] Skeletons de carga implementados
- [ ] Empty states con sugerencias alternativas
- [ ] Tema claro/oscuro funciona en toda la UI
- [ ] Sin errores de TypeScript
