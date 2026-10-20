# TP final — Efecto de la música

**Estudiante:** Emanuel Rodriguez Bedeman (`emanuel_rb`)

**Variante:** 2. Ejecución. **Opción B** — experimentación y comparación de modelos con MLflow.

## Objetivo

Predecir cómo dice la persona que la música le afecta la salud mental. La etiqueta es `Music effects`: `Improve`, `No effect` o `Worsen`. Es un problema de clasificación. La métrica de comparación es el F1 macro de test: el promedio del F1 de las tres clases, con el mismo peso para cada una. La accuracy no decide, porque `Improve` es cerca del 74 % de las respuestas.

## Contenido

| Archivo | Qué es |
|---|---|
| `data/mxmh_survey_results.csv` | Encuesta, 736 respuestas y 33 columnas |
| `01_experimentos.ipynb` | Carga con Spark y exploración. Todavía no imputa ni entrena |
| `README.md` | Esta guía |

## Datos

Fuente: [Music & Mental Health Survey Results](https://www.kaggle.com/datasets/catherinerasgaitis/mxmh-survey-results), Catherine Rasgaitis. Licencia CC0. El CSV está en el repo porque ya es público y así el notebook lo lee en Databricks sin una cuenta de Kaggle.

La encuesta salió de un formulario publicado en Reddit, Discord y redes. No es una muestra de la población. Ocho filas no tienen `Music effects`. `BPM` tiene faltantes y al menos un valor imposible. El arreglo de eso se decide después de mirar la exploración. El notebook no imputa ni descarta filas.

Los puntajes `Anxiety`, `Depression`, `Insomnia` y `OCD` se pueden usar como variables. En el notebook, el widget `puntajes_salud` elige `ninguna`, `algunas` o `todas`. Si es `algunas`, el widget `puntajes_algunas` lista las columnas separadas por coma.

## Preparación en Spark

La carga y, cuando se agreguen, los filtros y las columnas nuevas corren con Spark. Hoy la tabla tiene 736 filas. Queda escrita así para no rehacer la preparación si la encuesta crece. El entrenamiento de los modelos va a correr en el driver con scikit-learn: con este tamaño, un entrenamiento distribuido no cambia el resultado.

## Modelos previstos

Los cuatro van a ver la misma matriz y el mismo corte 80/20 estratificado (`random_state=7`). Random forest, LightGBM y CatBoost usan balanceo de clases. Gana el mayor F1 macro de test.

| Run | Modelo |
|---|---|
| `logistica` | Regresión logística, `C=1` |
| `random_forest` | Random forest |
| `lightgbm` | LightGBM |
| `catboost` | CatBoost |

Esas celdas todavía no están. Primero se mira la exploración.

## Cómo correr la exploración

1. En Databricks, abrir `01_experimentos.ipynb` con un cluster de notebooks, no con un SQL warehouse.
2. Dejar `csv_path` vacío. El notebook busca `data/mxmh_survey_results.csv` al lado del notebook, bajo `/Workspace`.
3. Elegir `puntajes_salud` y ejecutar de arriba hacia abajo.
4. El experimento de MLflow, cuando se entrene, va a ser `/Users/<usuario>/bigdata_emanuel_rb_tp_final`.
