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
   oc apply -f kafka-resources/
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
   ./bin/kafka-topics.sh --bootstrap-server my-kafka-cluster-kafka-bootstrap:9092 --list
   ```

2. **Send a Message:**
   ```bash
   ./bin/kafka-console-producer.sh --bootstrap-server my-kafka-cluster-kafka-bootstrap:9092 --topic my-topic
   ```

3. **Consume Messages:**
   ```bash
   ./bin/kafka-console-consumer.sh --bootstrap-server my-kafka-cluster-kafka-bootstrap:9092 --topic my-topic --from-beginning
   ```

### Real-Time Message Consumption

To simulate real-time messaging, open two terminal sessions:

- **Terminal 1 (Message Consumer):**
  ```bash
  oc -n kafka-example exec -it deploy/kafka-tools -- bash
  ./bin/kafka-console-consumer.sh --bootstrap-server my-kafka-cluster-kafka-bootstrap:9092 --topic my-topic --from-beginning
  ```

- **Terminal 2 (Message Producer):**
  ```bash
  oc -n kafka-example exec -it deploy/kafka-tools -- bash
  ./bin/kafka-console-producer.sh --bootstrap-server my-kafka-cluster-kafka-bootstrap:9092 --topic my-topic
  ```

Type messages in **Terminal 2** to send them to Kafka. Messages will appear in **Terminal 1**, allowing you to observe real-time message streaming, similar to a chat interface.

---