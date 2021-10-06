# kubeturbo-openshift

## Requirements
* Kubernetes version 1.8 or higher, OpenShift release 3.4 or higher, including any k8s upstream compliant distribution
* Turbonomic Server version 5.9 or higher is installed, running, and the following information:
** Turbonomic Server URL https://
** Turbonomic username with administrator role, and password
** Turbonomic Server Version. To get this from the UI, go to Settings -> Updates -> About and use the numeric version such as “6.2.11” or “6.3” (No minor version needed 6.3+. Build details not required) To determine Turbo Server, CWOM Server and the kubeturbo image tags, refer to Turbonomic - CWOM - Kubeturbo version mappings
### Access and Permissions to create all the resources required:
* User needs cluster-admin cluster role level access to be able to create the following resources if required: namespace, service account, and cluster role binding for the service account.
### Repo Access and Network requirements. Refer to the Figure below “Turbonomic and Kubernetes Network Detail”.
* Instructions assume the node you are deploying to has internet access to pull the kubeturbo image from the DockerHub repository, or your environment is configured with a repo accessible by the node.[RedHat Container Catalog] (https://access.redhat.com/containers/#/registry.connect.redhat.com/turbonomic/kubeturbo)
* Kubeturbo pod will have access to the kubelet on every node
* Network: https + port=10250 (default). Or http + port=10255.
* Kubeturbo pod will have https/tcp access to the Turbonomic Server
* Proxies between kubeturbo and Turbonomic Server need to allow websocket communication.
* One kubeturbo pod will be deployed per cluster or per control plane when using stretch clusters. Kubeturbo pod will run with a service account with a cluster-admin role
* This pod will typically run with no more than 512 Mg Memory, less than 1 core or 1 GHz CPU, and maximum volume space of 10 G.
