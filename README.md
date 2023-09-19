# Google Cloud Pub/Sub Group Kafka Connector - Emulator Fix


According to Google Cloud's documentation, when connecting to an Pub/Sub emulator, in all languages except for Java and C#, if you have set `PUBSUB_EMULATOR_HOST` as an environment variable, the Pub/Sub client library will automatically call the API running locally instead of using Pub/Sub. 

However, C# and Java (including Kafka-Connect connectors, which is this use case) do require you to modify your code to use the emulator.

This repository has the CloudPubSubSinkTask.java modified by the following changes:

* The credentials module is removed, as it is not used in the emulator, and `NoCredentialsProvider.create()` is used as drop-in replacement.

* A new ManagedChannel and TransportChannelProvider is used instead of the default channel, which will not work using the emulator due to HTTPS and HTTP/2 requirements from Pub/Sub that the emulator does not comply with. This channels require a different way to parametrize the emulator endpoint.

> `cps.endpoint`, when the connector is properly configured, should have the IP address
>  of the emulator (`127.0.0.1:5398`, or any other arbitrary hostname and port,
> `pubsub-emulator:5398`, if the emulator is not running in the same machine as the connector). 


> However, you could also hard-code the hostport directly in the connector
> (`String hostport=127.0.0.1:5398`), or read from an environment variable
> (`String hostport=System.getenv("PUBSUB_EMULATOR_HOST")`).


> I will take the first approach as it is the most flexible one.


### Build the connector

These instructions assume you are using [Maven](https://maven.apache.org/).

1. Clone this repository:
   ```shell
   > git clone https://github.com/elbisbe/java-pubsub-group-kafka-connector-emulator
   ```

2. Package the connector jar:
   ```shell
   > mvn clean package -DskipTests=True
   ```
   You should see the resulting jar at `target/pubsub-group-kafka-connector-${VERSION}-SNAPSHOT.jar`
   on success.