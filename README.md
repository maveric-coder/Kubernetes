# Kubernetes

## apply vs create

The key difference between kubectl apply and create is that apply creates Kubernetes objects through a declarative syntax, while the create command is imperative.

The command set kubectl apply is used at a terminal's command-line window to create or modify Kubernetes resources defined in a manifest file. This is called a declarative usage. The state of the resource is declared in the manifest file, then kubectl apply is used to implement that state.

In contrast, the command set kubectl create is the command you use to create a Kubernetes resource directly at the command line. This is an imperative usage. You can also use kubectl create against a manifest file to create a new instance of the resource. However, if the resource already exists, you will get an error.

### Example of kubectl apply
Let's explore the details of both kubectl usages. First, let's look at kubectl apply. Listing 1 below is a manifest file that describes a Kubernetes deployment that has three replicas of a nginx container image.

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mydeployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

The name of the deployment manifest file in Listing 1 is mydeployment.yaml. If you run the command below, it will create a deployment according to the contents of this manifest file.
```bash
kubectl apply -f mydeployment.yaml
```
Executing the command will create the following response:

`deployment/mydeployment created`

When you run the command `kubectl get deployment`, you'll get the following output:

NAME           READY   UP-TO-DATE   AVAILABLE   AGE
mydeployment   3/3     3            3           7m10s
Here, we've created the deployment named mydeployment, and it is running its three pods.

Example of kubectl create
Now, let's use kubectl create to try to create a deployment imperatively, like so:
```bash
kubectl create deployment mydeployment --image=nginx
```
When you execute the imperative command, you'll get the following result:

`Error from server (AlreadyExists): deployments.apps "mydeployment" already exists`

This makes sense. Remember, if you try to use kubectl create against a resource that already exists, you'll get an error.

However, let's try to execute kubectl create for a resource that doesn't exist. In this case, we'll create a Kubernetes deployment named yourdeployment. We'll create it using the following command:
```bash
kubectl create deployment yourdeployment --image=nginx
```
You'll get the following output, indicating success:

`deployment.apps/yourdeployment created`


```bash
sudo apt-get update
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y docker.io
sudo systemctl enable docker

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
```bash

kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter.yaml
```


Objects in Kubernetes:
* POD 
* Service
* ReplicaController
* ReplicaSet
* DS
* VolumeJobs

```bash
Kubectl get nodes
kubectl get ns


kubectl get componentstatuses

sudo ufw status verbose
sudo ufw disable
## turn on
sudo ufw enable
## make sure your kube-apiserver can get through port 6443
sudo ufw allow 6443/tcp


Services:
ClustureIP
NodePort
LoadBalancer


```
