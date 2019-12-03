# database should be accessible only from API on port 5984 and has outbound access to nothing:
kubectl apply -f db-netpol.yaml

# Verify that the database doesn't have access to anything:
$ns="demo"
$RANDOM = Get-Random -Maximum 32767
kubectl run curl-$RANDOM --image=radial/busyboxplus:curl --labels app=db --rm -it --generator=run-pod/v1 -n $ns
# curl --connect-timeout 2 http://web
# curl --connect-timeout 2 www.microsoft.com
# exit

# Want API to have access only to DB on port 5984 and be accessible only from WEB on port 8080:
kubectl apply -f api-netpol.yaml

# add a Label on the kube-system namespace:
kubectl label ns kube-system name="kube-system"
# To be able to reach out to another Pod via its service name exposure we need to add an Egress rule for the DNS resolver (with the label k8s-app=kube-dns) in the kube-system namespace. We saw that we need to add a label name=kube-system on the kube-system Namespace.

# Verify that the API has access to the database (DB) but doesn't have access to WEB or internet:
kubectl run curl-$RANDOM --image=radial/busyboxplus:curl --labels app=api --rm -it --generator=run-pod/v1
# curl http://db:15984
# curl --connect-timeout 2 http://web:80
# curl --connect-timeout 2 www.microsoft.com
# exit


# Configure WEB to have access only to API on port 3000 and be accessible only from internet on port 80:
kubectl apply -f web-netpol.yaml

# Verify that WEB has access to API but doesn't have access to DB or Internet:
kubectl run curl-$RANDOM --image=radial/busyboxplus:curl --labels app=web --rm -it --generator=run-pod/v1
# curl http://api:8080
# curl --connect-timeout 2 www.microsoft.com
# curl --connect-timeout 2 http://db:15984
# exit

# Verify that WEB is publicly accessible:
curl $(kubectl get svc web -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
