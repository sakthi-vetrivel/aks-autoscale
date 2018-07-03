# Cluster Autoscaler on AKS
The cluster autoscaler on Azure scales worker nodes within any specified autoscaling group. It will run as a Kubernetes deployment in your cluster. This README will go over some of the necessary steps required to get the cluster autoscaler up and running. Once you've got your AKS cluster up, follow these next steps.

### Kubernetes Version

Kubernetes v1.10.X and Cluster autoscaler v1.2+  are required to run on Azure.

### Permissions

Get Azure credentials by running the following command

```sh
# replace <subscription-id> with yours.
az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/<subscription-id>" --output json
```

### values.yaml
Fill the values of the values.yaml file, including

- ClientID: `<app-id>` from permissions step
- ClientSecret: `<app-password>` from permissions step
- ResourceGroup: `<resource-group-name>` (Note: Please use lower case)
- SubscriptionID: `<subscription-id` (Can be found using the portal)
- TenantID: `<tenant-id>`
- ClusterName: `<clustername>`
- NodeResourceGroup: `<node-resource-group>` (Please use the value of the kubernetes.azure.com/cluster label verbatim (case sensitive), usually of the form MC_{cluster-name}\_{resource-group}\_{location}). Note that this is different from the resource group above. Steps on how to retrieve this below.

The vmType param determines the kind of service we are interacting with. In this case, it's AKS. Next, we want to set the maximum and minimum number of nodes for our cluster autoscaler. This is set using MinNodes and MaxNodes, currently set at 1 and 5 for the default values. The NodePoolName parameter is the name of the node pool you're interested in scaling.

Get a node pool name by extracting the value of the label **agentpool**

  ```sh
  kubectl get nodes --show-labels
  ```

Get a node resource group name by extracting the value of the label **kubernetes.azure.com/cluster**

### Deployment
Then deploy cluster-autoscaler by running:

```sh
helm install ./aks-autoscale -f values.yaml
```
