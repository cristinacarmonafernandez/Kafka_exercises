# Notas - Exploring Apache Kafka

## 1. Objetivo

En este ejercicio se ha explorado un clúster Kafka sencillo formado por un único broker y controller.

El entorno se ha ejecutado mediante Docker y se ha utilizado un producer para enviar datos de posiciones de vehículos a Kafka.

Posteriormente, se ha utilizado un consumer para leer los mensajes almacenados en un topic.

El objetivo principal ha sido familiarizarme con el funcionamiento básico de Kafka y con el flujo:

```text
Fuente de datos
      ↓
   Producer
      ↓
    Kafka
      ↓
     Topic
      ↓
   Consumer
```
## 2. Entorno

Para realizar el ejercicio se ha utilizado:

Apache Kafka / Confluent Platform 7.8.9
Docker
Docker Compose
Kafka en modo KRaft
PowerShell
Visual Studio Code

El clúster utilizado está compuesto por un único nodo que actúa como broker y controller.
Esta configuración es adecuada para realizar el ejercicio y aprender los conceptos básicos, pero no es una configuración recomendada para un entorno de producción.

## 3. Levantar el clúster

El entorno Kafka se levanta mediante Docker Compose:

```poweshell
docker compose up -d
```

Para comprobar que el contenedor está funcionando:

```poweshell
docker ps
```

El contenedor Kafka debe aparecer con estado Up.

## 4. Topic vehicle-positions

El ejercicio utiliza un topic llamado:

vehicle-positions

El topic tiene:

6 particiones
Factor de replicación: 1

Para comprobar la información del topic se utiliza:

```poweshell
docker exec kafka kafka-topics --bootstrap-server kafka:29092 --describe --topic vehicle-positions
```

La información proporcionada permite observar:

Número de particiones.
Factor de replicación.
Leader de cada partición.
Réplicas.
ISR (In-Sync Replicas).

## 5. Producer

El producer utilizado en el ejercicio genera datos de posiciones de vehículos y los envía al topic vehicle-positions. Los datos representan posiciones de vehículos de transporte público y contienen información como:

Identificador del vehículo.
Línea.
Velocidad.
Dirección.
Latitud.
Longitud.
Timestamp.
Ruta.
Estado del vehículo.

Un ejemplo simplificado de un mensaje es:

{
  "VP": {
    "desi": "65",
    "veh": 268,
    "spd": 6.33,
    "hdg": 353,
    "lat": 60.233573,
    "long": 24.978816
  }
}

Estos datos representan eventos que llegan continuamente al sistema.

## 6. Consumer

Para leer los mensajes del topic se utilizó kafka-console-consumer.

El consumer se conecta al broker Kafka y escucha los mensajes publicados en vehicle-positions.

Conceptualmente:

```text
Producer
   │
   │ mensajes
   ▼
Kafka Topic
vehicle-positions
   │
   │ mensajes
   ▼
Consumer
```

Los mensajes aparecen continuamente en la terminal porque el producer está generando datos en tiempo real.

Para detener el consumer:
```poweshell
Ctrl + C
```

## 7. Arquitectura observada

El flujo del ejercicio puede representarse de la siguiente manera:
```text
      Public IoT Data
            │
            ▼
        Producer
            │
            ▼
     Kafka Broker
            │
            ▼
   vehicle-positions
      6 partitions
            │
            ▼
         Consumer
            │
            ▼
     Vehicle events
```
Este flujo representa un ejemplo sencillo de una arquitectura de streaming de datos.

## 8. Conceptos aprendidos
### Broker

El broker es el servidor que recibe, almacena y sirve los mensajes de Kafka. En este ejercicio se utiliza un único broker.

### Controller

El controller participa en la gestión del estado y los metadatos del clúster Kafka. En este entorno el mismo nodo actúa como broker y controller.

### Topic

Un topic es una categoría o flujo lógico donde Kafka almacena los mensajes.

En este ejercicio:

vehicle-positions

es el topic utilizado para los eventos de posiciones de vehículos.

### Partition

Los topics pueden dividirse en varias particiones. El topic utilizado en este ejercicio tiene 6 particiones. Las particiones permiten distribuir los datos y son fundamentales para la escalabilidad de Kafka.

### Producer

El producer es el componente que publica mensajes en Kafka. En este ejercicio genera los eventos de posiciones de vehículos.

### Consumer

El consumer es el componente que lee los mensajes almacenados en Kafka. En este ejercicio se utiliza kafka-console-consumer.

## 9. MQTT e IoT

Los datos utilizados en el ejercicio proceden de una fuente de datos de transporte público que utiliza un sistema de publicación de eventos. MQTT es un protocolo ligero de mensajería basado en el modelo publish/subscribe.

Es habitual en aplicaciones IoT porque está diseñado para trabajar con dispositivos y redes con recursos limitados. En este ejercicio se observa cómo datos procedentes de una fuente IoT pueden terminar siendo procesados mediante Kafka.

## 10. Troubleshooting y adaptación del laboratorio

El laboratorio original estaba diseñado para ejecutarse en un entorno proporcionado por Confluent, donde determinados scripts y herramientas ya estaban configurados. Para este portfolio se decidió reproducir el entorno utilizando Docker Compose en local.

En lugar de depender directamente de:

start.sh
stop.sh

se creó un docker-compose.yml propio para levantar el broker Kafka. Esto permite que el ejercicio sea más reproducible y facilita entender qué componentes forman realmente el entorno.

El clúster se puede iniciar con:

```poweshell
docker compose up -d
```

y detener con:
```poweshell
docker compose down
```

## 11. Limpieza del entorno

Al finalizar el ejercicio, los contenedores se pueden detener mediante:

```poweshell
docker compose down
```

Esto detiene y elimina los recursos creados por Docker Compose.

## 12. Principales conclusiones

Este ejercicio permitió comprender el flujo básico de datos en Apache Kafka:
```text
Producer → Kafka Topic → Consumer
```

Los principales conceptos aprendidos fueron:

Kafka almacena eventos en topics.
Los topics están formados por particiones.
Los producers publican mensajes.
Los consumers leen mensajes.
Los brokers almacenan y sirven los datos.
Los controllers gestionan información del clúster.
Kafka puede utilizarse para procesar datos en tiempo real.
Docker permite crear un entorno Kafka reproducible.
Kafka resulta especialmente útil para arquitecturas de streaming y sistemas orientados a eventos.

Este ejercicio constituye la base para los siguientes ejercicios del proyecto, donde se profundizará en topics, particiones, keys, offsets, producers y consumers.