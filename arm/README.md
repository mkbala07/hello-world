# Cloud Pak for Data 4.0 on Azure

Cloud Pak for Data is an end to end platform that helps organizations in their journey to AI. It enables data engineers, data stewards, data scientists, and business analysts to collaborate using an integrated multiple-cloud platform.
Cloud Pak for Data uses IBM’s deep analytics portfolio to help organizations meet data and analytics challenges. The required building blocks (collect, organize, analyze, infuse) for information architecture are available using Cloud Pak for Data on Azure.

Cloud Pak for Data uses cloud native services and features including VNets, VPCs, Availability Zones, security groups, Managed Disks, and Load Balancers to build a highly available, reliable, and scalable cloud platform.

This deployment guide provides step-by-step instructions for deploying IBM Cloud Pak for Data on a Azure ARO.

This reference deployment provides ARM templates to deploy Azure ARO Cluster with cloupak for data installation.

 - A Azure ARO cluster.
 - A highly available storage infrastructure with NFS or OpenShift Container Storage.
 - Scalable OpenShift compute nodes running Cloud Pak for Data services. See [Services](#cloud-pak-for-data-services) for the services that are enabled in this deployment.

## Cost and licenses
Cloud Pak for Data offers a try and buy experience.
The automated template deploys the Cloud Pak for Data environment by using Azure Resource Manager templates.
The deployment template includes configuration parameters that you can customize. Some of these settings, such as instance count, will affect the cost of the deployment. For cost estimates, see the pricing page for each Azure service you will be using. Prices are subject to change.

**TRIAL:**<br/>
To request a 60 day trial license of Cloud Pak for Data please use the following link - [IBM Cloud Pak for Data Trial](https://www.ibm.com/account/reg/us-en/signup?formid=urx-42212).
Instructions to use your trial license/key are provided in the section - [IBM Cloud Pak for Data Trial key](#IBM-Cloud-Pak-for-Data-Trial-key).
Beyond the 60 day period, you will need to purchase the Cloud Pak for Data by following the instructions in the 'Purchase' section below.

**PURCHASE:**<br/>
To get pricing information, or to use your existing Cloud Pak for Data entitlements, contact your IBM sales representative at 1-877-426-3774. 
Note: Cloud Pak for Data license will include entitlements to RHEL and Openshift.

## Deployment on Azure

### Prerequisites
- Azure Client (az cli)
- Azure Service Principal, with Contributor and User Access Administrator.

The Service Principal can be created by running the azure CLI commands from any host where azure CLI is installed.

  * Create Azure Service Principal with `Contributor` and `User Access Administrator` roles.
    * **Option 1:** using the script provided in the `scripts` folder:
      ```bash
      az login
      scripts/createServicePrincipal.sh -r "Contributor,User Access Administrator"
      ```
    * **Option 2:** running the commands manually:
      * Create Service Principal, using your Azure Subscription ID, and save the returned json:
        ```bash
        az login
        az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/<subscription_id>"
        ```
      * Get `Object ID`, using the AppId from the Service Principal just created:
        ```bash
        az ad sp list --filter "appId eq '<app_id>'"
        ```
      * Assign `User Access Administrator` roles, using the `Object Id`.
        ```bash
        az role assignment create --role "User Access Administrator" --assignee-object-id "<object_id>"
        ```
    * Save the `ClientID` and `ClientSecret` from either option.

2. [Download](https://cloud.redhat.com/openshift/install/pull-secret) a pull secret. Create a Red Hat account if you do not have one.

3. [Sign up](https://www.ibm.com/account/reg/us-en/signup?formid=urx-42212) for Cloud Pak for Data Trial Key if you don't have the entitlement api key

4. Read and agree to the [license terms](https://ibm.biz/BdqyB2)

### Deploy

* There are 2 methods to deploy this script. 

## Method 1:

[![Deploy To Azure](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.svg?sanitize=true)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmuthukumarbala07%2Fhello-world%2Fmaster%2Farm%2Faro%2Fazuredeploy.json)

hello-world/blob/master/arm/aro/azuredeploy.json

* Click on the above button and input the variables in the template in the azure portal 

## Method 2:

* Enter variables in `azuredeploy.parameters.json`. See the [PARAMETERS.md](./PARAMETERS.md) for detailed descriptions.
```bash
az login
./az-group-deploy.sh -a [folder_name] -l <region> -g <resource_group> -e /path/to/parameters_file
```
Example:
```bash
./az-group-deploy.sh -a aro -l westus2 -g myocp-rg -e aro/parameters.json
```

* The webconsole URL can be found in the `ResourceGroup`>`Deployments`>`azuredeploy`>`Outputs`.

* Access the respective console on a web browser.
* example:

![Alt text](images/Output.png?raw=true "output")

<br/>

Use the default credentials for Cloud Pak for Data `admin` / `password` to log in to CPD console. Ensure to change the password after your first login.

## Cloud Pak for Data Services
You can browse the various services that are available for use by navigating to the services catalog page in Cloud Pak for Data

![Alt text](images/services.png?raw=true "parameters2")

As part of the deployment, the following services can be enabled:
•	Watson Studio Local
•	Watson Knowledge Catalog
•	Watson Machine Learning
•	Data Virtualization
•	Watson Openscale
•	Cognos Dashboard
•	Apache Spark


To get information on various other services that are available, you can visit [Cloud Pak for Data Service Catalog](https://www.ibm.com/support/producthub/icpdata/docs/content/SSQNUZ_current/cpd/svc/services.html)

## Troubleshoot 

### Insufficient CPU/Memory (Scale Up the cluster)
**Switch to `openshift-machine-api` namespace:**
```
$oc project openshift-machine-api
```

**Fetch machineset:**
```
$oc get machineset
```

**Scale up particular machineset:**
```
$oc scale machineset <machine_set_from_above_cmd> --replicas=<number_of_replicas_you_want_to_create>
```
