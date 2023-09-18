We will implement ingress

deploy lb-java and lb-maven first and check the outputs

delete both deployments

single lb should be able to route

in both apps change from lb to ClusterIP

and deploy


other load balancers:
HAProxy,
GKE ingress
contour
ingress LB



go to kubernetes_ingress folder which is copied from https://github.com/kubernetes/ingress-nginx.git

in deployment apply common