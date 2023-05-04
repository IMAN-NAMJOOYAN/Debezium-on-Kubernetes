# Debezium-on-Kubernetes
Running Debezium 2.2 on kubernetes, Using Mysql Connector:

**LOM:**

- Kubernetes: Version 1.25.5
- Kafka Docker Image: quay.io/strimzi/kafka:latest-kafka-3.4.0
- Docker: Version 23.0.5 For Create Mysql Connector Image
- Maven "debezium-connector-mysql": https://repo1.maven.org/maven2/io/debezium/debezium-connector-mysql/2.2.0.Final/debezium-connector-mysql-2.2.0.Final-plugin.tar.gz
- StorageClass: Longhorn
- Private Docker Registry: NexusOSS or Harbor

**Prerequisites**
```
1- curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.20.0/install.sh | bash -s v0.20.0
2- kubectl create -f https://operatorhub.io/install/strimzi-kafka-operator.yaml
3- Creating Secrets for the Database
kubectl create -n debezium-example -f database-secret.yml
Note: The username and password contain base64-encoded credentials (debezium/dbz) for connecting to the MySQL database.
4- Now, we can create a role, which refers the secret created in the previous step:
kubectl create -n debezium-example -f connector-configuration-role.yml
5- We also have to bind this role to the Kafka Connect cluster service account so that Kafka Connect can access the secret:
kubectl create -n debezium-example -f connector-configuration-role-binding.yml
```

**Deploying Apache Kafka**
```
1- deploy a (single-node) Kafka cluster:
kubectl create -n debezium-example -f debezium-cluster.yml

2- Wait until it’s ready:
kubectl wait kafka/debezium-cluster --for=condition=Ready --timeout=300s -n debezium-example
```
**Deploying a Data Source**
```
1- As a data source, MySQL will be used in the following. Besides running a pod with MySQL, an appropriate service which will point to the pod with DB itself is needed. It can be created e.g. as follows:
kubectl create -n debezium-example -f mysql-example.yml



