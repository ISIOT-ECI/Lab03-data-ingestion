### Software Engineering for IoT and BigData

### Lab 3 - IoT devices communication

### Exercise to be done individually or by groups of two students.

### Required elements.

1. Lab-02 in working condition: ESP8266 transmitting data to the MQTT Broker.

### Lab Goal

In this exercise,instead of sending sensor data back and forth between edge-devices (the ESP8266) (M2M- Machine-to-Machine communication) like in Lab-02, you will be sending it to a back-end platform that, in turn, will process it. To do so, you will configure the data ingestion layer of the IoT reference-architecture that will be explored in the course using the Slack platform.

Although, in principle, you could just create a MQTT-Client running on a server, as a data-ingestion layer, in this exercise we are asumming that your service will eventually receive data not from a couple of edge devices, but from thousands of them, and so scalability and high performance is a must. 

Given that the protocol that work better on the IoT devices of previous exercises is MQTT, we will configure the data ingest layer so it will 'pull' the data posted by them into the MQTT Broker (CloudAMQP). This way, you won't have to do further changes -by now- on the devices firmware.

In this first iteration of the proof of concept of the architecture, the ingested data will be running locally in your development environment using Docker and Docker compose. Furthermore, the collected data will be processed by a basic Slack consumer. In this, and the following laboratories, you will 




### Requirements

0. Update your ESP8266's code, so it post the readings of the sensors to the MQTT broker's topic every time you press the push button.

