# Objetivo A

Este proyecto desarrolla un flujo completo en Python para deteccion de fraude en transacciones originadas desde aplicaciones moviles o canales digitales equivalentes. El objetivo principal es comparar metricas tradicionales y metricas custom en LightGBM para reducir falsos positivos sin perder capacidad de deteccion de fraude.

El cuaderno principal es `ObjetivoA.ipynb`. Este notebook carga el dataset `01_bo_vip_seed22_n100000.csv` o su equivalente local `Copia de 01_bo_vip_seed22_n100000.csv`, realiza EDA, construye una proxy de transacciones mobile app, genera variables predictivas, entrena modelos LightGBM y evalua resultados generales y por segmento mobile app.

## Version de Python

Se requiere Python 3.11 o superior. Se recomienda usar Python 3.12 para evitar conflictos con las versiones fijadas en `requirements.txt`.

## Instalacion

Desde la raiz del proyecto ejecutar:

```bash
pip install -r requirements.txt
```

Para abrir y ejecutar los notebooks se recomienda usar Jupyter Notebook, JupyterLab, VS Code o un entorno equivalente con kernel de Python 3.12.

## Modelos implementados

- Modelo baseline LightGBM con metricas tradicionales.
- Modelo LightGBM con metrica custom `mobile_app_false_positive_ratio`.
- Modelo LightGBM con metrica custom `mobile_app_recall_constrained_fp_ratio`.
- Modelo LightGBM con metrica custom `mobile_app_alert_quality`.
- Modelo final con tuning ligero de hiperparametros.

## Guia de uso

1. Confirmar que el dataset principal esta en la raiz del proyecto.
2. Instalar dependencias con `requirements.txt`.
3. Abrir `ObjetivoA.ipynb`.
4. Ejecutar todas las celdas de arriba hacia abajo.
5. Revisar resultados en la carpeta `outputs`.

## Outputs

El notebook genera resultados organizados en subcarpetas dentro de `outputs`:

- `01_data_checks`: validaciones y archivos de control.
- `02_eda`: resumenes del analisis exploratorio.
- `03_features`: lista de variables usadas por el modelo.
- `04_baseline`: metricas y predicciones del baseline.
- `05_custom_metrics`: comparacion de modelos con metricas custom.
- `06_tuning`: resultados del tuning ligero.
- `07_final_model`: modelo final, metricas finales y predicciones.
- `08_plots`: graficas generadas.
- `09_delivery`: espacio para archivos finales de entrega.

Tambien se genera `outputs/output_index.csv` como indice de los archivos producidos.

---

# Objetivo B

Esta fase implementa una simulacion federada para deteccion de fraude usando Banco 1 y Banco 2 como datasets etiquetados. Banco 3 se trata como dataset no etiquetado y se usa solo para inferencia.

El cuaderno principal es `ObjetivoB.ipynb`. Este notebook carga los tres bancos, realiza un miniEDA por archivo, valida el target en Banco 1 y Banco 2, construye variables comunes incluyendo contadores y acumuladores, controla fuga de informacion, entrena y compara multiples estrategias de modelado, optimiza hiperparametros y genera predicciones binarias para Banco 3.

El objetivo practico es producir inferencias preliminares sobre el primer 30 por ciento de Banco 3 y una inferencia final sobre el 100 por ciento de Banco 3. Como Banco 3 no tiene etiqueta real utilizable, las metricas reales se calculan solo con Banco 1 y Banco 2.

## Datos de entrada

Los archivos esperados pueden estar en la raiz del proyecto o en una carpeta `data`:

- `Copia de 01_bo_vip_seed22_n100000.csv`
- `Copia de 02_br_privado_seed33_n100000.csv`
- `Copia de 03_gt_estatal_seed3_n100000.csv`

## Metodologia

### EDA

Se realiza un miniEDA por banco: forma, tipos de datos, valores faltantes, cardinalidad y distribucion del target. Se incluye una grafica de desbalance de clases para Banco 1 y Banco 2.

### Ingenieria de variables

Se crean variables derivadas comunes entre los tres bancos:

- Transformaciones numericas: `log_amount`, `amount_to_baseline_ratio`, `log_distance_from_home`.
- Indicadores temporales: `night_transaction_flag`, `business_hour_transaction_flag`, `hour`, `month`, `day`, `weekday`.
- Proxies de canal: `ecommerce_channel_proxy`, `manual_or_ecommerce_entry_proxy`, `contactless_proxy`.
- Contadores por cliente, tarjeta y comercio: `customer_transaction_count`, `card_transaction_count`, `merchant_transaction_count`.
- Acumuladores de monto: `*_total_amount`, `*_mean_amount`, `*_amount_to_mean_ratio`.
- Cardinalidad: `customer_unique_merchant_count`, `card_unique_merchant_count`.

