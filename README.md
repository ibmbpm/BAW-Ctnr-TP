# IBM Business Automation Workflow on Containers 20.0.0.1 Tech Preview
Learn how to configure IBM Business Automation Workflow to run your applications, case solutions, and toolkits in a container environment.

## Table of contents
- [Introduction](#Introduction)
- [System requirements](#System-requirements)
- [Downloading the packages](#Downloading-the-packages)
- [Deployment options](#Deployment-options)
  - [Choosing a deployment type to install](#Choosing-a-deployment-type-to-install)
- [Troubleshooting](#Troubleshooting)
- [What is supported in the container environment](#What-is-supported-in-the-container-environment)
- [What is not supported in the container environment](#What-is-not-supported-in-the-container-environment)


## Introduction
When you target your Business Automation Workflow process app, case solution, or toolkit to the "Traditional or Container" environment, your process app, case solution, or toolkit is targeted to run in the Workflow Server in either the traditional WebSphere environment (IBM WebSphere Application Server Network Deployment) or the container environment (IBM WebSphere Liberty). The artifacts are validated in the web IBM Process Designer when you change the target environment of the process app, case solution, or toolkit to ensure that they are supported in this environment. You can include only those artifacts that are supported for containers.

For information about Business Automation Workflow in the container environment, see [What is supported in the container environment](#What-is-supported-in-the-container-environment).


## System requirements
The cluster administrator must check the system requirements and make sure that the system can host and run the deployment. The administrator and the installer (non-administrator user) should plan the deployment together.

The administrator must make sure that the target OpenShift cluster has the following tools and attributes.
   - Kubernetes 1.11+.
   - Kubernetes CLI. For more information, see [Install and Set Up kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/).
   - OpenShift Container Platform command line interface (CLI). The CLI has commands for managing your applications, and lower-level tools to interact with each component of your system. For an example, see [Get Started with the CLI for OpenShift 3.11](https://docs.openshift.com/container-platform/3.11/cli_reference/get_started_cli.html).
   - Dynamic storage created and ready.
     > **Tip**: For more information, see [Kubernetes NFS Client Provisioner](https://github.com/kubernetes-incubator/external-storage/tree/master/nfs-client), which includes instructions to configure an NFS server and client for the OCP nodes.
   - At least one non-administrator user that can be used to run the deployment script, for example `baw-user`.
   
The OpenShift cluster has the following minimum requirements for the deployment:

| Component 	| Master/Infra/Worker nodes | CPU per node | Memory per node | Storage |
| :---	| :---	| :---	| :---	| :--- |
| Business Automation Workflow    	| 1/1/3 | 4/4/4 | 16Gi | 66GB |

   > **Note**: The Master and Infrastructure nodes can be located on the same host as long as the host has enough resources. Masters with a co-located **etcd** need a minimum of 6 cores. OCP 3.11 does not support Docker alternative runtime environments that implement the Kubernetes CRI (Container Runtime Interface), such as CRI-O.

   
## Downloading the packages
Go to the IBM Business Automation Workflow on Containers Tech Preview page. The download tab provides archives (.tar.gz) for the software.

Download the provided archives to a server that is connected to your Docker registry.
- BAW-on-Ctnrs-20.0.0.1-baw.tar.gz
- BAW-on-Ctnrs-20.0.0.1-operator.tar.gz
- BAW-on-Ctnrs-20.0.0.1-Supplemental.tar.gz
- BAW-on-Ctnrs-20.0.0.1-scripts.tar.gz

The structure of the BAW-on-Ctnrs-20.0.0.1-baw.tar.gz package is
```
|-- images
|   |-- baw-jms-server_20.0.2.tar.gz
|   |-- dba-dbcompatibility-initcontainer_20.0.2.tar.gz
|   |-- dba-etcd_20.0.2.tar.gz
|   |-- dba-keytool-initcontainer_20.0.2.tar.gz
|   |-- dba-keytool-jobcontainer_20.0.2.tar.gz
|   |-- dba-umsregistration-initjob_20.0.2.tar.gz
|   |-- iaws-ps-content-integration_20.0.2.tar.gz
|   |-- pfs-bpd-database-init-prod_20.0.2.tar.gz
|   |-- pfs-elasticsearch-prod_20.0.2.tar.gz
|   |-- pfs-init-prod_20.0.2.tar.gz
|   |-- pfs-nginx-prod_20.0.2.tar.gz
|   |-- pfs-prod_20.0.2.tar.gz
|   |-- workflow-server_20.0.0.1.tar.gz
|   |-- workflow-server-case-initialization_20.0.0.1.tar.gz
|   `-- workflow-server-dbhandling_20.0.0.1.tar.gz
```

The structure of the BAW-on-Ctnrs-20.0.0.1-operator.tar.gz package is
```
|-- images
|   `-- icp4a-operator_20.0.2.tar
```

The structure of the BAW-on-Ctnrs-20.0.0.1-Supplemental.tar.gz package is
```
|-- images
|   |-- cmis_20.0.2.tar.gz
|   |-- cpe_20.0.2.tar.gz
|   |-- navigator-sso_20.0.2.tar.gz
|   `-- ums_20.0.2.tar.gz
```

The structure of the BAW-on-Ctnrs-20.0.0.1-scripts.tar.gz package is
```
|-- BAW-Ctnrs-TP
|   |-- descriptors
|   `-- scripts
```
In directory of `BAW-Ctnrs-TP`, you will find the scripts and Kubernetes descriptors that you need to install Business Automation Workflow.


## Deployment options
#### Choosing a deployment type to install
You can choose a demo or customized  deployment to install IBM Business Automation Workflow on Containers. A deployment that is intended for demonstration purposes is much quicker to install than a customized deployment. Scripts are provided to help an OpenShift cluster administrator set up the cluster and a non-administrator user to run the installation. The demo deployment provisions all of the required services, such as IBM Db2 and OpenLDAP, with the default values. With a customized deployment, you must prepare a Db2 and LDAP server before installing Business Automation Workflow.

Choose your deployment type:
- [Business Automation Workflow Demo Pattern](./BAW-on-Ctnrs-TP-Demo-pattern_v20.0.0.1.md)
- [Business Automation Workflow Customized](./BAW-on-Ctnrs-TP_v20.0.0.1.md)


## Troubleshooting
For installation issues, see [Troubleshooting a demo deployment](./BAW-on-Ctnrs-TP-Troubleshooting.md).


## What is supported in the container environment
Process apps, case solutions, and toolkits built with the web IBM Process Designer that do not use deprecated APIs or deprecated features.

However, because of differences between WebSphere ND and WebSphere Liberty with Kubernetes, high-availability disaster recovery (HADR) HADR will rely on Kubernetes. The fundamentals of application deployment onto the Workflow platform remain the same.


## What is not supported in the container environment

- Business Automation Insights
- Deprecated artifacts (except heritage human services)
- Advanced applications that are created in IBM Integration Designer such as BPEL, MFC, and SCA
- External services implemented by Java classes that rely on WebSphere ND-specific features. 
- Performance Data Warehouse, the Performance dashboard, and the Team dashboard
- Monitor models for IBM Business Monitor in process apps and toolkits
- Case Analyzer and Case Monitor
- Case forms
- wsadmin commands (replaced with REST APIs)
- Online deployment from IBM Workflow Center
- External Service Web Service invocation
- Inbound web services
- Inbound SCA invocations
- Advanced Integration Service (AIS) invocation
- SQL integration
- JMS and MQ integration
- Dynamic Event Framework (DEF) event emission

**Notes:** 
- To maximize compatibility, replace all deprecated features and APIs in your applications, case solutions, and toolkits, and upgrade all dependent system toolkits to the latest versions.
- Not all artifacts are disabled yet that can't be created and used in the Container environment.
- Check the Target Environment Conversion tab in Process Designer when converting an existing process application or toolkit.

For information about running IBM Business Automation Workflow, see  [IBM Business Automation Workflow documentation](https://www.ibm.com/support/knowledgecenter/en/SS8JB4/com.ibm.wbpm.workflow.main.doc/kc-homepage-workflow.html).

