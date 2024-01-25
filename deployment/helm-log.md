### etcd


NAME: etcd
LAST DEPLOYED: Wed Jan 24 17:04:33 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: etcd
CHART VERSION: 9.10.0
APP VERSION: 3.5.11

** Please be patient while the chart is being deployed **

etcd can be accessed via port 2379 on the following DNS name from within your cluster:

    etcd.default.svc.cluster.local

To create a pod that you can use as a etcd client run the following command:

    kubectl run etcd-client --restart='Never' --image docker.io/bitnami/etcd:3.5.11-debian-11-r3 --env ROOT_PASSWORD=$(kubectl get secret --namespace default etcd -o jsonpath="{.data.etcd-root-password}" | base64 -d) --env ETCDCTL_ENDPOINTS="etcd.default.svc.cluster.local:2379" --namespace default --command -- sleep infinity

Then, you can set/get a key using the commands below:

    kubectl exec --namespace default -it etcd-client -- bash
    etcdctl --user root:$ROOT_PASSWORD put /message Hello
    etcdctl --user root:$ROOT_PASSWORD get /message

To connect to your etcd server from outside the cluster execute the following commands:

    export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
    export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services etcd)
    echo "etcd URL: http://$NODE_IP:$NODE_PORT/"

 * As rbac is enabled you should add the flag `--user root:$ETCD_ROOT_PASSWORD` to the etcdctl commands. Use the command below to export the password:

    export ETCD_ROOT_PASSWORD=$(kubectl get secret --namespace default etcd -o jsonpath="{.data.etcd-root-password}" | base64 -d)





## Postgres
NAME: postgres
LAST DEPLOYED: Wed Jan 24 17:06:23 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: postgresql
CHART VERSION: 13.3.1
APP VERSION: 16.1.0

** Please be patient while the chart is being deployed **

PostgreSQL can be accessed via port 5432 on the following DNS names from within your cluster:

    postgres-postgresql.default.svc.cluster.local - Read/Write connection

To get the password for "postgres" run:

    export POSTGRES_PASSWORD=$(kubectl get secret --namespace default postgres-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

To connect to your database run the following command:

    kubectl run postgres-postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:16.1.0-debian-11-r20 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host postgres-postgresql -U postgres -d postgres -p 5432

    > NOTE: If you access the container using bash, make sure that you execute "/opt/bitnami/scripts/postgresql/entrypoint.sh /bin/bash" in order to avoid the error "psql: local user with ID 1001} does not exist"

To connect to your database from outside the cluster execute the following commands:

    export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
    export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services postgres-postgresql)
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host $NODE_IP --port $NODE_PORT -U postgres -d postgres

WARNING: The configured password will be ignored on new installation in case when previous PostgreSQL release was deleted through the helm command. In that case, old PVC will have an old password, and setting it through helm won't take effect. Deleting persistent volumes (PVs) will solve the issue.


# Kafka
NAME: kafka
LAST DEPLOYED: Wed Jan 24 17:07:00 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: kafka
CHART VERSION: 26.8.0
APP VERSION: 3.6.1

** Please be patient while the chart is being deployed **

Kafka can be accessed by consumers via port 9092 on the following DNS name from within your cluster:

    kafka.default.svc.cluster.local

Each Kafka broker can be accessed by producers via port 9092 on the following DNS name(s) from within your cluster:

    kafka-controller-0.kafka-controller-headless.default.svc.cluster.local:9092
    kafka-controller-1.kafka-controller-headless.default.svc.cluster.local:9092
    kafka-controller-2.kafka-controller-headless.default.svc.cluster.local:9092

The CLIENT listener for Kafka client connections from within your cluster have been configured with the following security settings:
    - SASL authentication

To connect a client to your Kafka, you need to create the 'client.properties' configuration files with the content below:

security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-256
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required \
    username="user1" \
    password="$(kubectl get secret kafka-user-passwords --namespace default -o jsonpath='{.data.client-passwords}' | base64 -d | cut -d , -f 1)";

To create a pod that you can use as a Kafka client run the following commands:

    kubectl run kafka-client --restart='Never' --image docker.io/bitnami/kafka:3.6.1-debian-11-r1 --namespace default --command -- sleep infinity
    kubectl cp --namespace default /path/to/client.properties kafka-client:/tmp/client.properties
    kubectl exec --tty -i kafka-client --namespace default -- bash

    PRODUCER:
        kafka-console-producer.sh \
            --producer.config /tmp/client.properties \
            --broker-list kafka-controller-0.kafka-controller-headless.default.svc.cluster.local:9092,kafka-controller-1.kafka-controller-headless.default.svc.cluster.local:9092,kafka-controller-2.kafka-controller-headless.default.svc.cluster.local:9092 \
            --topic test

    CONSUMER:
        kafka-console-consumer.sh \
            --consumer.config /tmp/client.properties \
            --bootstrap-server kafka.default.svc.cluster.local:9092 \
            --topic test \
            --from-beginning


# keycloak
NAME: keycloak
LAST DEPLOYED: Wed Jan 24 17:07:41 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: keycloak
CHART VERSION: 18.2.1
APP VERSION: 23.0.4

** Please be patient while the chart is being deployed **

Keycloak can be accessed through the following DNS name from within your cluster:

    keycloak.default.svc.cluster.local (port 80)

To access Keycloak from outside the cluster execute the following commands:

1. Get the Keycloak URL by running these commands:

    export HTTP_NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[?(@.name=='http')].nodePort}" services keycloak)
    export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")

    echo "http://${NODE_IP}:${HTTP_NODE_PORT}/"

2. Access Keycloak using the obtained URL.



