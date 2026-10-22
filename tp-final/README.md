# TP final — Efecto de la música

**Estudiante:** Emanuel Rodriguez Bedeman (`emanuel_rb`)

**Variante 2:** Ejecución. **Opción B** — experimentación y comparación de modelos con MLflow.

## Objetivo

Predecir si una persona dice que la música le mejora, no le hace nada o le empeora la salud mental, a partir de sus hábitos de escucha. Es una clasificación de la variable Music effects, con las clases Improve, No effect y Worsen. La métrica de comparación es el `F1` macro de test: el promedio del `F1` de las tres clases, con el mismo peso para cada una. La accuracy queda corta porque el 74 % de las respuestas ya es Improve, y un modelo que siempre elige esa clase acierta eso sin distinguir las otras dos.

## Contenido

| Archivo | Qué es |
|---|---|
| `data/mxmh_survey_results.csv` | Encuesta, 736 respuestas y 33 columnas |
| `00_eda_cleaning.ipynb` | Exploración y una limpieza mínima: tipos de dato y baja de `Permissions`. No imputa ni entrena |
| `README.md` | Esta guía |

## Datos

Fuente: [Music & Mental Health Survey Results](https://www.kaggle.com/datasets/catherinerasgaitis/mxmh-survey-results), Catherine Rasgaitis. Licencia CC0. El CSV está en el repo porque es público además no es necesario almacenar ningún tipo de credencial de kaggle para acceder a él en databricks.

Los datos se generaron a través de encuesta de un Google Forms publicado en Reddit, Discord y redes. No es una muestra de la población.

## Calidad de los datos

La exploración de `00_eda_cleaning.ipynb` muestra faltantes por columna, filas sin `Music effects` y valores de `BPM` que no son un tempo. El notebook marca BPM fuera de 40–250. Ese rango no viene de la encuesta: un tema suele ir de 60 a 180, y 40–250 es un margen ancho para ver valores imposibles, como 0 o 999999999. Todavía no es una regla que descarte filas. El notebook no imputa ni descarta filas.

La limpieza que sí queda hecha es de tipos y de una columna: `Permissions` se elimina. Solo registra si la persona aceptó que se publique su respuesta. No dice nada del hábito de escucha ni de `Music effects`, que es la etiqueta. En `Primary streaming service`, `I do not use a streaming service` pasa a `No streaming` y `Other streaming service` pasa a `Other`.

## Columnas

| Columna | Qué es |
|---|---|
| `Timestamp` | Fecha y hora en que envió el formulario |
| `Age` | Edad |
| `Primary streaming service` | Servicio de música que más usa |
| `Hours per day` | Horas de música por día |
| `While working` | Si escucha música mientras trabaja |
| `Instrumentalist` | Si toca un instrumento |
| `Composer` | Si compone música |
| `Fav genre` | Género favorito |
| `Exploratory` | Si busca artistas o géneros nuevos |
| `Foreign languages` | Si escucha música en otros idiomas |
| `BPM` | Pulsos por minuto que declaró (*beats per minute*) |
| `Frequency [Classical]` | Con qué frecuencia escucha classical |
| `Frequency [Country]` | Con qué frecuencia escucha country |
| `Frequency [EDM]` | Con qué frecuencia escucha EDM |
| `Frequency [Folk]` | Con qué frecuencia escucha folk |
| `Frequency [Gospel]` | Con qué frecuencia escucha gospel |
| `Frequency [Hip hop]` | Con qué frecuencia escucha hip hop |
| `Frequency [Jazz]` | Con qué frecuencia escucha jazz |
| `Frequency [K pop]` | Con qué frecuencia escucha K-pop |
| `Frequency [Latin]` | Con qué frecuencia escucha latin |
| `Frequency [Lofi]` | Con qué frecuencia escucha lofi |
| `Frequency [Metal]` | Con qué frecuencia escucha metal |
| `Frequency [Pop]` | Con qué frecuencia escucha pop |
| `Frequency [R&B]` | Con qué frecuencia escucha R&B |
| `Frequency [Rap]` | Con qué frecuencia escucha rap |
| `Frequency [Rock]` | Con qué frecuencia escucha rock |
| `Frequency [Video game music]` | Con qué frecuencia escucha música de videojuegos |
| `Anxiety` | Puntaje autodeclarado de ansiedad, de 0 a 10 |
| `Depression` | Puntaje autodeclarado de depresión, de 0 a 10 |
| `Insomnia` | Puntaje autodeclarado de insomnio, de 0 a 10 |
| `OCD` | Puntaje autodeclarado de trastorno obsesivo-compulsivo, de 0 a 10 |
| `Music effects` | Si la música le mejora (`Improve`), no le hace nada (`No effect`) o le empeora (`Worsen`) |
| `Permissions` | Si aceptó que se publique su respuesta. Se elimina en la limpieza |

## Preparación en Spark

La carga y, cuando se agreguen, los filtros y las columnas nuevas corren con Spark. Hoy la tabla tiene `736` filas. Queda escrita así para no rehacer la preparación si la encuesta crece. El entrenamiento de los modelos va a correr en el driver con scikit-learn: con este tamaño, un entrenamiento distribuido no cambia el resultado.

## Features

Qué puntajes de salud entran como variables se elige en el notebook de entrenamiento, no en la exploración. Ahí cada uno (`Anxiety`, `Depression`, `Insomnia`, `OCD`) se marca por separado.

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

1. En Databricks, abrir `00_eda_cleaning.ipynb` con un cluster de notebooks, no con un SQL warehouse.
2. Dejar el widget `Archivo CSV` como viene: apunta a `data/mxmh_survey_results.csv` al lado del notebook.
3. Ejecutar de arriba hacia abajo.
4. El experimento de MLflow, cuando se entrene, va a ser `/Users/<usuario>/bigdata_emanuel_rb_tp_final`.
