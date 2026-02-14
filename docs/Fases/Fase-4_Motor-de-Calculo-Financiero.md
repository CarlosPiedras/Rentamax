# Fase 4: Motor de Cálculo Financiero

---

## Objetivo

Implementar el motor de cálculo que convierte datos crudos de inmuebles en métricas de inversión accionables: ROI neto real, estimación de CAPEX, carga fiscal IRPF con reducciones, scoring de oportunidad y análisis de flipping. Este motor es el corazón de la propuesta de valor de Rentamax, basado en la documentación de "Lógica Matemática y Parámetros Técnicos".

---

## 4.1 Fórmula Principal: Rentabilidad Neta Real (ROI Neto)

### Implementación de la Fórmula

```
ROI_neto = ((Ingresos Anuales - Gastos Operativos - Impuestos) / Inversión Total Inicial) × 100
```

- [ ] Crear módulo `src/features/analysis/lib/roi-calculator.ts`
- [ ] Implementar función principal:
  ```typescript
  function calculateNetROI(params: ROICalculatorParams): ROIResult
  ```

### A. Cálculo de la Inversión Total Inicial (Denominador)

- [ ] Implementar `calculateTotalInvestment()`:
  - **Precio de Adquisición:** Valor pactado de compraventa
  - **Impuestos de Compra (variable según tipo de activo):**
    - Vivienda Nueva: IVA (10%) + AJD (0,5% - 1,5% según CCAA)
    - Vivienda Usada: ITP (6% - 11% según CCAA)
    - VPO: IVA superreducido (4%)
  - **Gastos de Cierre:** Notaría + Registro + Gestoría
    - Estimación por defecto: 10-15% extra sobre precio
    - Permitir personalización de cada concepto
  - **CAPEX / Reforma:** Coste total de obra (calculado en módulo de reformas)
  - **Fórmula:** `PrecioCompra + ImpuestosCompra + GastosCierre + CAPEX`

- [ ] Crear tabla de referencia estática de ITP por CCAA:

| CCAA | ITP General | ITP Reducido (vivienda habitual) |
|---|---|---|
| Andalucía | 7% | 6% |
| Aragón | 8% | - |
| Asturias | 8% | - |
| Baleares | 8-13% | - |
| Canarias | 6,5% | - |
| Cantabria | 10% | 8% |
| Castilla-La Mancha | 9% | - |
| Castilla y León | 8% | - |
| Cataluña | 10% | - |
| Comunidad Valenciana | 10% | - |
| Extremadura | 8% | 7% |
| Galicia | 9% | 8% |
| La Rioja | 7% | - |
| Madrid | 6% | - |
| Murcia | 8% | - |
| Navarra | 6% | - |
| País Vasco | 4% | - |
| Ceuta y Melilla | 6% | - |

- [ ] Crear tabla de referencia de AJD por CCAA (0,5% - 1,5%)

### B. Cálculo de Gastos Operativos Anuales (Numerador - Restar)

- [ ] Implementar `calculateAnnualOpex()`:
  - **Comunidad de propietarios:** Input manual o estimación por zona
  - **IBI:** Valor catastral × tipo impositivo (0,4% - 1,1%)
  - **Seguros:**
    - Seguro de hogar: estimación ~200-400€/año según valor
    - Seguro de impago: ~3-5% de la renta anual
  - **Mantenimiento:** Estimación ~1% del valor del inmueble/año
  - **Tasas municipales:** Basuras ~100-300€/año según municipio
  - **Suministros:** Solo si los paga el arrendador (input manual, default 0)
  - **Gestión externa:** Si externaliza, ~8-10% de la renta (input manual, default 0)

### C. Cálculo de la Carga Fiscal IRPF (Numerador - Restar)

- [ ] Implementar `calculateIRPF()`:
  1. Calcular **Rendimiento Neto** = Ingresos alquiler - Gastos deducibles
     - Gastos deducibles: IBI, comunidad, seguros, reparaciones, amortización (3% valor construcción), intereses hipoteca
  2. Aplicar **reducciones vigentes 2025/2026:**
     - Reducción General: **50%** del rendimiento neto
     - Reducción Zona Tensionada: **90%** (si se baja la renta un 5% respecto al anterior contrato)
     - Reducción Jóvenes (18-35 años): **70%**
     - Reducción por Rehabilitación: **60%** (si hubo obras en los 2 años anteriores)
  3. Calcular base imponible tras reducción
  4. Aplicar tipo marginal de IRPF según tramos:

