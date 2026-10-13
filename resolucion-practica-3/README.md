# Entrega práctica 3 - No SQL

- **Nombre:** Emanuel Rodriguez Bedeman
- **student_id:** emanuel_rb
- **Escala:** small

### CAP y PACELC (`01_cap_pacelc`)

1. Con la red partida, ¿qué respondió la réplica `C` en modo CP y qué respondió en modo AP? ¿Qué propiedad de CAP sacrifica cada modo?

Con la red partida, en modo `CP` la réplica `C` no responde: la lectura da `**ERROR**`: no disponible y la escritura ERROR: sin quórum. 

En modo `AP` sí responde: la lectura devuelve `100` y la escritura de `50` termina en OK.

`CP` sacrifica la disponibilidad. El lado minoritario deja de atender para no devolver un saldo distinto al del lado `AB`.

`AP` sacrifica la consistencia. `C` sigue atendiendo y devuelve `100` mientras `A` y `B` ya tienen `80`.

2. En modo `AP`, ¿qué saldo quedó en las tres réplicas después de la reparación y qué escritura se perdió?

Después de la reparación, las tres réplicas quedan en `50`. Se perdió la escritura `saldo=80` hecha en `A`. La reparación usa last-write-wins y se queda con la escritura posterior, la de `50` en `C`. En `CP` esa escritura fue rechazada por falta de quórum, así que el `80` sobrevivió.

3. Según el gráfico de latencia, ¿cuál es la p99 con `W=1` y con `W=3`? ¿Qué se gana a cambio de esperar más réplicas? (PACELC)

Con `W=1` el `p99=9.6ms`.

Con `W=3` el `p99=55,6ms`.

`W` es cuántas réplicas tienen que confirmar la escritura antes de darla por hecha. Esperar más réplicas sube la latencia y, a cambio, la escritura queda en más copias antes de responder. Eso es el tramo LC de PACELC: sin red partida, más consistencia cuesta más latencia.

4. Con cinco réplicas partidas en `ABC | DE`, ¿qué lado pudo seguir escribiendo en modo CP y por qué?

Siguió escribiendo el lado `ABC` porque la cantidad de nodos totales es mayor que la cantidad de escritura y lectura.

### Clave-valor (`02_clave_valor`)

5. ¿Cuántas veces más rápido fue el GET en memoria que el GET con Spark? ¿Por qué Redis guarda los datos en memoria?

Fué ~3,212,655 veces más rápido.

Redis guarda los datos en memoria es mucho más rápido que en disco y esto es necesario para tenerlos disponibles lo más rápido posible y responder  un GET en el acto, porque busca un valor en específico de una clave.  No está pensado para recorrer toda la tabla, como sí lo está Spark sobre una tabla Delta.

6. ¿Por qué para contar los clientes de AR hubo que leer y parsear todos los valores? Mencioná una ventaja y una desventaja del modelo clave-valor.

Hubo que leer y parsear todos los valores porque el país está dentro del valor, y el almacén clave-valor solo sabe buscar por clave. El valor es un texto opaco: no existe un `WHERE country = 'AR'`. Para contar los clientes de AR se parsea cada JSON y después se filtra.

La ventaja es el `GET` por clave: encuentra un perfil al instante. La desventaja es consultar por un campo de adentro del valor: hay que recorrer y parsear todas las claves.

7. Al pasar de 4 a 5 nodos, ¿qué porcentaje de claves se movió con `hash mod N` y con hashing consistente? ¿Por qué conviene el segundo?

Al pasar de 4 a 5 nodos, se movieron con:
- `hash mod N = 79.6%` de claves
- hashing consistente `= 21.2%` de claves

Conviene el segundo porque solo se mudan las claves del tramo que ocupa el nodo nuevo. Con `mod N` cambia el divisor y hay que mover casi todas.

### Documental (`03_documental`)

8. ¿Cuántas filas devolvió la consulta relacional del cliente 42 y cuántos documentos la documental? ¿Qué datos quedaron embebidos dentro del documento?

La consulta relacional del cliente 42 devolvió 11 filas. La documental devolvió un solo documento.

En el documento quedan embebidos `profile` (país, segmento, fecha de alta, email), `stats` (cantidad de compras, monto total, fraudes) y el array `transactions`, con el producto adentro de cada compra.

9.  Al leer documentos con campos distintos, ¿qué hizo Spark con los campos que faltaban en algunos? ¿Por qué se dice que el modelo documental tiene esquema flexible?

Spark lo que fué construir un esquema que es la unión de todos los campos y completa con null los que datos faltantes.

El modelo documental se dice que tiene esquema flexible porque puedo ir modificandolo sobre la marcha (agregando o sacando campos) y aún va a funcionar, no va a crashear.

10. ¿Cuánto pesa el documento más grande? ¿Qué problema aparece si un documento crece sin límite?

El documento más grande pesa `5.2 KB`.

Si un documento crece sin límite, puede llegar a su tamaño máximo (16 MB) y al intentar pasarlo, MongoDB rechaza la escritura.

11. Para calcular el monto por categoría hubo que usar `explode`. ¿Qué tipo de consultas resuelve bien el modelo documental y cuáles le cuestan más?

