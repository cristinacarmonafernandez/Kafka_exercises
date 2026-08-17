# Notas - Fundamentos de Apache Kafka

## 1. Entorno

Para este ejercicio se ha utilizado un clúster Kafka de un único nodo ejecutándose mediante Docker.

Kafka utiliza KRaft para la gestión de metadatos, con el broker y el controller ejecutándose en el mismo nodo.

### Versiones utilizadas

- Confluent Kafka: 7.8.9
- Docker
- Docker Compose
- KRaft

---

## 2. Gestión de Topics

### Listar topics

Para consultar los topics existentes:

```powershell
docker exec kafka kafka-topics --bootstrap-server kafka:29092 --list
```

### Listar topics

Se creó el topic vehicle-positions con:

6 particiones
Factor de replicación: 1

```powershell
docker exec kafka kafka-topics --bootstrap-server kafka:29092 --create --topic vehicle-positions --partitions 6 --replication-factor 1
```

### Describir un topic

Para consultar la confoguración y los metadatos del topic:

```powershell
docker exec kafka kafka-topics --bootstrap-server kafka:29092 --describe --topic vehicle-positions
```

Este comando permite consultar:

Número de particiones
Leader de cada partición
Réplicas
ISR (In-Sync Replicas)
Factor de replicación


### Crear y eliminar un topic

También se creó test-topic con 3 particiones y posteriormente se eliminó.

```powershell
docker exec kafka kafka-topics --bootstrap-server kafka:29092 --create --topic test-topic --partitions 3 --replication-factor 1
```

Para eliminarlo 
```powershell
docker exec kafka kafka-topics --bootstrap-server kafka:29092 --delete --topic test-topic
```

## 3. Producción y consumo de mensajes
Se creó el topic sample-topic con 3 particiones y un factor de replicación de 1.

```powershell
docker exec kafka kafka-topics --bootstrap-server kafka:29092 --create --topic sample-topic --patitions 3 --replication-factor 1
```

### Producer
Se utilizó kafka-console-producer para producir mensajes:

```powershell
docker exec -it kafka kafka-console-producer --bootstrap-server kafka:29092 --topic sample-topic
```

Se introdujeron mensajes como:
hello
world
Kafka
is
cool!!!!!
Consumer

Para consumir los mensajes desde el principio del topic:

```powershell
docker exec kafka kafka-console-consumer --bootstrap-server kafka:29092 --topic sample-topic --fro
```

## 4. Experimento sobre el orden de los mensajes

Para investigar en qué partición se almacenaban los mensajes, se utilizó:

```powershell
docker exec kafka kafka-console-consumer --bootstrap-server kafka:29092 --topic sample-topic --from-beginning --property print.partition=true --property print.offset=true
```

El resultado mostró que todos los mensajes producidos sin key estaban almacenados en la misma partición:
 | Partition | Messages | 
 |-----------|----------| 
 |  0 | 0 | 
 | 1 | 5 | 
 | 2 | 0 | 
 
Por ejemplo:

Partition:1    Offset:0    hello
Partition:1    Offset:1    world
Partition:1    Offset:2    Kafka
Partition:1    Offset:3    is
Partition:1    Offset:4    cool!!!!!

### Observación

Aunque el topic tenía tres particiones, los mensajes no se distribuyeron necesariamente entre las tres. En este experimento todos los mensajes acabaron en la partición 1.
Como todos los mensajes estaban en la misma partición, el consumer los mostró en el mismo orden en el que fueron producidos.

### Conclusión

Kafka garantiza el orden de los mensajes dentro de una misma partición, pero no garantiza un orden global entre diferentes particiones.
Tener varias particiones no significa que cada mensaje vaya a una partición diferente.

## 5. Producción de mensajes con keys

Se utilizó el producer configurado para aceptar una key y un valor separados por una coma:

```powershell
docker exec -it kafka kafka-console-producer --bootstrap-server kafka:29092 --topic sample-topic --property parse.key=true --property key.separator=,
```

Los mensajes se introdujeron utilizando el formato:
key,value

Por ejemplo:
1,apples
2,pears
3,walnuts
4,peanuts
5,oranges

La estructura de cada mensaje es:
Key   = 1
Value = apples

### Consumir mostrando las keys

Para visualizar las keys:
```powershell
docker exec kafka kafka-console-consumer --bootstrap-server kafka:29092 --topic sample-topic --from-beginning --property print.key=true
```

