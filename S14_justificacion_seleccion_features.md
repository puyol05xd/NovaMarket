# Justificación de la selección de features — Semana 14

Dataset de entrada: `S13_PD_Camacho_Serpa_novamarket_con_features.csv`
Variable objetivo: `fuga_cliente`
Dataset exportado: `novamarket_features_seleccionadas.csv`

Se aplicaron los tres métodos de selección (filtro por correlación, wrapper RFE con regresión logística, embedded con Random Forest) usando únicamente el conjunto de entrenamiento, y se cruzaron sus resultados para decidir el subconjunto final.

---

## Features seleccionadas

| Feature | Votos | Justificación |
|---|---|---|
| `nivel_lealtad_cod` | **3/3** | Seleccionada por los tres métodos. Es la feature con mayor correlación con `fuga_cliente` (0.22) y la segunda en importancia del Random Forest. Representa el nivel del cliente en el programa de lealtad (Bronce, Plata, Oro, Platino), señal directa de qué tan comprometido está con la marca. |
| `monto_compra` | **2/3** | Seleccionada por filtro y RFE. El valor del pedido es señal fundamental de comportamiento de compra. Clientes con montos típicos muy distintos del promedio suelen tener patrones de fuga distintos. |
| `precio_unitario` | **2/3** | Seleccionada por filtro y RFE. Precio efectivo pagado por el producto, útil para distinguir clientes que compran premium vs. clientes que compran productos de bajo valor. |
| `gasto_promedio` | **2/3** | Seleccionada por filtro y Random Forest. Representa el ticket promedio por cliente (la M del RFM). Clientes de alto ticket promedio tienden a tener menos fuga. |
| `edad_cliente` | **2/3** | Seleccionada por filtro y RFE. Variable demográfica básica. La edad del cliente influye en sus preferencias de compra y en el riesgo de abandono del programa. |
| `canal_App` | **2/3** | Seleccionada por filtro y RFE. Indica si el pedido vino por la App. El canal de compra es un predictor clásico de fuga: clientes multicanal suelen ser más leales que los que solo usan uno. |
| `es_venta_corporativa` | **2/3** | Seleccionada por filtro y RFE. Marca pedidos con monto originalmente muy alto. Las ventas corporativas tienen un patrón de fuga diferente al retail individual. |
| `dias_desde_ultima_compra` | **1/3** | Seleccionada solo por Random Forest, pero se incluye con justificación de negocio clara: es la R (Recencia) del modelo RFM, señal clave para predecir fuga de clientes. Ninguna otra feature de la selección cubre la recencia, así que descartarla dejaría un hueco grave en el modelo. |

**Total: 8 features**, dentro del rango 6–10 pedido por el enunciado.

**Verificación de redundancia:** se calculó la matriz de correlación sobre las 8 features finales y ninguna pareja supera 0.85 en valor absoluto, así que no hay redundancia dentro de la selección.

---

## Features descartadas

| Feature | Motivo de descarte |
|---|---|
| `monto_compra_boxcox` | **Redundante con `monto_compra`.** Aunque tuvo 2 votos (filtro y RFE), es simplemente la transformación Box-Cox de `monto_compra`, por lo que su correlación entre ambas es casi 1. Se conserva la original por ser más interpretable para negocio. |
| `menciona_demora` | **Varianza casi nula.** El Random Forest le asignó importancia exactamente 0.0, lo que confirma que casi no varía entre registros (prácticamente ningún comentario contiene la palabra "demora" literalmente). No aporta ninguna señal útil al modelo. |
| `z_monto_compra` | **Redundante con `monto_compra`.** Es una estandarización lineal de la columna original, por lo que su correlación es prácticamente 1 con `monto_compra`. Solo apareció en Random Forest con importancia media y no aporta señal nueva. |
| `canal_Web` | **Baja correlación con el objetivo y bajo ranking.** Solo apareció en RFE (1 voto). Tiene correlación muy baja con `fuga_cliente` (0.017) y una importancia modesta en Random Forest. Como ya tenemos `canal_App` para el efecto de canal digital, no aporta información adicional importante. |
| `peso_pedido_kg`, `codigo_postal`, `dias_desde_pedido`, `total_gastado` | **Solo apareció en Random Forest** y con importancias moderadas. Aunque son candidatas razonables, no tienen justificación de negocio tan clara como `dias_desde_ultima_compra` para la predicción de fuga, y agregarlas haría crecer el subconjunto más allá de lo que se necesita. |
| `score_M` | **Solo apareció en el filtro** y con correlación baja (0.028). La información de valor monetario ya está cubierta por `monto_compra` y `gasto_promedio`, que son más directas. |

---

## Conclusión

El subconjunto final de **8 features** combina las que tuvieron consenso entre los tres métodos de selección (`nivel_lealtad_cod`), las que tuvieron 2 votos sin redundancia (`monto_compra`, `precio_unitario`, `gasto_promedio`, `edad_cliente`, `canal_App`, `es_venta_corporativa`), y una feature de 1 voto con justificación de negocio clara (`dias_desde_ultima_compra` como señal de recencia).

Esta selección cubre las tres dimensiones principales que afectan la fuga de un cliente: su **comportamiento de compra** (monto, precio, gasto promedio), su **recencia** (días desde la última compra), su **relación con la marca** (nivel de lealtad), su **canal preferido** (App) y su **perfil** (edad, si es corporativa).
