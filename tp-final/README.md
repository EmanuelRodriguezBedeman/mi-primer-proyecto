# TP final — Efecto de la música

**Estudiante:** Emanuel Rodriguez Bedeman (`emanuel_rb`)

**Variante 2:** Ejecución. **Opción B** — experimentación y comparación de modelos con MLflow.

## Objetivo

Predecir si una persona dice que la música le mejora, no le hace nada o le empeora la salud mental, a partir de sus hábitos de escucha. Es una clasificación de la variable Music effects, con las clases Improve, No effect y Worsen. La métrica de comparación es el `F1` macro de test: el promedio del `F1` de las tres clases, con el mismo peso para cada una. La accuracy queda corta porque el 74 % de las respuestas ya es Improve, y un modelo que siempre elige esa clase acierta eso sin distinguir las otras dos.

## Contenido

| Archivo | Qué es |
|---|---|
| `data/mxmh_survey_results.csv` | Encuesta, 736 respuestas y 33 columnas |
| `01_experimentos.ipynb` | Carga con Spark y exploración. Todavía no imputa ni entrena |
| `README.md` | Esta guía |

## Datos

Fuente: [Music & Mental Health Survey Results](https://www.kaggle.com/datasets/catherinerasgaitis/mxmh-survey-results), Catherine Rasgaitis. Licencia CC0. El CSV está en el repo porque es público además no es necesario almacenar ningún tipo de credencial de kaggle para acceder a él en databricks.

Los datos se generaron a través de encuesta de un Google Forms publicado en Reddit, Discord y redes. No es una muestra de la población.

## Calidad de los datos

Pendiente. Se completa con la salida de `01_experimentos.ipynb`: faltantes por columna, filas sin `Music effects`, valores de `BPM` que no son un tempo y lo demás que muestre esa corrida. El notebook marca BPM fuera de 40–250. Ese rango no viene de la encuesta: un tema suele ir de 60 a 180, y 40–250 es un margen ancho para ver valores imposibles, como 0 o 999999999. No es todavía una regla de limpieza. El arreglo se decide después de ese relevamiento. El notebook no imputa ni descarta filas.

## Preparación en Spark

La carga y, cuando se agreguen, los filtros y las columnas nuevas corren con Spark. Hoy la tabla tiene `736` filas. Queda escrita así para no rehacer la preparación si la encuesta crece. El entrenamiento de los modelos va a correr en el driver con scikit-learn: con este tamaño, un entrenamiento distribuido no cambia el resultado.

## Features

Los puntajes `Anxiety`, `Depression`, `Insomnia` y `OCD` se pueden usar como variables. El widget `Puntajes que entran` elige `ninguno`, `seleccionados` o `los cuatro`. Si es `seleccionados`, el widget `Puntajes seleccionados` lista los nombres separados por coma, por ejemplo `Depression, Anxiety`.

## Modelos previstos

Los cuatro van a ver la misma matriz y el mismo corte 80/20 estratificado (`random_state=7`). **Random forest**, **LightGBM** y **CatBoost** usan balanceo de clases. Gana el mayor F1 macro de test.

| Run | Modelo |
|---|---|
| `logistica` | Regresión logística, `C=1` |
| `random_forest` | Random forest |
| `lightgbm` | LightGBM |
| `catboost` | CatBoost |

Esas celdas todavía no están. Primero se mira la exploración.

## Cómo correr la exploración

1. En Databricks, abrir `01_experimentos.ipynb` con un cluster de notebooks, no con un SQL warehouse.
2. Dejar el widget `Archivo CSV` como viene: apunta a `data/mxmh_survey_results.csv` al lado del notebook.
3. Elegir `Puntajes que entran` y ejecutar de arriba hacia abajo.
4. El experimento de MLflow, cuando se entrene, va a ser `/Users/<usuario>/bigdata_emanuel_rb_tp_final`.
