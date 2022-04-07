# strimzi-sasl-plaintext-nr
Kafka Strimzi configuration with SASL-PLAINTEXT and monitored by NewRelic Agent

## Installing the strimzi-operator

To install with helm

```
$ helm repo add strimzi https://strimzi.io/charts/
$ helm upgrade strimzi strimzi/strimzi-kafka-operator -n strimzi-kafka
```


## Setting up a cluster with 3 Kafka brokers one zookeeper and the EntityOperator

Executing:
```
$ kubectl apply -f strimzi-kafka.yaml
```

This in the manifest is making kafka to accept SASL_PLAINTEXT :
```YAML
      - name: tls
        port: 9093
        type: internal
        tls: false
        authentication:
          type: scram-sha-512
```

We specify the authorization type and we pass the user we will create as a super-user:

```YAML
    authorization:
      type: simple
      superUsers:
        - my-user
```

We are specifying the following so strimzi will create the entityOperator deployment that will be needed to crate the user and password for broker authentication:
```YAML
  entityOperator:
    topicOperator: {}
    userOperator: {}
```

## Creating the user

Executing:
```
$ kubectl apply -f strimzi-user.yaml
```

If the UserOperator container in the EntityOperator pod is running, a new user called `my-user` will be created and a secret with the same name will be created with the password (Note that k8s codifies the secrets in Base64, so you will need to decode it before putting it in your NewRelic agent config file)

## Launching the nri-kubernetes with the kafka integration monitoring this kakfa cluster

Executing (The last 4 --set flags are not mandatory  for the kafka metrics to appear but for k8s monitoring):
```
helm upgrade --install newrelic-bundle newrelic/nri-bundle \
--set global.licenseKey=<your-license-key> \
--set global.cluster=<your-cluster> \
--namespace=<your-namespace> \
--set newrelic-infrastructure.privileged=false \
--set global.lowDataMode=true \
--set ksm.enabled=true \
--set prometheus.enabled=true -f kafka-newrelic-values.yaml
```

This setup is already prepared to monitor the previously launched kafka cluster and user.