# DEMO SETUP

## Create a resource group
$RESOURCE_GROUP_NAME="AksNetworkPolicyDemoRg"
$CLUSTER_NAME="KubernetesNPDemo"
$LOCATION="canadaeast"

az group create --name $RESOURCE_GROUP_NAME --location $LOCATION

## Create a virtual network and subnet
az network vnet create `
    --resource-group $RESOURCE_GROUP_NAME `
    --name myVnet `
    --address-prefixes 10.0.0.0/8 `
    --subnet-name myAKSSubnet `
    --subnet-prefix 10.240.0.0/16

## Create a service principal and read in the application ID
$SP=$(az ad sp create-for-rbac --output json)
$SP_ID=$(echo $SP | jq -r .appId)
$SP_PASSWORD=$(echo $SP | jq -r .password)

## Wait 15 seconds to make sure that service principal has propagated
echo "Waiting for service principal to propagate..."
sleep 15

## Get the virtual network resource ID
$VNET_ID=$(az network vnet show --resource-group $RESOURCE_GROUP_NAME --name myVnet --query id -o tsv)

## Assign the service principal Contributor permissions to the virtual network resource
az role assignment create --assignee $SP_ID --scope $VNET_ID --role Contributor

## Get the virtual network subnet resource ID
$SUBNET_ID=$(az network vnet subnet show --resource-group $RESOURCE_GROUP_NAME --vnet-name myVnet --name myAKSSubnet --query id -o tsv)

## Create the AKS cluster and specify the virtual network and service principal information
**Enable network policy by using the `--network-policy` parameter**
az aks create `
    --resource-group $RESOURCE_GROUP_NAME `
    --name $CLUSTER_NAME `
    --node-count 1 `
    --generate-ssh-keys `
    --network-plugin azure `
    --service-cidr 10.0.0.0/16 `
    --dns-service-ip 10.0.0.10 `
    --docker-bridge-address 172.17.0.1/16 `
    --vnet-subnet-id $SUBNET_ID `
    --service-principal $SP_ID `
    --client-secret $SP_PASSWORD `
    --network-policy calico `
    --node-vm-size Standard_B2ms `

## Configure kubectl to connect to the Kubernetes cluster 
az aks get-credentials --resource-group $RESOURCE_GROUP_NAME --name $CLUSTER_NAME
