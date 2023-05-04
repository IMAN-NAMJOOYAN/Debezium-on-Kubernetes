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
- As a data source, MySQL will be used in the following. Besides running a pod with MySQL, an appropriate service which will point to the pod with DB itself is needed. It can be created e.g. as follows:
kubectl create -n debezium-example -f mysql-example.yml
```
**Deploying a Debezium Connector**
```
1- Create customize docker image for mysql kafka connector:
- curl -O https://repo1.maven.org/maven2/io/debezium/debezium-connector-mysql/2.2.0.Final/debezium-connector-mysql-2.2.0.Final-plugin.tar.gz
- tar xzfv debezium-connector-mysql-2.2.0.Final-plugin.tar.gz
- cat << EOF > Dockerfile 
FROM quay.io/strimzi/kafka:latest-kafka-3.4.0
USER root:root
RUN mkdir -p /opt/kafka/plugins/debezium
COPY ./debezium-connector-mysql/ /opt/kafka/plugins/debezium/
USER 1001
EOF
- docker build . -t <Private Registry IP Address>/debezium/connect-mysql-debezium-3.4.0:v1
- docker push <Private Registry IP Address>/debezium/connect-mysql-debezium-3.4.0:v1
2- Creating Kafka Connect Cluster:
kubectl create -n debezium-example -f debezium-connect-cluster.yml
Note: <Private Registry IP Address> replace with docker private registry ip address.
3- Creating a Debezium Connector:
kubectl create -n debezium-example -f debezium-connector-mysql.yml

Note: If the Kafka Connector could not recognize the username and password. To ensure that the connector works correctly, enter the username and password manually.
```







