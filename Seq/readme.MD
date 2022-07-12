<h1>Seq install ( 2021.2 docker )</h1>

<b>Install ns,pv,pvc</b>
<p>

```text
kubectl apply -f Seq/install-env.yaml
```
</p>

<b> xD 2021.2 </b>
<p>

```text
cd Seq/res
docker build . -t <YOUR_REPO>:<TAG>
```
</p>


<b> Install by Helm </b>
<p>

```text
helm repo add datalust https://helm.datalust.co
helm repo update

helm install seq datalust/seq \
  --set image.repository=<YOUR_REPO> \
  --set image.tag=<TAG> \
  --set persistence.existingClaim=seq-data-claim
```
</p>
