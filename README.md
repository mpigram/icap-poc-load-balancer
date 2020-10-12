
**Requirement for a load-balancer for the ICAP POC Solution**

**Objective**

In order to continue working on the ICAP performance testing with larger load until the full version is available, we need more deployments of the first C-ICAP POC solution and an Azure Load Balancer.

This public load balancer will then distribute ICAP port traffic 1433 to all the additional endpoints on port 1433

Repo containing the C-ICAP POC: https://github.com/filetrust/c-icap

**Key requirements:** 

- Additional Endpoints for ICAP
- Public load balancer for port 1444
- Initial config to distribute to all ICAP endpoints
- Documentation on how to add / remove endpoints from the load balancer
  
