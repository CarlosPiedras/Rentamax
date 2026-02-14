# Fase 6: Funcionalidades Avanzadas

---

## Objetivo

Implementar las funcionalidades complementarias que completan la experiencia del usuario: sistema de favoritos con organización, notificaciones y alertas inteligentes, exportación de datos en múltiples formatos y página de configuración de parámetros de cálculo.

---

## 6.1 Sistema de Favoritos / Guardados

### Lista de Favoritos

- [ ] Crear página `src/app/dashboard/favorites/page.tsx`
- [ ] Crear componente `src/features/favorites/components/FavoritesList.tsx`:
  - Listado de inmuebles guardados con misma tarjeta que el dashboard
  - Ordenar por: fecha guardado, rentabilidad, precio, prioridad
  - Filtrar por: tags, prioridad

### Gestión de Favoritos

- [ ] Crear componente `src/features/favorites/components/FavoriteActions.tsx`:
  - **Guardar/Quitar favorito:** Toggle desde cualquier tarjeta de propiedad
  - **Notas personales:** Textarea para añadir notas a cada favorito
  - **Tags/Etiquetas:** Chips añadibles (ej. "alta rentabilidad", "para reformar", "negociar precio", "zona buena")
  - **Prioridad:** Selector alta/media/baja con indicador visual (color)

### Hooks y Lógica

- [ ] Crear hook `src/features/favorites/hooks/useFavorites.ts`:
  - `addFavorite(propertyId, data)` — Guardar favorito
  - `removeFavorite(propertyId)` — Quitar favorito
  - `updateFavorite(id, data)` — Actualizar notas/tags/prioridad
  - `isFavorite(propertyId)` — Comprobar si ya es favorito
  - `getFavorites(filters)` — Obtener lista con filtros
- [ ] Cache de favoritos con React Query para UI instantánea
- [ ] Optimistic updates (marcar como favorito antes de confirmar con BD)

### Comparador de Favoritos

- [ ] Crear componente `src/features/favorites/components/CompareView.tsx`:
  - Seleccionar 2-4 favoritos para comparar lado a lado
  - Tabla comparativa con métricas clave:
    - Precio, yield neto, ROI, ROE, cashflow, scoring
    - CAPEX estimado, plusvalía
    - Zona, superficie, habitaciones
  - Destacar visualmente el mejor valor en cada métrica (verde)
  - Destacar el peor valor en cada métrica (rojo)

---

## 6.2 Sistema de Notificaciones y Alertas

### Centro de Notificaciones

- [ ] Crear página `src/app/dashboard/notifications/page.tsx`
- [ ] Crear componente `src/features/notifications/components/NotificationCenter.tsx`:
  - Listado de notificaciones ordenadas por fecha
  - Marcar como leída/no leída
  - Filtrar por tipo: nuevos inmuebles, bajadas de precio, matches de búsqueda
  - Marcar todas como leídas
  - Borrar notificaciones antiguas

### Indicador de Notificaciones

- [ ] Crear componente `src/features/notifications/components/NotificationBadge.tsx`:
  - Badge numérico en el icono de campana del header
  - Dropdown rápido con las últimas 5 notificaciones
  - Link "Ver todas" al centro de notificaciones

### Tipos de Alertas

- [ ] **Nuevos inmuebles que coinciden con búsquedas guardadas:**
  - Cuando el scraping detecta nuevos inmuebles que cumplen los filtros de una `saved_search` con `notify_on_new = true`
  - Notificación: "3 nuevos inmuebles en [zona] con rentabilidad >7%"

- [ ] **Bajadas de precio:**
  - Cuando un inmueble (especialmente si está en favoritos) baja de precio
  - Notificación: "[Dirección] ha bajado de X€ a Y€ (-Z%)"

- [ ] **Inmuebles retirados:**
  - Cuando un favorito desaparece del portal (posible venta)
  - Notificación: "[Dirección] ya no está disponible en [portal]"

- [ ] **Oportunidades destacadas:**
  - Inmuebles con scoring > 80 recién detectados
  - Notificación: "Nueva oportunidad destacada: Yield Neto X% en [zona]"

### Generación de Notificaciones

- [ ] Implementar lógica de generación post-scraping:
  - Tras cada ejecución del scraper, comparar resultados con:
    - Búsquedas guardadas (`saved_searches`)
    - Favoritos (para bajadas de precio y retiradas)
    - Umbrales de scoring
  - Insertar notificaciones en tabla `notifications`

### Hooks y Estado

- [ ] Crear hook `src/features/notifications/hooks/useNotifications.ts`:
  - `getNotifications(filters)` — Obtener notificaciones
  - `getUnreadCount()` — Contar no leídas
  - `markAsRead(id)` — Marcar como leída
  - `markAllAsRead()` — Marcar todas como leídas
  - `deleteNotification(id)` — Borrar notificación
- [ ] Polling o suscripción Supabase Realtime para notificaciones en tiempo real

---

## 6.3 Búsquedas Guardadas

### Gestión de Búsquedas

- [ ] Crear componente `src/features/properties/components/SavedSearches.tsx`:
  - Listado de búsquedas guardadas
  - Nombre, filtros aplicados (como chips), última ejecución
  - Toggle de notificaciones (activar/desactivar alertas)
  - Ejecutar búsqueda (aplicar filtros guardados)
  - Editar / Eliminar búsqueda

### Guardar Búsqueda