Los mensajes producidos originalmente sin key aparecen con: null mientras que los mensajes producidos con key muestran la clave correspondiente.

### Concepto aprendido

Las keys pueden utilizarse para influir en la partición a la que se envía un mensaje. Esto es especialmente importante cuando queremos mantener relacionados los mensajes de una misma entidad. Algunos ejemplos de posibles keys en un sistema real serían:

customer_id
order_id
device_id
account_id

Utilizar una key adecuada puede ayudar a mantener los eventos relacionados dentro de la misma partición y, por tanto, conservar su orden dentro de ella.

## 6. Particiones y offsets

Para observar conjuntamente las keys, particiones y offsets se utilizó:

```powershell
docker exec kafka kafka-console-consumer --bootstrap-server kafka:29092 --topic sample-topic --from-beginning --property print.key=true --property print.partition=true --property print.offset=true
```

Esto permite observar que cada mensaje tiene:

Una key, que puede ser null.
Una partición.
Un offset.
Un valor.

El offset identifica la posición del mensaje dentro de su partición. Por tanto, los offsets son independientes entre particiones. No existe un único contador global para todos los mensajes de un topic.

## 7. KRaft Metadata Shell

El laboratorio original utiliza el siguiente comando:

```powershell
kafka-metadata-shell --cluster-id ... --controllers ...
```

Sin embargo, esta sintaxis no es compatible con la versión de kafka-metadata-shell incluida en nuestro entorno Kafka 7.8.9. Para comprobar las opciones disponibles se utilizó:

```powershell
docker exec kafka kafka-metadata-shell --help
```

La herramienta disponible acepta:

--snapshot

en lugar de los parámetros utilizados en el laboratorio original.

### Investigación del snapshot

Se investigó el sistema de archivos del contenedor para localizar un snapshot de metadatos KRaft:

```powershell
docker exec kafka find / -name "*.checkpoint" -o -name "*.snapshot" 2>$null
```

El único archivo encontrado fue:

/var/lib/kafka/data/bootstrap.checkpoint

No se encontró un snapshot de metadatos KRaft que pudiera utilizarse directamente con kafka-metadata-shell.

### Conclusión

Esta parte del laboratorio no pudo reproducirse exactamente debido a la diferencia entre el entorno original del curso y la versión de Kafka utilizada en este proyecto.

Como alternativa, se utilizaron las herramientas de administración de Kafka para inspeccionar la información disponible sobre el clúster, especialmente:

```powershell
docker exec kafka kafka-topics --bootstrap-server kafka:29092 --describe --topic sample-topic
```

## 8. Troubleshooting

Durante el ejercicio se encontraron algunas diferencias entre el entorno original del laboratorio y el entorno creado para este proyecto. La principal diferencia fue la sintaxis disponible para kafka-metadata-shell. Para comprobar la versión de Kafka se utilizó:
 ```powershell
docker exec kafka kafka-topics --version
```

Resultado:

7.8.9-ccs

Esto permitió identificar que estábamos trabajando con una versión diferente del entorno utilizado originalmente en el laboratorio.

### Lección aprendida

Los comandos y herramientas de Kafka pueden cambiar entre versiones.
Cuando un comando de un laboratorio o documentación no funciona, es importante:

Comprobar la versión instalada.
Consultar la ayuda de la herramienta.
Comprobar las opciones disponibles.
Adaptar el procedimiento al entorno utilizado.
Documentar la diferencia.

## 9. Conceptos principales aprendidos

Durante este ejercicio he trabajado con los siguientes conceptos:

Topics
Particiones
Replicación
Leaders
ISR (In-Sync Replicas)
Producers
Consumers
Message keys
Offsets
Orden de mensajes
KRaft
Metadatos de Kafka
Administración de topics
Troubleshooting de herramientas Kafka
Principales conclusiones
Un topic puede dividirse en múltiples particiones.
Cada partición mantiene el orden de sus propios mensajes.
Kafka no garantiza un orden global entre diferentes particiones.
Los offsets indican la posición de los mensajes dentro de una partición.
Las keys pueden influir en la asignación de los mensajes a las particiones.
Los consumers pueden mostrar información adicional como keys, particiones y offsets.
El factor de replicación determina el número de réplicas de una partición.
ISR significa In-Sync Replicas.
KRaft gestiona los metadatos del clúster.
Las herramientas y comandos pueden variar entre versiones de Kafka.