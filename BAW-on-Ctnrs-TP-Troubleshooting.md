# Troubleshooting a demo deployment

The troubleshooting information is divided into the following sections:

 - [Cluster admin setup script issues](install_troubleshooting_ocp.md#cluster-admin-setup-script-issues)
 - [IBM Db2 issues](install_troubleshooting_ocp.md#ibm-db2-issues)
 - [Route issues](install_troubleshooting_ocp.md#route-issues)
 
## Cluster admin setup script issues

### Issue: You run the  cp4a-clusteradmin-setup.sh script but the Custom Resource Definition (CRD) fails to deploy

If you see the following message in the output, the user ('XYZ' in this example message) does not have cluster-admin permission:
```
Start to create CRD, service account and role ..Error from server (Forbidden): error when retrieving current configuration of:
"/root/git/cert-kubernetes/descriptors/ibm_cp4a_crd.yaml": customresourcedefinitions.apiextensions.k8s.io "icp4aclusters.icp4a.ibm.com" is forbidden: User "XYZ" cannot get customresourcedefinitions.apiextensions.k8s.io at the cluster scope: no RBAC policy matched
```
Fix the issue by following these steps:
1. Log out of the current OpenShift Cloud Platform (OCP) session (non-admin).

2. Log in to OCP with the OCP cluster admin user.
   ```bash
   $ oc login -u dbaadmin 
   ```
   where dbaadmin is the OCP cluster admin user

## IBM Db2 issues

Db2 is installed as a prerequisite of the demo pattern.

### Issue: Intermittent issue where the Db2 process is not listening on port 50000

If you find a `not listening on port 50000` message in the logs, fix the issue by following these steps:

1. Get the current running Db2 pod: 
   ```bash
   $ oc get pod
   ```
2. Go to the pod: 
   ```bash
   $ oc exec -it <db2 pod> bash
   ```
3. Switch to the db2inst1 user: 
   ```bash
   $ su - db2inst
   ```
4. Re-apply the configuration: 
   ```bash
   $ db2 update dbm cfg using SVCENAME 50000
   ```
5. Restart Db2:
   ```bash
   $ db2stop
   $ db2start
   ```

### Issue: Db2 pod fails to start when  db2u-release-db2u-0 pod shows 0/1 Ready

This issue has the following symptoms in the Db2 pods:
```
[5357278.440940] db2u_root_entrypoint.sh[20]: + sudo /opt/ibm/db2/V11.5.0.0/adm/db2licm -a /db2u/license/db2u-lic
[5357278.531782] db2u_root_entrypoint.sh[20]: LIC1416N  The license could not be added automatically.  Return code: "-100".
[5357278.535893] db2u_root_entrypoint.sh[20]: + [[ 156 -ne 0 ]]
[5357278.536085] db2u_root_entrypoint.sh[20]: + echo '(*) Unable to apply db2 license.'
[5357278.536177] db2u_root_entrypoint.sh[20]: (*) Unable to apply db2 license.
```

To mitigate the issue, you have the following options: 
 - Option 1: Stop Db2
 - Option 2: Clean up Db2 and redeploy
 - Option 3: Delete the project
 - Option 4: Reboot the cluster

**Option 1: Stop Db2**
1. Run the following command to get the worker node that db2u is running on: 
   ```bash
   $ oc get pods -o wide
   ```
2. Run a ssh command as root on the worker node hosting Db2u: 
   ```bash
   $ ssh root@<worker node>
   ```
3. Run the following command to stop the orphaned db2u semaphores:
   ```bash
   $ ipcrm -S 0x61a8 
   ```
4. Clean up the affected project (namespace) by running the following commands: 
   ```bash
   $ oc get icp4acluster to get  the custom resource name
   $ oc delete icpa4acluster $name from step(a)
   $ oc delete <operator-deployment-name>
   ```
5. Run the deployment script to start again.

**Option 2: Clean Db2 and redeploy**
1. Get the custom resource name for the icp4acluster:
   ```bash
   $ oc get icp4acluster  
   ```   
2. Delete the custom resource: 
   ```bash
   $ oc delete icp4acluster $name
   ```
   or
   ```bash
   $ oc delete -f $cr.yaml
   ```
   The `$cr.yaml` is generated in the ./tmp directory, so you must also delete the operator deployment by running the following command: 
   ```bash
   $ oc delete <operator-deployment-name>
   ```
3. Make sure there is nothing left by running the following commands:
   ```bash
   $ oc get sts
   $ oc get jobs
   $ oc get deployment
   $ oc get pvc | grep db2
   ```
4. Run the deployment script to start again.

**Option 3: Delete the project**
1. If options 1 or 2 don't work, delete the project (namespace) by running the following command:
   ```bash
   $ oc delete project $project_name
   ```
 2. Redeploy the project.  

**Option 4: Reboot the entire cluster**
1. If none of the other options work, get the names of the nodes and reboot them:
   ```bash
   $ oc get no --no-headers | awk '{print $1}'
   ```
2. Reboot all of the nodes listed (reboot the worker nodes first, then the infrastructure node, and then the master node).

### Issue: db2-release-db2u-restore-morph-job-xxxxx shows "Running" but never shows "Completed"

Run the following command to check and confirm this issue:
```bash
$ oc get pod
```

The command outputs a table showing the STATUS and READY columns:
```bash
NAME                                        READY        STATUS 
db2-release-db2u-restore-morph-job-xxxxx    1/1          Running
```

If the STATUS does not change to `Completed` after a few minutes, fix the issue by following these steps:

1. Delete the Db2 pod by running the `oc delete` command: 
   ```bash
   $ oc delete pod db2-release-db2u-restore-morph-job-xxxxx
   ```
2. Confirm that the Db2 job is terminated and a new pod is up and running:
   ```bash
   $ oc get pod -w
   ```
   When the job reads `Completed`, the deployment can continue.

### Issue: Error: failed to start container "demo-cpe-deploy" or "demo-navigator-deploy"
The detailed error message is something like "Error response from daemon: oci runtime error: container_linux.go:235: starting container process caused "container init exited prematurely"". This kind of error is caused by incorrect binding for IBM Content Navigator or Content Platform Engine persistent volumes (PVs) and persistent volume claims (PVCs)

To fix this issue, delete the IBM Content Navigator or Content Platform Engine PVC, and then delete the IBM Content Navigator or Content Platform Engine related PV and NFS folders. Then, re-create them in the reverse order.

### Issue: Failed to start Pod "demo-ibm-pfs-elasticsearch-0"
To fix this issue, check the value of the `pfs_configuration.elasticsearch.privileged` property in your custom resource configuration. If it's set to `true`, run the `oc describe pod demo-ibm-pfs-elasticsearch-0` command to check the SecurityContextConstraint of the `demo-ibm-pfs-elasticsearch-0` Pod. Also, ensure it’s set as `openshift.io/scc=pfs-privileged-scc`. 
```
# oc describe pod demo-ibm-pfs-elasticsearch-0
Name:               demo-ibm-pfs-elasticsearch-0
Namespace:          demo-project
Priority:           0
PriorityClassName:  <none>
Node:               rhel76/<OPENSHIFT_MASTER_IP>
Start Time:         Thu, 21 Nov 2019 18:10:11 +0800
Labels:             app.kubernetes.io/component=pfs-elasticsearch
                    app.kubernetes.io/instance=demo
                    app.kubernetes.io/managed-by=Operator
                    app.kubernetes.io/name=demo-ibm-pfs-elasticsearch
                    app.kubernetes.io/version=19.0.3
                    controller-revision-hash=demo-ibm-pfs-elasticsearch-8675f484d
                    role=elasticsearch
                    statefulset.kubernetes.io/pod-name=demo-ibm-pfs-elasticsearch-0
Annotations:        checksum/config=6a3747ddc8ce13afdfc85b6793b847d035e8edd5
                    openshift.io/scc=pfs-privileged-scc
                    productID=5737-I23
                    productName=IBM Cloud Pak for Automation
                    productVersion=19.0.3
Status:             Running
```

### Issue: How To enable Business Automation Workflow container logs
Use the following specification to enable container logs in the custom resource configuration:
```yaml
baw_configuration:
 - name: instance1
    logs:
    console_format: “json”
    console_log_level: “INFO”
    console_source: “message,trace,accessLog,ffdc,audit”
    message_format: “basic”
    trace_format: “ENHANCED”
    trace_specification: “WLE.=all:com.ibm.bpm.=all：com.ibm.workflow.*=all”
```

Run the `oc logs IAWS_pod_name` command to see the logs, or log into Business Automation Workflow to see the logs.

The following example shows how to check the Business Automation Workflow container logs:
```
$ oc exec -it demo-instance1-ibm-iaws-server-0 bash
$ cat /logs/application/liberty-message.log
```

### Issue: How to customize the Process Federation Server liberty server trace setting
Use the following specification to enable Process Federation Server container logs in the custom resource configuration:
```yaml
pfs_configuration:
   pfs:
     logs:
       console_format: "json"
       console_log_level: "INFO"
       console_source: "message,trace,accessLog,ffdc,audit"
       trace_format: "ENHANCED"
       trace_specification: "*=info"
```

Run the `oc logs PFS_pod_name` command to see the logs, or log into Process Federation Server to see the logs.

The following example shows how to check the Process Federation Server container logs:
```
$ oc exec -it demo-ibm-pfs-0 bash
$ cat /logs/application/liberty-message.log
```


## Route issues

### Issue: Generated routes do not work

In some environments, route URLs contain the string `apps.`. However, the cp4a-clusteradmin-setup.sh script returns the hostname of the infrastructure node without this string. If you entered the hostname in the cp4a-post-deployment.sh script in an environment that uses `apps.`, the routes do not work. 

**Workaround:**
When you run the cp4a-deployment.sh script, add `apps.` to the infrastructure hostname. 

For example, if the cp4a-clusteradmin-setup.sh script outputs the infrastructure hostname as `ocp-master.tec.uk.ibm.com`, enter `ocp-master.apps.tec.uk.ibm.com` when you run the cp4a-post-deployment.sh script.

Tip: You can find the existing route URL by running `oc get route --all-namespaces` and extracting the common pattern URL for the routes.
