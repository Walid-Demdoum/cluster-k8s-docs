## Minikube base commands
```bash
minikube start
minikube dashboard
minikube dashboard --url --port=5555
kubectl proxy --address='0.0.0.0' --disable-filter=true
```
# kind: 
1.    namespaces
2.    pods
3.    deployments
4.    services
5.    configmaps
6.    pv
7.    pvc

## Kubectl commands
```bash
kubectl get "kind"
kubectl delete "kind" "kind-name"
kubectl logs "kind-name"
```

## Namespace commands
### Check current namespace
```bash
kubectl config get-contexts
```
### Switch to a namespace
```bash
kubectl config set-context --current --namespace="namespace-name"
```
## Commands to manage pods
### Create kinds from a file
```bash
kubectl apply -f <file/to/*.yaml>
```
### Port forward a service
```bash
kubectl port-forward svc/<service> <service-port>:<desired-port>
```
### Docker registry connection/Must be applied for every namespace
```bash
kubectl create secret docker-registry my-registry-secret \
  --docker-server=registry.example.com \
  --docker-username=my-username \
  --docker-password=my-password \
  --docker-email=my-email@example.com
```