### Control de fuga de informacion

Se excluyen variables que comprometen la generalizacion entre bancos:

- Target `is_fraud` y variables directas de fraude.
- Identificadores de banco (codigo, nombre, pais, tier).
- Pais, ciudad y moneda local.
- Identificadores directos de cliente, tarjeta, comercio, terminal, cuenta, autorizacion y transaccion.
- Columnas con tasa de nulos superior al 95 por ciento o cardinalidad excesiva.

Esto busca que el modelo aprenda patrones transaccionales generales y no memorice informacion especifica de un banco o pais.

### Estrategias de modelado

Se entrenan y comparan las siguientes estrategias usando Banco 1 y Banco 2:

- Modelo local Banco 1 (ExtraTrees).
- Modelo local Banco 2 (ExtraTrees).
- Ensamble federado con pesos iguales.
- Ensamble federado con pesos por desempeno en validacion.
- Modelo centralizado baseline (ExtraTrees sobre datos combinados).
- Modelo logistico baseline centralizado.
- Modelo boosting centralizado (HistGradientBoosting).
- Modelo tuneado con hiperparametros (HistGradientBoosting + GridSearchCV).

### Optimizacion de hiperparametros

Se ejecuta una busqueda con `GridSearchCV` y validacion cruzada estratificada sobre una muestra del conjunto de entrenamiento central. El mejor estimador se reentrena con todos los datos disponibles y compite en la seleccion final junto al resto de estrategias.

### Seleccion de metodologia final

La estrategia final se selecciona automaticamente con una puntuacion interna que combina F1 promedio, F1 minimo entre bancos, ROC AUC, recall y penalizacion por false positive ratio. La estrategia ganadora se usa para la inferencia final sobre el 100 por ciento de Banco 3.

### Visualizaciones

El notebook genera las siguientes graficas de soporte:

- Desbalance de clases por banco.
- Distribuciones de variables clave comparadas entre los tres bancos.
- Curvas ROC por estrategia.
- Curvas Precision-Recall por estrategia.
- Comparativa de metricas entre estrategias.
- Distribucion de probabilidades predichas para Banco 3.

### Simulacion federada

La simulacion federada se usa porque el trabajo se ejecuta en un solo entorno local con archivos CSV. Un entrenamiento federado real requeriria clientes distribuidos, rondas de comunicacion, agregacion de parametros y una infraestructura que no forma parte del alcance practico del notebook.

## Evaluacion interna

Las metricas se calculan sobre las validaciones de Banco 1 y Banco 2:

- `bank_1_valid`
- `bank_2_valid`
- `central_valid`
- Validacion cruzada entre bancos para los modelos locales.

Banco 3 solo recibe inferencias. Se reporta la tasa esperada de fraude predicha y la distribucion de probabilidades, pero no se calculan metricas reales porque no existe etiqueta confiable.

## Guia de uso

1. Confirmar que los tres CSV estan en la raiz del proyecto o en la carpeta `data`.
2. Instalar dependencias con `pip install -r requirements.txt`.
3. Abrir `ObjetivoB.ipynb`.
4. Ejecutar todas las celdas de arriba hacia abajo.
5. Revisar los resultados en `outputs_fase_B`.

## Outputs

El notebook genera resultados en `outputs_fase_B`:

- `01_mini_eda`: resumenes iniciales por banco y mapeo de columnas.
- `02_data_checks`: validaciones de carga, target, balance de clases y columnas detectadas.
- `03_features`: variables seleccionadas, excluidas y resumen de ingenieria.
- `04_models`: modelos entrenados en formato joblib y pesos del ensamble federado.
- `05_predictions`: archivos Excel de entrega y auditorias CSV.
- `06_plots`: graficas generadas.
- `07_reports`: metricas por umbral, seleccion de modelo, tuning de hiperparametros y resumenes finales.

## Entregables principales

Los archivos de entrega quedan en `outputs_fase_B/05_predictions`:

- `bank_3_first_30_baseline_centralized_model.xlsx`
- `bank_3_first_30_federated_equal_weight_model.xlsx`
- `bank_3_first_30_federated_performance_weighted_model.xlsx`
- `bank_3_full_final_inference.xlsx`

Cada Excel contiene unicamente la columna `is_fraud` con valores `True` o `False`.