- [ ] Al pulsar "Guardar búsqueda" en el panel de filtros:
  - Dialog para nombrar la búsqueda
  - Checkbox "Notificarme cuando haya nuevos resultados"
  - Guardar en tabla `saved_searches` con los filtros serializados

---

## 6.4 Exportación de Datos

### Formatos de Exportación

- [ ] Crear módulo `src/features/export/lib/`:

#### Exportación a PDF

- [ ] Instalar librería: `@react-pdf/renderer` o `jspdf`
- [ ] Crear plantilla de informe PDF:
  - **Portada:** Logo + nombre de la propiedad + fecha
  - **Resumen ejecutivo:** KPIs principales (yield, ROI, cashflow, scoring)
  - **Desglose de inversión:** Tabla con todos los costes
  - **Análisis de gastos:** OPEX + fiscalidad
  - **Estimación de reforma:** Si aplica, desglose de CAPEX
  - **Simulación hipotecaria:** Si se configuró
  - **Contexto de zona:** Datos de mercado
  - **Disclaimer:** "Datos estimados, consultar con profesional"
- [ ] Generar PDF individual por propiedad
- [ ] Generar PDF comparativo de varios inmuebles

#### Exportación a Excel (XLSX)

- [ ] Instalar librería: `xlsx` o `exceljs`
- [ ] Crear exportación Excel:
  - **Hoja 1 "Resumen":** Tabla con métricas clave de todos los inmuebles filtrados
  - **Hoja 2 "Detalle Financiero":** Desglose completo de inversión, OPEX y fiscal por inmueble
  - **Hoja 3 "Reformas":** Estimaciones de CAPEX por inmueble
  - **Hoja 4 "Datos de Zona":** Market data de las zonas incluidas
  - Formato: cabeceras con color, columnas con formato numérico/porcentaje
- [ ] Exportar listado completo con filtros aplicados
- [ ] Exportar detalle de un inmueble específico

#### Exportación a CSV

- [ ] Implementar exportación CSV simple:
  - Listado de inmuebles con métricas principales
  - Separador configurable (coma o punto y coma)
  - Compatible con Excel/Google Sheets

### UI de Exportación

- [ ] Crear componente `src/features/export/components/ExportButton.tsx`:
  - Botón con dropdown de formatos (PDF, Excel, CSV)
  - Disponible en:
    - Dashboard (exportar listado filtrado)
    - Vista de detalle (exportar análisis individual)
    - Favoritos (exportar favoritos)
    - Comparador (exportar comparativa)
- [ ] Mostrar toast de progreso: "Generando PDF..." → "PDF listo"
- [ ] Almacenar exports generados en Supabase Storage (bucket `exports`)

---

## 6.5 Página de Configuración

- [ ] Crear página `src/app/dashboard/settings/page.tsx`

### Parámetros de Cálculo por Defecto

- [ ] Crear componente `src/features/analysis/components/CalculationSettings.tsx`:
  - **CCAA por defecto:** Selector (para ITP y AJD)
  - **Tipo de reducción IRPF:** Selector (general, tensionada, joven, rehabilitación)
  - **Tramo IRPF del contribuyente:** Selector o input
  - **Tasa de vacancia por defecto:** Input (%) — default 5%
  - **Margen de imprevistos reforma:** Slider (10-15%)
  - **% Negociación precio:** Input (%) — default 7-10%
  - **Gastos operativos por defecto:**
    - Seguro hogar (€/año)
    - Seguro impago (% renta)
    - Mantenimiento (% valor inmueble)
    - Tasas municipales (€/año)

### Parámetros de Hipoteca por Defecto

- [ ] **% Financiación:** Slider (60-100%) — default 80%
- [ ] **Tipo de interés:** Input (%) — default tipo actual mercado
- [ ] **Plazo:** Selector (15, 20, 25, 30 años) — default 25

### Preferencias de Interfaz

- [ ] **Tema:** Selector claro/oscuro/sistema
- [ ] **Moneda:** € (fijo, solo español)
- [ ] **Formato numérico:** Separadores (1.234,56 formato español)

### Persistencia de Configuración

- [ ] Guardar configuración en localStorage (no necesita BD al no haber auth)
- [ ] Crear hook `useSettings()` para acceder a la configuración globalmente
- [ ] Los valores de configuración se usan como defaults en todos los cálculos

---

## 6.6 Página de Estado del Scraping

- [ ] Crear sección en dashboard o settings para:
  - Ver última ejecución por portal (fecha, duración, resultados)
  - Botón "Ejecutar scraping ahora" (manual)
  - Historial de ejecuciones recientes (de `scraping_logs`)
  - Errores recientes
  - Estadísticas: total inmuebles, por portal, por ciudad

---

## Criterios de Aceptación (Definition of Done)

- [ ] Favoritos: guardar, quitar, notas, tags, prioridad funcionan
- [ ] Comparador: muestra tabla comparativa de 2-4 inmuebles
- [ ] Notificaciones: se generan tras scraping para búsquedas guardadas
- [ ] Badge de notificaciones muestra conteo correcto
- [ ] Búsquedas guardadas: crear, ejecutar, eliminar, toggle alertas
- [ ] Exportar a PDF genera informe completo y correcto
- [ ] Exportar a Excel genera libro con múltiples hojas formateadas
- [ ] Exportar a CSV genera archivo válido
- [ ] Configuración persiste en localStorage y se aplica en cálculos
- [ ] Estado del scraping visible y botón manual funcional
- [ ] Sin errores de TypeScript
