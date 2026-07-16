# Análisis de Rotación y Vejez por Región — Fundamento de Targets

**Forest Coffee · Modelo de Compensación Spot v2 · Julio 2026**

Este documento explica cómo se derivaron los targets de rotación y vejez por región, y el piso mínimo (X) desde el cual la vejez afecta el bono. Es la documentación de respaldo de los valores cableados en el simulador.

---

## 1. Fuentes de datos y metodología

| Fuente | Contenido | Periodo |
|---|---|---|
| Rotación por regiones (3,425 transacciones) | Ventas con Rotation Days, kg, región, categoría | Jun 2025 – Jul 2026 |
| Inventario actual por región (191 lotes) | Foto del inventario con días de vejez, kg, precio, status | Snapshot ~Q2 2026 |

Decisiones metodológicas:

- **Todo ponderado por kg** (kg = cantidad × tamaño de saco). Un lote de 1,540 kg pesa lo que debe pesar; el promedio simple sobre-representa microlotes pequeños.
- **Rotación** = ingreso a bodega (ETA) → venta. Se validó que la columna Rotation Days coincide con `Fecha venta − ETA` en el 96 % de los casos (±7 días, mediana de diferencia: 1 día). El dato es confiable.
- **Vejez** = solo inventario **On Spot** (disponible). Los 51 lotes On Float (en tránsito, días negativos) se excluyen: no son vendibles aún.
- **Mapeo de regiones**: UK → EU, Dubai → MENA, para alinear con las 4 regiones del modelo.
- **Percentiles ponderados** por kg (no por número de lotes).

---

## 2. Hallazgos: rotación histórica

### 2.1 Año completo vs. últimos 90 días (kg-ponderado)

| Región | Promedio año | Últimos 90d | p50 | p75 | p90 | % kg vendido con >150d |
|---|---|---|---|---|---|---|
| USA | 84.9d | **110.9d** | 63 | 120 | 192 | 17.0 % |
| EU | 82.4d | 87.4d | 52 | 112 | 206 | 16.9 % |
| AU | 76.7d | 79.5d | 45 | 104 | 177 | 16.4 % |
| MENA | 68.0d | 66.7d | 18 | 95 | 152 | 14.3 % |

### 2.2 Tendencia (rotación móvil 90d, sep-25 → jun-26)

| Región | Trayectoria | Lectura |
|---|---|---|
| USA | 53 → 73 → 85 → 96 → 95 → 97 → 100 → 112 → **127** → 117 | **Deterioro sostenido.** Único caso estructural. |
| EU | 78 → 71 → 58 → 54 → 57 → 92 → 96 → 101 → 92 → 90 | Empeoró en Q1-26, estabilizada ~90d. |
| AU | 56 → 89 → 117 → **143** → 126 → 94 → 65 → 53 → 72 → 75 | Pico y recuperación completa. Hoy sana. |
| MENA | 25 → 64 → 76 → 69 → 40 → 71 → 89 → 119 → 68 → 56 | Volátil por bajo volumen. Promedio sano. |

### 2.3 Rotación por categoría

COMMUNITY rota entre 56–89d según región y representa 53–85 % del kg vendido. MICROLOT, INNOVATION y RESERVE rotan mucho más lento (hasta 177–534d). **La cola lenta del portafolio está concentrada en las categorías premium, no en el volumen.**

---

## 3. Hallazgos: vejez del inventario disponible

Snapshot On Spot: 140 lotes, 41,846 kg, ~$628k.

| Región | Vejez wavg | Núcleo sano (≤150d) | % kg >150d | $ >150d | Cobertura (meses de venta) |
|---|---|---|---|---|---|
| AU | 144.0d | 70.1d | 32 % | $50.4k | 0.6 |
| USA | 117.0d | 71.7d | 34 % | $63.0k | 0.3 |
| MENA | 108.4d | 41.1d | 39 % | $61.3k | 1.2 |
| EU | 90.3d | 52.8d | 20 % | $50.2k | 0.6 |

**Insight central: el problema es la cola, no el flujo.** Excluyendo los lotes >150d, todas las regiones tienen vejez de 41–72d. Los ~$225k atascados en lotes >150d son casi todos MICROLOT / INNOVATION / RESERVE (COMMUNITY en inventario: 35d; RESERVE: 288d).

Casos extremos: AU tiene 21.5 % del kg con >270 días (Caturra Syrup 355d, Java Koji 436d). USA tiene 19 lotes >150d que, por la regla de +2d/lote, suman **+38 días** a su rotación de cierre hoy.

---

## 4. Targets propuestos y su justificación

**Regla de diseño:** el target de rotación se ancla entre el p50 histórico y el promedio 90d actual — exigente pero ganable. Un target inalcanzable mata el incentivo (bono nace muerto); uno regalado no cambia comportamiento.

