<h1>RabbitMQ install</h1>

<b>Install ns,pv,pvc</b>
<p>

```text
kubectl apply -f RabbitMQ/install-env.yaml
```
</p>

<b>Install rabbitmq by helm</b>
<p>

```text
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install rabbitmq --set persistence.existingClaim=rabbitmq-data-claim bitnami/rabbitmq

# check
kubectl get secret --namespace databases rabbitmq -o jsonpath="{.data.rabbitmq-password}" | base64 -d
```
</p>
