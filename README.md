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
- `07_final_model`: modelo final metricas finales y predicciones.
- `08_plots`: graficas generadas.
- `09_delivery`: espacio para archivos finales de entrega.

Tambien se genera `outputs/output_index.csv` como indice de los archivos producidos.
