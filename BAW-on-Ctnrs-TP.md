# IBM Business Automation Workflow on Containers Tech Preview
Learn how to configure and customize IBM Business Automation Workflow to run your applications, case solutions, and toolkits in a container environment.

## Table of contents
- [Introduction](#Introduction)
- [Component details](#Component-details)
- [Prerequisites](#Prerequisites)
- [Step 1: Preparing to install Business Automation Workflow for production](#Step-1-Preparing-to-install-Business-Automation-Workflow)
  - [Creating a namespace on OpenShift](#Creating-a-namespace-on-OpenShift)
  - [Loading Docker images](#Loading-Docker-images)
- [Step 2: Preparing databases for Business Automation Workflow](#Step-2-Preparing-databases-for-Business-Automation-Workflow)
  - [Creating databases for Business Automation Workflow](#Creating-databases-for-Business-Automation-Workflow)
  - [Creating databases for User Management Service](#Creating-databases-for-User-Management-Service)
  - [Creating databases for Business Automation Navigator](#Creating-databases-for-Business-Automation-Navigator)
  - [Creating databases for FileNet Content Manager](#Creating-databases-for-FileNet-Content-Manager)
- [Step 3: Preparing to configure LDAP](#Step-3-Preparing-to-configure-LDAP)
- [Step 4: Preparing storage](#Step-4-Preparing-storage)
  - [Disabling swapping and increasing the limit number of file descriptors](#Disabling-swapping-and-increasing-the-limit-number-of-file-descriptors)
  - [Preparing storage for the operator](#Preparing-storage-for-the-operator)
  - [Preparing storage for FileNet Content Manager](#Preparing-storage-for-FileNet-Content-Manager)
  - [Preparing storage for Business Automation Navigator](#Preparing-storage-for-Business-Automation-Navigator)
  - [Preparing storage for Java Messaging Service](#Preparing-storage-for-Java-Messaging-Service)
  - [Preparing storage for Process Federation Server](#Preparing-storage-for-Process-Federation-Server)
- [Step 5: Creating secrets to protect sensitive configuration data](#Step-5-Creating-secrets-to-protect-sensitive-configuration-data)
  - [Creating required secrets for Business Automation Workflow](#Creating-required-secrets-for-Business-Automation-Workflow)
  - [Creating required secrets for Resource Registry](#Creating-required-secrets-for-Resource-Registry)
  - [Creating required secrets for User Management Service](#Creating-required-secrets-for-User-Management-Service)
  - [Creating required secrets for FileNet Content Manager](#Creating-required-secrets-for-FileNet-Content-Manager)
  - [Creating required secrets for Business Automation Navigator](#Creating-required-secrets-for-Business-Automation-Navigator)
- [Step 6: Preparing a SecurityContextConstraint](#Step-6-Preparing-a-SecurityContextConstraint)
- [Step 7: Deploying Business Automation Workflow](#Step-7-Deploying-Business-Automation-Workflow)
  - [Creating a secret to pull images](#Creating-a-secret-to-pull-images)
  - [Deploying the operator manifest files to your cluster](#Deploying-the-operator-manifest-files-to-your-cluster)
  - [Deploying the Custom Resource file](#Deploying-the-custom-resource-file)
  - [Custom configuration](#Custom-configuration)
- [Step 8: Verifying Business Automation Workflow](#Step-8-Verifying-Business-Automation-Workflow)
- [Step 9: Uninstalling Business Automation Workflow](#Step-9-Uninstalling-Business-Automation-Workflow)
- [Limitations](#Limitations)


## Introduction
This repository includes folders and resources to help you install IBM Business Automation Workflow on Containers on Red Hat OpenShift Cloud Platform (OCP). With a customized deployment, you must prepare databases, storage, and secrets in advance before installing Business Automation Workflow.

Business Automation Workflow uses an operator, which is a Kubernetes feature that makes it simpler to install and update. The operator captures the expert knowledge of administrators on how the system ought to behave, how to deploy it, and how to react if problems occur.

## Component details
The standard configuration includes these components:

- IBM Business Automation Workflow Server component
- IBM Java Messaging Service (JMS) component
- IBM Process Federation Server component
- IBM User Management Service (UMS) component
- IBM Resource Registry component
- IBM FileNet Content Manager component
- IBM Business Automation Navigator component

To support those components, a standard installation generates the following content:

- 1 StatefulSet running JMS
- 1 StatefulSet running Workflow Server
- 1 StatefulSet running Process Federation Server
- 1 StatefulSet running Elasticsearch Server
- 1 Deployment running UMS
- 2 Deployments running FileNet Content Manager
- 1 Deployment running Business Automation Navigator
- 8 jobs for Business Automation Workflow
- 8 service accounts with related role and role binding
- 20 secrets to gain access during installation
- 21 ConfigMaps that manage the configuration
- 25 services and Routes


## Prerequisites
- [OpenShift 3.11 or later](https://docs.openshift.com/container-platform/3.11/welcome/index.html)
- [IBM DB2 11.5](https://www.ibm.com/products/db2-database)


## Step 1: Preparing to install Business Automation Workflow
### Creating a namespace on OpenShift
Run the following steps on the master node in your OpenShift cluster to create a namespace.

1. Log in to your cluster:
   ```bash
   $ oc login https://<CLUSTERIP>:8443 -u <ADMINISTRATOR>
   ```
   > **Note**: You should use port `6443` if you are on OCP 4.x.
2. Create an OpenShift project (namespace) to install the operator in:
   ```bash
   $ oc new-project <my-project>
   ```
3. Add privileges to the project:
   ```bash
   $ oc adm policy add-scc-to-user privileged -z ibm-cp4a-operator -n <my-project>
   ```

### Loading Docker images
1. Check that you can run a Docker command:
   ```bash
   $ docker ps
   ```
   > **Note**: You can use `podman` if you are on OCP 4.x.
   
2. Log in to the Docker registry with a token:
   ```bash
   $ docker login $(oc registry info) -u <ADMINISTRATOR> -p $(oc whoami -t)
   ```

   You can also log in to an external Docker registry using the following command:
   ```bash
   $ docker login <registry_url> -u <your_account>
   ```
3. Run a `kubectl` command to make sure that you have access to Kubernetes:
   ```bash
   $ kubectl cluster-info
   ```
4. Run the `scripts/loadimages.sh` script to load the images into your Docker registry. Specify the two mandatory parameters on the command line:

   ```
   -p  PPA archive files location or archive filename
   -r  Target Docker registry and namespace
   -l  Optional: Target a local registry
   ```

   The following example shows the input values on the command line:

   ```
   chmod +x ./scripts/loadimages.sh
   ./scripts/loadimages.sh -p <PPA-ARCHIVE>.tar.gz -r <registry_url>/<my-project>
   ```

   > **Note**: You must have pull request privileges to the registry where the images are loaded.

5. Check that the images are pushed correctly to the registry:
    ```bash
    $ oc get is
   
## Step 2: Preparing databases for Business Automation Workflow
Before you can install, you must create databases for Workflow Server, User Management Service (UMS), Business Automation Navigator, and FileNet Content Manager. The IBM Db2 server for Business Automation Navigator must be installed on Linux.

### Creating databases for Business Automation Workflow
Create the database for Business Automation Workflow by copying the `scripts/CREATE_BAW_DB.sql` to your Db2 server and running the following commands:
```sh
su - <Db2_OS_USER>
db2 -tvf CREATE_BAW_DB.sql
```

**Notes:**
- Replace `<BAW_DB_NAME>` with the Business Automation Workflow Server database name you want, for example BPMDB.
- Replace `<DB_USER>` with the user you will use for the database.


#### (Optional) Db2 SSL Configuration
To ensure that all communications between Workflow Server and Db2 are encoded, you must import the database CA certificate to the Workflow Server. First, create a secret to store the certificate:
```
kubectl create secret generic ibm-dba-baw-db2-cacert --from-file=cacert.crt=
```

**Note:** You must modify the part that points to the certificate file. Do not modify the part --from-file=cacert.crt=.

You can then use the resulting secret to set the `baw_configuration[x].database.sslsecretname: ibm-dba-baw-db2-cacert` property in the custom resource configuration YAML file. Also set  `baw_configuration[x].database.ssl` to `true`.

#### (Optional) Db2 HADR Configuration
To configure high availability disaster recovery (HADR), you must ensure that the Workflow Server automatically retrieves the necessary failover server information when it first connects to the database. As part of the setup, you must provide a comma-separated list of failover servers and failover ports.

For example, if there are two failover servers, such as:

    server1.db2.customer.com on port 50443
    server2.db2.customer.com on port 51443

you can specify these hosts and ports in the custom resource configuration YAML file as follows:
```yaml
database:
  ... ...
    hadr:
      standbydb_host: server1.db2.customer.com, server2.db2.customer.com
      standbydb_port: 50443,51443
      retryintervalforclientreroute: <default value is 10 min>
      maxretriesforclientreroute: <default value is 5>
  ... ...
```

### Creating databases for User Management Service
Create the database for User Management Service by copying the `scripts/CREATE_UMS_DB.sql` to your Db2 server and running the following commands:
```sh
su - <Db2_OS_USER>
db2 -tvf CREATE_UMS_DB.sql
```

**Note:**
- Replace `<UMS_DB_NAME>` with the UMS database name you want, for example UMSDB.

### Creating databases for Business Automation Navigator
To create the database for Business Automation Navigator, copy the `scripts/CREATE_BAN_DB.sql` files to your Db2 server and run the following commands:
```sh
su - <Db2_OS_USER>
db2 -tvf CREATE_BAN_DB.sql
```

**Notes:**
- Replace `<BAN_DB_NAME>` with the Navigator database name you want, for example ICNDB.
- Replace `<DB_USER>` with the user you will use for the database.


### Creating databases for FileNet Content Manager
To create databases for FileNet Content Manager, create a database for the Content Platform Engine global configuration database (GCD) and three databases for the Content Platform Engine onbect stores.
#### Creating databases for GCD
To create the database for GCD, copy the `scripts/CREATE_GCD_DB.sql` files to your Db2 server and run the following commands:
```sh
su - <Db2_OS_USER>
db2 -tvf CREATE_GCD_DB.sql
```

**Notes:**
- Replace `<GCD_DB_NAME>` with the GCD database name you want, for example GCDDB.
- Replace `<DB_USER>` with the user you will use for the database.


#### Creating databases for the object stores
To create the three databases for the object stores, copy the `scripts/CREATE_OBJECTSTORE_DB.sql` to your Db2 server and run the following commands:
```sh
su - <Db2_OS_USER>
db2 -tvf CREATE_OBJECTSTORE_DB.sql
```

**Notes:**
- Replace `<DB_USER>` with the user you will use for the databases.
- Because Content Platform Engine requires three databases, you must run the command `db2 -tvf CREATE_OBJECTSTORE_DB.sql` three times with a different `<OBJECTSTORE_DB_NAME>` each time. Replace `<OBJECTSTORE_DB_NAME>` with the DOCS database name, Target Object Store database name, and Design Object Store database name, for example DOCSDB/TOSDB/DOSDB. 



## Step 3: Preparing to configure LDAP
An LDAP server is required before you install Business Automation Workflow. Save the following content in a file named `ldap-bind-secret.yaml`. Then, apply it by running the `oc apply -f ldap-bind-secret.yaml` command:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ldap-bind-secret
type: Opaque
data:
  ldapUsername: <LDAP_BIND_DN>
  ldapPassword: <LDAP_PASSWORD>
```

**Notes:**
- Ensure `ldapUsername` corresponds to the **bindDN** property of your LDAP server with **base64** encoded.
- Ensure `ldapPassword` corresponds to the **bindPassword** property of your LDAP server with **base64** encoded.
- Specify the hostname of your LDAP server as the `shared_configuration.ldap_configuration.lc_ldap_server` property.
- Specify the secret name you created above as the `shared_configuration.ldap_configuration.lc_bind_secret` property.


## Step 4: Preparing storage
### Disabling swapping and increasing the limit number of file descriptors
For node stability and performance, disable memory swapping on all worker nodes. For detailed information, see [Disable swapping for Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/setup-configuration-memory.html). Also, because Elasticsearch uses a lot of file descriptors and running out of file descriptors can lead to data loss, be sure to increase the limit on the number of open file descriptors for the user running Elasticsearch.

By default, `pfs_configuration.elasticsearch.privileged` is set to `true` and you can perform this configuration yourself.

If privileged containers are not allowed, set the `pfs_configuration.elasticsearch.privileged` property to `false` in the custom resource configuration. Then, ask the cluster administrator to run the following command to change the swap and max_map_count:

```
sysctl -w vm.max_map_count=262144 && sed -i '/^vm.max_map_count /d' /etc/sysctl.conf && echo 'vm.max_map_count = 262144' >> /etc/sysctl.conf && sysctl -w vm.swappiness=1 && sed -i '/^vm.swappiness /d' /etc/sysctl.conf && echo 'vm.swappiness=1' >> /etc/sysctl.conf
```

### Preparing storage for the operator

The operator requires a persistent volume (PV) and persistent volume claim (PVC) to store the Db2 JDBC driver.

The following example illustrates the procedure using Network File System (NFS). An existing NFS server is required before you create PVs and PVCs.

- Create a directory for the operator on an NFS server. 

Run the following command on the NFS server:
```
mkdir -p <NFS_STORAGE_DIRECTORY>/db/jdbc/db2
chown -R :65534 <NFS_STORAGE_DIRECTORY>/db
chmod -R g+rw <NFS_STORAGE_DIRECTORY>/db
```

Then, copy `db2jcc4.jar` and `db2jcc_license_cu.jar` to `<NFS_STORAGE_DIRECTORY>/db/jdbc/db2`. In the `/etc/exports` configuration file, add the following line at the end:
```
<NFS_STORAGE_DIRECTORY> *(rw,sync,no_subtree_check)
```

After editing and saving the `/etc/exports` configuration file, you must restart the NFS service for the changes to take effect. The following example shows restarting the NFS server on RHEL 7:
```sh
systemctl stop nfs
systemctl stop rpcbind
systemctl start rpcbind
systemctl start nfs
```

- Create the PV and PVC for the operator

To create the shared PV and PVC that are required by the operator, run the following command on the OpenShift master node:
```
oc apply -f operator-shared-pv-pvc.yaml
```

The following content should be in the  `operator-shared-pv-pvc.yaml` file:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: operator-shared-pv
spec:
  storageClassName: "ecm-openshift-nfs"
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
  nfs:
    path: <NFS_STORAGE_DIRECTORY>/db
    server: <NFS_SERVER_IP>
  persistentVolumeReclaimPolicy: Recycle
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: operator-shared-pvc
  annotations:
    volume.beta.kubernetes.io/storage-class: "ecm-openshift-nfs"
  labels:
    app.kubernetes.io/instance: ibm-dba
    app.kubernetes.io/managed-by: ibm-dba
    app.kubernetes.io/name: ibm-dba
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```

**Notes:**
- Replace `<NFS_STORAGE_DIRECTORY>` with the operator folder on your NFS server.
- Replace `<NFS_SERVER_IP>` with the IP address of your NFS server.

### Preparing storage for FileNet Content Manager
- Create the folders required by Content Platform Engine and Content Management
Interoperability Services by running the following commands on the NFS server:

```sh
mkdir -p <NFS_STORAGE_DIRECTORY>/cpe/configDropins/overrides
mkdir -p <NFS_STORAGE_DIRECTORY>/cpe/logs
mkdir -p <NFS_STORAGE_DIRECTORY>/cpe/asa
mkdir -p <NFS_STORAGE_DIRECTORY>/cpe/bootstrap
mkdir -p <NFS_STORAGE_DIRECTORY>/cpe/textext
mkdir -p <NFS_STORAGE_DIRECTORY>/cpe/icmrules
mkdir -p <NFS_STORAGE_DIRECTORY>/cpe/cpefnlogstore 
mkdir -p <NFS_STORAGE_DIRECTORY>/cmis/configDropins/overrides
mkdir -p <NFS_STORAGE_DIRECTORY>/cmis/logs

chown -R :65534 <NFS_STORAGE_DIRECTORY>/cpe
chmod -R g+rw <NFS_STORAGE_DIRECTORY>/cpe
chown -R :65534 <NFS_STORAGE_DIRECTORY>/cmis
chmod -R g+rw <NFS_STORAGE_DIRECTORY>/cmis
```

- Create the PVs and PVCs required by Content Platform Engine and Content Management Interoperability Services by saving the following YAML files on the OpenShift master node and then running the `oc apply -f ecm-pv-pvc.yaml` command on the master node.

ecm-pv-pvc.yaml
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: cpe-cfgstore-pv
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
  nfs:
    path: <NFS_STORAGE_DIRECTORY>/cpe/configDropins/overrides
    server: <NFS_SERVER_IP>
  persistentVolumeReclaimPolicy: Retain
  storageClassName: cpe-cfgstore-pv
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cpe-cfgstore-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: cpe-cfgstore-pv
  volumeName: cpe-cfgstore-pv
status:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: cpe-logstore-pv
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
  nfs:
    path: <NFS_STORAGE_DIRECTORY>/cpe/logs
    server: <NFS_SERVER_IP>
  persistentVolumeReclaimPolicy: Retain
  storageClassName: cpe-logstore-pv
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cpe-logstore-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: cpe-logstore-pv
  volumeName: cpe-logstore-pv
status:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: cpe-filestore-pv
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
  nfs:
    path: <NFS_STORAGE_DIRECTORY>/cpe/asa
    server: <NFS_SERVER_IP>
  persistentVolumeReclaimPolicy: Retain
  storageClassName: cpe-filestore-pv
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cpe-filestore-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: cpe-filestore-pv
  volumeName: cpe-filestore-pv
status:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: cpe-bootstrap-pv
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
  nfs:
    path: <NFS_STORAGE_DIRECTORY>/cpe/bootstrap
    server: <NFS_SERVER_IP>
  persistentVolumeReclaimPolicy: Retain
  storageClassName: cpe-bootstrap-pv
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cpe-bootstrap-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: cpe-bootstrap-pv
  volumeName: cpe-bootstrap-pv
status:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: cpe-textext-pv
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
  nfs:
    path: <NFS_STORAGE_DIRECTORY>/cpe/textext
    server: <NFS_SERVER_IP>
  persistentVolumeReclaimPolicy: Retain
  storageClassName: cpe-textext-pv
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cpe-textext-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: cpe-textext-pv
  volumeName: cpe-textext-pv
status:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: cpe-icmrules-pv
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
  nfs:
    path: <NFS_STORAGE_DIRECTORY>/cpe/icmrules
    server: <NFS_SERVER_IP>
  persistentVolumeReclaimPolicy: Retain
  storageClassName: cpe-icmrules-pv
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cpe-icmrules-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: cpe-icmrules-pv
  volumeName: cpe-icmrules-pv
status:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: cpe-fnlogstore-pv
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
  nfs:
    path: <NFS_STORAGE_DIRECTORY>/cpe/cpefnlogstore
    server: <NFS_SERVER_IP>
  persistentVolumeReclaimPolicy: Retain
  storageClassName: cpe-fnlogstore-pv
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cpe-fnlogstore-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: cpe-fnlogstore-pv
  volumeName: cpe-fnlogstore-pv
status:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: cmis-cfgstore-pv
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
  nfs:
    path: <NFS_STORAGE_DIRECTORY>/cmis/configDropins/overrides
    server: <NFS_SERVER_IP>
  persistentVolumeReclaimPolicy: Retain
  storageClassName: cmis-cfgstore-pv
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cmis-cfgstore-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: cmis-cfgstore-pv
  volumeName: cmis-cfgstore-pv
status:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: cmis-logstore-pv
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
  nfs:
    path: <NFS_STORAGE_DIRECTORY>/cmis/logs
    server: <NFS_SERVER_IP>
  persistentVolumeReclaimPolicy: Retain
  storageClassName: cmis-logstore-pv
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cmis-logstore-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: cmis-logstore-pv
  volumeName: cmis-logstore-pv
status:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
```

**Notes:**
- Replace `<NFS_STORAGE_DIRECTORY>` with the operator folder on your NFS server.
- Replace `<NFS_SERVER_IP>` with the IP address of your NFS server.


### Preparing storage for Business Automation Navigator
- Create the required folders for Business Automation Navigator by running the following commands on the NFS server:

```sh
mkdir -p <NFS_STORAGE_DIRECTORY>/icn
mkdir -p <NFS_STORAGE_DIRECTORY>/icn/cfgstore
mkdir -p <NFS_STORAGE_DIRECTORY>/icn/logstore
mkdir -p <NFS_STORAGE_DIRECTORY>/icn/pluginstore
mkdir -p <NFS_STORAGE_DIRECTORY>/icn/vwcachestore
mkdir -p <NFS_STORAGE_DIRECTORY>/icn/vwlogstore
mkdir -p <NFS_STORAGE_DIRECTORY>/icn/asperastore

chown -R :65534 <NFS_STORAGE_DIRECTORY>/icn
chmod -R g+rw <NFS_STORAGE_DIRECTORY>/icn
```

- Create the PVs and PVCs required by Business Automation Navigator by saving the following YAML files on the OpenShift master node and then running the  `oc apply -f navigator-pv-pvc.yaml` command on the master node.

navigator-pv-pvc.yaml
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
   name: icn-cfgstore
spec:
   capacity:
    storage: 1Gi
   accessModes:
    - ReadWriteMany
   persistentVolumeReclaimPolicy: Retain
   storageClassName: icn-cfgstore
   nfs:
     path:  <NFS_STORAGE_DIRECTORY>/icn/cfgstore
     server: <NFS_SERVER_IP>
---
apiVersion: v1
kind: PersistentVolume
metadata:
   name: icn-logstore
spec:
   capacity:
    storage: 2Gi
   accessModes:
    - ReadWriteMany
   persistentVolumeReclaimPolicy: Retain
   storageClassName: icn-logstore
   nfs:
     path: <NFS_STORAGE_DIRECTORY>/icn/logstore
     server: <NFS_SERVER_IP>
---
apiVersion: v1
kind: PersistentVolume
metadata:
   name: icn-pluginstore
spec:
   capacity:
    storage: 1Gi
   accessModes:
    - ReadWriteMany
   persistentVolumeReclaimPolicy: Retain
   storageClassName: icn-pluginstore
   nfs:
     path: <NFS_STORAGE_DIRECTORY>/icn/pluginstore
     server: <NFS_SERVER_IP>
---
apiVersion: v1
kind: PersistentVolume
metadata:
   name: icn-vw-cachestore
spec:
   capacity:
    storage: 1Gi
   accessModes:
    - ReadWriteMany
   persistentVolumeReclaimPolicy: Retain
   storageClassName: icn-vw-cachestore
   nfs:
     path: <NFS_STORAGE_DIRECTORY>/icn/vwcachestore
     server: <NFS_SERVER_IP>
---
apiVersion: v1
kind: PersistentVolume
metadata:
   name: icn-vw-logstore
spec:
   capacity:
    storage: 1Gi
   accessModes:
    - ReadWriteMany
   persistentVolumeReclaimPolicy: Retain
   storageClassName: icn-vw-logstore
   nfs:
     path: <NFS_STORAGE_DIRECTORY>/icn/vwlogstore
     server: <NFS_SERVER_IP>
---
apiVersion: v1
kind: PersistentVolume
metadata:
   name: icn-asperastore
spec:
   capacity:
    storage: 1Gi
   accessModes:
    - ReadWriteMany
   persistentVolumeReclaimPolicy: Retain
   storageClassName: icn-asperastore
   nfs:
     path: <NFS_STORAGE_DIRECTORY>/icn/asperastore
     server: <NFS_SERVER_IP>
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: icn-cfgstore
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: icn-cfgstore
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: icn-logstore
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 2Gi
  storageClassName: icn-logstore
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: icn-pluginstore
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: icn-pluginstore
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: icn-vw-cachestore
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: icn-vw-cachestore
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: icn-vw-logstore
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: icn-vw-logstore
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: icn-asperastore
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: icn-asperastore
```

**Notes:**
- Replace `<NFS_STORAGE_DIRECTORY>` with the operator folder on your NFS server.
- Replace `<NFS_SERVER_IP>` with the IP address of your NFS server.

### Preparing storage for Java Messaging Service
The Java Messaging Service (JMS) component requires a PV and related folder to be created before you can deploy. 

The following example illustrates the procedure using NFS. An existing NFS server is required before creating PVs.

- Create related folders on an NFS server. You must grant minimal privileges to the NFS server.

Give the least privilege to the mounted directories using the following commands:
```bash
mkdir -p <NFS_STORAGE_DIRECTORY>/jms
chown -R :65534 <NFS_STORAGE_DIRECTORY>/jms
chmod -R g+rw <NFS_STORAGE_DIRECTORY>/jms
```

- Create the PV required by JMS by saving the following YAML files on the OpennShift master node and running the `oc apply -f jms-pv.yaml` command:

jms-pv.yaml
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jms-pv-baw
spec:
  storageClassName: "jms-storage-class"
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 2Gi
  nfs:
    path: <NFS_STORAGE_DIRECTORY>/jms
    server: <NFS_SERVER_IP>
  persistentVolumeReclaimPolicy: Recycle
```

**Notes:**
- Replace `<NFS_STORAGE_DIRECTORY>` with the operator folder on your NFS server.
- `accessModes` should be set to the same value as the `baw_configuration[x].jms.storage.access_modes` property in the custom resource configuration file.
- Replace `<NFS_SERVER_IP>` with the IP address of your NFS server.


### Preparing storage for Process Federation Server
The Process Federation Server component requires PVs, PVCs, and related folders to be created before you deploy. The deployment process uses these volumes and folders during the deployment.

The following example illustrates the procedure using NFS. An existing NFS server is required before creating PVs and PVCs.

- Create related folders on an NFS server. You must grant minimal privileges to the NFS server.

Give the least privilege to the mounted directories using the following commands: 
```bash
mkdir -p <NFS_STORAGE_DIRECTORY>/pfs-es-0
mkdir -p <NFS_STORAGE_DIRECTORY>/pfs-es-1
mkdir -p <NFS_STORAGE_DIRECTORY>/pfs-logs-0
mkdir -p <NFS_STORAGE_DIRECTORY>/pfs-logs-1

chown -R :65534 <NFS_STORAGE_DIRECTORY>/pfs-*
chmod -R g+rw <NFS_STORAGE_DIRECTORY>/pfs-*
```

- Create the Process Federation Server required PVs by saving the following YAML files on the OpenShift master node and then running the `oc apply -f <YAML_FILE_NAME>` command on the files in the following order:

1. pfs-pv-pfs-es-0.yaml
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pfs-es-0
spec:
  storageClassName: "pfs-es"
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 10Gi
  nfs:
    path: <NFS_STORAGE_DIRECTORY>/pfs-es-0
    server: <NFS_SERVER_IP>
  persistentVolumeReclaimPolicy: Recycle
```

2. pfs-pv-pfs-es-1.yaml
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pfs-es-1
spec:
  storageClassName: "pfs-es"
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 10Gi
  nfs:
    path: <NFS_STORAGE_DIRECTORY>/pfs-es-1
    server: <NFS_SERVER_IP>
  persistentVolumeReclaimPolicy: Recycle
```

3. pfs-pv-pfs-logs-0.yaml
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pfs-logs-0
spec:
  storageClassName: "pfs-logs"
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 5Gi
  nfs:
    path: <NFS_STORAGE_DIRECTORY>/pfs-logs-0
    server: <NFS_SERVER_IP>
  persistentVolumeReclaimPolicy: Recycle
```

4. pfs-pv-pfs-logs-1.yaml
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pfs-logs-1
spec:
  storageClassName: "pfs-logs"
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 5Gi
  nfs:
    path: <NFS_STORAGE_DIRECTORY>/pfs-logs-1
    server: <NFS_SERVER_IP>
  persistentVolumeReclaimPolicy: Recycle
```


**Notes:**
- Replace `<NFS_STORAGE_DIRECTORY>` with the Process Federation Server storage folder on your NFS server.
- Replace `<NFS_SERVER_IP>` with the IP address of your NFS server.


## Step 5: Creating secrets to protect sensitive configuration data
A secret is an object that contains a small amount of sensitive data such as a password, a token, or a key. Before you install Business Automation Workflow, you must create the following secrets manually.
### Creating required secrets for Business Automation Workflow
Save the following content in a separate YAML file for each secret and run the `oc apply -f <YAML_FILE_NAME>` command on the OpenShift master node.

**Note:** All values under `data` in secret must be **base64** encoded, and you can get a **base64** encoded string by running command `echo -n "<sample_string> | base64`, the output is namely the base64 encoded result.

Shared encryption key secret:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: icp4a-shared-encryption-key
type: Opaque
data:
  encryptionKey: <ENCRYPTION_KEY>
```
**Notes:**
- So that the confidential information is shared only between the components that hold the key, use the encryptionKey to encrypt the confidential information at the Resource Registry.
- Ensure the `encryptionKey` is **base64** encoded.

Business Automation Workflow Server database secret:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ibm-baw-wfs-server-db-secret
type: Opaque  
data:
  dbUser: <DB_USER>
  password: <DB_USER_PASSWORD>
```
**Notes:**
- `dbUser` and `password` are the database user name and password.
- Ensure all values under `data` are **base64** encoded.

Process Federation Server:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ibm-pfs-admin-secret
type: Opaque
data:
  ltpaPassword: <LTPA_PASSWORD>
  oidcClientPassword: <OIDC_CLIENT_PASSWORD>
  sslKeyPassword: <SSL_KEY_PASSWORD>
```

**Notes:**
- `ltpaPassword` is used to set LTPA password.
- `oidcClientPassword` is registered at UMS as the oidc client password.
- `sslKeyPassword` is used as the keystore and trust store password.
- Ensure all values under `data` are **base64** encoded.

### Creating required secrets for Resource Registry
Save the following content in a YAML file and run the `oc apply -f <YAML_FILE_NAME>` command on the OpenShift master node.

Resource Registry secret:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: rr-admin-secret
type: Opaque
data:
  rootPassword: <ROOT_PASSWORD>
  readUser: <READER_USER>
  readPassword: <READER_PASSWORD>
  writeUser: <WRITER_USER>
  writePassword: <WRITER_PASSWORD>
```

**Notes:**
- `rootPassword` is used as root password.
- `readUser` is used as read user.
- `readPassword` is used as read user password.
- `writeUser` is used as write user.
- `writePassword` is used as write user password.
- Ensure all values under `data` are **base64** encoded.

### Creating required secrets for User Management Service
Save the following content in a YAML file and run the `oc apply -f <YAML_FILE_NAME>` command on the OpenShift master node.

User Management Service secret:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ibm-dba-ums-secret
type: Opaque
data:
  adminUser: <UMS_USER>
  adminPassword: <UMS_PASSWORD>
  oauthDBUser: <UMS_DB_USER>
  oauthDBPassword: <UMS_DB_PASSWORD>
  tsDBUser: <UMS_DB_USER>
  tsDBPassword: <UMS_DB_PASSWORD>
```

**Notes:**
- `adminUser` is used as UMS admin user.
- `adminPassword` is used as UMS admin user password.
- `oauthDBUser` is used as UMS database user.
- `oauthDBPassword` is used as UMS database password.
- `tsDBUser` is used as UMS database user.
- `tsDBPassword` is used as UMS database password.
- Ensure all values under `data` are **base64** encoded.

### Creating required secrets for FileNet Content Manager
Save the following content in a YAML file and run the `oc apply -f <YAML_FILE_NAME>` command on the OpenShift master node.

FileNet Content Manager secret:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ibm-fncm-secret
type: Opaque
data:
  gcdDBUsername: <GCD_DB_USER>
  gcdDBPassword: <GCD_DB_PASSWORD>
  osDBUsername: <CPE_DB_USER>
  osDBPassword: <CPE_DB_PASSWORD>
  ltpaPassword: <LTPA_PASSWORD>
  ldapUsername: <LDAP_BIND_DN>
  ldapPassword: <LDAP_BIND_PASSWORD>
  keystorePassword: <KEYSTORE_PASSWORD>
  navigatorDBUsername: <NAVIGATOR_DB_USER>
  navigatorDBPassword: <NAVIGATOR_DB_PASSWORD>
  appLoginUsername: <LDAP_USER>
  appLoginPassword: <LDAP_USER_PASSWORD>
```

**Notes:**
- `gcdDBUsername` is used as GCD database user.
- `gcdDBPassword` is used as GCD database password.
- `osDBUsername` is used as CPE database user.
- `osDBPassword` is used as CPE database password.
- `ltpaPassword` is used as LTPA user password.
- `ldapUsername` is used as LDAP Bind DN property.
- `ldapPassword` is used as LDAP Bind password.
- `keystorePassword` is used as keystore password.
- `navigatorDBUsername` is used as Navigator database user.
- `navigatorDBPassword` is used as Navigator database password.
- `appLoginUsername` is used as LDAP user.
- `appLoginPassword` is used as LDAP user password.
- Ensure all values under `data` are **base64** encoded.

### Creating required secrets for Business Automation Navigator
Save the following content in a YAML file and run the `oc apply -f <YAML_FILE_NAME>` command on the OpenShift master node.

Business Automation Navigator secret:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ibm-ban-secret
type: Opaque
data:
  navigatorDBUsername: <NAVIGATOR_DB_USER>
  navigatorDBPassword: <NAVIGATOR_DB_PASSW0RD>
  ldapUsername: <LDAP_BIND_DN>
  ldapPassword: <LDAP_BIND_PASSWORD>
  appLoginUsername: <LDAP_USER>
  appLoginPassword: <LDAP_USER_PASSWORD>
  ltpaPassword: <LTPA_PASSWORD>
  keystorePassword: <KEYSTORE_PASSWORD>
```

**Notes:**
- `navigatorDBUsername` is used as Navigator database user.
- `navigatorDBPassword` is used as Navigator database password.
- `ldapUsername` is used as LDAP Bind DN property.
- `ldapPassword` is used as LDAP Bind password.
- `appLoginUsername` is used as LDAP user.
- `appLoginPassword` is used as LDAP user password.
- `ltpaPassword` is used as LTPA password.
- `keystorePassword` is used as keystore password.
- Ensure all values under `data` are **base64** encoded.


## Step 6: Preparing a SecurityContextConstraint
Process Federation Server requires a SecurityContextConstraint with the following content. Save it to the `ibm-pfs-privileged-scc.yaml` file.
```yaml
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: ibm-pfs-privileged-scc
allowHostDirVolumePlugin: true
allowHostIPC: true
allowHostNetwork: true
allowHostPID: true
allowHostPorts: true
allowPrivilegedContainer: true
allowPrivilegeEscalation: true
allowedCapabilities:
- '*'
allowedFlexVolumes: []
allowedUnsafeSysctls:
- '*'
defaultAddCapabilities: []
defaultAllowPrivilegeEscalation: true
forbiddenSysctls: []
fsGroup:
  type: RunAsAny
readOnlyRootFilesystem: false
requiredDropCapabilities: []
runAsUser:
  type: RunAsAny
seccompProfiles:
- '*'
seLinuxContext:
  type: RunAsAny
supplementalGroups:
  type: RunAsAny
volumes:
- '*'
priority: 2
```

Then, to create a service account named `ibm-pfs-es-service-account` and add the file you just created to the Process Federation Server Elasticsearch default service account in the current namespace, run the following commands in order:

```sh
$ oc create serviceaccount ibm-pfs-es-service-account
$ oc apply -f ibm-pfs-privileged-scc.yaml
$ oc adm policy add-scc-to-user ibm-pfs-privileged-scc -z ibm-pfs-es-service-account
```

**Note:** 
- Specify the value of the `elasticsearch_configuration.service_account` property as the newly created service account `ibm-pfs-es-service-account` in your custom resource configuration file.



## Step 7: Deploying Business Automation Workflow
### Creating a secret to pull images
Run the following command on the master node of the OpenShift cluster:
```yaml
oc create secret docker-registry <my_secret_name> --docker-server=image-registry.openshift-image-registry.svc:5000 --docker-username=<ADMINISTRATOR> --docker-password=$(oc whoami -t)
```

### Deploying the operator manifest files to your cluster
The operator has a number of descriptors that must be applied:
  - `descriptors/ibm_cp4a_crd.yaml` contains the description of the Custom Resource Definition.
  - `descriptors/operator.yaml` defines the deployment of the operator code.
  - `descriptors/role.yaml` defines the access of the operator.
  - `descriptors/role_binding.yaml` defines the access of the operator.
  - `descriptors/service_account.yaml` defines the identity for processes that run inside the pods of the operator.

1. Deploy the icp4a-operator on your cluster.

   Use the `scripts/deployOperator.sh` script to deploy these descriptors:
   ```bash
   $ chmod +x ./scripts/deployOperator.sh
   $ ./scripts/deployOperator.sh -i <registry_url>/<my-project>/icp4a-operator:20.0.2 -p '<my_secret_name>' -a accept
   ```
**Notes:**
  - `registry_url` is the value for your internal Docker registry and you can get it by running `oc registry info`.
  - `my_secret_name` is the secret created above to access the registry, and *accept* means that you accept the license.


2. Monitor the pod until it shows a STATUS of *Running*:
   ```bash
   $ kubectl get pods -w
   ```
   > **Note**: When started, you can monitor the operator logs by using the following command:
   ```bash
   $ kubectl logs -f deployment/ibm-cp4a-operator -c operator
   ```

### Deploying the Custom Resource file
You must set the configuration paramters for Business Automation Workflow, Resource Registry, Process Federation Server, JMS, FileNet Content Manager, and Business Automation Navigator in your custom resource file. There is a  Custom Resource template `descriptors/ibm_baw_cr_template.yaml` with default values. Using the template, you need to update only the minimum required values.


Run the following command to deploy the custom resource YAML file:
```sh
oc apply -f ibm_baw_cr_template.yaml
```

### Custom configuration
If you want to customize your custom resource YAML file, see [Business Automation Workflow parameters](./BAW-on-Ctnrs-TP-Parameters.md) to update the required value of each parameter according to your environment.



## Step 8: Verifying Business Automation Workflow
Wait for about 45 minutes, depending on your machine's performance. Then, check the status of the pods.

1. Get the names of the pods that were deployed by running the following command:
```
oc get pod -n <my-project>
```

<div>
<details>
<summary>Click to show a successful pod status. </summary>
<p>

```
NAME                                                         READY     STATUS      RESTARTS   AGE
demo-cmis-deploy-7d78dc77d7-6z5sg                            1/1       Running     0          5h
demo-cpe-deploy-d5f94b7d9-snntz                              1/1       Running     0          5h
demo-dba-rr-1b76b38a9d                                       1/1       Running     0          5h
demo-dba-rr-b19e269677                                       1/1       Running     0          5h
demo-dba-rr-d6579dbd81                                       1/1       Running     0          5h
demo-elasticsearch-statefulset-0                             2/2       Running     0          4h
demo-instance1-baw-case-init-job-rxtxd                       0/1       Completed   0          4h
demo-instance1-baw-content-init-job-9z8nf                    0/1       Completed   0          4h
demo-instance1-workflow-server-0                             1/1       Running     0          4h
demo-instance1-workflow-server-database-init-job-6wpnl       0/1       Completed   0          11m
demo-instance1-workflow-server-database-init-job-pfs-xqjtz   0/1       Completed   0          11m
demo-instance1-workflow-server-jms-0                         1/1       Running     0          4h
demo-instance1-workflow-server-ltpa-xbshm                    0/1       Completed   0          4h
demo-instance1-workflow-server-umsregistry-job-fjq2g         0/1       Completed   0          4h
demo-navigator-deploy-6969c5754b-v55fn                       1/1       Running     0          5h
demo-pfs-instance1-0                                         0/1       Running     0          4h
demo-pfs-instance1-dbareg-6879dcb5db-2w8xc                   1/1       Running     0          4h
demo-pfs-instance1-umsregistry-job-j79ps                     0/1       Completed   0          4h
demo-rr-setup-pod                                            0/1       Completed   0          5h
demo-ums-deployment-cbfb9b8f-mpl86                           1/1       Running     0          5h
demo-ums-ltpa-creation-job-btx5c                             0/1       Completed   0          5h
ibm-cp4a-operator-547b95dfdc-p7vks                           2/2       Running     0          5h
```

</p>
</details>
</div>

2. For each pod, check under Events to see that the images were successfully pulled and the containers were created and started, by running the following command with the specific pod name:
```
oc describe pod <POD_NAME> -n <my-project>
```

3. Access Process Portal and Case Client

- Access PFS at first to federate Workflow server

PFS URL is `https://<pfs_hostname>/rest/bpm/federated/v1/systems` and `<pfs_hostname>` is corresponding to `pfs_configuration.hostname` property in your Custom Resource file. Remember to login with you LDAP user and password.

- Access Process Portal

Process Portal URL is `https://<baw_hostname>/ProcessPortal ` and `<baw_hostname>` is corresponding to `baw_configuration.hostname` property in your Custom Resource file.

- Access Case Client

Case Client URL is `https://<navigator_hostname>/navigator` and `<navigator_hostname>` is corresponding to `navigator_configuration.hostname` property in your Custom Resource file.



## Step 9: Uninstalling Business Automation Workflow
### Deleting your automation instances
You can delete your custom resource deployments by deleting the custom resource YAML file or the custom resource instance. The name of the instance is taken from the value of the `name` parameter in the YAML file. Use the following command to delete an instance:

```bash
  $ kubectl delete ICP4ACluster <MY-INSTANCE>
```

> **Note**: You can get the names of the ICP4ACluster instances by using the following command:
  ```bash
    $ kubectl get ICP4ACluster
  ```

### Deleting the operator instance and all associated automation instances
Use the `scripts/deleteOperator.sh` to delete all the resources that are linked to the operator:

```bash
   $ ./scripts/deleteOperator.sh
```

Verify that all the pods created with the operator are terminated and deleted.


## Limitations

* Business Automation Workflow supports only the IBM Db2 database.

* Elasticsearch limitation

  **Note:** The following limitation applies only if you are updating a containers deployment that uses the embedded Elasticsearch statefulset.
  
  * Scaling Elasticsearch statefulet
  
  In the Elasticsearch configuration, the operator automatically sets the [discovery.zen.minimun_master_nodes property](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/discovery-settings.html#minimum_master_nodes) to the quorum of replicas of the Elasticsearch statefulset. If, during an update, the pfs_configuration.elasticsearch.replicas value is changed, and this leads to a new computed value for the discovery.zen.minimun_master_nodes configuration property, then all currently running Elasticsearch pods must be restarted. While the pods are restarting, Elasticsearch and Process Federation Server services will be temporarily interrupted.
  * Elasticsearch High Availability

  In the Elasticsearch configuration, the operator automatically sets the [discovery.zen.minimun_master_nodes property](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/discovery-settings.html#minimum_master_nodes) to the quorum of replicas of the Elasticsearch statefulset. If some Elasticsearch pods fail and the number of running Elasticsearch pods is less than the quorum of replicas of the Elasticsearch statefulset, Elasticsearch and Process Federation Server services will be interrupted, at least until the quorum of running Elasticsearch pods is satisfied again.

* Resource Registry limitation

  Because of the design of etcd, it's recommended that you don't change the replica size after you create the Resource Registry cluster to prevent data loss. If you must set the replica size, set it to an odd number. If you reduce the pod size, the pods are destroyed one by one slowly to prevent data loss or the cluster getting out of sync.
  * If you update the Resource Registry admin secret to change the username or password, first delete the <instance_name>-dba-rr-<random_value> pods to cause Resource Registry to enable the updates. Alternatively, you can enable the update manually with etcd commands.
  * If you update the Resource Registry configurations in the icp4acluster custom resource instance, the update might not affect the Resource Registry pod directly. It will affect the newly created pods when you increase the number of replicas.

* The App Engine trusts only Certification Authority (CA) because of a Node.js server limitation. If an external service is used and signed with another root CA, you must add the root CA as trusted instead of the service certificate.

  * The certificate can be self-signed, or signed by a well-known CA.
  * If you're using a depth zero self-signed certificate, it must be listed as a trusted certificate.
  * If you're using a certificate signed by a self-signed CA, the self-signed CA must be in the trusted list. Using a leaf certificate in the trusted list is not supported.
  * If you're adding the root CA of two or more external services to the App Engine trust list, you can't use the same common name for those root CAs.