| Región | Rotación hoy (90d) | Target 3M | Target 6M | Vejez hoy | Target vejez | Justificación |
|---|---|---|---|---|---|---|
| USA | 110.9d | **95d** | **85d** | 117d | **85d** | Glide path de −15d/trimestre. USA estuvo en 53–73d hace 9 meses: la capacidad existe. Target directo a 85 hoy = castigo garantizado, target 95 = recuperación creíble. Liquidar los 19 lotes >150d baja 38d de un golpe. |
| EU | 87.4d | **75d** | 75d | 90.3d | **70d** | Promedio anual 82d y núcleo sano en 53d. La brecha completa se cierra limpiando 13 lotes >150d. |
| AU | 79.5d | **70d** | 70d | 144.0d | **90d** | Rotación ya recuperada (~75d); 70 consolida. La vejez de 144d está inflada por la cola >270d: el target de 90 obliga a liquidarla sin ser imposible (núcleo sano = 70d). |
| MENA | 66.7d | **60d** | 60d | 108.4d | **75d** | Mejor rotación del grupo. Target mantiene disciplina; vejez 75 exige limpiar 6 lotes viejos ($61k, el mayor $ en riesgo relativo). |

---

## 5. Piso X: valor mínimo desde el que la vejez afecta

**Pregunta:** ¿desde qué nivel de inventario tiene sentido penalizar por vejez? Si la bodega está casi vacía, la vejez ponderada es ruido estadístico de 2–3 lotes y no refleja gestión.

**Respuesta: X = k % del movimiento mensual de la región, con k = 33 %.** (≈ inventario menor a 1/3 de mes de ventas ⇒ inmunidad.)

| Región | Mov. mensual | X @ 33 % | Inventario spot hoy | ¿Aplica vejez hoy? |
|---|---|---|---|---|
| USA | 43,213 kg | 14,260 kg | 15,056 kg | Sí (justo en el borde) |
| EU | 24,474 kg | 8,076 kg | 15,102 kg | Sí |
| MENA | 5,712 kg | 1,885 kg | 7,057 kg | Sí |
| AU | 8,345 kg | 2,754 kg | 4,631 kg | Sí |

**Por qué NO el contenedor fijo (19,200 kg):** ese piso equivale a 230 % del movimiento mensual de AU y 336 % del de MENA — en esas regiones la vejez *nunca* aplicaría (inventarios actuales: 4.6t y 7.1t). En USA equivale al 44 % — aplicaría siempre. Un piso fijo crea inmunidad estructural en regiones chicas y castigo estructural en las grandes. El k % escala con el tamaño real de cada operación.

**Calibración de k:** con k=33 % las 4 regiones están hoy por encima del piso (la vejez aplica en todas, correcto dado que todas tienen cola >150d). k=50 % dejaría a USA (15.0t vs 21.6t) inmune hoy pese a tener $63k atascados — demasiado laxo. **k=33 % es el valor recomendado.**

---

## 6. Plan de mejora por región

1. **USA — prioridad 1.** Deterioro 53→127d con solo 0.3 meses de cobertura: no es sobre-stock, son lotes atascados. Acción Q3-2026: liquidación dirigida de los 19 lotes >150d (~$63k). Efecto inmediato: −38d en rotación de cierre. Luego sostener con el glide path 95→85.
2. **AU — cola crónica.** Liquidar/reasignar el 21.5 % del kg con >270d (descuento agresivo o traslado a región con demanda). Sin esa cola, AU ya cumple el target de vejez.
3. **EU — mantener.** Núcleo más sano (53d en el 80 % del inventario). Solo limpiar 13 lotes >150d.
4. **MENA — estabilizar mix.** INNOVATION es 46 % del inventario y es la categoría que más se atasca. Límite de asignación por categoría atado a rotación histórica.
5. **Regla estructural de compras:** MICROLOT / INNOVATION / RESERVE generan casi toda la cola >150d en las 4 regiones. Tope de asignación por región en estas categorías en función de su rotación demostrada de los últimos 90d.

**Revisión:** targets se recalibran trimestralmente con la rotación móvil 90d (misma metodología de este análisis).

---

## 7. Caveats de datos

- Columna `Dias` del inventario inconsistente con ETA en algunos lotes (snapshot implícito varía entre oct-25 y jun-26; mediana abr-2026). Recomendado: estandarizar la fecha de corte del reporte de inventario.
- 2 transacciones sin kg excluidas del análisis de rotación.
- Rotation Days validada contra ETA: 96 % de coincidencia ±7d — dato confiable.
- Movimientos mensuales (base del Piso X) calculados sobre el año completo jun-25→jul-26; recalcular cada trimestre.
