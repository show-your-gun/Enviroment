<h1>Enviroment</h1>
<h2>Cluster unfolding on a single server</h2>
<h4>Server params</h4>
<hr>

> PRETTY_NAME="Debian GNU/Linux 11 (bullseye)"
>
> Architecture:                    x86_64
> 
> CPU op-mode(s):                  32-bit, 64-bit
>
> Model name:                      Intel(R) Pentium(R) CPU G3240 @ 3.10GHz
> 
> MemTotal:        8033040 kB

<h4>Cluster prepare</h4>
<hr>

<b>Ensure that the server is up to date</b>
<p>

```text
sudo apt update
sudo apt -y upgrade
```
</p>

<b>Install kubelet, kubeadm and kubectl</b>
<p>

```text
sudo apt install -y gnupg2
sudo apt -y install curl apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
</p>

<b>Install required packages</b>
<p>

```text
sudo apt update
sudo apt -y install nano git curl wget kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
</p>

<b>Confirm installation by checking the version of kubectl</b>
<p>

```text
# My versions
Client Version: version.Info{Major:"1", Minor:"24", GitVersion:"v1.24.2", GitCommit:"f66044f4361b9f1f96f0053dd46cb7dce5e990a8", GitTreeState:"clean", BuildDate:"2022-06-15T14:22:29Z", GoVersion:"go1.18.3", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v4.5.4
kubeadm version: &version.Info{Major:"1", Minor:"24", GitVersion:"v1.24.2", GitCommit:"f66044f4361b9f1f96f0053dd46cb7dce5e990a8", GitTreeState:"clean", BuildDate:"2022-06-15T14:20:54Z", GoVersion:"go1.18.3", Compiler:"gc", Platform:"linux/amd64"}
```
</p>

<b>Disable Swap and Enable Kernel modules</b>
<p>

```text
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo swapoff -a

sudo modprobe overlay
sudo modprobe br_netfilter
```
</p>

<b>Prepare sysctl</b>
<p>

```text
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```
</p>

<b>Install Container runtime ( Current is CRI-O )</b>
<p>

```text
sudo -i
OS=Debian_10
VERSION=1.23

echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list

curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | apt-key add -
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | apt-key add -


sudo apt update
sudo apt install cri-o cri-o-runc

```
</p>

<b>Installation confirm</b>
<p>

```text
apt-cache policy cri-o

#my std out

cri-o:
  Installed: 1.23.3~0
  Candidate: 1.23.3~0
  Version table:
 *** 1.23.3~0 500
        500 http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/1.23/Debian_10  Packages
        100 /var/lib/dpkg/status
```
</p>

<b>Start and enable CRI-O</b>
<p>

```text
sudo systemctl daemon-reload
sudo systemctl enable crio --now
```
</p>

<b>Confirm CRI-O status</b>
<p>

```text
sudo systemctl status crio

#my std out
● crio.service - Container Runtime Interface for OCI (CRI-O)
     Loaded: loaded (/lib/systemd/system/crio.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2022-07-07 02:42:56 EDT; 4h 5min ago
       Docs: https://github.com/cri-o/cri-o
   Main PID: 353 (crio)
      Tasks: 20
     Memory: 526.0M
        CPU: 3min 57.140s
     CGroup: /system.slice/crio.service
             └─353 /usr/bin/crio
             ...
```
</p>

<b>Initialize master node</b>
<p>

```text
lsmod | grep br_netfilter

#my std out
br_netfilter           32768  0
bridge                253952  1 br_netfilter
```
</p>

<b>Let us also enable the kubelet service to start on boot</b>
<p>

```text
sudo systemctl enable kubelet
```
</p>

<b>Pull the required images</b>
<p>

```text
sudo kubeadm config images pull
```
</p>

<b>Create cluster</b>
<p>

```text
kubeadm init
```
</p>

<b>Copy config</b>
<p>

```text
mkdir -p $HOME/.kube
sudo cp -f /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
</p>

<b>Check your cluster status</b>
<p>

```text
kubectl cluster-info

#my std out
Kubernetes control plane is running at https://188.44.124.36:6443
CoreDNS is running at https://188.44.124.36:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
...
```
</p>

<b>Let us scheduling pods on Master</b>
<p>

```text
kubectl taint nodes --all node-role.kubernetes.io/master-
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```
</p>

<b>Install network plugin on Master ( current is Calico)</b>
<p>

```text
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```
</p>

<b>Install ingress</b>
<p>

```text
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.1/deploy/static/provider/baremetal/deploy.yaml
```
</p>

<b>Install LoadBalancer ( MetalLB )</b>
<p>

```text
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/metallb.yaml
```

<b> ATTENTION ***  13.2 version doesn't work </b>


```text
#kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.2/config/manifests/metallb-native.yaml
```
</p>

<b>Next step you should apply configmap manifest with your server ip</b>
<p>

```text
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 188.44.124.36
```
</p>

<b>Change ingress-nginx-controller type from NodePort to LoadBalancer</b>
<p>

```text
...
  type: LoadBalancer
...
```
</p>

<b>Check your server</b>
<p>

```text
$ curl -D- http://188.44.124.36 -H 'Host: paradox-server.ru'

#my std out
HTTP/1.1 200 OK
Server: nginx/1.15.2
```
</p>

<b>Install cert manager ( LetsEncrypt )</b>
<p>

```text
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm repo update
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.8.2 \
  --set installCRDs=true
```
</p>




<b>Example manifest</b>
<p>

```text
apiVersion: v1
kind: Namespace
metadata:
  name:  app-test
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
  namespace: app-test
spec:
  selector:
    matchLabels:
      app: app-pod
  replicas: 3
  template:
    metadata:
      labels:
        app: app-pod
    spec:
      containers:
      - name: app-pod
        image: nginx
        resources:
          limits:
            memory: "128Mi"
            cpu: "0.1"
        ports:
        - containerPort: 80
          name: http
---
apiVersion: v1
kind: Service
metadata:
  name: app-service
  namespace: app-test
spec:
  selector:
    app: app-pod
  ports:
  - port: 8080
    name: http
    targetPort: 80
---
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: "letsencrypt-prod"
  namespace: app-test
spec:
  acme:
    email: alexanderdolgosheev@gmail.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: example-issuer-account-key
    solvers:
    - http01:
        ingress:
          class: nginx
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: app-test
  labels:
    name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - paradox-server.ru
    secretName: paradox-server-tls
  rules:
  - host: paradox-server.ru
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: app-service
            port: 
              name: http
  ingressClassName: nginx
```
</p>