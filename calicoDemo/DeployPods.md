# Deploy pods for network policy demo

$ns="demo"
kubectl create ns $ns
kubectl config set-context --current --namespace $ns
kubectl apply -f db-api-web-deployments.yaml

# We have 3 Pods and 3 Services:
kubectl get pod,svc -n $ns

# check if web pod is accessible
curl $(kubectl get svc web -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

# see that any pods could communicate with each others even externally, let's run few successful commands:
# $RANDOM is a bash function (not a constant) that returns a random signed 16 bit integer (from 0 through 32767).

$RANDOM = Get-Random -Maximum 32767
kubectl run curl-$RANDOM --image=radial/busyboxplus:curl --rm -it --generator=run-pod/v1

# curl www.microsoft.com
# curl http://db:15984
# exit
