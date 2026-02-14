# Fase 2: Base de Datos y Modelos de Datos

---

## Objetivo

Diseñar e implementar el esquema completo de base de datos en Supabase (PostgreSQL) que soporte todas las entidades del negocio: inmuebles, análisis financieros, favoritos, notificaciones y datos de mercado. Generar los tipos TypeScript correspondientes y establecer las políticas de acceso.

---

## 2.1 Diseño del Esquema de Base de Datos

### Tabla `properties` (Inmuebles)

Almacena los inmuebles obtenidos por scraping de portales inmobiliarios.

- [ ] Crear tabla con los siguientes campos:

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | `uuid` (PK) | Identificador único |
| `external_id` | `text` | ID del portal de origen (Idealista, Fotocasa...) |
| `source` | `text` | Portal de origen (`idealista`, `fotocasa`, `pisos.com`) |
| `source_url` | `text` | URL original del anuncio |
| `title` | `text` | Título del anuncio |
| `description` | `text` | Descripción completa |
| `price` | `numeric` | Precio de oferta (listing price) |
| `estimated_closing_price` | `numeric` | Precio de cierre estimado (7-10% menos) |
| `price_per_sqm` | `numeric` | Precio por m² |
| `property_type` | `text` | Tipo: `piso`, `casa`, `local`, `garaje`, `trastero` |
| `surface_area` | `numeric` | Superficie útil en m² |
| `built_area` | `numeric` | Superficie construida en m² |
| `rooms` | `integer` | Número de habitaciones |
| `bathrooms` | `integer` | Número de baños |
| `floor` | `text` | Planta (ej. "3ª", "bajo", "ático") |
| `has_elevator` | `boolean` | Tiene ascensor |
| `has_terrace` | `boolean` | Tiene terraza/balcón |
| `has_parking` | `boolean` | Incluye garaje |
| `has_storage` | `boolean` | Incluye trastero |
| `energy_cert` | `text` | Certificación energética (A-G) |
| `year_built` | `integer` | Año de construcción |
| `condition` | `text` | Estado: `nuevo`, `buen_estado`, `a_reformar`, `reforma_integral` |
| `address` | `text` | Dirección completa |
| `neighborhood` | `text` | Barrio |
| `city` | `text` | Ciudad |
| `province` | `text` | Provincia |
| `postal_code` | `text` | Código postal |
| `latitude` | `numeric` | Coordenada latitud |
| `longitude` | `numeric` | Coordenada longitud |
| `images` | `text[]` | Array de URLs de imágenes |
| `is_tensioned_zone` | `boolean` | Zona tensionada (Ley de Vivienda) |
| `days_on_market` | `integer` | Días en el mercado (DOM) |
| `status` | `text` | Estado del listing: `active`, `sold`, `withdrawn`, `expired` |
| `scraped_at` | `timestamptz` | Fecha de extracción |
| `created_at` | `timestamptz` | Fecha de inserción en BD |
| `updated_at` | `timestamptz` | Última actualización |

- [ ] Crear índices:
  - `idx_properties_city` en `city`
  - `idx_properties_province` en `province`
  - `idx_properties_price` en `price`
  - `idx_properties_source_external` en `(source, external_id)` UNIQUE
  - `idx_properties_coordinates` en `(latitude, longitude)`
  - `idx_properties_status` en `status`

### Tabla `market_data` (Datos de Mercado por Zona)

Datos agregados por barrio/zona para el heatmap y análisis macro.

- [ ] Crear tabla con los siguientes campos:

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | `uuid` (PK) | Identificador único |
| `zone_name` | `text` | Nombre del barrio/zona |
| `city` | `text` | Ciudad |
| `province` | `text` | Provincia |
| `postal_code` | `text` | Código postal |
| `avg_price_sqm_sale` | `numeric` | Precio medio €/m² venta |
| `avg_price_sqm_rent` | `numeric` | Precio medio €/m² alquiler |
| `avg_gross_yield` | `numeric` | Rentabilidad bruta media (%) |
| `avg_net_yield` | `numeric` | Rentabilidad neta media (%) |
| `avg_dom` | `numeric` | Media de días en mercado (DOM) |
| `absorption_rate` | `numeric` | Tasa de absorción (%) |
| `supply_count` | `integer` | Número de inmuebles en oferta |
| `demand_index` | `numeric` | Índice de demanda (calculado) |
| `is_tensioned_zone` | `boolean` | Zona tensionada |
| `avg_ibi_rate` | `numeric` | Tipo medio de IBI en la zona (%) |
| `itp_rate` | `numeric` | Tipo de ITP de la CCAA (%) |
| `latitude` | `numeric` | Centro geográfico de la zona |
| `longitude` | `numeric` | Centro geográfico de la zona |
| `geojson` | `jsonb` | Polígono GeoJSON de la zona |
| `period` | `text` | Periodo temporal (ej. "2025-Q1") |
| `updated_at` | `timestamptz` | Última actualización |

