## What is Ingress?

The Ingress is a Kubernetes resource that lets you configure an HTTP load balancer for applications running on Kubernetes, represented by one or more [Services](https://kubernetes.io/docs/concepts/services-networking/service/). Such a load balancer is necessary to deliver those applications to clients outside of the Kubernetes cluster.

<img src = "https://github.com/maveric-coder/Kubernetes/blob/main/files/img/nginx-ingress.jpeg" width="700" height="950"/></a> <br>
The Ingress resource supports the following features:
* **Content-based routing**:
    * *Host-based routing*. For example, routing requests with the host header `foo.example.com` to one group of services and the host header `bar.example.com` to another group.
    * *Path-based routing*. For example, routing requests with the URI that starts with `/serviceA` to service A and requests with the URI that starts with `/serviceB` to service B.

See the [Ingress User Guide](http://kubernetes.io/docs/user-guide/ingress/) to learn more about the Ingress resource.

## What is Ingress Controller?

The Ingress controller is an application that runs in a cluster and configures an HTTP load balancer according to Ingress resources. The load balancer can be a software load balancer running in the cluster or a hardware or cloud load balancer running externally. Different load balancers require different Ingress controller implementations. 

In the case of NGINX, the Ingress controller is deployed in a pod along with the load balancer.


## Installing the Ingress Controller In EKS

### 1. Clone Kubernetes Nginx Ingress Manifests into server where you have kubectl

```
git clone https://github.com/maveric-coder/Kubernetes.git

cd Kubernetes/kubernetes-ingress/deployments
```
**1.1 Deploy java and maven web applications if not deployed already**
```
cd Kubernetes/Manifest-files/ingress-demo
kubectl apply -f lb-java.yml
kubectl apply -f lb-maven.yml
```
By default the yml files are having `LoadBalancer` service. Test both the applications by using the loadbanacer `<PublicIP>/java-web-app`
and `<PublicIP>/maven-web-application`.

Post that remove the `LoadBalancer` services running for each application and run them as `ClusterIP`. Re-deploy both the applications, with modified yml files.


### 2. Create a Namespace and Service-Account
Now we have deployed our two applications which can be accessed by ClustureIP. We will proceed with Nginx-Ingress deployment.
* Swith to `Kubernetes\kubernetes_ingress\deployments` folder which contains files for deploying the Ingress controller and load balancer.

In below step we are creating namespace and Service account for Nginx-Ingress, post which we will deploy RBAC and configMap.
```
kubectl apply -f common/ns-and-sa.yaml
```
### 3. Create RBAC, Default Secret And Config Map

```
kubectl apply -f common/
```

### 4. Deploy the Ingress Controller

We include two options for deploying the Ingress controller:
 * *Deployment*. Use a Deployment if you plan to dynamically change the number of Ingress controller replicas.
 * *DaemonSet*. Use a DaemonSet for deploying the Ingress controller on every node or a subset of nodes.

 **4.1 Create a DaemonSet**

When you run the Ingress Controller by using a DaemonSet, Kubernetes will create an Ingress controller pod on every node of the cluster.

```
kubectl apply -f daemon-set/nginx-ingress.yaml
```

### 5. Check that the Ingress Controller is Running

Check that the Ingress Controller is Running
Run the following command to make sure that the Ingress controller pods are running:
```
kubectl get pods --namespace=nginx-ingress
```

### 6. Get Access to the Ingress Controller

 **If you created a daemonset**, ports 80 and 443 of the Ingress controller container are mapped to the same ports of the node where the container is running. To access the Ingress controller, use those ports and an IP address of any node of the cluster where the Ingress controller is running.


 **6.1 Service with the Type LoadBalancer**

 Create a service with the type **LoadBalancer**. Kubernetes will allocate and configure a cloud load balancer for load balancing the Ingress controller pods.

**For AWS, run:**
```
kubectl apply -f service/loadbalancer-aws-elb.yaml
```

To get the DNS name of the ELB, run:
```
kubectl describe svc nginx-ingress --namespace=nginx-ingress
```

`OR`

```
kubectl get svc -n nginx-ingress 
```

You can resolve the DNS name into an IP address using `nslookup`:
```
nslookup <dns-name>
```
To map your domain mavenwebapp.com to the AWS LoadBalancer DNS name (a41d8b744dd6d462c883f1ed05decfdb-1245958895.ap-south-1.elb.amazonaws.com) on your local machine, you will need to update your hosts file. This will allow your local machine to resolve the domain name mavenwebapp.com to the IP address that your LoadBalancer is using.

Please note, this only works on your local machine. If you want this setup to be accessible by other machines, you'll need to use a DNS service like Amazon Route 53 or Google DNS to setup an A or CNAME record that points to the load balancer.

nslookup a41d8b744dd6d462c883f1ed05decfdb-1245958895.ap-south-1.elb.amazonaws.com   

sudo yum install bind-utils

The bind-utils package includes both dig and nslookup tools. After installation, you should be able to use the dig or nslookup command.

**In Windows:**

Run cmd as Administrator -> ipconfig /flushdns
This will flush the saved dns in the local system

Defile DNS for the local system, open the below path as admin:
C:\Windows\System32\Drivers\etc\hosts
```cmd
13.234.79.225 javawebapptest.com
13.126.61.94 javawebapptest.com
13.234.79.225 mavenwebapptest.com
13.126.61.94 mavenwebapptest.com
```

**In Mac:**
To flush the DNS
```bash
sudo dscacheutil -flushcache
```
Then add the IP address and desired URLs:
```bash
sudo nano /etc/hosts
```
### 7. Ingress Resource:

**7.1 Define path based or host based routing rules for your services.**

**Single DNS Sample with host and servcie place holders**
``` yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-resource-1
spec:
  ingressClassName: nginx
  rules:
  - host: javawebapp.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: javawebapp-service
            port:
              number: 80
``` 

**Multiple DNS Sample with hosts and servcies place holders**
``` yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-resource-1
spec:
  ingressClassName: nginx
  rules:
  - host: javawebapp.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: javawebapp-service
            port:
              number: 80
  - host: mavenwebapp.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: mavenwebapp-service
            port:
              number: 80	
``` 		  

**Path Based Routing Example**
``` yaml		  
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-resource-1
spec:
  ingressClassName: nginx
  rules:
  - host: "javawebapp.com"
    http:
      paths:
      - pathType: Prefix
        path: "/maven-web-application"
        backend:
          service:
            name: mavenwebapp-service
            port:
              number: 80
      - pathType: Prefix
        path: "/java-web-app"
        backend:
          service:
            name: javawebapp-service
            port:
              number: 80
	
``` 


`Make sure you have services created in K8's with type ClusterIP for your applications. Which your are defining in Ingress Resource`.

### Uninstall the Ingress Controller

 Delete the `nginx-ingress` namespace to uninstall the Ingress controller along with all the auxiliary resources that were created:
 ```
 kubectl delete namespace nginx-ingress
 ```

 **Note**: If RBAC is enabled on your cluster and you completed step 2, you will need to remove the ClusterRole and ClusterRoleBinding created in that step:

 ```
 kubectl delete clusterrole nginx-ingress
 kubectl delete clusterrolebinding nginx-ingress
 ```
