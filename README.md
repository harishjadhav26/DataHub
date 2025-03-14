# DataHub

***DataHub Modules***

Please ensure the PV and PVC

Datahub contains 4 main components:

```
acryldata/datahub-ingestion
acryldata/datahub-gms
acryldata/datahub-frontend-react
acryldata/datahub-mae-consumer (optional)
acryldata/datahub-mce-consumer (optional)
```

Datahub setups for external dependencies:

```
acryldata/datahub-upgrade
acryldata/datahub-kafka-setup
acryldata/datahub-elasticsearch-setup
acryldata/datahub-mysql-setup
acryldata/datahub-postgres-setup
```

The main components are powered by 4 external dependencies:

```
- Kafka, Zookeeper, and Schema Registry
- Local DB (MySQL, Postgres, MariaDB)
- Search Index (Elasticsearch)
- Graph Index (Optional Neo4j)
```

Deployment Sequesnces:

We are using the default namespace `default` for deployment, you can also use different namespace like `datahub`.

1. Deploy MySQL/MariaDB/PostGreSQL any one from these
2. Deploy Elasticsearch
3. Deploy Zookeeper
4. Deploy Kafka
5. Deploy Schema-Registry
6. Deploy Neo4J (OPTIONAL NOT REQUIRED)
7. Deploy Kafka-Setup
9. Deploy Elasticsearch-Setup
10. Deploy DataHub-Upgrade
11. Deploy DataHub-GMS
12. Deploy DataHub-Frontend-React
13. Deploy DatHub-Ingestion
14. Deploy DatHub-MAE-Consumer (OPTIONAL)
15. Deploy DatHub-MCE consumer (OPTIONAL)

Please verify every deployment up and running without any error

***Kafka and Kubernetes: Challenges***
```
Using Kafka with Kubernetes has multiple benefits, especially if the organization already deploys stateful applications in the cluster. However, the following is a list of potential challenges to consider before deciding to deploy Kafka in a container environment:

- Kubernetes works best with stateless apps. While stateful applications can operate inside a cluster, running them requires administrators to give up many features that make Kubernetes popular.

- Kafka requires an entire operating system. Kafka stores the message queue data using a file system cache, which requires an OS kernel. While Kafka can run inside a container, it still needs to communicate with the rest of the OS, so it cannot benefit from containerization as a practice.

- Kafka cannot work with a single load balancer. As explained before, Kafka brokers are not interchangeable, so putting a load balancer before them results in Kafka breaking. A solution to this problem is setting up one load balancer for each broker, but this setup can create a complicated deployment when many brokers are present.
```

```
You can update the following details in deployment files

Mysql Default Host: 192.168.1.111
Mysql Default User: root
Mysql Default Passwod: admin
```
MariaDB POC with Galera:
https://github.com/mariadb-operator/mariadb-poc/tree/main/galera/kubernetes


`OPTIONAL` DataHub uses Neo4j as graph db in the backend to serve graph queries.
[Official Neo4j image](https://hub.docker.com/_/neo4j) found in Docker Hub is used without 
any modification.

## Neo4j Browser(OPTIONAL)
To be able to debug and run Cypher queries against your Neo4j image, you can open up `Neo4j Browser` which is running at
[http://localhost:7474/browser/](http://localhost:7474/browser/). Default username is `neo4j` and password is `datahub`.


# Create Local Storage for Kubeadm

```
sudo rm -rf /mnt/data/{mysql-db,kafka,zookeeper,elasticsearch}
sudo mkdir -p /mnt/data/{mysql-db,kafka,zookeeper,elasticsearch}
sudo chmod 777 /mnt/data/{mysql-db,kafka,zookeeper,elasticsearch}
sudo chown -R harish. /mnt/data/{mysql-db,kafka,zookeeper,elasticsearch}
```

***Deployment Phases***

1. Deploy the Zookeeper
```
kubectl apply -f kafka-zookeeper-schema-registry/zookeeper.yaml

kubectl get pod

kubectl logs -f -l service=zookeeper
```

2. Deploy the kafka 
```
kubectl apply -f kafka-zookeeper-schema-registry/kafka.yaml

kubectl get pod

kubectl logs -f -l app=kafka
```

3. Deploy the schema-registry
```
kubectl apply -f kafka-zookeeper-schema-registry/schema-registry.yaml

kubectl get pod

kubectl logs -f -l app=schema-registry
```

4. Deploy the Elasticsearch
```
kubectl apply -f elasticsearch/elasticsearch.yaml

kubectl get pod

kubectl logs -f -l app=elasticsearch
```

5. Add job for datahub kafka setup
```
kubectl apply -f datahub-app/datahub-kafka-setup.yaml

kubectl get pod

kubectl logs -f job/datahub-kafka-setup-job
```
6. Add job for datahub elasticsearch setup
```
kubectl apply -f datahub-app/datahub-elasticsearch-setup.yaml

kubectl get pod

kubectl logs -f job/datahub-elasticsearch-setup-job
```

7. Add job for datahub mysql setup
```
kubectl apply -f datahub-app/datahub-mysql-setup.yaml

kubectl get pod

kubectl logs -f job/datahub-mysql-setup-job
```

8. Add job for datahub-upgrade
```
kubectl apply -f datahub-app/datahub-upgrade.yaml

kubectl get pod

kubectl logs -f job/datahub-upgrade
```

9. Deploy the Datahub GMS
```
kubectl apply -f datahub-app/datahub-gms.yaml

kubectl get pod

kubectl logs -f -l app=datahub-gms
```

10. Deploy the Datahub Frontend
```
kubectl apply -f datahub-app/datahub-frontend-react.yaml

kubectl get pod

kubectl logs -f -l app=datahub-frontend-react
```

Finalyy try to access the URL 

kubectl port-forward svc/datahub-frontend 9002:9002

Open URL in browser `URL: http://localhost:9002` Default username `datahub` and password `datahub`


If DataHub is still showing the `"Before we begin"` onboarding screen, follow these debugging steps to resolve the issue:

```
Run this GraphQL mutation in Postman or GraphiQL:

kubectl port-forward svc/datahub-frontend 9002:9002

URL: http://localhost:9002/api/graphql

Method: POST

Body:

{
  "query": "mutation { updateGlobalSettings(input: {showOnboarding: false}) }"
}


# Refresh your browser and see if the onboarding screen disappears.
#  Manually Modify DataHub MySQL DB (Alternative)

If you're using MySQL as the metadata store, you can manually change the onboarding setting.

USE datahub;

UPDATE metadata_aspect_v2 SET aspect = JSON_SET(aspect, '$.showOnboarding', false) WHERE urn = 'urn:li:dataset:(urn:li:dataPlatform:datahub,global_settings,PROD)';

kubectl rollout restart deployment datahub-gms
kubectl rollout restart deployment datahub-frontend-react

```