- [ ] Crear índices:
  - `idx_market_data_zone_city` en `(zone_name, city)`
  - `idx_market_data_coordinates` en `(latitude, longitude)`

### Tabla `property_analysis` (Análisis Financiero por Inmueble)

Almacena el análisis financiero calculado para cada propiedad.

- [ ] Crear tabla con los siguientes campos:

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | `uuid` (PK) | Identificador único |
| `property_id` | `uuid` (FK → properties) | Inmueble analizado |
| **Inversión Total Inicial** | | |
| `purchase_price` | `numeric` | Precio de compra pactado/estimado |
| `purchase_tax_type` | `text` | Tipo impuesto: `iva_nueva`, `itp_usada`, `iva_vpo` |
| `purchase_tax_rate` | `numeric` | Porcentaje aplicado (%) |
| `purchase_tax_amount` | `numeric` | Importe impuesto de compra (€) |
| `ajd_amount` | `numeric` | Actos Jurídicos Documentados (€) |
| `notary_fees` | `numeric` | Gastos de notaría (€) |
| `registry_fees` | `numeric` | Gastos de registro (€) |
| `agency_fees` | `numeric` | Gastos de gestoría (€) |
| `total_closing_costs` | `numeric` | Total gastos de cierre (€) |
| `capex_total` | `numeric` | Coste total de reforma (€) |
| `total_investment` | `numeric` | Inversión Total Inicial (€) |
| **Ingresos** | | |
| `monthly_rent_estimate` | `numeric` | Alquiler mensual estimado (€) |
| `annual_gross_income` | `numeric` | Ingresos brutos anuales (€) |
| `vacancy_rate` | `numeric` | Tasa de vacancia estimada (%) |
| `annual_effective_income` | `numeric` | Ingresos efectivos tras vacancia (€) |
| **Gastos Operativos Anuales** | | |
| `community_fees` | `numeric` | Comunidad de propietarios (€/año) |
| `ibi_amount` | `numeric` | IBI (€/año) |
| `insurance_amount` | `numeric` | Seguros: hogar + impago (€/año) |
| `maintenance_amount` | `numeric` | Mantenimiento y reparaciones (€/año) |
| `municipal_taxes` | `numeric` | Tasas municipales / basuras (€/año) |
| `management_fees` | `numeric` | Gestión externa si aplica (€/año) |
| `total_opex` | `numeric` | Total gastos operativos (€/año) |
| **Fiscalidad IRPF** | | |
| `taxable_income` | `numeric` | Rendimiento neto antes de reducciones (€) |
| `irpf_reduction_type` | `text` | Tipo reducción: `general_50`, `tensionada_90`, `joven_70`, `rehabilitacion_60` |
| `irpf_reduction_rate` | `numeric` | Porcentaje de reducción aplicado (%) |
| `irpf_taxable_base` | `numeric` | Base imponible tras reducción (€) |
| `estimated_irpf` | `numeric` | IRPF estimado a pagar (€/año) |
| **Métricas Resultado** | | |
| `gross_yield` | `numeric` | Rentabilidad bruta (%) |
| `net_yield` | `numeric` | Rentabilidad neta real (%) |
| `roi` | `numeric` | Return on Investment (%) |
| `roe` | `numeric` | Return on Equity (%) si hay hipoteca |
| `monthly_cashflow` | `numeric` | Cashflow mensual neto (€) |
| `annual_cashflow` | `numeric` | Cashflow anual neto (€) |
| `payback_years` | `numeric` | Años para recuperar inversión |
| **Plusvalía / Revalorización** | | |
| `estimated_value_post_reform` | `numeric` | Valor estimado tras reforma (€) |
| `estimated_appreciation_pct` | `numeric` | Revalorización estimada (%) |
| `flip_profit` | `numeric` | Beneficio potencial de flipping (€) |
| **Scoring** | | |
| `opportunity_score` | `numeric` | Puntuación del algoritmo (0-100) |
| `risk_level` | `text` | Nivel de riesgo: `bajo`, `medio`, `alto` |
| **Meta** | | |
| `calculated_at` | `timestamptz` | Fecha del cálculo |
| `assumptions` | `jsonb` | Parámetros/supuestos usados en el cálculo |
| `created_at` | `timestamptz` | Fecha de creación |
| `updated_at` | `timestamptz` | Última actualización |

