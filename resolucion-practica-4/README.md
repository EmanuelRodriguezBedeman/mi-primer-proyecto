# Entrega práctica 4 — Streaming

- **Nombre:** Emanuel Rodriguez Bedeman
- **student_id:** emanuel_rb
- **Escala:** small

## Log y offsets (`01_log_y_offsets`)

1. En el ejemplo en Python, ¿cuántos mensajes quedaron en la cola y cuántos en el log después de leer tres? ¿Qué diferencia hay entre una cola tradicional y un log como Kafka?

Después de leer tres, en la cola quedan 2 mensajes (`pago_4` y `pago_5`). En el log siguen los 5.

En una cola tradicional, leer un mensaje lo saca de la cola. En un log como Kafka, leer no borra nada: el consumidor solo avanza su offset (acá quedó en 3). Por eso el log conserva los mensajes y otro consumidor puede leerlos a su ritmo, o releerlos desde el inicio.

2. Después de la llegada 2, ¿cuántos mensajes había leído el consumidor `antifraude` y cuántos `reportes`? ¿El atraso de `reportes` afectó a `antifraude`? ¿Por qué?

Después de la llegada 2, `antifraude` había leído 180 mensajes y `reportes` 0. El topic ya tenía esos 180.

El atraso de `reportes` no afectó a `antifraude`. Cada consumidor guarda su propio offset: `antifraude` siguió leyendo lo nuevo (60 y después 120) mientras `reportes` no leía nada. En la llegada 3, `reportes` leyó de una los 220 que tenía pendientes y `antifraude` solo los 40 nuevos.

3. ¿Qué es un offset y dónde lo guarda cada consumidor en esta práctica?

El offset es hasta dónde leyó un consumidor en el log. En esta práctica el topic es una tabla Delta y cada append es una versión: esa versión es el offset.

Cada consumidor lo guarda en su propio checkpoint (`checkpoints/antifraude` y `checkpoints/reportes`), no en el topic. El número que guarda es la próxima versión a leer. Por eso, al reejecutar sin mensajes nuevos, los dos leen 0: la última versión del topic es 5 y el offset de ambos ya está en 6.

4. ¿Cuántos mensajes leyó el replay? ¿Qué propone la arquitectura Kappa para reprocesar datos y en qué se diferencia de Lambda?

El replay leyó 280 mensajes, todos los del topic.Releyó el log desde el principio.

Kappa propone un solo código de streaming: para reprocesar se lanza una consulta nueva que lee el log desde el inicio con la lógica nueva (acá, el monto total por canal). Lambda mantiene dos sistemas, un batch y un streaming, y hay que escribir la misma lógica en los dos.

### Ingesta y enriquecimiento (`02_ingesta_y_enriquecimiento`)

5. ¿Cuántas filas nuevas procesó Bronze en cada llegada y cuántas al reejecutar sin archivos nuevos? ¿Qué componente evita leer dos veces el mismo archivo?

Bronze procesó, por llegada, 60, 120, 40, 40 y 20 filas nuevas (stream_001 a stream_005). En cada vuelta sumó solo el archivo que acababa de llegar.

Al reejecutar sin archivos nuevos, Bronze y Silver suman 0.

El checkpoint de Auto Loader (`cloudFiles`, en `checkpoints/bronze`) recuerda qué archivos ya leyó, así que no los vuelve a ingerir.

6. ¿Qué motivos de rechazo aparecieron en la cuarentena y en qué llegada? ¿Por qué Bronze guarda el texto original en lugar de descartar los registros con errores?

La cuarentena apareció solo en `stream_002`: 20 eventos con `INVALID_AMOUNT` y 20 con `UNKNOWN_CUSTOMER`. Las otras llegadas no tuvieron rechazos.

Bronze guarda el texto original porque el descarte se hace en Silver, al aplicar el contrato. Si después cambia una regla o el formato, la línea cruda sigue disponible y se puede reprocesar. Si se tirara en la ingesta, ese dato se pierde.

7. ¿Qué columnas agregó el join con `silver_customers` y `silver_products`? ¿Qué tipo de join es (stream-stream, stream-tabla o tabla-tabla) según la teoría?

El join agregó `country`, que sale de `silver_customers`, y `category`, que sale de `silver_products`. El evento solo traía `customer_id` y `product_id`. Si el cliente no existe, `country` queda null y la fila va a cuarentena como `UNKNOWN_CUSTOMER` (en la muestra, `bad_customer_000`).

Es un join stream-tabla: el stream es Bronze y las dos dimensiones son tablas estáticas. No es stream-stream ni tabla-tabla.

### Tiempo y ventanas (`03_tiempo_y_ventanas`)

8. ¿Qué diferencia hay entre event time y processing time? Usá `late_ok` como ejemplo.

El event time es cuándo ocurrió el evento (`event_ts`). El processing time es cuándo lo procesó el pipeline (la llegada).

`late_ok` ocurrió a las 12:03, pero se procesó en la llegada 2. En la llegada 1 ya habían entrado compras de alrededor de las 12:00. Por eso llega "tarde": su hora de compra es anterior al momento en que el sistema lo ve. Las ventanas lo cuentan por las 12:03, no por la llegada 2.

9. ¿Cuánto valía el watermark al terminar cada llegada? ¿Cómo se calcula a partir del máximo event time visto?

El watermark al terminar cada llegada fue 11:54, 11:58, 12:16, 12:17 y 12:35.

Se calcula como el máximo event time visto hasta esa llegada, menos 10 minutos. Esos máximos son 12:04 (`e003`), 12:08 (`e005`), 12:26 (`e007`), 12:27 (`e008`) y 12:45 (`clock`).

10.  ¿En qué llegada aparecieron las primeras ventanas tumbling en modo append y por qué no antes?

