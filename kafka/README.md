# Zookeeper, Kafka, Schema-Registry and Kafka Netwoork Policy

How to install Zookeepr, Kafaka and Schema-Registry

1. Create `datahub` namespace
2. Verify and confirm the PV and PVC for ZKS
3. Deploy the services 1 by 1, and confirm every services is up and running without any error
4. If you want to connect another pod with ZKS the please add the following field in manifest file
```
.
.
  template:
    metadata:
      labels:
        network/kafka-network: "true" # Only If you are using Network Policy else comment this line while deploymment
        service: XYZ-SERVICE
.
.
```

List the pods available in the cluster:
```
kubectl get po,svc,deploy,statefulset -n datahub
```

Find the kcat pod and copy its name.

Enter the pod by executing the following command:
```
kubectl exec --stdin --tty [pod-name] -- /bin/sh
/ #  <----- Enter the command below to send Kafka a test message to ingest:
echo "Test Message" | kcat -P -b kafka-0.kafka.datahub.svc.cluster.local:29092 -t testtopic -p -1
```
If successful, the command prints no output.

Switch to the consumer role and query Kafka for messages by typing:
```
kcat -C -b kafka-2.kafka.datahub.svc.cluster.local:29092 -t testtopic -p -1
```

Execute the following to create a new topic
```
kafka-topics --bootstrap-server [KAFKA BROKER]:29092 --topic [topic-name] --create --partitions [number] --replication-factor [number]

EX:
kafka-topics --bootstrap-server kafka-2.kafka.datahub.svc.cluster.local:29092 --topic mytopic --create --partitions 3 --replication-factor 3
```

***Kafka and Kubernetes: Challenges***

1. Ensure the PV,PVC correctly created or not
2. Ensure if you are not familier with networking policy please comment `network/kafka-network: "true"` in manifest to avoid connectivity
3. Ensure and verify the ZKS FQDN is working or not else you will face the erro `Error connecting to node kafka-1.kafka.datahub.svc.cluster.local:29092 (id: 1020 rack: null) (org.apache.kafka.clients.NetworkClient)
java.net.UnknownHostException: kafka-1.kafka.datahub.svc.cluster.local`
4. Ensure PROTOCOL Map in Kafka and Schema Regitry, else you will face `The error "Unable to parse PLAINTEXT://${HOSTNAME}.kafka.datahub.svc.cluster.local:9092 to a broker endpoint" means that the ${HOSTNAME} variable is not being properly resolved when setting up Kafka listeners.`
5. Ensure the StatefulSet-generated Hostnames, Kafka StatefulSets generate predictable hostnames (kafka-0, kafka-1, etc.), so instead of using ${HOSTNAME}, explicitly use: `PLAINTEXT://kafka-0.kafka.datahub.svc.cluster.local:9092`
6. Ensure the Schema-Regitry ENV Variables in manifests

***If you are facing the any kafka related issues please raise the Issues***
