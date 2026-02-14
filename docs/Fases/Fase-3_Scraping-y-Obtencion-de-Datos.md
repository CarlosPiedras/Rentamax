# Fase 3: Scraping y Obtención de Datos

---

## Objetivo

Implementar el sistema de extracción de datos de portales inmobiliarios (Idealista, Fotocasa, pisos.com), normalización de los datos obtenidos, almacenamiento en Supabase y detección de cambios (nuevos inmuebles, bajadas de precio, inmuebles retirados). Este es el motor de alimentación de datos de toda la plataforma.

---

## 3.1 Estrategia de Scraping

### Investigación y Decisiones Técnicas

- [ ] Investigar las APIs públicas/semi-públicas disponibles:
  - API de Idealista (requiere solicitud de acceso)
  - Datos abiertos de catastro (Sede Electrónica del Catastro)
  - API del INE para datos demográficos
  - Datos del Ministerio de Vivienda (Índice de referencia)
- [ ] Evaluar herramientas de scraping para Next.js/Node.js:
  - **Puppeteer** / **Playwright** para scraping dinámico (páginas con JS)
  - **Cheerio** para parsing de HTML estático
  - **Axios** / **Fetch** para peticiones HTTP
- [ ] Definir la estrategia anti-bloqueo:
  - Rotación de User-Agents
  - Delays aleatorios entre peticiones (2-5 segundos)
  - Respetar `robots.txt` de cada portal
  - Limitar la frecuencia de scraping (1 ejecución/día máximo)
- [ ] Decidir dónde se ejecuta el scraping:
  - **Opción A:** Supabase Edge Functions (cron jobs)
  - **Opción B:** API Routes de Next.js (ejecutado manualmente o con cron local)
  - **Opción C:** Script Node.js independiente que se ejecuta con cron del sistema

### Arquitectura del Scraper

- [ ] Diseñar arquitectura modular de scrapers:

```
src/features/properties/lib/scrapers/
├── base-scraper.ts          # Clase/interfaz base para todos los scrapers
├── idealista-scraper.ts     # Scraper específico de Idealista
├── fotocasa-scraper.ts      # Scraper específico de Fotocasa
├── pisos-scraper.ts         # Scraper específico de pisos.com
├── normalizer.ts            # Normalización de datos al formato unificado
├── deduplicator.ts          # Detección de duplicados entre portales
└── index.ts                 # Orquestador principal
```

---

## 3.2 Implementación de Scrapers

### Scraper Base (Interfaz Común)

- [ ] Crear interfaz `BaseScraper` con métodos:
  ```typescript
  interface BaseScraper {
    source: string
    scrape(params: ScrapeParams): Promise<RawProperty[]>
    parse(html: string): RawProperty[]
    normalize(raw: RawProperty): NormalizedProperty
  }
  ```
- [ ] Definir tipo `ScrapeParams`:
  - `city`: Ciudad objetivo
  - `propertyType`: Tipo de inmueble
  - `priceRange`: Rango de precios
  - `minRooms`: Mínimo de habitaciones
  - `maxPages`: Máximo de páginas a scrapear
- [ ] Definir tipo `RawProperty` (datos crudos del portal)
- [ ] Definir tipo `NormalizedProperty` (formato unificado para BD)

### Scraper de Idealista

- [ ] Implementar `idealista-scraper.ts`:
  - Construcción de URLs de búsqueda con filtros
  - Extracción de listado de resultados (precio, ubicación, superficie, habitaciones)
  - Extracción de detalle de cada inmueble (descripción, imágenes, características)
  - Extracción de coordenadas (lat/lng) del mapa
  - Detección del estado del inmueble (nuevo, bajada de precio, etc.)
  - Parsing de la etiqueta energética
- [ ] Implementar paginación (recorrer todas las páginas de resultados)
- [ ] Manejar errores y reintentos (máximo 3 reintentos por página)

### Scraper de Fotocasa

- [ ] Implementar `fotocasa-scraper.ts` con la misma interfaz
- [ ] Adaptar el parsing a la estructura HTML de Fotocasa
- [ ] Mapear campos al formato normalizado

### Scraper de pisos.com

- [ ] Implementar `pisos-scraper.ts` con la misma interfaz
- [ ] Adaptar el parsing a la estructura HTML de pisos.com
- [ ] Mapear campos al formato normalizado

---

## 3.3 Normalización y Deduplicación

### Normalización de Datos

