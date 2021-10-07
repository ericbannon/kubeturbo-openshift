# Quickstart Install Guide: Turbonomic with Kubeturbo on Openshift 

The following guide in intended to provide quick start intructions on how to install and configure Turbonomic with Kubeturbo on Openshift. For full intstructions on Multi-Node Deployment of Turbonomic on Kubernetes, please visit the [Turbonomic Wiki Instructions](https://github.com/turbonomic/t8c-install/wiki/1.-Turbonomic-MultiNode-Deployment-Overview "tc8s instructions").

![image](https://user-images.githubusercontent.com/34694236/136449047-31b6d00e-18f5-4854-86dd-c9dbd9c78948.png)

## Turbonomic 

The [Turbonomic Platform Operator](https://operatorhub.io/operator/t8c "Turbonomic Platform Operator") can be quickly installed on your Openshfit cluster(s). This can be done through Openshift Console under the operator catalog, or directly through the CLI. The Turbonomic Platform is deployed as a set of containerized micro-services owned and managed by the Operator, and covers varios components across mediation, abtraction, analysis, automation, presentation, and more. For a more detailed description, the Turbonomic architecture can be viewed [here](https://github.com/turbonomic/t8c-install/wiki/1.-Turbonomic-MultiNode-Deployment-Overview#planning-the-deployment "tc8s-arch"). The following meaningful resources will be installed into the Openshift Cluster:

* Operator Deployment (t8c-operator)
* Custom Resource Definition (xl-release)
 * 21 Deployments & Services
 * 5 ConfigMaps
 * 10 PersistentVolumeClaims 
 * Service Account
 * Cluster Role 
 * RoleBinding
 * 2 Openshift Routes (one for api service, and one for topology-processing service) 
 * Secrets

For detailed pre-requisites, please visit [wiki page here](https://github.com/turbonomic/t8c-install/wiki/2.-Prerequisites "pre-reqs"), otherwise continue to the proceeding sections. 

## Kubeturbo

The [Kubeturbo Operator](https://operatorhub.io/operator/kubeturbo "Kubeturbo Operator") can be quickly installed on your Openshfit cluster(s). This can be done through Openshift Console under the operator catalog, or directly through the CLI. Kubeturbo operator will run as a single pod deployment of kubeturbo per cluster, with the following meaningful resources installed into the Openshift Cluster:

* Operator Deployment (kubeturbo-operator)
* Custom Resource Definition (kubeturbo-release)
 * 1 Deployment (kubeturbo pod) & Service	
 * 1 ConfigMap (for kubeturbo to connect to the Turbonomic topology processing server)
 * Service Account
 * Cluster Role 
 * RoleBinding
 * Secrets

### Requirements
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

## I. Install The Turbonomic Platform

## II. Install Kubeturbo

#### Step 1: Create a namespace for kubeturbo

Create a namespace with any name you would like. In our example, we will use the name 'turbo'. 

```
oc create ns turbo
```

#### Step 2: Deploy kubeturbo Operator through Openshift Console (preferred method)

From the Openshift Console, go to OperatorHub, select the project you just created from step 1, and search for kubeturbo. Select the non-community, non-marketplace option. Proceed with the installation of the default configuration in your project. 

![image](https://user-images.githubusercontent.com/34694236/136431873-4f63f032-5198-445f-9b27-1bcefa65d820.png)

#### Step 3: Create a kubeturbo CRD resource from the Installed Operator

1. Go to Installed Operators, and click into the Kubeturbo operator that was installed in step 2
2. Click 'create instance' under Provided APIs for kubeturbo operator 

![image](https://user-images.githubusercontent.com/34694236/136433519-1ed63794-1271-4b44-ac18-b4ff6a2db960.png)

3. In the next screen, you must define the following values for the CRD. 

Key                 |  Value                   | Description           
-----------------   | --------------------     | -------------
serverMeta.turboServer         | https://Turbo_server_URL | The turbonomic platform's https address (lb, route, external IP, etc...) 
restAPIConfig. opsManagerUserName: | Turbo_username           | default is administrator
restAPIConfig.opsManagerPassword: | Turbo_password           | configured during setup of t8c
targetConfig.targetName:         | Name_Each_Cluster        | a unique name for the managed kubeturbo cluster
args.sccsupport                  | *                        | Include a value of * in order to enable support of actions that move container pods

```
Note: It is reccomended to configure the argument for sccsupport if you plan on testing move actions for pods. 
```  

