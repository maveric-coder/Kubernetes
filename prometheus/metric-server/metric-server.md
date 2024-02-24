# prometheus-kube-state-metrics

kube-state-metrics (KSM) is a simple service that listens to the Kubernetes API server and generates metrics about the state of the objects. 

kube-state-metrics is about generating metrics from Kubernetes API objects without modification. This ensures that features provided by kube-state-metrics have the same grade of stability as the Kubernetes API objects themselves.

It provides much more information than what can be captured by prometheus. It gives RAW data from kube-API server.

Let's expose the prometheus-kube-state-metrics service and observe the output.

```sh
kubectl expose service prometheus-kube-state-metrics --type=NodePort --target-port=8080 --name=state-metric-ext
```
Observe the dedicated port
```sh
kubectl get all
```
Head to browser and click on metrics and observe the data.