# Entregable `01_ingesta_bronze`

`student_id: emanuel_rb`

1. CSV, Parquet y Delta.

Observando el esquema por defecto de **CSV** el único tipo de dato es `string`. Esto Sucede porque `.csv` no guarda tipos de datos.

En cambio, al utilizar el esquema físico del **Parquet** que contiene toda la metadata, cada columna tiene su tipo de dato correcto con el que fué escrito.

Al utilizar inferencia de tipos en `transactions`, se puede ver que el campo `amount`, aún no tiene el tipo de dato correcto sino `string` y además, el runtime para esto fue de `9.870s` por lo cual no es recomendable utilizar la inferencia.

Las ventajas de tener un **Delta** para los archivos parquet son: que nos permite llevar un historial (log) de estos archivos, sabiendo (por ej) cuando fué la última vez que se creó, modificó, numero de archivos, el tamaño del registro (log), etc. y, además, nos permite llevar el versionado de los `.parquet` brindando informacion como por ejemplo qué usuario modificó algo por ultima vez, su timestamp, que operacion realizó y con qué id de notebook. También agrega estadísticas, features y ahislamiento de escritura.

1. Las cinco V (Volumen, Variedad, Velocidad, Veracidad y Valor)

• **Volumen**: Con escala small hay 5.000 clientes, 500 productos, 50.011 transacciones y 200.000 eventos. bronze_transactions pesa 687.083 bytes en un solo archivo.

• **Velocidad**: La tabla se creó el 2026-09-26 y se recargó el 2026-10-05. La segunda corrida sacó el archivo viejo y escribió otro, otra vez con 50.011 filas. Es reproceso por lotes, no un stream.

• **Variedad**: Clientes y transacciones llegan en CSV y quedan como string. Productos llegan en Parquet, con product_id long y price decimal. Eventos llegan en JSON y context es una estructura.

• **Veracidad**: Al medir sin limpiar: 50.011 filas, 50.000 transaction_id distintos y 52 amount que no pasan a número. Bronze los conserva.

• **Valor**: Ya se puede agrupar por payment_channel y ver duplicados e importes rotos antes de corregir en Silver.