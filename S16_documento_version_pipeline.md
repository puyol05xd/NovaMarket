# Documento de versión — Pipeline NovaMarket

**Versión:** v1
**Archivo:** `novamarket_pipeline_v1.joblib`
**Fecha de la versión:** 18 de noviembre de 2026
**Integrantes:** Juan Esteban Camacho Castro, Juan Diego Serpa Ruiz

---

## Qué contiene esta versión

El pipeline empaqueta, en un solo objeto reutilizable, todas las transformaciones de preprocesamiento del dataset de NovaMarket más un modelo de validación al final. Se construyó a partir del dataset `novamarket_features_seleccionadas.csv` de la semana 14 (8 features finales + `fuga_cliente` + `id_cliente`).

### Estructura del pipeline

```
Pipeline
├── preprocesador (ColumnTransformer)
│   ├── continuas → Pipeline(SimpleImputer(median) + StandardScaler)
│   └── binarias → "passthrough"
└── modelo_validacion → LogisticRegression(max_iter=1000, class_weight="balanced")
```

### Columnas incluidas

**Continuas (6):** se les aplica imputación con la mediana y luego escalado con `StandardScaler`.

- `nivel_lealtad_cod`
- `edad_cliente`
- `gasto_promedio`
- `precio_unitario`
- `monto_compra`
- `dias_desde_ultima_compra`

**Binarias (2):** pasan tal cual (`passthrough`), porque ya están codificadas como 0/1 desde la semana 9 y no necesitan escalado.

- `canal_App`
- `es_venta_corporativa`

---

## Validación del pipeline

Se evaluó con `cross_val_score(cv=5, scoring="recall")` sobre el conjunto de entrenamiento (`X_entrenamiento`, `y_entrenamiento`). Se usó `recall` en vez de `accuracy` porque la variable objetivo está desbalanceada (~21 % de fuga), y la exactitud daría una falsa sensación de buen desempeño si el modelo ignorara la clase minoritaria.

- **Recall promedio (5 folds):** [completar con `resultados.mean()`]
- **Desviación estándar:** [completar con `resultados.std()`]

El balanceo de clases se hizo con `class_weight="balanced"` dentro del `LogisticRegression`, no con SMOTE. Esto evita la fuga de datos que ocurriría si se balanceara antes de la validación cruzada: con `class_weight`, el ajuste se calcula fold por fold, solo con el entrenamiento de cada ronda.

---

## Confirmación de no-fuga de datos

- La división en entrenamiento y prueba se hizo con `train_test_split(..., stratify=y, random_state=42)` antes de cualquier ajuste del pipeline.
- `cross_val_score` ajusta el pipeline completo (imputador + escalador + modelo) solo con el entrenamiento de cada fold, y evalúa con la parte restante que ese fit nunca vio.
- El `.fit()` final se hizo únicamente con `X_entrenamiento` y `y_entrenamiento`. El conjunto de prueba (`X_prueba`, `y_prueba`) solo se usó para observar el `recall` final con `.predict()`, nunca para ajustar.

---

## Cómo usar este pipeline con datos nuevos

```python
import joblib
pipeline_cargado = joblib.load("novamarket_pipeline_v1.joblib")
predicciones = pipeline_cargado.predict(datos_nuevos)
```

Los datos nuevos deben traer las mismas 8 columnas con los mismos nombres y en el mismo formato que el dataset de la semana 14. Si llegan con columnas crudas sin codificar (por ejemplo, `canal_compra` en texto), este pipeline no lo sabe convertir porque las binarias van en `passthrough`.
