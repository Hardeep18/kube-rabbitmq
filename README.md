# RabbitMQ on Kubernetes

Create a cluster with [kind](https://kind.sigs.k8s.io/docs/user/quick-start/)

```
kind create cluster --name rabbit --image kindest/node:v1.18.4
```

## Namespace

```
kubectl create ns rabbitmq
```

## Storage Class

```
kubectl get storageclass
NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
standard (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  84s
```

## Deployment

```
kubectl apply -n rabbitmq -f .\kubernetes\rabbit-rbac.yaml
kubectl apply -n rabbitmq -f .\kubernetes\rabbit-configmap.yaml
kubectl apply -n rabbitmq -f .\kubernetes\rabbit-secret.yaml
kubectl apply -n rabbitmq -f .\kubernetes\rabbit-statefulset.yaml
```

## Access the UI

```
kubectl -n rabbitmq port-forward rabbitmq-0 8080:15672
```
Go to htttp://localhost:8080 <br/>
Username: `guest` <br/>
Password: `guest` <br/>

# Message Publisher

```

cd messaging\rabbitmq\applications\publisher
docker build . -t aimvector/rabbitmq-publisher:v1.0.0

kubectl apply -f rabbitmq deployment.yaml
```

# Automatic Synchronization

https://www.rabbitmq.com/ha.html#unsynchronised-mirrors

```
rabbitmqctl set_policy ha-fed \
    ".*" '{"federation-upstream-set":"all", "ha-sync-mode":"automatic", "ha-mode":"nodes", "ha-params":["rabbit@rabbitmq-0.rabbitmq.rabbitmq.svc.cluster.local","rabbit@rabbitmq-1.rabbitmq.rabbitmq.svc.cluster.local","rabbit@rabbitmq-2.rabbitmq.rabbitmq.svc.cluster.local"]}' \
    --priority 1 \
    --apply-to queues
```