- [ ] Implementar `normalizer.ts`:
  - Limpiar y estandarizar precios (eliminar "€", puntos, comas)
  - Estandarizar superficies (m² útiles vs construidos)
  - Normalizar direcciones (formato consistente)
  - Estandarizar tipos de inmueble al enum de la BD
  - Mapear estados/condiciones al enum de la BD
  - Calcular `price_per_sqm` automáticamente
  - Calcular `estimated_closing_price` (precio de oferta - 7-10% según mercado)
  - Detectar y etiquetar valores atípicos (outliers) en precios

### Deduplicación

- [ ] Implementar `deduplicator.ts`:
  - Detección por `(source, external_id)` — mismo portal, mismo ID
  - Detección cross-portal por similitud:
    - Misma dirección + superficie similar (±5%) + precio similar (±10%)
    - Mismas coordenadas (radio de 50m) + misma superficie
  - Marcar duplicados y mantener el registro más completo
  - Actualizar datos si el inmueble ya existe (precio, estado, DOM)

---

## 3.4 Datos Complementarios

### Datos de Catastro

- [ ] Investigar acceso a la Sede Electrónica del Catastro
- [ ] Extraer datos catastrales por referencia catastral o dirección:
  - Valor catastral del suelo y construcción
  - Año de construcción real
  - Superficie registrada
  - Uso del inmueble
- [ ] Almacenar datos catastrales como campos complementarios en `properties`

### Datos de Zonas Tensionadas

- [ ] Obtener listado oficial de zonas tensionadas por CCAA
- [ ] Mapear códigos postales / zonas a flag `is_tensioned_zone`
- [ ] Mantener actualizado con cada cambio normativo

### Datos de Mercado por Zona

- [ ] Calcular y alimentar tabla `market_data`:
  - Precio medio €/m² de venta por zona (de los datos scrapeados)
  - Precio medio €/m² de alquiler por zona
  - Rentabilidad bruta media por zona
  - DOM medio por zona
  - Tasa de absorción
  - Índice de demanda
- [ ] Calcular tipos de ITP por CCAA (tabla de referencia estática)
- [ ] Calcular tipos medios de IBI por municipio

---

## 3.5 Orquestador y Persistencia

### Orquestador de Scraping

- [ ] Implementar orquestador principal que:
  1. Ejecuta cada scraper configurado
  2. Normaliza los resultados
  3. Deduplica entre portales
  4. Inserta/actualiza en Supabase
  5. Detecta cambios (nuevos, actualizados, retirados)
  6. Genera notificaciones si hay matches con búsquedas guardadas
  7. Registra la ejecución en `scraping_logs`

### Persistencia en Supabase

- [ ] Implementar lógica de upsert (insertar o actualizar):
  - Si `(source, external_id)` no existe → INSERT
  - Si existe y hay cambios (precio, estado) → UPDATE + registrar cambio
  - Si existía y ya no aparece → marcar como `withdrawn` o `expired`
- [ ] Calcular `days_on_market` en cada actualización
- [ ] Actualizar `market_data` tras cada ejecución de scraping

### Ejecución Programada

- [ ] Configurar ejecución automática:
  - **Opción local:** Cron del sistema o script manual
  - **Opción Supabase:** Edge Function con pg_cron
- [ ] Implementar endpoint/API route para ejecutar scraping manualmente desde el dashboard
- [ ] Añadir botón "Actualizar datos" en la UI (para ejecución manual)

---

## 3.6 Logging y Monitoreo

- [ ] Registrar cada ejecución en `scraping_logs`:
  - Portal, duración, inmuebles encontrados/nuevos/actualizados
  - Errores encontrados (con detalle)
- [ ] Mostrar estado del último scraping en el dashboard:
  - Última actualización por portal
  - Número de inmuebles en BD
  - Errores recientes

---

## 3.7 Manejo de Errores y Resiliencia

- [ ] Implementar retry con backoff exponencial para peticiones fallidas
- [ ] Manejar cambios en la estructura HTML de los portales (alertar si el parser falla)
- [ ] Timeout configurable por petición (30 segundos máximo)
- [ ] No detener la ejecución completa si un portal falla (continuar con los demás)
- [ ] Rate limiting propio (máximo X peticiones por minuto por portal)

---

## Criterios de Aceptación (Definition of Done)

- [ ] Al menos 1 scraper funcional (Idealista) extrayendo datos reales
- [ ] Datos normalizados e insertados correctamente en Supabase
- [ ] Deduplicación funcionando (no hay inmuebles duplicados)
- [ ] `estimated_closing_price` calculado automáticamente
- [ ] `days_on_market` calculado correctamente
- [ ] `market_data` alimentada con datos agregados
- [ ] `scraping_logs` registrando cada ejecución
- [ ] Ejecución manual desde el dashboard funcional
- [ ] Sin errores de TypeScript en toda la feature
