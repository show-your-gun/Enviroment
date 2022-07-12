<h1>Postgres install</h1>

<b>Install ns,pv,pvc</b>
<p>

```text
kubectl apply -f Postgres/install-env.yaml
```
</p>

<b>Install postgres by helm</b>
<p>

```text
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install postgresql -f Postgres/values.yaml bitnami/postgresql

# check
kubectl get secret --namespace databases postgresql -o jsonpath="{.data.postgres-password}" | base64 -d
```
</p>
