<h1 align="center">
  <br>
  Aks and Application Gateway Ingress Controller
  
  <br>
</h1>

# Architecture For traffic Routing 

<img src='https://learn.microsoft.com/en-us/azure/application-gateway/media/application-gateway-ingress-controller-overview/architecture.png'/>


# Step by step

```
# Export your variables
export rgName=dev
export aksName=dev-aks
export pipName=dev-pip
export appgwVnetName=dev-appgw-vnet
export appgwSnetName=dev-appgw-snet-01
export location=westeurope
export appgwName=dev-appgw
export wafPolicyName=dev-waf-policy
export aksVnetName=dev-vnet


```


# Create public ip
```bash
az network public-ip create -n $pipName -g $rgName -l $location --allocation-method Static --sku Standard
```


# Create vnet
```bash
az network vnet create -n $appgwVnetName -g $rgName -l $location --address-prefix 10.0.0.0/16 --subnet-name $appgwSnetName --subnet-prefix 10.0.0.0/24
```


# Create WAF policy
```bash
az network application-gateway waf-policy create --name $wafPolicyName --resource-group $rgName
```


# Create application gateway
```bash
az network application-gateway create -n $appgwName -l westeurope -g $rgName --sku WAF_v2 --public-ip-address $pipName --vnet-name $appgwVnetName --subnet $appgwSnetName --priority 100 --waf-policy $wafPolicyName
```


# Enable Application Gateway Ingress Controller on AKS
```bash
appgwId=$(az network application-gateway show -n $appgwName -g $rgName -o tsv --query "id")
az aks enable-addons -n $aksName -g $rgName -a ingress-appgw --appgw-id $appgwId
```

# Create vnet peerings (only if the appGateway and the cluster belong to different vnets)
```bash
aksVnetId=$(az network vnet show -n $aksVnetName -g $rgName -o tsv --query "id")
az network vnet peering create -n AppGWtoAKSVnetPeering -g $rgName --vnet-name $appgwVnetName --remote-vnet $aksVnetId --allow-vnet-access

appGWVnetId=$(az network vnet show -n $appgwVnetName -g $rgName -o tsv --query "id")
az network vnet peering create -n AKStoAppGWVnetPeering -g $rgName --vnet-name $aksVnetName --remote-vnet $appGWVnetId --allow-vnet-access
```


# Let's deploy a simple webserver application  (git clone repo)
```bash 
cd deployments
kubectl create namespace nginx
kubectl apply -f nginx-deployment.yaml -n nginx
kubectl apply -f nginx-service.yaml -n nginx
kubectl apply -f nginx-ingress.yaml -n nginx
```


# Getting your Ingress IP
```bash 
kubectl get ingress -n nginx
```



