# kubeturbo-openshift

The [Kubeturbo Operator](https://operatorhub.io/operator/kubeturbo "Kubeturbo Operator") can be quickly installed on your Openshfit cluster(s). This can be done through Openshift Console under the operator catalog, or directly through the CLI. Kubeturbo operator will run as a single pod deployment of kubeturbo per cluster, with the following resources installed in the Openshift Cluster:

* Custom Resource Definition (kubeturbo)
* Namespace
* Deployment	
* ReplicaSet
* Pod (kubeturbo)
* ConfigMap for kubeturbo to connect to the Turbonomic server
* Service Account
* Cluster Role 
* RoleBinding

## Requirements
* Kubernetes version 1.8 or higher, OpenShift release 3.4 or higher, including any k8s upstream compliant distribution
* Turbonomic Server version 5.9 or higher is installed, running, and the following information:
  * Turbonomic Server URL https://
  * Turbonomic username with administrator role, and password
  * Turbonomic Server Version. To get this from the UI, go to Settings -> Updates -> About and use the numeric version such as “6.2.11” or “6.3” (No minor version needed 6.3+. Build details not required) To determine Turbo Server, CWOM Server and the kubeturbo image tags, refer to Turbonomic - CWOM - Kubeturbo version mappings
* User needs cluster-admin cluster role level access to be able to create the following resources if required: 
  * namespace
  * service account,
  * cluster role binding for the service account.
   * Note: kubeturbo will run the with the [following cluster level permissions](https://github.com/ericbannon/kubeturbo-openshift/blob/main/deploy/clusterole.yaml "Permissions") 
* Instructions assume the node you are deploying to has internet access to pull the kubeturbo image from the DockerHub repository, and your environment is configured with a repo accessible by the node.
  * (https://access.redhat.com/containers/#/registry.connect.redhat.com/turbonomic/kubeturbo)
* Kubeturbo pod will have access to the kubelet on every node
* Network: https + port=10250 (default). Or http + port=10255.
* Kubeturbo pod will have https/tcp access to the Turbonomic Server
* Proxies between kubeturbo and Turbonomic Server need to allow websocket communication.
* One kubeturbo pod will be deployed per cluster or per control plane when using stretch clusters. Kubeturbo pod will run with a service account with a cluster-admin role
* This pod will typically run with no more than 512 Mg Memory, less than 1 core or 1 GHz CPU, and maximum volume space of 10 G.

## Instructions

### Step 1: Create a namespace for kubeturbo

Create a namespace with any name you would like. In our example, we will use the name 'turbo'. 

```
oc create ns turbo
```

### Step 2: Deploy kubeturbo Operator through Openshift Console (preferred method)

From the Openshift Console, go to OperatorHub, select the project you just created from step 1, and search for kubeturbo. Select the non-community, non-marketplace option. Proceed with the installation of the default configuration in your project. 

![image](https://user-images.githubusercontent.com/34694236/136431873-4f63f032-5198-445f-9b27-1bcefa65d820.png)

### Step 3: Create a kubeturbo CRD resource from the Installed Operator

1. Go to Installed Operators, and click into the Kubeturbo operator that was installed in step 2
2. Click 'create instance' under Provided APIs for kubeturbo operator 

![image](https://user-images.githubusercontent.com/34694236/136433519-1ed63794-1271-4b44-ac18-b4ff6a2db960.png)

3. In the next screen, you must define the following values for the CRD. 

Key                 |  Value                   | Description           
-----------------   | --------------------     | -------------
turboServer         | https://Turbo_server_URL | The turbonomic platform's https address (lb, route, external IP, etc...) 
opsManagerUserName: | Turbo_username           | default is administrator
opsManagerPassword: | Turbo_password           | configured during setup of t8c
targetName:         | Name_Each_Cluster        | a unique name for the managed kubeturbo cluster



