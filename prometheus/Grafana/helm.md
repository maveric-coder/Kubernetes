# Install using Helm

## Add helm repo

`helm repo add grafana https://grafana.github.io/helm-charts`

## Update helm repo

`helm repo update`

## Install helm 

`helm install grafana grafana/grafana`

```zsh
kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```
```zsh
AJYzgFIBregRCHHduMC7XTMFD0VFr5Nl7t8lNULX
```

## Expose Grafana Service
```zsh
kubectl expose service grafana --type=NodePort --target-port=3000 --name=grafana-ext
```
Once inside, select prometheus as your data source.

And in dashboard, import some existing dashboards eg. - 3662, 1860, **315**