1. Install [Docker](https://docs.docker.com/get-docker/) and [Docker compose](https://docs.docker.com/compose/install/) in your development environment.

2. Docker-container file descriptor with the basic configuratin [Docker-compose](https://docs.docker.com/compose/) configuration that will handle three Docker containers:

	- Kafka: the streaming processing software platform (a.k.a. our data ingestion layer).
	- Kafka connect: the platform that allows the integration with third-party brokers/data sources (e.g. ActiveMQ, JDBC, **MQTT**, etc).
	- Zookeeper: a configuration service required by Kafka.


	```
	docker-compose up
	```

3. Check it works correctly. [Check that three containers were created by running](https://docs.docker.com/engine/reference/commandline/ps/)


5. Check [Kafka-connect REST API documentation](https://docs.confluent.io/current/connect/references/restapi.html). Check in the Docker-compose file in which port this service is listening to. As you can see in the section 'port' of the container configuration, this port is mapped to the same port in the host machine. Hence, you can use

'ports:
      - "8083:8083"

6. Register the following connector [MqttSourceConnector](https://docs.confluent.io/current/connect/kafka-connect-mqtt/mqtt-source-connector/index.html). Use a HTTP client or the *curl* commant to POST. Verifique que el conector haya quedado configurado consultando nuevamente 

	```json
	{
    "name": "cloud-mqtt-source",
    "config": {
        "connector.class": "io.confluent.connect.mqtt.MqttSourceConnector",
        "tasks.max": 1,
        "mqtt.server.uri": "tcp://YOUR-MQTT-BROKER-ADDRESS:1883",
        "mqtt.password" : "YOUR-MQTT-BROKER-PASSWORD",
        "mqtt.username": "USER-NAME",
        "mqtt.topics": "YOUR-MQTT-TOPIC-NAME",
        "kafka.topic": "YOUR-KAFKA-TOPIC-NAME",
        "value.converter": "org.apache.kafka.connect.converters.ByteArrayConverter",
        "confluent.topic.bootstrap.servers": "kafka:9092",
        "confluent.topic.replication.factor": 1
	    }
	}
	```

7. Now you must crate a Kafka topic, with the same name used for '"kafka.topic"' when registering the MQTT connector (previous step). To do so, this command must be executed from the container running the Docker service (not from the host machine), replacing *ZOOKEEPER-HOST* with the address assigned by Docker's networking system to the container running Zookeeper:

```
kafka-topics --create --zookeeper ZOOKEEPER-HOST:2181 --replication-factor 1 --partitions 1 --topic feeds.data
```

Check [how to list the running containers and their identifiers](https://docs.docker.com/engine/reference/commandline/ps/), and how to [run a command in a container](https://docs.docker.com/engine/reference/commandline/exec/). After doing so, to create the topic you can either (a) execute the command 'bash' with *docker exec* to open a shell in the container and run the command from there, or (b) execute the command directly with *docker exec*. Regarding Zookeeper's address, you can use the value given in the *hostname* attribute, in the docker-compose configuration file (only within the containers, these hostnames wouldn't be solved by the host).


8. To verify the configuration:
	-  Run a *kafka-console-consumer* (a simple kafka client that shows in console all the data posted in a given Docker topic) from the Kafka container
	```
	kafka-console-consumer --topic THE-KAFKA-TOPIC --from-beginning --bootstrap-server localhost:9092
	```
	- Publish some data on the MQTT broker, and check that the kafka-console shows it.

9. Once you are sure that Docker was correctly configured, write your own Kafka client in Java. Use as a reference the [examples provided by *confluent*](https://docs.confluent.io/current/tutorials/examples/clients/docs/java.html#client-examples-java), in particular the [sample client](https://github.com/confluentinc/examples/blob/6.0.0-post/clients/cloud/java/src/main/java/io/confluent/examples/clients/cloud/ConsumerExample.java). 

10. Test the application by running it in a shell in Kafka's container. 

To do so, create a runnable jar and execute it from the Docker's container.

10. 


https://docs.docker.com/compose/



the easiest way is to run it in the Kafka container (it has a jvm installed)


Rather than getting the IP address required by the above command, take a look at the 'hostname' section in the Docker-compose configuration file. 


To know the address of the 

hostname: kafka



You can either (a) execute

You can execute commands on a container with the *docker execute* command](https://docs.docker.com/engine/reference/commandline/exec/), so 

You can do it either by 
8. running a linux shell on the container, and then executing the command, or e







See the docker-connect configuration, each machine has a domain name 

section hostname: kafka in the docker-compose , has the address of the other containers.

The command to create a new topic. It should be executed 




https://github.com/kaiwaehner/kafka-connect-iot-mqtt-connector-example/blob/master/live-demo-kafka-connect-iot-mqtt-connector.adoc


### Esta guía no nos sirve (no usa docker), pero fue donde descubrí que faltaba el paso de hacer el tópico en kafka
https://github.com/kaiwaehner/kafka-connect-iot-mqtt-connector-example/blob/master/live-demo-kafka-connect-iot-mqtt-connector.adoc


### Se debe crear el topico en kafka despues de registrar el conector en kafka-connect. Esto se hace a traves de zookeeper.Como se hace desde la vm de kafka-connect, en vez de localhost se usa la direccion IP de la vm de zookeeper (en mi caso era 172.20.0.2)
kafka-topics --create --zookeeper 172.20.0.2:2181 --replication-factor 1 --partitions 1 --topic feeds.data


### Para obtener la dirección IP
https://www.freecodecamp.org/news/how-to-get-a-docker-container-ip-address-explained-with-examples/

ip -4 -o address

docker exec -it 735962a36c39 ip -4 -o address

### Prueba para ejecutarse dentro del contenedor de Kafka
kafka-console-consumer --topic feeds.data --from-beginning --bootstrap-server localhost:9092



docker exec -it f5147c9ff4e9 kafka-console-consumer --topic feeds.data --from-beginning --bootstrap-server localhost:9092


Use the [sample client code](https://github.com/smallnest/kafka-example-in-scala/blob/master/src/main/java/com/colobu/kafka/ConsumerExample.java)


mvn -f ./java-app/pom.xml  exec:java -Dexec.mainClass="edu.eci.isiot.kafka.simple.cli.SimpleKafkaClient"

Containerize your app


Minimal Consumer [app](https://kafka.apache.org/26/javadoc/index.html?org/apache/kafka/clients/consumer/KafkaConsumer.html)





docker exec -it f5147c9ff4e9 kafka-console-producer --topic thetopic --bootstrap-server localhost:9092

kafka-console-producer --topic example-topic --broker-list broker:9092


[Code examples of Kafka client](https://docs.confluent.io/current/tutorials/examples/clients/docs/java.html#client-examples-java)





Use the sample [consumer from confluent](https://docs.confluent.io/current/tutorials/examples/clients/docs/java.html#client-examples-java)