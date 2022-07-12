<h1>Redis install</h1>

<b>Install ns,pv,pvc</b>
<p>

```text
kubectl apply -f Redis/install-env.yaml
```
</p>

<b>Install redis by helm</b>
<p>

```text
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install redis --set master.persistence.existingClaim=redis-data-claim-master --set replica.persistence.existingClaim=redis-data-claim-replica bitnami/redis

# check
kubectl get secret --namespace databases redis -o jsonpath="{.data.redis-password}" | base64 -d
```
</p>