Las primeras ventanas tumbling en modo append aparecieron en la llegada 3. En las llegadas 1 y 2 no se emitió ninguna.

En append la ventana se emite cuando el watermark supera su fin. Tras la llegada 1 el watermark era 11:54 y tras la 2 era 11:58, ambos antes de las 12:05, así que [12:00, 12:05) y [12:05, 12:10) seguían abiertas. En la llegada 3 el máximo event time pasa a 12:26, el watermark a 12:16, y esas dos ventanas se cierran.

11. ¿Qué pasó con `late_bad`? ¿Por qué se aceptó `late_ok` y se descartó `late_bad`?

`late_bad` llegó en la llegada 4 con event time 12:02 y se descartó: no sumó en ninguna ventana.

`late_ok` (12:03) llegó en la llegada 2, cuando el watermark todavía estaba en 11:54, así que entró en la ventana [12:00, 12:05). `late_bad` llegó después de la llegada 3, cuando el watermark ya estaba en 12:16. Su event time queda por debajo y se descarta.

12. Mirando el gráfico de ventanas, ¿en qué se diferencian tumbling, hopping y session? Da un ejemplo de negocio para cada una.

**Tumbling** son ventanas fijas de tiempo de 5 minutos, sin solaparse. Sirve para el total vendido cada 5 minutos.

**Hopping** mide 10 minutos y avanza de a 5, así que se solapan. Una compra cuenta en más de una ventana. Sirve para contabilizar las compras de *los últimos 10 minutos*, recalculado cada 5.

**Session** agrupa por cliente y se cierra tras 5 minutos sin compras de ese cliente. El cliente 0 tiene dos sesiones (4 y 3 compras) porque hubo una pausa. Sirve para una visita o una ráfaga de pagos del mismo cliente.

13.  Para la ventana [12:00, 12:05) del canal card, ¿cuántas veces y con qué valores se emitió en modo update y en modo append? ¿Qué ventaja y qué desventaja tiene cada modo?

Las ventas en modo `update` se emitió dos veces primero con monto `6000` y luego corregido al llegar `late_ok` con valor `8400`.

Las ventas en modo `append`se emitió una sola vez (valor: `8400`).

La ventaja del modo `update` es que se emite rápidamente las ventas, pero la desventaja es que el consumidor debe tolerar recibir la misma venta dos veces.

La ventaja del modo `append` es que se emite una sola vez, pero tiene mayor demora (latencia).

14.  ¿Qué se gana y qué se pierde si el watermark fuera de 1 minuto en lugar de 10?

Con un watermark de 1 minuto se gana latencia: las ventanas se cierran apenas el máximo event time las supera por un minuto, en vez de esperar 10. *El resultado sale antes*.

**Se pierde completitud**. El margen para tardíos pasa de 10 minutos a 1. Un evento como `late_ok`, que hoy entra porque llega dentro de esos 10 minutos, queda mucho más cerca del corte y uno que llegue varios minutos tarde se descarta.

### Garantías y fallas (`04_garantias_y_fallas`)

15. ¿Cuántas filas quedaron en el destino sin deduplicar y cuántas deduplicando por `event_id`? ¿Por qué el productor puede enviar el mismo evento dos veces?

Quedaron en el destino sin deduplicar: `220` y deduplicando por `event_id 200`.

El productor puede enviar el mismo evento dos veces porque usa at-least-once: si no sabe si el envío llegó, lo reintenta. En los datos, `e002` aparece en dos llegadas. Sin deduplicar esas copias se escriben. Con `dropDuplicatesWithinWatermark` se conserva una sola.

16. ¿Qué pasó al reiniciar con el mismo checkpoint y qué pasó al perderlo? ¿Qué garantía de entrega se observa en cada caso?

Al reiniciar con el mismo checkpoint, el job no vuelve a escribir nada. El checkpoint tiene registro de hasta dónde se procesó. Eso es `exactly-once`.

Al perder el checkpoint, el consumer no tiene registro de hasta donde se procesó. Como el destino es un append, entonces las filas se escriben otra vez. Se duplican los resultados. La garantía que se observa es at-least-once: no se pierden eventos, pero se duplican.

17.  ¿Por qué con `MERGE` el resultado no cambió aunque se perdiera el checkpoint? ¿Qué significa que una escritura sea idempotente?

Con `MERGE` el resultado no cambió porque la segunda ejecución, aunque perdió el checkpoint y releyó todo, inserta solo los `event_id` que todavía no están. Esos 200 ya estaban, así que no se agregó ninguna fila.

Una escritura es idempotente cuando aplicarla dos veces deja el mismo resultado que aplicarla una. El `MERGE` por `event_id` lo es. Un `append` no: la segunda pasada duplica las filas.

### CDC (`05_cdc`)

18. ¿Cuántos eventos de cambio generó el log y de qué tipos? ¿Por qué un `UPDATE` genera dos eventos?

El log generó 13 eventos de cambio: 6 `insert` (los 5 clientes iniciales y el alta del 100), 3 `update_preimage`, 3 `update_postimage` y 1 `delete`. 

Un `UPDATE` genera dos eventos porque el log guarda la fila antes (`update_preimage`) y después (`update_postimage`). Así se ve qué cambió, no solo el valor final.

19.  ¿Se pudo reconstruir la tabla a partir del log? ¿Qué información tiene el log que la tabla actual no tiene?

Sí. Quedándose con el último cambio de cada `customer_id` y descartando los que terminan en `delete`, la tabla reconstruida coincide con la actual.

El log guarda la historia que la tabla ya pisó. Del cliente 2 se ve que estuvo en `CL` antes de `BR`. Del cliente 3 se ve que existió y después fue borrado. La tabla solo muestra el estado final.