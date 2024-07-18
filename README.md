This is a sample application where I use a Kafka messaging service to emulate a message queue.
The message queue is populated by a producer (in this case the cab-book-driver-app) with location information. This location information is random generated numbers.

The messge queue is then subscribed by a consumer(in this case a cab-book-user). Then I just print the location co-ordinates subscribed to by the consumer.


Important commands - (For Windows Setup)


The Kafka Ecosystem has 2 main components , The Kafka Cluster and a Zookeeper. The zookeeper manages the kafka cluster and its metadata.

1) We need to get the zookeeper up by saying
  bin\windows\zookeeper-server-start.bat config\zookeeper.properties ( This command starts the zookeeper )

2) We then need to get the kafka server up
    bin\windows\kafka-server-start.bat config\server.properties
   
These 2 commnds get out kafka system up and running. To test this out we can create out own producer and consumer to see if the messages are being put on queue and being subscribed to
Producer -> bin\windows\kafka-console-producer.bat --topic first-event --bootstrap-server localhost:9092
Consumer -> bin\windows\kafka-console-consumer.bat --topic first-event --from-beginning --bootstrap-server localhost:9092

3) After testing to see if the messages are flowing properly we can use it in out spring application.

4) Create 2 apps one for producer and one for consumer
   
application.properties for producer

spring.kafka.producer.bootstrap-server=localhost:9092
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
server.port=8082

application.properties for consumer

spring.kafka.consumer.bootstrap-server=localhost:9092
spring.kafka.consumer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.consumer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.consumer.group-id=user-group
server.port=8081
spring.kafka.consumer.auto-offset-reset=earliest

We then Need to create a service in the producer that can place the messages on the message queue. We use the KafkaTemplate package provided by spring boot.

private KafkaTemplate<String, Object> kafkaTemplate;
kafkaTemplate.send(AppConstant.CAB_LOCATION, location);

Once put in we need to subscribe. In our consumer application
@KafkaListener(topics = "cab-location", groupId = "user-group") // thistopic is the same as the AppConstant.CAB_LOCATION (this denotes the same topic)
    public void cabLocation(String location) {
        System.out.println("location is" +location);
    }

Run the service in postman and you can test to see if the events are being consumed using 
bin/windows/kafka-console-consumer.bat --topic cab-location --from-beginning --bootstrap-server localhost:9092 (here topic is cab-location since that's the topic we use in our producer and consumer)