El modelo documental resuelve bien consultas por entidad (dentro de cada documento), por ejemplo leer un cliente con su perfil y su historial es un solo documento.

Las que le cuesta más son aquellas que son las que cruzan varios documentos  (de analítica transversal) por ejemplo "*Monto total por categoría*". Esta última sería más fácil de resolver en un modelo relacional - SQL.

### Grafos (`04_grafos`)

12. ¿Cuántos nodos `Customer`, nodos `Device` y aristas `USES` tiene el grafo?

`Customer` tiene `5000` nodos y `Device` tiene `2361`. 

El grafo tiene 5,400 aristas `USES`.

13. ¿Cuántos joins necesitó SQL para el patrón de 2 saltos y para el de 4? ¿Por qué Neo4j recorre relaciones más eficientemente que una base relacional?

SQL para el patrón de **dos saltos** necesitó `3 JOINS` y para el de **cuatro saltos** necesita `4 JOINS`.

`Neo4j` lo hace más eficientemente porque ya guarda en cada nodo punteros a sus vecinos (*index-free adjacency*), lo que le permite pasar de un cliente a un dispositivo sin necesidad de buscar en una tabla.


14.  ¿Cuántas iteraciones tardó en converger la búsqueda de componentes conexos y cuántos nodos tiene el componente más grande?

La búsqueda convergió en la iteración 14, cuando ningún nodo cambió de etiqueta.

El componente más grande tiene 23 nodos. Su etiqueta es `c:122`: el id más chico de esa red, no la cantidad de nodos.

15. ¿Qué porcentaje de aristas quedó entre máquinas distintas al repartir los nodos con `hash(id) mod 4`? ¿Por qué es difícil distribuir una base de grafos?

El porcentaje de aristas que quedó entre máquinas distintas al repartir los nodos con `hash(id) mod 4` fué de `74,4%`.

Es difícil de distribuir porque el extremo de las aristas queda en distintas máquinas, lo que luego dificulta su recorrido e inevitablemente lleva a recorrer la red para obtener el nodo vecino.

### Vectorial (`05_vectorial`)

16. ¿Qué representa el vector de cada cliente y qué mide la similitud coseno?

El vector de cada cliente resume su comportamiento de la siguiente manera:
- Proporción del gasto en cada categoría (`electronics`, `home`, `books`, `sports`, `fashion`).
- Proporción de pagos por canal (`card`, `wallet`, `transfer`).
- Ticket promedio (dividido por `2.000`) y proporción de compras de más de `1.500`.

La similitud coseno mide qué tan parecida es la dirección de dos vectores. El resultado va de 0 a 1, donde 1 significa la misma dirección.

17.  ¿Qué porcentaje de clientes marcados hay entre los vecinos de clientes marcados y cuál es la tasa base? ¿Para qué sirve buscar "vecinos parecidos"?

El porcentaje de vecinos marcados que hay entre los vecinos de clientes marcados es de `31.6%` y el de la tasa base es de `14.8%`.

Buscar "vecinos parecidos" sirve para encontrar clientes con comportamientos de consumo similares (categorías, canales, ticket promedio y compras caras).

18. En la curva del índice IVF, ¿qué recall y qué porcentaje de vectores escaneados se obtienen con `nprobe=4`? ¿Qué se gana y qué se pierde con la búsqueda aproximada (ANN)?

Con `nprobe=4` el recall que se obtiene es de `0.872` y el porcentaje de vectores escaneados es de `5.985%`.

Al utilizar una búsqueda aproximada (ANN) la ventaja es poder agrupar los vectores en `n_lists` para poder compararlos, porque es muy costoso comparar todos los vectores contra todos. Al comparar, solamente se comparan los nprobe clusters más cercanos a la consulta.

Por lo tanto, escanea menos pero la desventaja es que puede quedar algún vecino afuera. El trade-off es **recall vs costo**.

### Columnar (`06_columnar`)

19. ¿Cuánto ocupan los datos en CSV, JSON y Parquet? ¿Qué columnas leyó Spark para la consulta de `amount` sobre Parquet (`ReadSchema`)? ¿Cuántos archivos se pueden saltear con los datos ordenados?

El tamaño que ocupan en disco expresados en `MB`, los distintos formatos es:
- Parquet: `5.3`
- JSON: `121.1`
- CSV: `50.0`

Spark para la consulta de `amount` sobre Parquet, solamente leyó la columna de `amount`, no hizo falta leer todas las columnas ni siquiera separar por filas.

Con los datos ordenados se pueden saltear 7 de 8 archivos.

20. ¿Por qué el formato columnar es bueno para analítica y poco conveniente para modificar una fila por vez?

El formato columnar es bueno para la analítica porque guarda juntos los valores de cada columna. Solamente se leen la/las columna/s de la consulta, y como en Parquet están en un bloque: no se leen las demás columnas.

Es poco conveniente para modificar una fila por vez porque los campos de esa fila viven en columnas distintas. Cambiar una transacción obliga a tocar cada una de esas columnas. En un formato por filas, los cuatro valores están juntos y se reescribe solo esa fila.