- [ ] Crear índices:
  - `idx_analysis_property` en `property_id`
  - `idx_analysis_net_yield` en `net_yield`
  - `idx_analysis_score` en `opportunity_score`

### Tabla `reform_estimates` (Estimación de Reformas / CAPEX)

Desglose detallado del presupuesto de reforma por partidas.

- [ ] Crear tabla con los siguientes campos:

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | `uuid` (PK) | Identificador único |
| `property_id` | `uuid` (FK → properties) | Inmueble asociado |
| `analysis_id` | `uuid` (FK → property_analysis) | Análisis asociado |
| `reform_type` | `text` | Tipo: `integral`, `parcial`, `cosmetica` |
| `surface_sqm` | `numeric` | Superficie a reformar (m²) |
| `cost_per_sqm` | `numeric` | Coste base €/m² aplicado |
| **Partidas** | | |
| `kitchen_cost` | `numeric` | Cocina (€) |
| `bathroom_cost` | `numeric` | Baños (€) |
| `electrical_cost` | `numeric` | Electricidad (€) |
| `plumbing_cost` | `numeric` | Fontanería (€) |
| `demolition_cost` | `numeric` | Demoliciones/desescombro (€) |
| `masonry_cost` | `numeric` | Albañilería/tabiquería (€) |
| `flooring_cost` | `numeric` | Suelos (€) |
| `painting_cost` | `numeric` | Pintura (€) |
| `carpentry_cost` | `numeric` | Carpintería (€) |
| `other_costs` | `numeric` | Otros costes (€) |
| `subtotal` | `numeric` | Subtotal partidas (€) |
| **Factores de Corrección** | | |
| `contingency_rate` | `numeric` | Margen de imprevistos (10-15%) |
| `contingency_amount` | `numeric` | Imprevistos (€) |
| `license_rate` | `numeric` | Licencias/ICIO (3-7%) |
| `license_amount` | `numeric` | Licencias (€) |
| `architect_rate` | `numeric` | Honorarios técnicos (6-10%) |
| `architect_amount` | `numeric` | Honorarios (€) |
| `total_cost` | `numeric` | Coste total reforma (€) |
| **Factor Temporal** | | |
| `estimated_weeks` | `integer` | Duración estimada (semanas) |
| `vacancy_cost` | `numeric` | Coste de oportunidad por vacancia (€) |
| **Meta** | | |
| `created_at` | `timestamptz` | Fecha de creación |
| `updated_at` | `timestamptz` | Última actualización |

### Tabla `favorites` (Favoritos / Guardados)

- [ ] Crear tabla con los siguientes campos:

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | `uuid` (PK) | Identificador único |
| `property_id` | `uuid` (FK → properties) | Inmueble guardado |
| `notes` | `text` | Notas personales del usuario |
| `tags` | `text[]` | Etiquetas (ej. "alta rentabilidad", "para reformar") |
| `priority` | `text` | Prioridad: `alta`, `media`, `baja` |
| `created_at` | `timestamptz` | Fecha de guardado |

- [ ] Crear índice en `property_id`

### Tabla `saved_searches` (Búsquedas Guardadas)

- [ ] Crear tabla con los siguientes campos:

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | `uuid` (PK) | Identificador único |
| `name` | `text` | Nombre de la búsqueda |
| `filters` | `jsonb` | Filtros aplicados (serializado) |
| `notify_on_new` | `boolean` | Notificar cuando haya nuevos resultados |
| `last_result_count` | `integer` | Resultados en la última ejecución |
| `created_at` | `timestamptz` | Fecha de creación |
| `updated_at` | `timestamptz` | Última actualización |

### Tabla `notifications` (Notificaciones)

