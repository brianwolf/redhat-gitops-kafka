# Red Hat AMQ Streams

---

## Agregar role a admin para generar instancia de Kafka, tópicos, etc.

> Este paso permite que la cuenta de servicio utilizada por ArgoCD genere los objetos dentro del cluster.

oc adm policy add-role-to-user admin \
system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller \
-n <kafka-namespace>

# Ejecutar pruebas desde entorno local hacia tópico de Kafka dentro del cluster de OpenShift

---

## Precondiciones
### 1 - Descargar de [AMQ Streams](https://access.redhat.com/jbossnetwork/restricted/listSoftware.html?downloadType=distributions&product=jboss.amq.streams) el link de Red Hat AMQ Streams 2.6.0

### 2 - Extraer certificado del cluster de Kafka

```sh
oc extract secret/<kafka-cluster-name>-cluster-ca-cert --keys=ca.crt --to=- > ca.crt
```

```sh
keytool -keystore client.truststore.jks -alias CARoot -import -file ca.crt
```

### 3 - Crear un password
### 4 - Confirmar password
### 5 - Confiar en el certificado a agregar (ca.crt extraído del cluster de OpenShift del cluster de AMQ Streams)

## 6 - Generar ruta para acceso externo al cluster

verificar que kafka.listeners.name[tls]. esté en true y el tipo sea ruta para generar una URL al tópico que sea accesible por fuera del cluster de OpenShift

## Obtener dominio de la ruta

```sh
oc get routes <kafka-cluster-name>-kafka-tls-bootstrap -o=jsonpath='{.status.ingress[0].host}{"\n"}'
```

## Producir desde local

```sh
bin/kafka-console-producer.sh --broker-list <cluster-kafka-domain>:443 --producer-property security.protocol=SSL --producer-property ssl.truststore.password=<pwd-trustore> --producer-property ssl.truststore.location=./client.truststore.jks --topic <topic-name>
```

## Consumir desde local

```sh
bin/kafka-console-consumer.sh --bootstrap-server <cluster-kafka-domain>:443 --consumer-property security.protocol=SSL --consumer-property ssl.truststore.password=<pwd-trustore> --consumer-property ssl.truststore.location=./client.truststore.jks --topic <topic-name> --from-beginning
```

## Producir desde cluster

oc run kafka-producer -ti \
--image=registry.redhat.io/amq-streams/kafka-36-rhel8:2.6.0 \
--rm=true \
--restart=Never \
-- bin/kafka-console-producer.sh \
--bootstrap-server my-cluster-kafka-bootstrap:9092 \
--topic my-topic

## Consumir desde cluster

oc run kafka-consumer -ti \
--image=registry.redhat.io/amq-streams/kafka-36-rhel8:2.6.0 \
--rm=true \
--restart=Never \
-- bin/kafka-console-consumer.sh \
--bootstrap-server my-cluster-kafka-bootstrap:9092 \
--topic my-topic \
--from-beginning