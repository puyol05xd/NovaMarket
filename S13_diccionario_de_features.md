# Diccionario de features — Semana 13

Dataset: `S13_PD_Camacho_Serpa_novamarket_con_features.csv`
Fecha de corte del análisis: **2027-12-29** (un día después del pedido más tardío del dataset, 2027-12-28).

---

## Familia 1 — Variables derivadas

### `dias_desde_pedido`
- **Fórmula:** `(fecha_corte - fecha_pedido).dt.days`
- **Tipo:** entero (días).
- **Justificación de negocio:** convierte una fecha en un número comparable. Permite ordenar y filtrar pedidos por qué tan recientes son, sin tener que leer fechas.

### `gasto_por_unidad`
- **Fórmula:** `monto_compra / unidades_vendidas` (dejando nulo cuando `unidades_vendidas = 0`).
- **Tipo:** decimal (pesos por unidad).
- **Justificación de negocio:** mide el precio efectivo que pagó el cliente por cada unidad. Es útil para detectar pedidos con descuento (precio bajo por unidad) o compras de productos premium (precio alto por unidad), señales que el `monto_compra` solo no muestra.

### `antiguedad_cliente_dias`
- **Fórmula:** `(fecha_corte - fecha_inscripcion_club).dt.days`
- **Tipo:** entero (días). Queda nulo para clientes que no están en el club.
- **Justificación de negocio:** indica hace cuánto el cliente se inscribió al programa de fidelidad. Los clientes antiguos del club suelen tener comportamientos de compra distintos a los recién inscritos.

### `dominio_correo`
- **Fórmula:** parte del `correo_cliente` después del símbolo `@`.
- **Tipo:** texto (gmail.com, hotmail.com, novamarket.co, etc.).
- **Justificación de negocio:** el dominio del correo puede revelar patrones (clientes corporativos con dominio propio, clientes jóvenes en gmail, clientes del propio NovaMarket). Es información que estaba oculta dentro de la columna de correo.

---

## Familia 2 — Agregaciones por cliente

### `numero_compras`
- **Fórmula:** `df.groupby("id_cliente")["id_pedido"].count()`
- **Tipo:** entero.
- **Justificación de negocio:** cuántas veces ha comprado cada cliente. Es la base de la **F** (Frecuencia) del RFM y es señal directa de fidelidad.

### `total_gastado`
- **Fórmula:** `df.groupby("id_cliente")["monto_compra"].sum()`
- **Tipo:** decimal (pesos).
- **Justificación de negocio:** cuánto dinero ha dejado cada cliente en la empresa en total. Es la base de la **M** (Valor monetario) del RFM.

### `gasto_promedio`
- **Fórmula:** `df.groupby("id_cliente")["monto_compra"].mean()`
- **Tipo:** decimal (pesos).
- **Justificación de negocio:** el ticket promedio del cliente. Permite distinguir entre clientes de muchas compras pequeñas y clientes de pocas compras grandes.

### `dias_desde_ultima_compra`
- **Fórmula:** `df.groupby("id_cliente")["dias_desde_pedido"].min()`
- **Tipo:** entero (días).
- **Justificación de negocio:** cuánto tiempo lleva el cliente sin comprar. Es la base de la **R** (Recencia) del RFM y la señal más fuerte para detectar riesgo de fuga.

---

## Familia 3 — Features de dominio (RFM)

### `score_R`
- **Fórmula:** `pd.qcut(dias_desde_ultima_compra, q=4, labels=[4, 3, 2, 1])`
- **Tipo:** entero (1 a 4).
- **Justificación de negocio:** puntaje de Recencia. Un cliente que compró hace poco recibe 4; uno inactivo recibe 1.

### `score_F`
- **Fórmula:** `pd.qcut(numero_compras.rank(method="first"), q=4, labels=[1, 2, 3, 4])`
- **Tipo:** entero (1 a 4).
- **Justificación de negocio:** puntaje de Frecuencia. Más compras = puntaje más alto.

### `score_M`
- **Fórmula:** `pd.qcut(total_gastado, q=4, labels=[1, 2, 3, 4])`
- **Tipo:** entero (1 a 4).
- **Justificación de negocio:** puntaje de Valor monetario. Más gasto total = puntaje más alto.

### `rfm_score`
- **Fórmula:** `score_R + score_F + score_M`
- **Tipo:** entero (3 a 12).
- **Justificación de negocio:** un solo número que resume el valor del cliente para la empresa. 12 es el cliente ideal; 3 es el cliente perdido.

### `segmento_rfm`
- **Fórmula:** etiqueta según el `rfm_score`:
  - 10-12 → "Campeón"
  - 7-9 → "Leal"
  - 5-6 → "En riesgo"
  - 3-4 → "Perdido"
- **Tipo:** texto.
- **Justificación de negocio:** traduce el puntaje numérico a un segmento accionable por marketing. Permite diseñar campañas distintas para cada grupo (retención para "Campeones", reactivación para "En riesgo", recuperación para "Perdidos").

### `es_cliente_club`
- **Fórmula:** `fecha_inscripcion_club.notnull()`
- **Tipo:** booleano (True / False).
- **Justificación de negocio:** señala si el cliente pertenece al programa de fidelidad. Es la señal más directa de pertenencia al club, útil para cualquier análisis que compare clientes del club contra los que compran sin estar inscritos.
