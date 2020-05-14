# IBM Business Automation Workflow on Containers Demo Pattern Tech Preview

This repository includes folders and resources to help you install IBM Business Automation Workflow on Containers for demonstration purposes on Red Hat OpenShift Cloud Platform (OCP).

Business Automation Workflow uses an operator, which is a Kubernetes feature that makes it simpler to install and update. To install the demo pattern with the Business Automation Workflow on Containers operator, an OCP administrator user runs a script to set up a cluster and then works with an installer (a non-administrator user) to help them run a deployment script.  

> **Note**: The scripts can be used only on a Linux-based operating system: Red Hat (RHEL), CentOS, and macOS.

The configuration include these components:

- IBM Business Automation Workflow Server component
- IBM Java Messaging Service (JMS) component
- IBM Process Federation Server component
- IBM User Management Service (UMS) component
- IBM Resource Registry component
- IBM FileNet Content Manager component
- IBM Business Automation Navigator component

The demo pattern deployment provisions all the required services, such as IBM Db2 and OpenLDAP, with the default values so there is no need to prepare these in advance. 

Use the following sections to install or uninstall the demo pattern:
- [Install a deployment pattern](#install-a-deployment-pattern)
- [Uninstall a deployment pattern](#uninstall-a-deployment-pattern)

# Install a deployment pattern

- [Step 1: Plan and prepare (OCP cluster administrator)](#step-1-plan-and-prepare-ocp-cluster-administrator)
- [Step 2: Get access to the container images (installer)](#step-2-get-access-to-the-container-images-installer)
- [Step 3: Run the deployment script (installer)](#step-3-run-the-deployment-script-installer)
- [Step 4: Verify that the containers are running](#step-4-verify-that-the-containers-are-running)
- [Step 5: Access the services](#step-5-access-the-services)
- [Step 6: List the default LDAP users and passwords](#step-6-list-the-default-ldap-users-and-passwords)
- [Step 7: Post-installation tasks](#step-7-post-installation-tasks)
- [Troubleshoot](#troubleshoot)
- [Uninstall a deployment pattern](#uninstall-a-deployment-pattern)

## Step 1: Plan and prepare (OCP cluster administrator)

The cluster administrator must check the system requirements and make sure that the system can host and run the deployment. The administrator and the installer (non-administrator user) should plan the deployment together.

The cluster setup script creates an OpenShift project (namespace), applies the custom resource definition (CRD), adds the specified user to the ibm-cp4a-operator role, binds the role to the service account, and applies a security context constraint (SCC) for the Business Automation Workflow on Containers. 

The script also prompts the administrator to take note of the cluster host name and a dynamic storage class on the cluster. These names must be provided to the installer who runs the deployment script. 

Use the following steps to complete the preparation:

1. Go to the `BAW-Ctnrs-TP` directory:
   ```bash
   $ cd BAW-Ctnrs-TP
   ```
2. Log in to the target cluster as the `<cluster-admin>` user:
   ```bash
   $ oc login https://<cluster-ip>:<port> -u <cluster-admin> -p <password>
   ```
3. Run the cluster setup script from where you downloaded the GitHub repository, and follow the prompts in the command window:
   ```bash
   $ cd scripts
   $ ./cp4a-clusteradmin-setup.sh
   ```
   
   1. Enter the name for a new project or an existing project (namespace), for example `cp4a-demo`.
   2. Enter an existing non-administrator username in your cluster to run the deployment script, for example `cp4a-user`.
   
   When the script is finished, all the available storage class names and the infrastructure node name are shown. Keep track of the hostname and the class name that you want to use for the installation, because they are both needed for the deployment script.

## Step 2: Get access to the container images (installer)

To get access to the Business Automation Workflow on Containers images, you must download the package (.tgz file) from Passport Advantage (PPA) and push the images to a local Docker registry. The deployment script asks for the entitlement key or user credentials for the local registry.

### Load the downloaded PPA images to the local Docker registry

For instructions for downloading the PPA images, see [Downloading the packages](README.md).

1. Check that you can run a Docker command:
   ```bash
   $ docker ps
   ```
2. Log in to the Docker registry with a token:
   ```bash
   $ docker login $(oc registry info) -u <ADMINISTRATOR> -p $(oc whoami -t)
   ```
   > **Note**: You can connect to a node in the cluster to resolve the `docker-registry.default.svc` parameter.

3. Run a `kubectl` command to make sure that you have access to Kubernetes:
   ```bash
   $ kubectl cluster-info
   ```
4. Identify the local registry:
   ```bash
   $ oc registry info --internal
   ```
5. Run the [`scripts/loadPrereqImages.sh`] script to load Db2 and OpenLDAP images into your Docker registry. Specify the -r parameter on the command line and use the result of step 4 as the value:
   ```bash
   $ ./loadPrereqImages.sh -r docker-registry.default.svc:5000/<project-name>
   ```
6. Run the [`scripts/loadimages.sh`] script to load the images into your Docker registry. Specify the two mandatory parameters on the command line:

   ```
   -p  PPA archive files location or archive filename
   -r  Target Docker registry and namespace
   -l  Optional: Target a local registry
   ```

   The following example shows the input values on the command line:
   ```
   $ ./loadimages.sh -p <PPA-ARCHIVE>.tgz -r docker-registry.default.svc:5000/<project-name>
   ```

   > **Note**: The `project-name` variable is the name of the project created by the cluster setup script. To use an external Docker registry, keep track of the OCP Docker registry service name or the URL to the Docker registry, so that you can enter it in the deployment script. If you connect remotely to the OCP cluster from a Linux host or VM, you must have Docker and the OpenShift command line interface (CLI) installed. If you have access to the master node on the OCP v3.11 cluster, the CLI and Docker are already installed. 

   > **Note**: Make sure that you use the same Docker registry in step 5 and step 6.

7. Check that the images are pushed correctly to the registry:
    ```bash
    $ oc get is
    ```

## Step 3: Run the deployment script (installer)

Based on the pattern, the deployment script prepares the environment before installing the containers. The script applies a customized custom resource file, which is deployed by the Business Automation Workflow on Containers operator. The deployment script prompts you to enter values to get access to the container images and to select what is installed with the deployment.

1. Log in to the OCP 3.11 cluster with the non-administrator user that the cluster administrator used in Step 1, for example:
   ```bash
   $ oc login -u cp4a-user -p cp4a-user
   ```
2. Run the deployment script from the local directory where you downloaded the GitHub repository, and follow the prompts in the command window:
   ```bash
   $ cd scripts
   $ ./cp4a-deployment.sh
   ```
   
> **Note:** The deployment script uses a custom resource template file for the demo pattern. The template name includes "demo" and is found in the [descriptors/patterns] folder. The custom resource files are configured by the deployment script. However, you can copy the template, configure it by hand, and apply the file from the kubectl command line if you want to run the steps manually.

## Step 4: Verify that the containers are running

The operator reconciliation loop can take some time. It will be about an hour before the whole system is running. 

1. You can open the operator log to view the progress:
   ```bash
   $ oc logs <operator pod name> -c operator -n <project-name>
   ```
   
2. Monitor the status of your pods by using the following command:
   ```bash
   $ oc get pods -w
   ```

3. When all the pods are *Running*, you can access the status of your services by using the following command:
   ```bash
   $ oc status
   ```

## Step 5: Access the services

When all the containers are running, you can run a post-deployment script to get access to the services.

1. Go to the `BAW-Ctnrs-TP` directory on your local machine.
   ```bash
   $ cd BAW-Ctnrs-TP
   ```
2. Log in to the OCP cluster with the non-administrator user that the administrator created in Step 1. For example:
   ```bash
   $ oc login -u cp4a-user -p cp4a-user
   ```
3. Run the following commands, which print out the routes created by the pattern and the user credentials that you need to log in to the web applications to get started:
   ```bash
   $ oc get route
   $ oc get secret ibm-fncm-secret -o template --template '{{ index .data "appLoginUsername" }}' | base64 -d
   $ oc get secret ibm-fncm-secret -o template --template '{{ index .data "appLoginPassword" }}' | base64 -d
   ```
   
## Step 6: List the default LDAP users and passwords

After you get the service URLs and admin user credentials, you can also get a list of LDAP users.

1. Get the `<deployment-name>` by running the following command. The `<deployment-name>` is the name of the pattern that you installed:
   ```bash
   $ oc get icp4acluster
   ```
2. Get the usernames and passwords for the LDAP users:
   ```bash
   $ oc get cm <deployment-name>-openldap-customldif -o yaml
   ```
   
## Step 7: Post-installation tasks

If the pattern that you installed includes IBM Business Automation Navigator and the User Management Service (UMS), you must configure the single sign-on (SSO) logout for the Admin desktop. For more information, see [Configuring Logout SSO between IBM Business Automation Navigator and UMS](https://www.ibm.com/support/knowledgecenter/SSYHZ8_20.0.x/com.ibm.dba.install/op_topics/tsk_configbanumsssok8s.html]).

To accept the certificate, you must first visit the Process Federation Server federated Process Portal and then visit the Business Automation Workflow Process Portal. For example, log in at https://pfs.demo-project.example1.fyre.ibm.com/rest/bpm/federated/v1/systems and then log in at https://baw.demo-project.example1.fyre.ibm.com/ProcessPortal.

   
### Known Issue: Case Init job failed to complete

To fix this issue, use the following  command to get the Business Automation Navigator URL:
```bash
$ oc get route
```
Use the user and password from step 6 to log in to the Navigator desktop, for example https://navigator-demo-project.example1.fyre.ibm.com/navigator/?desktop=admin
Select "admin Desktop" and edit, then select "Set as default desktop" and click Save.

Delete the current case init job:
```bash
$ oc delete job xxxbaw-case-init-jobxxx
```


## Troubleshoot 
If you experience issues, see [Troubleshooting a deployment for demonstration purposes](BAW-on-Ctnrs-TP-Troubleshooting.md).

# Uninstall a deployment pattern

To uninstall the deployment, you can delete the namespace by running the following command:
```bash
$ oc delete project <project-name>
```
To uninstall the cluster role, cluster role binding, and the custom resource definition, run the following commands:
```bash
$ oc delete clusterrolebinding <NAMESPACE>-cp4a-operator 
$ oc delete clusterrole ibm-cp4a-operator 
$ oc delete crd icp4aclusters.icp4a.ibm.com
```