- [ ] Crear tabla con los siguientes campos:

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | `uuid` (PK) | Identificador único |
| `type` | `text` | Tipo: `new_property`, `price_change`, `search_match`, `system` |
| `title` | `text` | Título de la notificación |
| `message` | `text` | Mensaje/cuerpo |
| `property_id` | `uuid` (FK → properties, nullable) | Inmueble relacionado |
| `search_id` | `uuid` (FK → saved_searches, nullable) | Búsqueda relacionada |
| `is_read` | `boolean` | Leída/no leída |
| `metadata` | `jsonb` | Datos adicionales |
| `created_at` | `timestamptz` | Fecha de creación |

- [ ] Crear índices en `is_read` y `created_at`

### Tabla `scraping_logs` (Registro de Ejecuciones de Scraping)

- [ ] Crear tabla con los siguientes campos:

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | `uuid` (PK) | Identificador único |
| `source` | `text` | Portal scrapeado |
| `status` | `text` | Estado: `running`, `completed`, `failed` |
| `properties_found` | `integer` | Inmuebles encontrados |
| `properties_new` | `integer` | Inmuebles nuevos insertados |
| `properties_updated` | `integer` | Inmuebles actualizados |
| `errors` | `jsonb` | Errores encontrados |
| `started_at` | `timestamptz` | Inicio de ejecución |
| `finished_at` | `timestamptz` | Fin de ejecución |
| `duration_ms` | `integer` | Duración en milisegundos |

---

## 2.2 Migrations y SQL

- [ ] Crear archivo SQL de migración inicial con todas las tablas
- [ ] Ejecutar migración en Supabase (SQL Editor o CLI)
- [ ] Verificar que todas las tablas se crearon correctamente
- [ ] Verificar que los índices están activos
- [ ] Insertar datos de prueba (seed) para desarrollo:
  - 10-20 propiedades de ejemplo
  - 2-3 zonas de market_data
  - 1-2 análisis financieros de ejemplo

---

## 2.3 Tipos TypeScript

- [ ] Generar tipos automáticos desde Supabase:
  - Usar `npx supabase gen types typescript` o configurar generación automática
- [ ] Crear archivo `src/lib/supabase/types.ts` con los tipos generados
- [ ] Crear tipos de dominio adicionales en cada feature:
  - `src/features/properties/types/index.ts`
  - `src/features/analysis/types/index.ts`
  - `src/features/map/types/index.ts`
  - `src/features/favorites/types/index.ts`
  - `src/features/notifications/types/index.ts`
- [ ] Crear tipos para los enums:
  ```typescript
  type PropertyType = 'piso' | 'casa' | 'local' | 'garaje' | 'trastero'
  type PropertyCondition = 'nuevo' | 'buen_estado' | 'a_reformar' | 'reforma_integral'
  type PropertyStatus = 'active' | 'sold' | 'withdrawn' | 'expired'
  type ReformType = 'integral' | 'parcial' | 'cosmetica'
  type IRPFReductionType = 'general_50' | 'tensionada_90' | 'joven_70' | 'rehabilitacion_60'
  type RiskLevel = 'bajo' | 'medio' | 'alto'
  type NotificationType = 'new_property' | 'price_change' | 'search_match' | 'system'
  ```

---

## 2.4 Helpers y Queries Base

- [ ] Crear funciones helper para queries frecuentes en `src/lib/supabase/`:
  - `getProperties(filters)` — Listado con filtros
  - `getPropertyById(id)` — Detalle de un inmueble
  - `getMarketDataByZone(zone)` — Datos de mercado por zona
  - `getAnalysisByPropertyId(id)` — Análisis financiero de un inmueble
  - `getFavorites()` — Listado de favoritos
  - `getNotifications()` — Listado de notificaciones
- [ ] Verificar que las queries funcionan con los datos seed

---

## 2.5 Configuración de Storage (Supabase Storage)

- [ ] Crear bucket `property-images` para almacenar imágenes de inmuebles (si se descargan del scraping)
- [ ] Crear bucket `exports` para almacenar PDFs/Excel generados
- [ ] Configurar políticas de acceso a los buckets

---

## Criterios de Aceptación (Definition of Done)

- [ ] Todas las tablas creadas en Supabase con sus índices
- [ ] Tipos TypeScript generados y sin errores de compilación
- [ ] Datos seed insertados y verificados
- [ ] Queries base funcionando correctamente
- [ ] Buckets de Storage creados
- [ ] El proyecto compila sin errores con los nuevos tipos
