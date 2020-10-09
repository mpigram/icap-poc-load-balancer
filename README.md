
**Requirement for a load-balancer for the ICAP POC Solution**

**Objective**

In order to continue working on the ICAP performance testing with larger load until the full version is available, we need more deployments of the first C-ICAP POC solution and an Azure Load Balancer.

This public load balancer will then distribute ICAP port traffic 1433 to all the additional endpoints on port 1433

Repo containing the C-ICAP POC: https://github.com/filetrust/c-icap

**Key requirements:** 

- Additional Endpoints for ICAP
- Public load balancer for port 1433
- Initial config to distribute to all ICAP endpoints
- Documentation on how to add / remove endpoints from the load balancer
  
## Azure Traffic Manager solution
The traffic can be distributed among all the deployed ICAP servers using an Azure Traffic Manager. Here traffic manager acts as a load balancer in front of ICAP servers.
Different routing methods are available in Traffic manager

Performance: Use this method when your endpoints are deployed in different geographic locations, and you want to use the one with the lowest latency. 
Priority: Use this method when you want to select an endpoint which has highest priority and is available. 
Weighted: Use this method when you want to distribute traffic across a set of endpoints as per the weights provided.
Geographic: Use this method if you want to direct users to specific endpoints based on the geo locations where they are connecting from.
MultiValue: Use this method when you want to return multiple IPV4 or IPV6 endpoints. Works with external IPV4, IPV6 endpoints only.
Subnet: Use this method when you want to return an endpoint for a specific set of IPV4 range.

In our use case, as all servers are deployed in the same region, "Weighted" routing method is the simplest by giving equal weight to all servers.

### Populate below variables in the script given in next section

RESOURCE_GROUP - The resource group where traffic manager needs to be deployed

PROFILE_NAME - Name of the Traffic manager profile to use. 

    e.g. PROFILE_NAME="icap-server-tm"

DNS_NAME - DNS name to be used for the traffic manager. 

    e.g. DNS_NAME="icap-server"
    If "icap-server" is given here, the traffic manager gets the DNS as icap-server.trafficmanager.net

CONTAINER_FQDNS - Popluate the list of IP addresses or domain names of the ICAP servers.

    e.g. CONTAINER_FQDNS=("gw-icap01.westeurope.azurecontainer.io" "gw-icap02.westeurope.azurecontainer.io" "gw-icap03.westeurope.azurecontainer.io" "gw-icap04.westeurope.azurecontainer.io")


### Run below commands to create the Traffic manager profile and add all the ICAP server endpoints to it.

```
RESOURCE_GROUP=explore
PROFILE_NAME=icap-tm
DNS_NAME=icap-server

CONTAINER_FQDNS=("gw-icap01.westeurope.azurecontainer.io" "gw-icap02.westeurope.azurecontainer.io" "gw-icap03.westeurope.azurecontainer.io" "gw-icap04.westeurope.azurecontainer.io")

az network traffic-manager profile create --name $PROFILE_NAME \
    --resource-group $RESOURCE_GROUP \
    --routing-method Weighted \
    --unique-dns-name $DNS_NAME \
    --protocol HTTP \
    --status enabled \
    --port 80 \
    --status Enabled \
    --ttl 10

for CONTAINER_FQDN in "${CONTAINER_FQDNS[@]}" ; do
    echo $CONTAINER_FQDN
    az network traffic-manager endpoint delete -g $RESOURCE_GROUP --profile-name $PROFILE_NAME -n  $(echo $CONTAINER_FQDN | cut -d. -f1) --type externalEndpoints
    az network traffic-manager endpoint create -g $RESOURCE_GROUP -n $(echo $CONTAINER_FQDN | cut -d. -f1) \
        --profile-name $PROFILE_NAME \
        --type externalEndpoints \
        --weight 1 \
        --target $CONTAINER_FQDN
done

```

### To see the list of endpoints attached to the traffic manager, run below command - 

`az network traffic-manager endpoint list --profile-name $PROFILE_NAME --resource-group $RESOURCE_GROUP`

### To remove an endpoint(e.g. gw-icap05) from the traffic manager, run below command - 

`az network traffic-manager endpoint delete -g $RESOURCE_GROUP --profile-name $PROFILE_NAME -n gw-icap05 --type externalEndpoints`

### To add an endpoint to the traffic manager, run below command

```
FQDN=gw-icap05.westeurope.azurecontainer.io
ENDPOINT_NAME=gw-icap05
az network traffic-manager endpoint create -g $RESOURCE_GROUP -n $ENDPOINT_NAME \
        --profile-name $PROFILE_NAME \
        --type externalEndpoints \
        --weight 1 \
        --target $FQDN
```

### To test if the traffic manager is connecting to ICAP servers, run below command - 

`c-icap-client -i $DNS_NAME.trafficmanager.net -req any -s 'info?view=text'`

