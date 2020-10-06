### Software Engineering for IoT and BigData

### Lab 3 - IoT devices communication

### Exercise to be done individually or by groups of two students.

### Required elements.

1. Lab-02 in working condition: ESP8266 transmitting data to the MQTT Broker.

### Lab Goal

The goal of this exercise is to send the sensor readings of the IoT devices to a back-end service for their processing, unlike Lab-02, which was focused on sending sensor data back and forth between edge-devices (the ESP8266), that is to say, following a M2M- Machine-to-Machine communication approach. To do so, you will configure the data ingestion layer of the IoT reference-architecture explored in the course using the Slack platform. Although, in principle, you could just create a MQTT-Client running on a server, as a data-ingestion layer, in this exercise we are asumming that your service will eventually receive data not from a couple of edge devices, but from thousands of them, and so scalability and high performance is a must. 

Given that MQTT is the protocol that works better on the devices used on previous exercises, we will configure the data ingest layer so it will 'pull' the data from the MQTT broker (CloudAMQP) used before. In this first iteration on the proof of concept of the architecture, the data ingestion platform will be running locally in your development environment using Docker and Docker compose. Furthermore, the collected data will be processed by a basic Slack consumer. 

### Steps

0. Update your ESP8266's code, so it post the readings of the sensors to the MQTT broker's topic with a given frequency, or (even better) every time you press the push button.

1. Install [Docker](https://docs.docker.com/get-docker/) and [Docker compose](https://docs.docker.com/compose/install/) in your development environment.

2. The file docker-compose.yml included in this repository contains the minimum configuratin to run the three containers required in this exercise, and integrating them using [Docker-compose](https://docs.docker.com/compose/). The three containers are:
	- Kafka: the streaming processing software platform (a.k.a. our data ingestion layer).
	- Kafka connect: the platform that allows the integration with third-party brokers/data sources (e.g. ActiveMQ, JDBC, **MQTT**, etc).
	- Zookeeper: a configuration service required by Kafka.

	Through the following exercises an laboratories, we will be exploring some details of Docker and Docker-compose. By now, let's fire the three containers by running:

	```bash
	docker-compose up
	```

3. Check that [all the three containers are 'up and running'](https://docs.docker.com/engine/reference/commandline/ps/). 

4. Kafka-connect works in a service-oriented fashion. Hence, its configuration is [made through a REST API](https://docs.confluent.io/current/connect/references/restapi.html). Check in the Docker-compose file in which port this service is listening to. As you can see in the section 'port' of the container configuration, this port is mapped to the same port in the host machine. Hence, you can use any REST client (curl, Postman, etc) and send requests from the host (your actual computer, not the containers). Use the API to *GET* the connectors registered in the platform.
If the [MqttSourceConnector](https://docs.confluent.io/current/connect/kafka-connect-mqtt/mqtt-source-connector/index.html) is still not registered, you should do it by sending a POST request with the folowing body (remember to include in the request header that the *Content-Type* of the request is 'application/json'. Remember to update the fields based on the configuration of the MQTT broker used on Lab-02. In "kafka.topic", choose a name only with letters, . (dot), and _ (underscore).

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

7. Now you must crate a topic in the kafka server, that correspond to the one selected in the previous step, in the '"kafka.topic"' parameter. To do so, this command must be executed from the container running the Docker service (not from the host machine), replacing *ZOOKEEPER-HOST* with the address assigned by Docker's networking system to the container running Zookeeper:

	```
	kafka-topics --create --zookeeper ZOOKEEPER-HOST:2181 --replication-factor 1 --partitions 1 --topic feeds.data
	```

	Check [how to list the running containers and their identifiers](https://docs.docker.com/engine/reference/commandline/ps/), and how to [run a command in a container](https://docs.docker.com/engine/reference/commandline/exec/). After doing so, to create the topic you can either (a) execute the command 'bash' with *docker exec* to open a shell in the container and run the command from there, or (b) execute the command directly with *docker exec*. Regarding Zookeeper's address, you can use the value given in the *hostname* attribute, in the docker-compose configuration file (only within the containers, these hostnames wouldn't be solved by the host).


8. To verify the configuration:
	-  Run a *kafka-console-consumer* (a simple kafka client that shows in console all the data posted in a given Docker topic) in the Kafka container.
	```
	kafka-console-consumer --topic THE-KAFKA-TOPIC --from-beginning --bootstrap-server localhost:9092
	```
	- Publish some data on the MQTT broker (e.g. through your ESP8266 device), and check that the kafka-console shows it.


9. Once you are sure that Docker was correctly configured, write your own Kafka client in Java. Use as a reference the [examples provided by *confluent*](https://docs.confluent.io/current/tutorials/examples/clients/docs/java.html#client-examples-java), in particular the [sample client](https://github.com/confluentinc/examples/blob/6.0.0-post/clients/cloud/java/src/main/java/io/confluent/examples/clients/cloud/ConsumerExample.java). Write a program that continuously calculates and prints the average value of the sensor readings received. Create a self-contained runnable jar for your application, so it can be easily executed from one of the containers.


10. Test the application by running it in the Kafka's container shell. To let the container access your runnable jar, add an entry into the 'volumes' section of the 'kafka' service in the docker-compose.yml. For example, with the following configuration, if your executable jar is in the folder 'client', it will be available for the container in the folder /app. Remember to restart 

	```yml
	...
	volumes:
	      - ./kafka/data:/var/lib/kafka/data
	      - ./client:/app
	...
	```

