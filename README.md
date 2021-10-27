# WSO2 AsyncAPI Demo Artifacts

This repo contains resources that are needed to demo AsyncAPI with WSO2 API Manager 4.0.0 and WSO2 Streaming Integrator 4.0.0.

## Prerequisite

- Java 8 or above
- Docker and Docker compose (Allocate a minimum of 4 CPU cores and 4GB Memory for Docker resources)
- WSO2 Streaming Integrator Tooling (Download link: https://wso2.com/integration/streaming-integrator)

## Setup

run following command to start the environment

```
docker-compose up
```

This will start WSO2 API Manager 4.0.0, WSO2 SI 4.0.0, Zookeeper and Kafka.

Once you are done with the demo, use

```
docker-compose down
```

## Demo

1. Extract WSO2 Streaming Integrator Tooling to a preferred location (referred as `<SI_TOOLING_HOME>`).
2. Start Streaming Integrator Tooling by executing the following command from `<SI_TOOLING_HOME>/bin` directory.
    - For Window: `tooling.bat`
    - For Linux/Mac: `tooling.sh`

3. In the browser, open Streaming Integrator Tooling by navigating to `http://localhost:9390/editor`.
4. Create a new Siddhi application and copy the following content there and save.

```
@App:name('PizzaTrackingApp')
@App:description('Pizza tracking Siddhi app.')

@App:asyncAPI("""asyncapi: 2.0.0
info:
  title: PIzzaTrackingAPI
  version: 1.0.0
  description: This exposes an API from WSO2 SI
servers:
  production:
    url: 'si-runtime:8830'
    protocol: ws
    security: []
channels:
  /:
    subscribe:
      message:
        $ref: '#/components/messages/WebSocketOutputStreamPayload'
components:
  messages:
    WebSocketOutputStreamPayload:
      payload:
        type: object
        properties:
          orderId:
            $ref: '#/components/schemas/orderId'
          status:
            $ref: '#/components/schemas/status'
  schemas:
    orderId:
      type: integer
    status:
      type: string
  securitySchemes: {}
""")

@source(type='kafka',bootstrap.servers = "kafka:9092",topic.list = "pizza-tracks",group.id = "trackingConsumers",threading.option = "single.thread",
	@map(type='json'))
define stream KafkaInputStream (orderId int,status string);


@sink(type='websocket-server',host = "si-runtime",port = "8830",
	@map(type='json'))
define stream WebSocketOutputStream (orderId int,status string);


@info(name='query1')
from KafkaInputStream
select *
insert  into WebSocketOutputStream;

```

### Deploy Siddhi application in WSO2 Streaming Integrator Runtime

1. In the Tooling, go to "Deploy" from the menu, then "Deploy To Server".
2. From the "Deploy Siddhi Apps To Server" window, add new server with following details.

```
Host: localhost
HTTPS Port: 9448
User Name: admin
Password: admin
```

3. Select the above Siddhi app from the "Siddhi Apps To Deploy" section and the newly added server from "Servers" section.
4. Click "Deploy".
5. This will deploy the Siddhi app to Streaming Integrator Runtime and also publish to the Service catalog in WSO2 API Manager.

### Publish API

1. Login to the Publisher Portal (http://localhost:9443/publisher)
2. Go to the Service catalog and create AsyncAPI from there with the published artifact.
3. Deploy and invoke the API.

### Testing the API

1. Once the API is deployed and subscribed, copy the `wscat` command from the "Try Out" section in the API Manager Developer Portal.
2. Open a terminal window and execute the copied `wscat` command.
3. In another terminal window, login to the Kafka docker image by executing the following command. Replace the <DOCKER_KAFKA_CONTAINER_ID> with the correct container ID of the Kafka image.

    ```
    docker exec -it <DOCKER_KAFKA_CONTAINER_ID> /bin/bash
    ```
4. From the container command line, execute the following command to publish a message to the Kafka topic called `pizza-tracks`.

```
kafka-console-producer.sh --topic pizza-tracks --bootstrap-server localhost:9092
> {"event": {"orderId": 1,"status": "pending"}}
```

5. Once the message is published, the same content will be appeared from the above `wscat` terminal.
