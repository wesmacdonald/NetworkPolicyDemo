# Let's apply the first Network Policy that should be for any namespace
# Deny all ingress and egress traffic
kubectl apply -f deny-all-netpol.yaml

# check web
curl --connect-timeout 2 $(kubectl get svc web -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

$RANDOM = Get-Random -Maximum 32767
kubectl run curl-$RANDOM --image=radial/busyboxplus:curl --rm -it --generator=run-pod/v1
# curl --connect-timeout 2 www.microsoft.com
# curl --connect-timeout 2 http://db:15984
# exit