| Base imponible | Tipo marginal |
|---|---|
| 0 - 12.450€ | 19% |
| 12.450 - 20.200€ | 24% |
| 20.200 - 35.200€ | 30% |
| 35.200 - 60.000€ | 37% |
| 60.000 - 300.000€ | 45% |
| > 300.000€ | 47% |

  - *Nota: El tipo marginal depende de la base imponible total del contribuyente. Permitir que el usuario introduzca su tramo o una estimación.*

- [ ] Implementar selector de reducción IRPF aplicable (el usuario elige cuál aplica a su caso)

### D. Cálculo del ROI, ROE y Cashflow

- [ ] Implementar `calculateROI()`:
  ```
  ROI = ((Ingresos Efectivos - OPEX - IRPF) / Inversión Total) × 100
  ```

- [ ] Implementar `calculateROE()` (Return on Equity):
  ```
  ROE = ((Ingresos Efectivos - OPEX - IRPF - Cuota Hipoteca Anual) / Capital Propio Aportado) × 100
  ```
  - Capital Propio = Inversión Total - Importe Hipoteca
  - Incluir parámetros de hipoteca:
    - Importe financiado
    - Tipo de interés (fijo/variable)
    - Plazo en años
    - Cuota mensual resultante

- [ ] Implementar `calculateCashflow()`:
  ```
  Cashflow Mensual = (Ingresos Efectivos - OPEX - IRPF - Cuota Hipoteca) / 12
  ```

- [ ] Implementar `calculatePayback()`:
  ```
  Payback (años) = Inversión Total / Cashflow Anual Neto
  ```

---

## 4.2 Algoritmo de Estimación de Reformas (CAPEX)

### Implementación del Estimador

- [ ] Crear módulo `src/features/analysis/lib/reform-calculator.ts`

### A. Coste Base por Metro Cuadrado

- [ ] Implementar estimación por nivel de reforma:

| Tipo de Reforma | Coste €/m² | Descripción |
|---|---|---|
| Cosmética | 150€ - 400€ | Pintura, suelos, detalles |
| Parcial | 400€ - 600€ | Cocina o baño + cosmética |
| Integral Estándar | 600€ - 1.200€ | Todo: instalaciones, distribución, acabados |

### B. Desglose por Partidas

- [ ] Implementar cálculo por partidas con rangos configurables:

| Partida | Rango (€) | Variable |
|---|---|---|
| Cocina | 6.000 - 12.000 | Por unidad |
| Baños | 3.000 - 6.000 | Por unidad |
| Electricidad | 4.000 - 6.000 | Total vivienda |
| Fontanería | 3.500 - 5.000 | Total vivienda |
| Demoliciones | 15 - 20 €/m² | Por superficie |
| Albañilería/Tabiquería | 60 - 90 €/m² | Por superficie |
| Suelos | 30 - 80 €/m² | Por superficie |
| Pintura | 8 - 15 €/m² | Por superficie |
| Carpintería | 2.000 - 5.000 | Total vivienda |

- [ ] Permitir activar/desactivar partidas según necesidad
- [ ] Permitir ajustar los valores por defecto manualmente

### C. Factores de Corrección

- [ ] Implementar `applyCorrections()`:
  1. **Margen de Imprevistos:** +10% - 15% sobre subtotal (configurable)
  2. **Licencias y Tasas (ICIO):** +3% - 7% sobre presupuesto de ejecución material
  3. **Honorarios Técnicos:** +6% - 10% si se requiere arquitecto (cambio distribución/estructural)

### D. Factor Temporal (Coste de Oportunidad)

- [ ] Implementar `calculateOpportunityCost()`:
  - Duración estimada: 8-16 semanas para integral, 4-8 semanas para parcial
  - Meses sin alquiler = duración obra
  - Coste de oportunidad = (Alquiler mensual estimado × meses de obra) + (Gastos fijos mensuales × meses de obra)
  - Gastos fijos durante obra: comunidad + IBI mensualizado

