# HighThroughputJavaApp
A java application capable of performing deduplication at high throughput when deployed in multi-instance behind load balancers 

### Requirements

1. Java-17
2. Maven-3.9.9
3. Docker
4. Redis server
5. Cassandra database
6. Kafka broker
7. Jmeter-5.6.3

### To Build the application
```bash
mvn clean package
```

### To run the application
```bash
java -jar dedupIds-0.0.1-SNAPSHOT
```

### Functionalities
The application accepts a HTTP Get request with Id as mandatory param and endpoint as a optinal parameter. 

For each minute the application uses `redis` as a distributed in-memory cache store. The incoming id's are pushed into a `set` and using schedule for each minute the unique number of id's are logged in a file and also published to kafka.

The application is designed to process at least `10,000 requests per second` and supports various features, including HTTP request deduplication and logging.
