# kafka-example

## Installation

1. **Clone the Repository:**
   ```bash
   git clone https://github.com/itmwiw/kafka-example
   cd kafka-example/
   ```

2. **Install Strimzi Kafka Operator with ArgoCD:**
   ```bash
   oc apply -f argocd-apps/strimzi-kafka-operator.yaml
   ```

3. **Deploy Kafka Resources on OpenShift:**
   Once the operator is installed, deploy the Kafka resources:
   ```bash
   oc apply -f argocd-apps/kafka-resources.yaml
   ```
   Alternatively, you can directly deploy the resources in the `kafka-resources` folder using:
   ```bash
   oc create ns kafka-example
   oc -n kafka-example apply -f kafka-resources/
   ```

---

## Demo

### Access Kafka Tools Pod
Open a bash session on the `kafka-tools` deployment:
```bash
oc -n kafka-example exec -it deploy/kafka-tools -- bash
```

### Basic Kafka Commands

1. **List Topics:**
   ```bash
   ./bin/kafka-topics.sh --bootstrap-server my-simple-cluster-kafka-bootstrap:9092 --list
   ```

2. **Send a Message:**
   ```bash
   ./bin/kafka-console-producer.sh --bootstrap-server my-simple-cluster-kafka-bootstrap:9092 --topic my-topic
   ```

3. **Consume Messages:**
   ```bash
   ./bin/kafka-console-consumer.sh --bootstrap-server my-simple-cluster-kafka-bootstrap:9092 --topic my-topic --from-beginning
   ```

### Real-Time Message Consumption

To simulate real-time messaging, open two terminal sessions:

- **Terminal 1 (Message Consumer):**
  ```bash
  oc -n kafka-example exec -it deploy/kafka-tools -- bash
  ./bin/kafka-console-consumer.sh --bootstrap-server my-simple-cluster-kafka-bootstrap:9092 --topic my-topic --from-beginning
  ```

- **Terminal 2 (Message Producer):**
  ```bash
  oc -n kafka-example exec -it deploy/kafka-tools -- bash
  ./bin/kafka-console-producer.sh --bootstrap-server my-simple-cluster-kafka-bootstrap:9092 --topic my-topic
  ```

Type messages in **Terminal 2** to send them to Kafka. Messages will appear in **Terminal 1**, allowing you to observe real-time message streaming, similar to a chat interface.

### Authentication and authorization

Up until now we used 'my-kafka-cluster' resources along with the plain listener without any authentication nor authorization. It is possible to restrict access to Kafka though. In this example we'll use the 'my-oauth-kafka-cluster' resources which implent both authentiation and authorization. 

As a prerequisite, please import the kafka-authz-realm.json to your keaycloak instance.
This will configure the authorization server in keycloak as well as give 'Dev Team A' and 'Dev Team B' some permissions.

Open a bash session on the `kafka-tools` deployment:
```bash
oc -n kafka-example exec -it deploy/kafka-tools -- bash
```

- **Producer:**


### Configuring Authentication and Authorization with OAuth in Kafka

Up until now, we have been using the `my-simple-cluster` resources with the plain listener, without any authentication or authorization. However, it is possible to restrict access to Kafka. In this example, we will use the `my-oauth-cluster` resources, which implement both authentication and authorization.

#### Prerequisites

Before proceeding, import the `kafka-authz-realm.json` file into your Keycloak instance. This will configure the authorization server in Keycloak and grant appropriate permissions to 'Dev Team A' and 'Dev Team B'.

#### Client configuration

Open a bash session in the `kafka-tools` deployment:

```bash
oc -n kafka-example exec -it deploy/kafka-tools -- bash
```

Create a properties file in the `/tmp` folder for the producer and consumer configuration:

```bash
cat > /tmp/team-a-client.properties << EOF
security.protocol=SASL_PLAINTEXT
sasl.mechanism=OAUTHBEARER
sasl.jaas.config=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required \
  oauth.client.id="team-a-client" \
  oauth.client.secret="team-a-client-secret" \
  oauth.token.endpoint.uri="http://keycloak.apps.okd.advatys.com/realms/kafka-authz/protocol/openid-connect/token" ;
sasl.login.callback.handler.class=io.strimzi.kafka.oauth.client.JaasClientOauthLoginCallbackHandler
EOF
```

#### Kafka Producer


Next, let's try sending a message to the Kafka topic `my-topic` using the `team-a-client`:

```bash
./bin/kafka-console-producer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic my-topic --producer.config=/tmp/team-a-client.properties
```

Type the message `Hello World` and hit Enter. You should encounter an error similar to:

```
Not authorized to access topics: [my-topic].
```
In fact, 'team-a-client' can only push to topics that start with ‘a_’.

**Adjust the Topic Name:**

Since 'Dev Team A' is only authorized to produce messages to topics starting with `a_`, let's change the topic name and try again:

```bash
./bin/kafka-console-producer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic a_messages --producer.config=/tmp/team-a-client.properties
```

You can now successfully send the message, and you should see an output similar to:


#### Kafka Consumer

Next, let's consume the messages from the `a_messages` topic. Run the following command:

```bash
./bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic a_messages --from-beginning --consumer.config=/tmp/team-a-client.properties
```

Since a consumer group is not specified, a random group name will be generated. However, you'll encounter an error:

```
Not authorized to access the group: console-consumer-55841
```

This happens because 'Dev Team A' only has access to consumer groups with names starting with `a_`. Let's specify a custom consumer group name that starts with `a_`:

```bash
./bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic a_messages \
  --from-beginning --consumer.config=/tmp/team-a-client.properties --group a_consumer_group_1
```

At this point, you should be able to consume all messages from the `a_messages` topic.