### E. Cálculo de Plusvalía Post-Reforma

- [ ] Implementar `calculatePostReformValue()`:
  - Revalorización estimada: 24-33% según documentación
  - Valor post-reforma = Precio compra + (Precio compra × % revalorización)
  - Beneficio flipping = Valor post-reforma - Inversión Total (compra + reforma + impuestos)
  - ROI del flip = Beneficio / Inversión Total × 100

---

## 4.3 Algoritmo de Scoring (Puntuación de Oportunidad)

- [ ] Crear módulo `src/features/analysis/lib/scoring-engine.ts`

### Criterios de Scoring (0-100 puntos)

- [ ] Implementar sistema de puntuación ponderada:

| Criterio | Peso | Descripción |
|---|---|---|
| Rentabilidad neta | 30% | >7% = máximo, <3% = mínimo |
| Precio vs mercado | 20% | Descuento sobre media de zona |
| DOM | 10% | Muchos días = posible negociación |
| Estado/Reforma | 10% | Potencial de revalorización |
| Zona | 15% | Demanda, absorción, tendencia |
| Cashflow mensual | 15% | Flujo de caja positivo |

- [ ] Clasificar en niveles de riesgo:
  - Score 70-100: Riesgo **bajo** — Oportunidad alta
  - Score 40-69: Riesgo **medio** — Analizar con detalle
  - Score 0-39: Riesgo **alto** — Precaución

- [ ] Identificar outliers positivos (oportunidades excepcionales):
  - Rentabilidad neta > media de zona + 2 desviaciones estándar
  - Precio < media de zona - 15%

---

## 4.4 Simulador de Hipoteca

- [ ] Crear módulo `src/features/analysis/lib/mortgage-calculator.ts`
- [ ] Implementar cálculo de cuota mensual (sistema francés):
  ```
  Cuota = Capital × (i × (1+i)^n) / ((1+i)^n - 1)
  ```
  - `i` = tipo de interés mensual
  - `n` = número de cuotas (meses)
- [ ] Implementar tabla de amortización
- [ ] Parámetros configurables:
  - Porcentaje de financiación (ej. 80% del precio)
  - Tipo de interés fijo
  - Plazo en años (15, 20, 25, 30)
- [ ] Calcular impacto en ROE y cashflow

---

## 4.5 Persistencia de Análisis

- [ ] Guardar cada análisis calculado en `property_analysis`
- [ ] Guardar desglose de reforma en `reform_estimates`
- [ ] Permitir recalcular con diferentes parámetros (crear nuevo análisis)
- [ ] Almacenar los supuestos/parámetros usados en el campo `assumptions` (JSON)

---

## 4.6 Validación y Testing

- [ ] Crear tests unitarios para cada calculadora:
  - `roi-calculator.test.ts` — Casos con datos reales
  - `reform-calculator.test.ts` — Verificar rangos de costes
  - `scoring-engine.test.ts` — Verificar ponderaciones
  - `mortgage-calculator.test.ts` — Verificar cuotas con calculadoras externas
- [ ] Casos de prueba con datos reales de la documentación:
  - Piso 90m² a reformar: verificar que CAPEX está en rango 60.000-100.000€
  - Verificar que reducciones IRPF se aplican correctamente
  - Verificar que la revalorización post-reforma está en rango 24-33%
- [ ] Verificar edge cases:
  - Precio 0, superficie 0
  - Sin reforma (CAPEX = 0)
  - Sin hipoteca (ROE = ROI)
  - Zona tensionada con reducción 90%

---

## Criterios de Aceptación (Definition of Done)

- [ ] `calculateNetROI()` devuelve resultados correctos con datos de prueba
- [ ] Tabla de ITP por CCAA completa y correcta
- [ ] Reducciones IRPF 2025/2026 implementadas correctamente
- [ ] Estimador de reformas funcional con desglose por partidas
- [ ] Scoring asigna puntuaciones coherentes
- [ ] Simulador de hipoteca calcula cuotas verificadas
- [ ] Tests unitarios pasando (>90% cobertura en esta feature)
- [ ] Análisis se persisten correctamente en Supabase
- [ ] Sin errores de TypeScript
