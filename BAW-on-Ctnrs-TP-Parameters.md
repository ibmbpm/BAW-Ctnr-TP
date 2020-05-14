## Business Automation Workflow Server parameters

Each container needs a set of values for its configuration parameters to create a Kubernetes deployment. The following tables provide the description and default value for each parameter. Complete the configuration files for your deployment by supplying values for your environment and configuration.

## Table of contents
- [Shared configuration parameters](#Shared-configuration-parameters)
  - [Mandatory configuration parameters](#Mandatory-configuration-parameters)
  - [LDAP configuration parameters](#LDAP-configuration-parameters)
- [User Management Service parameters](#User-Management-Service-parameters)
  - [UMS datasource parameters](#UMS-datasource-parameters)
  - [UMS configuration parameters](#UMS-configuration-parameters)
- [Resource Registry parameters](#Resource-Registry-parameters)
- [Business Automation Workflow Server parameters](#Business-Automation-Workflow-Server-parameters)
  - [Workflow Services configuration parameters](#Workflow-Services-configuration-parameters)
  - [Java Messaging Service configuration parameters](#Java-Messaging-Service-configuration-parameters)
  - [Process Federation Server configuration parameters](#Process-Federation-Server-configuration-parameters)
  - [Elasticsearch configuration parameters](#Elasticsearch-configuration-parameters)
- [FileNet Content Manager parameters](#FileNet-Content-Manager-parameters)
  - [FileNet Content Manager datasource parameters](#FileNet-Content-Manager-datasource-parameters)
  - [FileNet Content Manager common parameters](#FileNet-Content-Manager-common-parameters)
  - [Monitoring parameters](#Monitoring-parameters)
  - [Logging parameters](#Logging-parameters)
  - [Content Platform Engine parameters](#Content-Platform-Engine-parameters)
  - [Content Management Interoperability Services parameters](#Content-Management-Interoperability-Services-parameters)
  - [Initialization parameters](#Initialization-parameters)
- [Business Automation Navigator parameters](#Business-Automation-Navigator-parameters)
  - [Business Automation Navigator Datasource parameters](#Business-Automation-Navigator-Datasource-parameters)
  - [Business Automation Navigator configuration parameters](#Business-Automation-Navigator-configuration-parameters)



## Shared configuration parameters
Some configuration parameters can be shared by an installation of multiple automation components.
### Mandatory configuration parameters
You must set the parameters in the custom resource file to access the docker images in your environment.

The following table lists the configurable parameters and default values. All properties are mandatory.

| Parameter                              | Description                                           | Default             |
| -------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------- |
| shared_configuration.<br>root_ca_secret| Root certificate authority (CA) secret name to store the root CA TLS key and certificate.| icp4a-root-ca|
| shared_configuration.<br>encryption_key_secret| Shared encryption key secret name that is used for Workflow Services and Process Federation Server integration.| icp4a-shared-encryption-key|
| shared_configuration.<br>sc_deployment_type| Digital Business Automation can be installed for non-production and production. Set the value to "non-production" in your non-production environments only.| production|
| shared_configuration.<br>image_pull_secrets| Shared image pull secrets.                            | []                                                   |
| shared_configuration.images.<br>keytool_job_container.repository | Image name for Transport Layer Security (TLS) job container.  | dba-keytool-jobcontainer |
| shared_configuration.images.<br>keytool_job_container.tag| Image tag for TLS job container | 	20.0.2|
| shared_configuration.images.<br>dbcompatibility_init_container.repository| Image name for database compatibility init container.| dba-dbcompatibility-initcontainer|
| shared_configuration.images.<br>dbcompatibility_init_container.tag| Image tag for database compatibility init container.| 20.0.2|
| shared_configuration.images.<br>keytool_init_container.repository| Image name for TLS init container.| dba-keytool-initcontainer|
| shared_configuration.images.<br>keytool_init_container.tag| Image tag for TLS init container.| 20.0.2|
| shared_configuration.images.<br>umsregistration_initjob.repository| Image name for OpenID Connect (OIDC) registration job container.| dba-umsregistration-initjob|
| shared_configuration.images.<br>umsregistration_initjob.tag| Image tag for OIDC registration job container.| 20.0.2|
| shared_configuration.images.<br>pull_policy| Pull policy for all containers.| IfNotPresent|



### LDAP configuration parameters
A server that runs the Lightweight Directory Access Protocol (LDAP) can be configured by more than one component on Kubernetes.

Download the sample configuration XML files from the folders on GitHub and modify a file to match your existing LDAP server. Follow the instructions to apply the modified configuration file in your deployment. Options include IBM Security Directory Server and Active Directory.

| Parameter                              | Description                                           | IBM Security Directory Server example values             |
| -------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------- |
| ldap_configuration.<br>lc_selected_ldap_type| The type of the directory.| IBM Security Directory Server|
| ldap_configuration.<br>lc_ldap_server| The hostname must be either the fully qualified domain name or IP address of your LDAP server.| HOSTNAME|
| ldap_configuration.<br>lc_ldap_port| 	The LDAP server host port number.| 389|
| ldap_configuration.<br>lc_bind_secret| User name and password for the bind user. The LDAP bind secret must have ldapUsername and ldapPassword keys.| ldap-bind-secret|
| ldap_configuration.<br>lc_ldap_base_dn| The LDAP base distinguished name (DN). The base DN subtree is used when you search for user entries on the LDAP server.| o=mycompany,c=us|
| ldap_configuration.<br>lc_ldap_ssl_enable| Specifies whether SSL is used to access LDAP server.| true, false|
| ldap_configuration.<br>lc_ldap_ssl_secret_name| Specifies a secret name that includes an SSL certificate to use when SSL is used to access LDAP server.| ldap-ssl-cert|
| ldap_configuration.<br>lc_ldap_user_name_attribute| The LDAP attribute that represents the full name of the user.| *:cn or *:uid|
| ldap_configuration.<br>lc_ldap_user_display_name_attr| The LDAP attribute to display for the full name of the user.	| cn or uid|
| ldap_configuration.<br>lc_ldap_group_base_dn| The LDAP group base distinguished name (DN). The base DN subtree is used when you search for group entries on the LDAP server.| dc=hqpsidcdom,dc=com|
| ldap_configuration.<br>lc_ldap_group_name_attribute| The LDAP attribute that represents the group name.|	*:cn |
| ldap_configuration.<br>lc_ldap_group_display_name_attr| The LDAP attribute to display the full name of the group.| 	cn|
| ldap_configuration.<br>lc_ldap_group_membership_search_filter| Search filter for finding group membership.| (&(cn=%v)(|(objectclass=groupOfNames)(objectclass=groupOfUniqueNames)(objectclass=groupOfURLs))|
| ldap_configuration.<br>lc_ldap_group_member_id_map| Identifies the group member.| groupofnames:member|
| ldap_configuration.<br>lc_ldap_max_search_results| Specify a higher value if you expect more search results.| |
| ldap_configuration.<br>tds.lc_user_filter| Search filter for finding entries in the IBM Directory Server base DN users subtree that match the username.| (&(cn=%v)(objectclass=person))|
| ldap_configuration.<br>tds.lc_group_filter| Search filter for finding entries in the IBM Directory Server base DN group subtree that match the group name.| (&(cn=%v)(|(objectclass=groupofnames)(objectclass=groupofuniquenames)(objectclass=groupofurls)))	|

For components that require LDAP, use the `lc_bind_secret` parameter in the template YAML file to locate a secret that includes the ldapUsername and ldapPassword keys. Specify the secret name that you create in the `lc_bind_secret` parameter.
```yaml
ldap_configuration:
  lc_bind_secret: ldap-bind-secret
```

The following YAML shows an example `ldap_configuration` section:
```yaml
ldap_configuration:
    # the candidate value is "IBM Security Directory Server" or "Microsoft Active Directory"
    lc_selected_ldap_type: "IBM Security Directory Server"
    lc_ldap_server: "<ldap_server_host>"
    lc_ldap_port: "389"
    lc_ldap_base_dn: "dc=hqpsidcdom,dc=com"
    lc_ldap_ssl_enabled: false
    lc_ldap_ssl_secret_name: ""
    lc_ldap_user_name_attribute: "*:cn"
    lc_ldap_user_display_name_attr: "cn"
    lc_ldap_group_base_dn: "dc=hqpsidcdom,dc=com"
    lc_ldap_group_name_attribute: "*:cn"
    lc_ldap_group_display_name_attr: "cn"
    lc_ldap_group_membership_search_filter: "(|(&(objectclass=groupofnames)(member={0}))(&(objectclass=groupofuniquenames)(uniquemember={0})))"
    lc_ldap_group_member_id_map: "groupofnames:member"
    lc_ldap_max_search_results: 4500
    ad:
      lc_ad_gc_host: ""
      lc_ad_gc_port: ""
      lc_user_filter: "(&(cn=%v)(objectclass=person))"
      lc_group_filter: "(&(cn=%v)(|(objectclass=groupofnames)(objectclass=groupofuniquenames)(objectclass=groupofurls)))"
    tds:
      lc_user_filter: "(&(cn=%v)(objectclass=person))"
      lc_group_filter: "(&(cn=%v)(|(objectclass=groupofnames)(objectclass=groupofuniquenames)(objectclass=groupofurls)))"
```



## User Management Service parameters
Configuration parameters for the User Management Service (UMS) on Kubernetes. 
### UMS datasource parameters
Provide appropriate values for the User Management Service (UMS) datasource configuration parameters. These are specified in the section dc_ums_datasource.

Most UMS datasource configuration parameters are optional, the following parameters are required:
- datasource_configuration.dc_ums_datasource.dc_ums_oauth_type
  - datasource_configuration.dc_ums_datasource.dc_ums_oauth_host
  - datasource_configuration.dc_ums_datasource.dc_ums_oauth_port
  - datasource_configuration.dc_ums_datasource.dc_ums_oauth_name

- datasource_configuration.dc_ums_datasource.dc_ums_teamserver_type
  - datasource_configuration.dc_ums_datasource.dc_ums_teamserver_host
  - datasource_configuration.dc_ums_datasource.dc_ums_teamserver_port
  - datasource_configuration.dc_ums_datasource.dc_ums_teamserver_name


| Parameter                              | Description                                           | Default             |
| -------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------- |
| datasource_configuration.<br>dc_ums_datasource.dc_ums_oauth_type| Required: The type of OAuth database (db2).| db2|
| datasource_configuration.<br>dc_ums_datasource.dc_ums_oauth_host| Required if the OAuth database is db2 or oracle: The host name of the OAuth database.| None|
| datasource_configuration.<br>dc_ums_datasource.dc_ums_oauth_port| Required if the OAuth database is db2 or oracle: OAuth database port. For example, 50000| None|
| datasource_configuration.<br>dc_ums_datasource.dc_ums_oauth_name| Required if the OAuth database is db2 or oracle: The name of the OAuth database. For example UMSDB| None|
| datasource_configuration.<br>dc_ums_datasource.dc_ums_oauth_schema| Optional parameter, can be specified if a schema was created.| |
| datasource_configuration.<br>dc_ums_datasource.dc_ums_oauth_ssl| Optional: Specify true if SSL will be used to secure the OAuth database connection.| false|
| datasource_configuration.<br>dc_ums_datasource.dc_ums_oauth_ssl_secret_name| Optional: If SSL will be used to secure the database connection, specify the name of the SSL secret. For example ibm-dba-ums-db2-cacert| None|
| datasource_configuration.<br>dc_ums_datasource.dc_ums_oauth_driverfiles| Optional: If you are using a database of type other than Db2® or derby, copy the driver files to the connected persistent volume (PV). Use the property spec.ums_configuration.existing_claim_name to point to the PV claim. During the deployment Operator will pick up the driver files and configure the connection to the database| db2jcc4.jar db2jcc_license_cu.jar.|
| datasource_configuration.<br>dc_ums_datasource.dc_ums_oauth_alternate_hosts| Optional: Alternate OAuth database hosts in a high-availability (HA) environment.	| None|
| datasource_configuration.<br>dc_ums_datasource.dc_ums_oauth_alternate_ports	| Optional: Alternate ports of OAuth database hosts in an HA environment.| None|
| datasource_configuration.<br>dc_ums_datasource.dc_ums_teamserver_type	| Required: The type of UMS Teams database (db2).| db2|
| datasource_configuration.<br>dc_ums_datasource.dc_ums_teamserver_host| Required if the UMS Teams database is db2: The host name of the UMS Teams database.| None|
| datasource_configuration.<br>dc_ums_datasource.dc_ums_teamserver_port| Required if the UMS Teams database is db2: UMS Teams database port. For example, 50000| None|
| datasource_configuration.<br>dc_ums_datasource.dc_ums_teamserver_name| Required if the UMS Teams database is db2: The name of the UMS Teams database. For example UMSTEAMSDB| None|
| datasource_configuration.<br>dc_ums_datasource.dc_ums_teamserver_ssl| Optional: Specify true if SSL will be used to secure the UMS Teams database connection.| false|
| datasource_configuration.<br>dc_ums_datasource.dc_ums_teamserver_ssl_secret_name| Optional: If SSL will be used to secure the UMS Teams database connection, specify the name of the SSL secret. For example ibm-dba-ums-db2-cacert| None|
| datasource_configuration.<br>dc_ums_datasource.dc_ums_teamserver_driverfiles| Optional: During the deployment Operator will pick up the driver files and configure the connection to the UMS Teams database| db2jcc4.jar db2jcc_license_cu.jar.|
| datasource_configuration.<br>dc_ums_datasource.dc_ums_teamserver_alternate_hosts| Optional: Alternate UMS Teams database hosts in a high-availability (HA) environment.| None|
| datasource_configuration.<br>dc_ums_datasource.dc_ums_teamserver_alternate_ports| Optional: Alternate ports of UMS Teams database hosts in an HA environment.	| None|


### UMS configuration parameters
Provide appropriate values for the User Management Services (UMS) configuration parameters. These are specified in the section ums_configuration.

Most configuration parameters are optional, only two parameters are required:
- ums_configuration.images.ums.repository: The repository from where the UMS image is pulled.
- ums_configuration.images.ums.tag: The UMS image tag.

| Parameter                              | Description                                           | Default             |
| -------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------- |
| ums_configuration.<br>existing_claim_name| Optional: The name of the Persistent Volume Claim for JDBC drivers and custom binaries.| None|
| ums_configuration.<br>replica_count| Optional: The number of pod replicas running by default.| 2|
| ums_configuration.<br>service_type| Optional: The type to expose the service as. For example, Route for external access or NodePort for internal tests.If you specify NodePort, the value for ums_configuration.port must be between 30000 and 32767.| Route|
| ums_configuration.<br>hostname | Optional: The name of the host where the User Management Service will run. | If not specified, hostname is generated from shared_configuration.sc_deployment_hostname_suffix|
| ums_configuration.<br>port| Optional: The port that will be used to access the User Management Service, for example, 443 when using SSL.| 443|
| ums_configuration.<br>images.ums.repository| Required: The repository from where the UMS image is pulled.| None|
| ums_configuration.<br>images.ums.tag| Required: The UMS image tag.| None|
| ums_configuration.<br>admin_secret_name| Optional: The name of the secret that was generated for the UMS secret and database secret.| If not specified, the secret ibm-dba-ums-secret must be created.|
| ums_configuration.<br>external_tls_secret_name| Optional: Enables SSL with an existing certificate. If you don't want to provide an external TLS certificate, leave this empty.| None|
| ums_configuration.<br>external_tls_ca_secret_name| Optional: Certificate Authority (CA) used to sign the external TLS secret. If you don't want to provide a CA to sign the external TLS certificate, leave this empty.| None|
| ums_configuration.<br>oauth.client_manager_group| Optional: The full DN of an LDAP group that is authorized to manage OIDC clients, in addition to the primary admin from the admin secret.| None|
| ums_configuration.<br>oauth.token_manager_group| Optional: The full DN of an LDAP group that is authorized to manage tokens, in addition to the primary admin from the admin secret.| None|
| ums_configuration.<br>oauth.access_token_lifetime| Optional: The lifetime of OAuth access_tokens.| 7200s|
| ums_configuration.<br>oauth.app_token_lifetime| Optional: The lifetime of app-tokens.| 366d|
| ums_configuration.<br>oauth.app_password_lifetime| Optional: The lifetime of app-passwords.| 366d|
| ums_configuration.<br>oauth.app_token_or_password_limit| Optional: The maximum number of app-tokens or app-passwords per client.| 100|
| ums_configuration.<br>oauth.client_secret_encoding| Optional: The encoding / encryption when storing client secrets in the OAuth database.| Default is xor for compatibility. Recommended value is PBKDF2WithHmacSHA512|
| ums_configuration.<br>custom_secret_name| Optional: The name of the existing secret for sensitive Liberty configuration, specified in XML format.| None|
| ums_configuration.<br>resources| Optional: Kubernetes controls resources such as CPU and memory using requests and limits mechanisms. Requests are what the container is guaranteed to get. Limits make sure a container never goes above a certain value. A limit value cannot be lower than the corresponding request value. For more information about determining values for cpu and memory, see the planning section of the UMS readme.| |
| ums_configuration.<br>resources.limits.cpu| Optional: The maximum CPU limit.| 500m|
| ums_configuration.<br>resources.limits.memory| Optional: The maximum memory limit.| 512Mi|
| ums_configuration.<br>resources.requests.cpu| Optional: The minimum CPU.| 200m|
| ums_configuration.<br>resources.requests.memory| Optional: The minimum memory.| 256Mi|
| ums_configuration.<br>autoscaling.enabled| Optional: If true, pods are automatically scaled within the specified range.| true|
| ums_configuration.<br>autoscaling.min_replicas| Optional: The minimum number of replicas for autoscaling.| 2|
| ums_configuration.<br>autoscaling.max_replicas| Optional: The maximum number of replicas for autoscaling.| 5|
| ums_configuration.<br>autoscaling.target_average_utilization| Optional: The average CPU utilization for autoscaling.| 98|
| ums_configuration.<br>use_custom_jdbc_drivers| Optional: Specify if JDBC drivers other than Db2® are used.| false|
| ums_configuration.<br>use_custom_binaries| Optional: Specify if any custom binaries are used.| false|
| ums_configuration.<br>custom_secret_name| Optional: The name of the existing secret for sensitive Liberty configuration, specified in XML format.| None|
| ums_configuration.<br>custom_xml| Optional: Custom configuration settings (optional, multi-line value). For LDAP configuration use spec.ldap_configuration parameters.| None|
| ums_configuration.<br>logs.console_format| Optional: The format of the UMS logs console.| json|
| ums_configuration.<br>logs.console_log_level| Optional: The log level for the UMS logs console.| INFO|
| ums_configuration.<br>logs.console_source| Optional: UMS logs console source.| message,trace,accessLog,ffdc,audit|
| ums_configuration.<br>logs.trace_format| Optional: The format of the UMS logs trace.| ENHANCED|
| ums_configuration.<br>logs.trace_specification| Optional: The UMS logs trace specification.| *=info|


## Resource Registry parameters
Provide the details that are relevant to your Resource Registry component and your decisions for the deployment of the container.

| Parameter                              | Description                                           | Default             |
| -------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------- |
| resource_registry_configuration.<br>admin_secret_name| Existing Resource Registry administrative secret for sensitive configuration data| rr-admin-secret|
| resource_registry_configuration.<br>hostname| Resource Registry external hostname| |
| resource_registry_configuration.<br>service_type| How the HTTPS endpoint service should be published. Possible values are ClusterIP, NodePort, Route.| Route|
| resource_registry_configuration.<br>port| Resource Registry port for using the NodePort service| 443|
| resource_registry_configuration.<br>replica_size| Number of etcd nodes in the cluster| 3|
| resource_registry_configuration.<br>images.pull_policy| Pull policy of the Resource Registry container.| dba-etcd|
| resource_registry_configuration.<br>images.resource_registry.repository| Image name of the Resource Registry container.| dba-etcd|
| resource_registry_configuration.<br>images.resource_registry.tag| Image tag of the Resource Registry container.| 20.0.2|
| resource_registry_configuration.<br>tls.tls_secret| Existing TLS secret that contains tls.key and tls.crt| rr-tls-client-secret|
| resource_registry_configuration.<br>probe.liveness.initial_delay_seconds| Number of seconds after the container starts before the liveness probe is initiated| 60|
| resource_registry_configuration.<br>probe.liveness.period_seconds| How often (in seconds) to perform the probe| 10|
| resource_registry_configuration.<br>probe.liveness.timeout_seconds| Number of seconds after which the probe times out| 5|
| resource_registry_configuration.<br>probe.liveness.success_threshold| Minimum consecutive successes for the probe to be considered successful after failing. Minimum value is 1.| 1|
| resource_registry_configuration.<br>probe.liveness.failure_threshold| When a pod starts and the probe fails, Kubernetes tries this number of times before giving up. Minimum value is 1.| 3|
| resource_registry_configuration.<br>probe.readiness.initial_delay_seconds| Number of seconds after the container starts before the liveness probe is initiated| 10|
| resource_registry_configuration.<br>probe.readiness.period_seconds| How often (in seconds) to perform the probe| 10|
| resource_registry_configuration.<br>probe.readiness.timeout_seconds| Number of seconds after which the probe times out| 5|
| resource_registry_configuration.<br>probe.readiness.success_threshold| Minimum consecutive successes for the probe to be considered successful after failing. Minimum value is 1.| 1|
| resource_registry_configuration.<br>probe.readiness.failure_threshold| When a pod starts and the probe fails, Kubernetes tries this number of times before giving up. Minimum value is 1.| 3|
| resource_registry_configuration.<br>resource.limits.cpu| CPU limit for Resource Registry configuration| 500m|
| resource_registry_configuration.<br>resource.limits.memory| Memory limit for Resource Registry configuration| 512Mi|
| resource_registry_configuration.<br>resource.requests.cpu| Requested CPU for Resource Registry configuration| 200m|
| resource_registry_configuration.<br>resource.requests.memory| Requested memory for Resource Registry configuration| 256Mi|



## Business Automation Workflow Server parameters
Provide the details that are relevant to your Business Automation Workflow environment and your decisions for the deployment of the container.

The following tables list the parameters for configuring Workflow Services.
- Workflow Services configuration parameters
- Process Federation Server configuration parameters
- Elasticsearch configuration parameters


### Workflow Services configuration parameters
| Parameter                              | Description                                           | Default             |
| -------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------- |
| baw_configuration[x].<br>name| Name of the instance. The name for each item in the array must be different.| instance1|
| baw_configuration[x].<br>host_federated_portal| | |
| baw_configuration[x].<br>service_type| Workflow server service type.| Route|
| baw_configuration[x].<br>hostname| Workflow server hostname. This parameter is required.| <hostname>|
| baw_configuration[x].<br>port| Workflow server port| 443|
| baw_configuration[x].<br>env_type| | |
| baw_configuration[x].<br>replicas| Workflow server replica count.| 1|
| baw_configuration[x].<br>admin_secret_name| | |
| baw_configuration[x].<br>admin_user| Administrator user for Workflow server. This parameter is required.| |
| baw_configuration[x].<br>tls.tls_secret_name| Workflow server TLS secret that contains tls.key and tls.crt.| |
| baw_configuration[x].<br>tls.tls_trust_list| Workflow server TLS trust list.| |
| baw_configuration[x].<br>image.repository| Workflow server (process server) image repository URL.| workflow-server|
| baw_configuration[x].<br>image.tag| Image tag for Workflow server container.| 20.0.0.1|
| baw_configuration[x].<br>image.pullPolicy| Pull policy for Workflow server container.| Always|
| baw_configuration[x].<br>pfs_bpd_database_init_job.repository| Database initialization image repository URL for Process Federation Server.| pfs-bpd-database-init-prod|
| baw_configuration[x].<br>pfs_bpd_database_init_job.tag| Database initialization image repository tag for Process Federation Server.| 20.0.2|
| baw_configuration[x].<br>pfs_bpd_database_init_job.pullPolicy| Pull policy for Process Federation Server database initialization image.| Always|
| baw_configuration[x].<br>upgrade_job.repository| Workflow server database handling image repository URL.| workflow-server-dbhandling|
| baw_configuration[x].<br>upgrade_job.tag| Workflow server database handling image repository tag.| 20.0.0.1|
| baw_configuration[x].<br>upgrade_job.pullPolicy| Pull policy for database handling.| Always|
| baw_configuration[x].<br>database.ssl| Whether to enable Secure Sockets Layer (SSL) support for the Workflow server database connection| false|
| baw_configuration[x].<br>database.sslsecretname| Secret name for storing the database TLS certificate when an SSL connection is enabled| ibm-dba-baw-db2-cacert|
| baw_configuration[x].<br>database.type| Workflow server database type| DB2|
| baw_configuration[x].<br>database.server_name| Workflow server database server name. This parameter is required.| <db2_server>|
| baw_configuration[x].<br>database.database_name| Workflow server database name. This parameter is required.| BPMDB|
| baw_configuration[x].<br>database.port| Workflow server database port. This parameter is required.| 50000|
| baw_configuration[x].<br>database.secret_name| Workflow server database secret name. This parameter is required.| ibm-baw-wfs-server-db-secret|
| baw_configuration[x].<br>database.dbcheck.wait_time| Workflow server database check wait time| 900|
| baw_configuration[x].<br>database.dbcheck.interval_time| Workflow server database interval time| 15|
| baw_configuration[x].<br>database.hadr.standbydb_host| Database standby host for high availability disaster recovery (HADR). To enable database HADR, configure both standby host and port.| |
| baw_configuration[x].<br>database.hadr.standbydb_port| Database standby port for HADR. To enable database HADR, configure both standby host and port.| |
| baw_configuration[x].<br>database.hadr.retryinterval| Retry interval for HADR| |
| baw_configuration[x].<br>database.hadr.maxretries| Maximum retries for HADR| |
| baw_configuration[x].<br>database.hadr.maxretries| Maximum retries for HADR| |
| baw_configuration[x].<br>content_integration.init_job_image.repository| Image name for content integration container.| iaws-ps-content-integration|
| baw_configuration[x].<br>content_integration.init_job_image.tag| Image tag for content integration container.| 20.0.2|
| baw_configuration[x].<br>content_integration.init_job_image.pull_policy| Pull policy for content integration container.| Always|
| baw_configuration[x].<br>content_integration.domain_name| Domain name for content integration.| P8DOMAIN|
| baw_configuration[x].<br>content_integration.object_store_name| Object Store name for content integration.| DOCS|
| baw_configuration[x].<br>content_integration.cpe_admin_secret| Admin secret for content integration.| ibm-fncm-secret|
| baw_configuration[x].<br>case.init_job_image.repository| Image name for CASE init job container.| workflow-server-case-initialization|
| baw_configuration[x].<br>case.init_job_image.tag| Image tag for CASE init job container.| 20.0.0.1|
| baw_configuration[x].<br>case.init_job_image.pull_policy| Pull policy for CASE init job container.| Always|
| baw_configuration[x].<br>case.domain_name| Domain name for CASE.| P8DOMAIN|
| baw_configuration[x].<br>case.object_store_name_dos| Design Object Store name of CASE.| DOS|
| baw_configuration[x].<br>case.object_store_name_tos| Target Object Store name of CASE.| TOS|
| baw_configuration[x].<br>case.connection_point_name_tos| Connection point name for Target Object Store.| cpe_conn_tos|
| baw_configuration[x].<br>case.network_shared_directory_pvc| PVC name for CASE.| pluginstore|
| baw_configuration[x].<br>resources.limits.cpu| Memory limit for Workflow server.| 3|
| baw_configuration[x].<br>resources.limits.memory| CPU limit for Workflow server.| 2096Mi|
| baw_configuration[x].<br>resources.requests.cpu| Requested amount of memory for Workflow server.| 2|
| baw_configuration[x].<br>resources.requests.memory| Requested amount of CPU for Workflow server.| 1048Mi|
| baw_configuration[x].<br>probe.ws.liveness_probe.initial_delay_seconds| Number of seconds after the Workflow server container starts before the liveness probe is initiated| 240|
| baw_configuration[x].<br>probe.ws.readinessProbe.initial_delay_seconds| Number of seconds after the Workflow server container starts before the readiness probe is initiated| 180|
| baw_configuration[x].<br>logs.console_format| Format for printing logs on the console.| json|
| baw_configuration[x].<br>logs.console_log_level| Log level for printing logs on the console.| INFO|
| baw_configuration[x].<br>logs.console_source| Source of the logs for printing on the console.| message,trace,accessLog,ffdc,audit|
| baw_configuration[x].<br>logs.message_format| Format for printing message logs on the console.| basic|
| baw_configuration[x].<br>logs.trace_format| Format for printing trace logs on the console.| ENHANCED|
| baw_configuration[x].<br>logs.trace_specification| Specification for printing trace logs.| *=info|
| baw_configuration[x].<br>custom_xml_secret_name| Workflow server custom XML secret name.| |
| baw_configuration[x].<br>custom_xml_secret_name| Workflow server Lombardi custom XML secret name.| |


### Java Messaging Service configuration parameters
| Parameter                              | Description                                           | Default             |
| -------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------- |
| baw_configuration[x].<br>jms.image.repository| Image name for Java Messaging Service container.| baw-jms-server|
| baw_configuration[x].<br>jms.image.tag| Image tag for Java Messaging Service container.| 20.0.2|
| baw_configuration[x].<br>jms.image.pull_policy| Pull policy for Java Messaging Service container.| Always|
| baw_configuration[x].<br>jms.tls.tls_secret_name| TLS secret name for Java Message Service (JMS)| ibm-jms-tls-secret|
| baw_configuration[x].<br>jms.resources.limits.memory| Memory limit for JMS configuration.| 2Gi|
| baw_configuration[x].<br>jms.resources.limits.cpu| CPU limit for JMS configuration.| 1000m|
| baw_configuration[x].<br>jms.resources.requests.memory| Requested amount of memory for JMS configuration.| 512Mi|
| baw_configuration[x].<br>jms.resources.requests.cpu| Requested amount of CPU for JMS configuration.| 200m|
| baw_configuration[x].<br>jms.storage.persistent| Whether to enable persistent storage for JMS.| true|
| baw_configuration[x].<br>ms.storage.size| Size for JMS persistent storage.| 1Gi|
| baw_configuration[x].<br>jms.storage.use_dynamic_provisioning| Whether to enable dynamic provisioning for JMS persistent storage.| false|
| baw_configuration[x].<br>jms.storage.access_modes| Access modes for JMS persistent storage. Refer to Kubernetes documentation for available options.| ReadWriteOnce|
| baw_configuration[x].<br>jms.storage.storage_class| Storage class name for JMS persistent storage. This parameter is required.| jms-storage-class|



### Process Federation Server configuration parameters
| Parameter                              | Description                                           | Default             |
| -------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------- |
| pfs_configuration.<br>hostname| Process Federation Server hostname.| <pfs_hostname>|
| pfs_configuration.<br>port| Process Federation Server port.| 443|
| pfs_configuration.<br>service_type| How the HTTPS endpoint service should be published. Possible values are ClusterIP, NodePort, Route.| Route|
| pfs_configuration.<br>admin_secret_name| Name of the Kubernetes secret containing the Process Federation Server administration passwords, such as keystorePassword, ltpaPassword, oidcClientPassword, sslKeyPassword, truststorePassword.| ibm-pfs-admin-secret|
| pfs_configuration.<br>config_dropins_overrides_secret| Name of the Kubernetes secret containing the files that will be mounted in the /config/configDropins/overrides folder| |
| pfs_configuration.<br>image.repository| Process Federation Server image.| pfs-prod|
| pfs_configuration.<br>image.tag| Process Federation Server image tag.| 20.0.2|
| pfs_configuration.<br>image.pull_policy| Process Federation Server image pull policy| IfNotPresent|
| pfs_configuration.<br>liveness_probe.initial_delay_seconds| Number of seconds after Process Federation Server container starts before the liveness probe is initiated.| 60|
| pfs_configuration.<br>readiness_probe.initial_delay_seconds| Number of seconds after Process Federation Server container starts before the readiness probe is initiated.| 60|
| pfs_configuration.<br>replicas| Number of initial Process Federation Server pods.| 1|
| pfs_configuration.<br>service_account| Service account name for the Process Federation Server pod.| |
| pfs_configuration.<br>anti_affinity| Whether Kubernetes can (soft) or must not (hard) deploy Process Federation Server pods onto the same node. Possible values are soft and hard.| hard|
| pfs_configuration.<br>resources_security_secret| Name of the Kubernetes secret containing the files that will be mounted in the /config/resources/security folder| |
| pfs_configuration.<br>external_tls_secret| The secret that contains the Transport Layer Security (TLS) key and certificate for external https visits. You can enter the secret name here. If you do not want to use the customized external TLS certificate, leave it empty.| |
| pfs_configuration.<br>external_tls_ca_secret| Certificate authority (CA) used to sign the external TLS secret. It is stored in the secret with the TLS key and certificate. You can enter the secret name here. If you don't want to use the customized CA to sign the external TLS certificate, leave it empty.| |
| pfs_configuration.<br>tls.tls_secret_name| Existing TLS secret containing tls.key and tls.crt.| |
| pfs_configuration.<br>tls.tls_trust_list| Existing TLS trust secret.| |
| pfs_configuration.<br>resources.requests.cpu| Requested CPU for Process Federation Server Resource Registry configuration| 200m|
| pfs_configuration.<br>resources.requests.memory| Minimum memory required (including JVM heap and file system cache) to start an Elasticsearch pod| 500Mi|
| pfs_configuration.<br>resources.limits.cpu| CPU limit for Process Federation Server Resource Registry configuration| 500m|
| pfs_configuration.<br>resources.limits.memory| Maximum memory (including JVM heap and file system cache) to allocate to each Elasticsearch pod| 2Gi|
| pfs_configuration.<br>saved_searches.index_name| Name of the Elasticsearch index used to store saved searches| ibmpfssavedsearches|
| pfs_configuration.<br>saved_searches.index_number_of_shards| Number of shards of the Elasticsearch index used to store saved searches| 3|
| pfs_configuration.<br>saved_searches.index_number_of_replicas| Number of replicas (pods) of the Elasticsearch index used to store saved searches| 2|
| pfs_configuration.<br>saved_searches.index_batch_size| Batch size used when retrieving saved searches| 100|
| pfs_configuration.<br>saved_searches.update_lock_expiration| Amount of time before considering an update lock as expired. Valid values are numbers with a trailing 'm' or 's' for minutes or seconds| 5m|
| pfs_configuration.<br>saved_searches.unique_constraint_expiration| Amount of time before considering a unique constraint as expired. Valid values are numbers with a trailing 'm' or 's' for minutes or seconds| 5m|
| pfs_configuration.<br>security.sso.domain_name| ssoDomainNames property of the <webAppSecurity> tag| |
| pfs_configuration.<br>security.sso.cookie_name| ssoCookieName property of the <webAppSecurity> tag| |
| pfs_configuration.<br>security.sso.ltpa.filename| keysFileName property of the <ltpa> tag| ltpa.keys|
| pfs_configuration.<br>security.sso.ltpa.expiration| expiration property of the <ltpa> tag| 120m|
| pfs_configuration.<br>security.sso.ltpa.monitor_interval| monitorIntervalproperty of the <ltpa> tag| 60s|
| pfs_configuration.<br>security.ssl_protocol| sslProtocol property of the <ssl> tag that is used as default SSL configuration| SSL|
| pfs_configuration.<br>executor.max_threads| Value of the maxThreads property of the <executor> tag| 80|
| pfs_configuration.<br>executor.core_threads| Value of the coreThreads property of the <executor> tag| 40|
| pfs_configuration.<br>rest.user_group_check_interval| Value of the userGroupCheckInterval property of the <ibmPfs_restConfig> tag| 300s|
| pfs_configuration.<br>rest.system_status_check_interval| Value of the systemStatusCheckInterval property of the <ibmPfs_restConfig> tag| 60s|
| pfs_configuration.<br>rest.bd_fields_check_interval| Value of the bdFieldsCheckInterval property of the <ibmPfs_restConfig> tag| 300s|
| pfs_configuration.<br>custom_env_variables.names| Names of the custom environment variables defined in the secret referenced in customEnvVariables.secret| |
| pfs_configuration.<br>custom_env_variables.secret| Secret holding custom environment variables| |
| pfs_configuration.<br>logs.console_format| Format for printing logs on the console. The valid values are basic or json format.| json|
| pfs_configuration.<br>logs.console_log_level| Log level for printing logs on the console, The valid values are INFO, AUDIT, WARNING, ERROR, and OFF.| INFO|
| pfs_configuration.<br>logs.console_source| Source of the logs for printing on the console. This property applies only when the consoleFormat is json. Valid values are message, trace, accessLog, ffdc, and audit.| message,trace,accessLog,ffdc,audit|
| pfs_configuration.<br>logs.trace_format| Format for printing trace logs. The default format for the Liberty server is ENHANCED. The BASIC and ADVANCED formats are for WebSphere® Application Server.| ENHANCED|
| pfs_configuration.<br>logs.trace_specification| Specification for printing trace logs. An example value is *=info:com.ibm.bpm.federated.*=all| *=info|
| pfs_configuration.<br>logs.storage.use_dynamic_provisioning| Uses Dynamic Provisioning for Process Federation Server Logs Data Storage| false|
| pfs_configuration.<br>logs.storage.size| The minimum size of the persistent volume used mounted as Process Federation Server Liberty server /logs folder| 5Gi|
| pfs_configuration.<br>logs.storage.storage_class| Storage class of the persistent volume used mounted as Process Federation Server Liberty server /logs folder| |
| pfs_configuration.<br>dba_resource_registry.lease_ttl| Duration of the lease that creates the Process Federation Server entry in the DBA Service Registry, in seconds.| 120|
| pfs_configuration.<br>dba_resource_registry.pfs_check_interval| Interval at which to check that Process Federation Server is running, in seconds.| 10|
| pfs_configuration.<br>dba_resource_registry.pfs_connect_timeout| The number of seconds after which Process Federation Server will be considered as not running if no connection can be established.| 10|
| pfs_configuration.<br>dba_resource_registry.pfs_response_timeout| The number of seconds after which Process Federation Server will be considered as not running if it has not yet responded.| 30|
| pfs_configuration.<br>dba_resource_registry.pfs_registration_key| The key under which Process Federation Server should be registered in the DBA Service Registry when running| /dba/appresources/IBM_PFS/PFS_SYSTEM|


### Elasticsearch configuration parameters
| Parameter                              | Description                                           | Default             |
| -------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------- |
| elasticsearch_configuration.<br>es_image.repository| Elasticsearch image| pfs-elasticsearch-prod|
| elasticsearch_configuration.<br>es_image.tag| Elasticsearch image tag| 20.0.2|
| elasticsearch_configuration.<br>es_image.pull_policy| Elasticsearch image pull policy| IfNotPresent|
| elasticsearch_configuration.<br>es_init_image.repository|  	Process Federation Server init image| pfs-init-prod|
| elasticsearch_configuration.<br>es_init_image.tag| Process Federation Server init image tag| 20.0.2|
| elasticsearch_configuration.<br>es_init_image.pull_policy| Process Federation Server init image pull policy| IfNotPresent|
| elasticsearch_configuration.<br>es_nginx_image.repository| Elasticsearch ngnix image| pfs-nginx-prod|
| elasticsearch_configuration.<br>es_nginx_image.tag| Elasticsearch ngnix image tag| 20.0.2|
| elasticsearch_configuration.<br>es_nginx_image.pull_policy| Elasticsearch ngnix image pull policy| IfNotPresent|
| elasticsearch_configuration.<br>replicas| Number of initial Elasticsearch pods| 2|
| elasticsearch_configuration.<br>service_type| How the HTTPS endpoint service should be published.| ClusterIP|
| elasticsearch_configuration.<br>external_port| The port to which the Elasticsearch server HTTPS endpoint will be exposed externally. This parameter is relevant only if pfs_configuration.elasticsearch.service_type is set to NodePort| |
| elasticsearch_configuration.<br>admin_secret_name| Name of the Kubernetes secret containing the Elastic Search administration passwords, such as username, password| |
| elasticsearch_configuration.<br>anti_affinity| Whether Kubernetes may (soft) or must not (hard) deploy Elasticsearch pods onto the same node. Possible values are softand hard| hard|
| elasticsearch_configuration.<br>service_account| Name of a service account to use. If elasticsearch_configuration.privileged is set to true, then this service account must allow running privileged containers. If not provided, <CUSTOM_RESOURCE_NAME>-ibm-pfs-es-service-account is used.| Custom_Resource_Name-ibm-pfs-es-service-account|
| elasticsearch_configuration.<br>privileged| When set to true, a privileged container is created to run the appropriate sysctl commands so that the node running the pods matches the Disable swapping and increasing limit number of files descriptors part.| false|
| elasticsearch_configuration.<br>probe_initial_delay| Initial delay for liveness and readiness probes of Elasticsearch pods| 90|
| elasticsearch_configuration.<br>heap_size| JVM heap size to allocate to each Elasticsearch pod| 1024m|
| elasticsearch_configuration.<br>resources.limits.memory| Maximum memory (including JVM heap and file system cache) to allocate to each Elasticsearch pod| 2Gi|
| elasticsearch_configuration.<br>resources.limits.cpu| Maximum amount of CPU to allocate to each Elasticsearch pod| 1000m|
| elasticsearch_configuration.<br>resources.requests.memory| Minimum memory required (including JVM heap and file system cache) to start an Elasticsearch pod| 500Mi|
| elasticsearch_configuration.<br>resources.requests.cpu| Minimum amount of CPU required to start an Elasticsearch pod| 100m|
| elasticsearch_configuration.<br>storage.persistent| Whether to enable persistent Elasticsearch storage for Process Federation Server. Set to false for non-production or trial-only deployment.| false|
| elasticsearch_configuration.<br>storage.use_dynamic_provisioning| Set to true to use GlusterFS or other dynamic storage provisioner.| false|
| elasticsearch_configuration.<br>storage.size| The minimum Resource quanities.| 10Gi|
| elasticsearch_configuration.<br>storage.storage_class| See the Persistent Volumes.| |
| elasticsearch_configuration.<br>snapshot_storage.enabled| Set to true for production deployment.| false|
| elasticsearch_configuration.<br>snapshot_storage.use_dynamic_provisioning| Set to true to use GlusterFS or other dynamic storage provisioner.| false|
| elasticsearch_configuration.<br>snapshot_storage.size| The minimum Resource quantities| 30Gi|
| elasticsearch_configuration.<br>snapshot_storage.storage_class_name| See the Persistent Volumes.| ""|
| elasticsearch_configuration.<br>snapshot_storage.existing_claim_name| By default, a new persistent volume claim is be created. Specify an existing claim here if one is available.| ""|


## FileNet Content Manager parameters
Configuration parameters for the FileNet Content Manager component.

### FileNet Content Manager datasource parameters
Update the custom YAML file to provide the details that are relevant for your Content Manager. The default parameter list assumes one Content Platform Engine Global Configuration database, two object store databases. Some parameters are specific to database vendors.

| Parameter                              | Description                                           | Default             |
| -------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------- |
| datasource_configuration.<br>dc_gcd_datasource.dc_database_type| Specify the type for your GCD database| db2|
| datasource_configuration.<br>dc_gcd_datasource.dc_common_gcd_datasource_name| The JNDI name of the non-XA JDBC data source associated with the global configuration table space or database. The name must be unique.| FNGCDDS|
| datasource_configuration.<br>dc_gcd_datasource.dc_common_gcd_xa_datasource_name| The JNDI name of the XA JDBC data source associated with the global configuration table space or database. The name must be unique.| FNGCDDSXA|
| datasource_configuration.<br>dc_gcd_datasource.database_servername| The host name of the server where the database software is installed.| <hostname>|
| datasource_configuration.<br>dc_gcd_datasource.database_name| Provide the database name.| GCDDB|
| datasource_configuration.<br>dc_gcd_datasource.database_port| Provide the database port.| 50000|
| datasource_configuration.<br>dc_os_datasources[x].dc_database_type| Specify the type for your Object Store database| db2|
| datasource_configuration.<br>dc_os_datasources[x].dc_common_os_datasource_name| The JNDI name of the non-XA JDBC data source associated with the object store table space or database. The name must be unique.| FNOS1DS|
| datasource_configuration.<br>dc_os_datasources[x].dc_common_os_xa_datasource_name| The JNDI name of the XA JDBC data source associated with the object store table space or database. The name must be unique.| FNOS1DSXA|
| datasource_configuration.<br>dc_os_datasources[x].database_servername| The host name of the server where the database software is installed.| <hostname>|
| datasource_configuration.<br>dc_os_datasources[x].database_name| Provide the database name.| OS1DB|
| datasource_configuration.<br>dc_os_datasources[x].database_port| Provide the database port.| 50000|


### FileNet Content Manager common parameters
| Parameter                              | Description                                           | Default             |
| -------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------- |
| ecm_configuration.fncm_secret_name| Contains the information about the LDAP user and password for components.| ibm-fncm-secret|


### Monitoring parameters
| Parameter                              | Description                                           | Default             |
| -------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------- |
| monitoring_configuration.<br>mon_metrics_writer_option| Provide the monitoring metrics option.Specify 0 for Graphite and 4 for Prometheus.| 4|
| monitoring_configuration.<br>mon_enable_plugin_pch| Performance metrics for FileNet® Content Manager components.(Do not update)| false|
| monitoring_configuration.<br>mon_enable_plugin_mbean| Performance metrics for JMX. (Do not update.)| false|
| monitoring_configuration.<br>collectd_plugin_write_graphite_host| The hostname for the Graphite service.| localhost|
| monitoring_configuration.<br>collectd_plugin_write_graphite_port| The port for the Graphite service.| 2003|
| monitoring_configuration.<br>collectd_interval| The interval seconds in which to query the read plugins. If set, will use the specified interval for collectd and plugins.| 10|
| monitoring_configuration.<br>collectd_disable_host_monitoring| If set to true, disables the collectd cpu, interface, load, memory, and prometheus plugins.| false|
| monitoring_configuration.<br>collectd_plugin_write_prometheus_port| The port of the collectd embedded webserver should listen on that can be scraped by using Prometheus.| 9103|


### Logging parameters
| Parameter                              | Description                                           | Default             |
| -------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------- |
| logging_configuration.<br>mon_log_parse| Set this parameter to "true" to enable log parsing. This allows you to filter and query logs using parsed parameters.| false|
| logging_configuration.<br>mon_log_service_endpoint| This is the endpoint to the Elasticsearch server. For example, <elastic_search_server_host>:9200| <hostname>:9200|
| logging_configuration.<br>private_logging_enabled| Specify whether to use private logging. Setting to true (for Filebeat) writes the console log, message log, trace log, and ffdc log to the folder /logs/application. Setting to false (for Red Hat OpenShift) writes the console log, message log, trace log, and ffdc log to the folder stdout.| false|
| logging_configuration.<br>logging_type| default, enable logging at the DBA container as backend service. sidecar, enable logging as the DBA container's sidecar. node-logging, enable logging as DaemonSet to collect Kubernetes node logging.| default|
| logging_configuration.<br>mon_log_path| Colon (:) separated list of paths to logs that Filebeat should forward. By default, the principle logs for the IBM components, including Liberty and the containerized applications, are forwarded automatically when logging is enabled for a component. To have Filebeat forward other logs, use the MON_LOG_PATH parameter can be used to provide the list of file paths as seen by Filebeat from inside the container.| /path_to_extra_log|


### Content Platform Engine parameters
| Parameter                              | Description                                           | Default             |
| -------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------- |
| ecm_configuration.<br>cpe.replica_count| How many Content Platform Engine replicas to deploy.| 1|
| ecm_configuration.<br>cpe.image.repository| Image name for Content Platform Engine container.| cpe|
| ecm_configuration.<br>cpe.image.tag| Image tag for Content Platform Engine container.| 20.0.2|
| ecm_configuration.<br>cpe.image.pull_policy| Pull policy for Content Platform Engine container.| Always|
| ecm_configuration.<br>cpe.log.format| The format for workload logging.| json|
| ecm_configuration.<br>cpe.resources.requests.cpu| Specifies a CPU request for the container.| 500m|
| ecm_configuration.<br>cpe.resources.requests.memory| Specify a memory request for the container.| 512Mi|
| ecm_configuration.<br>cpe.resources.limits.cpu| Specify a CPU limit for the container.| 1|
| ecm_configuration.<br>cpe.resources.limits.memory| Specify a memory limit for the container.| 3072Mi|
| ecm_configuration.<br>cpe.auto_scaling.enabled| Specify whether to enable auto scaling.| false|
| ecm_configuration.<br>cpe.auto_scaling.max_replicas| The upper limit for the number of pods that can be set by the autoscaler. Required.| 3|
| ecm_configuration.<br>cpe.auto_scaling.min_replicas| The lower limit for the number of pods that can be set by the autoscaler. If it is not specified or negative, the server will apply a default value.| 1|
| ecm_configuration.<br>cpe.auto_scaling.target_cpu_utilization_percentage| The target average CPU utilization (represented as a percent of requested CPU) over all the pods. If it is not specified or negative, a default autoscaling policy is used.| 80|
| ecm_configuration.<br>cpe.hostname| Provide a hostname that the operator uses to create an OpenShift route definition to access the application, most often the host name of the infrastructure node in Open Shift.| <hostname>|
| ecm_configuration.<br>cpe.cpe_production_setting.time_zone| The time zone for the container deployment.| Etc/UTC|
| ecm_configuration.<br>cpe.cpe_production_setting.jvm_initial_heap_percentage| The initial use of available memory.| 18|
| ecm_configuration.<br>cpe.cpe_production_setting.jvm_max_heap_percentage| The maximum percentage of available memory to use.| 33|
| ecm_configuration.<br>cpe.cpe_production_setting.jvm_customize_options| Optionally specify JVM arguments using comma separation. For example:jvm_customize_options="-Dmy.test.jvm.arg1=123,-Dmy.test.jvm.arg2=abc,-XX:+SomeJVMSettings,XshowSettings:vm"| None|
| ecm_configuration.<br>cpe.cpe_production_setting.gcd_jndi_name| JNDI name for the Global Configuration Database.| FNGCDDS|
| ecm_configuration.<br>cpe.cpe_production_setting.gcd_jndixa_name| JNDI XA name for the Global Configuration Database| FNGCDDSXA|
| ecm_configuration.<br>cpe.cpe_production_setting.license_model| Choose the licensing model. Required. The expected values are ICF.PVUNonProd, ICF.PVUProd, ICF.UVU, ICF.CU, FNCM.PVUNonProd, FNCM.PVUProd, FNCM.UVU, or FNCM.CU.| FNCM.PVUNonProd|
| ecm_configuration.<br>cpe.cpe_production_setting.license| The value must be set to accept to deploy.| accept|
| ecm_configuration.<br>cpe.monitor_enabled| Specify whether to use the built-in monitoring capability.| false|
| ecm_configuration.<br>cpe.logging_enabled| Specify whether to use the built-in logging capability.| false|
| ecm_configuration.<br>cpe.datavolume.existing_pvc_for_cpe_cfgstore| The persistent volume claim for Content Platform Engine configuration.| cpe-cfgstore-pvc-ctnrs|
| ecm_configuration.<br>cpe.datavolume.existing_pvc_for_cpe_logstore| The persistent volume claim for Content Platform Engine logs.| cpe-logstore-pvc-ctnrs|
| ecm_configuration.<br>cpe.datavolume.existing_pvc_for_cpe_filestore| The persistent volume claim for the Content Platform Engine files.| cpe-filestore-pvc-ctnrs|
| ecm_configuration.<br>cpe.datavolume.existing_pvc_for_cpe_icmrulestore| The persistent volume claim for the IBM Case Manager rules.| cpe-icmrules-pvc-ctnrs|
| ecm_configuration.<br>cpe.datavolume.existing_pvc_for_cpe_textextstore| The persistent volume claim for text extraction| cpe-textext-pvc-ctnrs|
| ecm_configuration.<br>cpe.datavolume.existing_pvc_for_cpe_bootstrapstore| The persistent volume claim for upgrade and startup.| cpe-bootstrap-pvc-ctnrs|
| ecm_configuration.<br>cpe.datavolume.existing_pvc_for_cpe_fnlogstore| The persistent volume claim for FileNet logs.| cpe-fnlogstore-pvc-ctnrs|
| ecm_configuration.<br>cpe.probe.readiness.initial_delay_seconds| The behavior of readiness probes to know when the containers are ready to start accepting traffic.| 120|
| ecm_configuration.<br>cpe.probe.readiness.period_seconds| The behavior of readiness probes to know when the containers are ready to start accepting traffic.| 5|
| ecm_configuration.<br>cpe.probe.readiness.timeout_seconds| The behavior of readiness probes to know when the containers are ready to start accepting traffic.| 10|
| ecm_configuration.<br>cpe.probe.readiness.failure_threshold| The behavior of readiness probes to know when the containers are ready to start accepting traffic.| 6|
| ecm_configuration.<br>cpe.probe.liveness.initial_delay_seconds| The behavior of liveness probes to know when to restart a container.| 600|
| ecm_configuration.<br>cpe.probe.liveness.period_seconds| The behavior of liveness probes to know when to restart a container.| 5|
| ecm_configuration.<br>cpe.probe.liveness.timeout_seconds| The behavior of liveness probes to know when to restart a container.| 5|
| ecm_configuration.<br>cpe.probe.liveness.failure_threshold| The behavior of liveness probes to know when to restart a container.| 6|
| ecm_configuration.<br>cpe.image_pull_secrets| The secrets to be able to pull images.| admin.registrykey|


### Content Management Interoperability Services parameters
| Parameter                              | Description                                           | Default             |
| -------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------- |
| ecm_configuration.<br>cmis.replica_count| How many replicas to deploy.| 1|
| ecm_configuration.<br>cmis.image.repository| Image name for Content Management Interoperability Services container.| cmis|
| ecm_configuration.<br>cmis.image.tag| Image tag for Content Management Interoperability Services container.| 20.0.2|
| ecm_configuration.<br>cmis.image.pull_policy| Pull policy for Content Management Interoperability Services container.| Always|
| ecm_configuration.<br>cmis.log.format| The format for workload logging.| json|
| ecm_configuration.<br>cmis.resources.requests.cpu| Specifies a CPU request for the container.| 500m|
| ecm_configuration.<br>cmis.resources.requests.memory| Specify a memory request for the container.| 256Mi|
| ecm_configuration.<br>cmis.resources.limits.cpu| Specify a CPU limit for the container.| 1|
| ecm_configuration.<br>cmis.resources.limits.memory| Specify a memory limit for the container.| 1536Mi|
| ecm_configuration.<br>cmis.auto_scaling.enabled| Specify whether to enable auto scaling.| false|
| ecm_configuration.<br>cmis.auto_scaling.max_replicas| The upper limit for the number of pods that can be set by the autoscaler. Required.| 3|
| ecm_configuration.<br>cmis.auto_scaling.min_replicas| The lower limit for the number of pods that can be set by the autoscaler. If it is not specified or negative, the server will apply a default value.| 1|
| ecm_configuration.<br>cmis.auto_scaling.target_cpu_utilization_percentage| The target average CPU utilization (represented as a percent of requested CPU) over all the pods. If it is not specified or negative, a default autoscaling policy is used.| 80|
| ecm_configuration.<br>cmis.hostname| Provide a hostname that the operator uses to create an OpenShift route definition to access the application, most often the host name of the infrastructure node in Open Shift.| <hostname>|
| ecm_configuration.<br>cmis.cmis_production_setting.time_zone| The time zone for the container deployment.| Etc/UTC|
| ecm_configuration.<br>cmis.cmis_production_setting.jvm_initial_heap_percentage| The initial use of available memory.| 40|
| ecm_configuration.<br>cmis.cmis_production_setting.jvm_max_heap_percentage| The maximum percentage of available memory to use.| 66|
| ecm_configuration.<br>cmis.cmis_production_setting.jvm_customize_options| Optionally specify JVM arguments using comma separation. For example:jvm_customize_options="-Dmy.test.jvm.arg1=123,-Dmy.test.jvm.arg2=abc,-XX:+SomeJVMSettings,XshowSettings:vm"| None|
| ecm_configuration.<br>cmis.cmis_production_setting.checkout_copycontent| Determines whether the content-stream of the Private Working Copy should be copied from the Document that was checked out.| true|
| ecm_configuration.<br>cmis.cmis_production_setting.default_maxitems| The default value for the optional maxItems input argument on paging-related services.| 25|
| ecm_configuration.<br>cmis.cmis_production_setting.cvl_cache| Determines whether ChoiceLists will be cached once for all users.| true|
| ecm_configuration.<br>cmis.cmis_production_setting.secure_metadata_cache| Determines whether secure metadata cache shoule be enabled| true|
| ecm_configuration.<br>cmis.cmis_production_setting.filter_hidden_properties| Determines whether hidden P8 domain properties should appear in CMIS type definitions and folder or document instance data.| true|
| ecm_configuration.<br>cmis.cmis_production_setting.querytime_limit| Timeout in seconds for the queries that specify timeout.| 180|
| ecm_configuration.<br>cmis.cmis_production_setting.resumable_queries_forrest| If true, then a faster response time for REST next line. If false, the next link for REST will re-issue query.| true|
| ecm_configuration.<br>cmis.cmis_production_setting.escape_unsafe_string_characters| Specifies whether to escape characters that are not valid for XML unicode as specified by the XML 1.0 standard.| false|
| ecm_configuration.<br>cmis.cmis_production_setting.max_soap_size| Limits the maximum allowable Web Service SOAP message request size.| 180|
| ecm_configuration.<br>cmis.cmis_production_setting.print_pull_stacktrace| Prints the full stack trace in the response.| false|
| ecm_configuration.<br>cmis.cmis_production_setting.folder_first_search| Configures the sequence in which CMIS tries to identify objects (folder or document first).| false|
| ecm_configuration.<br>cmis.cmis_production_setting.ignore_root_documents| To ignore reading or writing contents in root folder, set ignoreRootDocuments to true.| false|
| ecm_configuration.<br>cmis.cmis_production_setting.supporting_type_mutability| Determines whether to support type mutability.| false|
| ecm_configuration.<br>cmis.cmis_production_setting.license| The value must be set to accept to deploy.| accept|
| ecm_configuration.<br>cmis.monitor_enabled| Specify whether to use the built-in monitoring capability.| false|
| ecm_configuration.<br>cmis.logging_enabled| Specify whether to use the built-in logging capability.| false|
| ecm_configuration.<br>cmis.datavolume.existing_pvc_for_cmis_cfgstore| The persistent volume claim for Content Platform Engine configuration.| cmis-cfgstore-pvc-ctnrs|
| ecm_configuration.<br>cmis.datavolume.existing_pvc_for_cmis_logstore| The persistent volume claim for Content Platform Engine logs.| cmis-logstore-pvc-ctnrs|
| ecm_configuration.<br>cmis.probe.readiness.initial_delay_seconds| The behavior of readiness probes to know when the containers are ready to start accepting traffic.| 120|
| ecm_configuration.<br>cmis.probe.readiness.period_seconds| The behavior of readiness probes to know when the containers are ready to start accepting traffic.| 5|
| ecm_configuration.<br>cmis.probe.readiness.timeout_seconds| The behavior of readiness probes to know when the containers are ready to start accepting traffic.| 10|
| ecm_configuration.<br>cmis.probe.readiness.failure_threshold| The behavior of readiness probes to know when the containers are ready to start accepting traffic.| 6|
| ecm_configuration.<br>cmis.probe.liveness.initial_delay_seconds| The behavior of liveness probes to know when to restart a container.| 600|
| ecm_configuration.<br>cmis.probe.liveness.period_seconds| The behavior of liveness probes to know when to restart a container.| 5|
| ecm_configuration.<br>cmis.probe.liveness.timeout_seconds| The behavior of liveness probes to know when to restart a container.| 5|
| ecm_configuration.<br>cmis.probe.liveness.failure_threshold| The behavior of liveness probes to know when to restart a container.| 6|
| ecm_configuration.<br>cmis.image_pull_secrets| The secrets to be able to pull images.| admin.registrykey|


### Initialization parameters

| Parameter                              | Description                                           | Default             |
| -------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------- |
| initialize_configuration.<br>ic_domain_creation.domain_name| Provide a name for the domain.| P8DOMAIN|
| initialize_configuration.<br>ic_domain_creation.encryption_key| The encryption strength.| 128|
| initialize_configuration.<br>ic_ldap_creation.ic_ldap_admin_user_name| Administrator user.| |
| initialize_configuration.<br>ic_ldap_creation.ic_ldap_admins_groups_name| Administrator group.| |
| initialize_configuration.<br>ic_ldap_creation.ic_ldap_name| Name of the LDAP directory.| ldap_name|
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_display_name| Display name for the object store to create.| |
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_symb_name| Symbolic name for the object store to create.| |
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_conn.name| Object store connection name.| <objectstore1_connection>|
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_conn.site_name| The name of the site.| InitialSite|
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_conn.dc_os_datasource_name| Add the name of the object store database.| |
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_conn.dc_os_xa_datasource_name| The XA datasource.| |
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_admin_user_groups| Admin user group.| |
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_basic_user_groups| An array of users with access to the object store.| |
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_addons| Specify whether to enable add-ons.| true|
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_addons_list| Add-ons to enable for Content Platform Engine| (Add-on values)|
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_asa_name| Provide a name for the device| demo_storage|
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_asa_file_systems_storage_device_name| | |
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_asa_root_dir_path| The root directory path for the object store storage area.| /opt/ibm/asa/os01_storagearea1|
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_enable_workflow| Specify whether to enable workflow for the object store.| false|
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_workflow_region_name| Specify a name for the workflow region| design_region_name|
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_workflow_region_number| Specify the number of the workflow region.| 1|
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_workflow_data_tbl_space| Specify a table space for the workflow data.| VWDATA_TS|
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_workflow_index_tbl_space| Optionally specify a table space for the workflow index.| |
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_workflow_blob_tbl_space| Optionally specify a table space for the workflow blob.| |
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_workflow_admin_group| Designate an LDAP group for the workflow admin group.| |
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_workflow_config_group| Designate an LDAP group for the workflow config group.| |
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_workflow_date_time_mask| Default format for date and time| mm/dd/y hh:tt am|
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_workflow_locale| Locale for the workflow| en|
| initialize_configuration.<br>ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_workflow_pe_conn_point_name| Provide a name for the connection point| |



## Business Automation Navigator parameters
Update the custom YAML file to provide the details that are relevant to your Business Automation Navigator component and your decisions for the deployment of the container.

If you plan to use logging and monitoring with your Business Automation Navigator, check the lists of parameters for those components to compile the required values:
- [Monitoring parameters](#Monitoring-parameters)
- [Logging parameters](#Logging-parameters)

### Business Automation Navigator Datasource parameters
Update the custom YAML file to provide the details that are relevant for your Content Navigator datasource environment. The default parameter list assumes one Content Navigator database. Some parameters are specific to database vendors.

| Parameter                              | Description                                           | Default             |
| -------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------- |
| datasource_configuration.<br>dc_icn_datasource.dc_database_type| Specify the type for your IBM Content Navigator database| db2|
| datasource_configuration.<br>dc_icn_datasource.dc_oracle_icn_jdbc_url| Oracle: Provide the URL for the IBM Content Navigator database.| jdbc:oracle:thin:@//<hostname>:1521/orcl|
| datasource_configuration.<br>dc_icn_datasource.dc_common_icn_datasource_name| The JNDI name of the non-XA JDBC data source associated with the IBM Content Navigator table space or database. The name must be unique.| ICMClientDS|
| datasource_configuration.<br>dc_icn_datasource.database_servername| The host name of the server where the database software is installed.| <hostname>|
| datasource_configuration.<br>dc_icn_datasource.database_port| Provide the database port.| 50000|
| datasource_configuration.<br>dc_icn_datasource.database_name| Provide the database name.| ICNDB|
| datasource_configuration.<br>dc_icn_datasource.dc_hadr_standby_servername| Enter the standby server name.| <hostname>|
| datasource_configuration.<br>dc_icn_datasource.dc_hadr_standby_port| Enter the standby server port.| 50000|
| datasource_configuration.<br>dc_icn_datasource.dc_hadr_validation_timeout| Specify the validation timeout entry.| |
| datasource_configuration.<br>dc_icn_datasource.dc_hadr_retry_interval_for_client_reroute| Specify the time in seconds between connection attempts made by the automatic client reroute if the primary connection to the server fails.| 3|
| datasource_configuration.<br>dc_icn_datasource.dc_hadr_max_retries_for_client_reroute| The maximum number of connection retries attempted by automatic client reroute if the primary connection to the server fails. This property is used only if the Retry interval for client reroute property is set.| 3|


### Business Automation Navigator configuration parameters

| Parameter                              | Description                                           | Default             |
| -------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------- |
| navigator_configuration.<br>ban_secret_name| Contains the information about the LDAP user and password for components.| ibm-ban-secret|
| navigator_configuration.<br>replica_count| How many IBM Business Automation Navigator replicas to deploy.| 1|
| navigator_configuration.<br>image.repository| Image name for navigator container.| navigator-sso|
| navigator_configuration.<br>image.tag| Image tag for navigator container.| 20.0.2|
| navigator_configuration.<br>image.pull_policy| Pull policy for navigator container.| Always|
| navigator_configuration.<br>image.arbitrary_uid_enabled| | true|
| navigator_configuration.<br>log.format| The format for workload logging.| json|
| navigator_configuration.<br>resources.requests.cpu| Specifies a CPU request for the container.| 500m|
| navigator_configuration.<br>resources.requests.memory| Specify a memory request for the container.| 512Mi|
| navigator_configuration.<br>resources.limits.cpu|Specify a CPU limit for the container. | 1|
| navigator_configuration.<br>resources.limits.memory| Specify a memory limit for the container.| 1536Mi|
| navigator_configuration.<br>auto_scaling.enabled| Specify whether to enable auto scaling.| false|
| navigator_configuration.<br>auto_scaling.max_replicas| The upper limit for the number of pods that can be set by the autoscaler. Required.| 3|
| navigator_configuration.<br>auto_scaling.min_replicas| The lower limit for the number of pods that can be set by the autoscaler. If it is not specified or negative, the server will apply a default value.| 1|
| navigator_configuration.<br>auto_scaling.target_cpu_utilization_percentage| The target average CPU utilization (represented as a percent of requested CPU) over all the pods. If it is not specified or negative, a default autoscaling policy is used.| 80|
| navigator_configuration.<br>hostname| Provide a hostname that the operator uses to create an OpenShift route definition to access the application, most often the host name of the infrastructure node in Open Shift.| <hostname>|
| navigator_configuration.<br>icn_production_setting.timezone| The time zone for the container deployment.| Etc/UTC|
| navigator_configuration.<br>icn_production_setting.jvm_initial_heap_percentage| The initial use of available memory.| 40|
| navigator_configuration.<br>icn_production_setting.jvm_max_heap_percentage| The maximum percentage of available memory to use.| 66|
| navigator_configuration.<br>icn_production_setting.jvm_customize_options| Optionally specify JVM arguments using comma separation. For example:jvm_customize_options="-Dmy.test.jvm.arg1=123,-Dmy.test.jvm.arg2=abc,-XX:+SomeJVMSettings,XshowSettings:vm"| None|
| navigator_configuration.<br>icn_production_setting.icn_db_type| Type of the IBM Business Automation Navigator database.| db2|
| navigator_configuration.<br>icn_production_setting.icn_jndids_name| Name for the Navigator JNDI datasource.| ECMClientDS|
| navigator_configuration.<br>icn_production_setting.icn_schema| Schema for IBM Business Automation Navigator. If you plan to use Task Manager with Business Automation Navigator, this value must be ICNDB.| <ICNDB>|
| navigator_configuration.<br>icn_production_setting.icn_table_space| Table space for IBM Business Automation Navigator.| <ICNDB>|
| navigator_configuration.<br>icn_production_setting.icn_admin| IBM Business Automation Navigator administrator user.| <LDAP_USER>|
| navigator_configuration.<br>icn_production_setting.license|The value must be set to accept to deploy.| accept|
| navigator_configuration.<br>icn_production_setting.enable_appcues| Internal use only. Do not change the value.| false|
| navigator_configuration.<br>icn_production_setting.allow_remote_plugins_via_http| It is recommended not to change this setting.| false|
| navigator_configuration.<br>monitor_enabled| Specify whether to use the built-in monitoring capability.| false|
| navigator_configuration.<br>logging_enabled| Specify whether to use the built-in logging capability.| false|
| navigator_configuration.<br>collectd_enable_plugin_write_graphite| If you use Graphite database for metrics or use IBM Cloud™ monitoring, set to true.| false|
| navigator_configuration.<br>datavolume.existing_pvc_for_icn_cfgstore| The persistent volume claim for IBM Business Automation Navigator configuration.| icn-cfgstore-ctnrs|
| navigator_configuration.<br>datavolume.existing_pvc_for_icn_logstore| The persistent volume claim for IBM Business Automation Navigator logs.| icn-logstore-ctnrs|
| navigator_configuration.<br>datavolume.existing_pvc_for_icn_pluginstore|The persistent volume claim for the plug-ins. | icn-pluginstore-ctnrs|
| navigator_configuration.<br>datavolume.existing_pvc_for_icnvw_cachestore| The persistent volume claim for the viewer cache.| icn-vw-cachestore-ctnrs|
| navigator_configuration.<br>datavolume.existing_pvc_for_icnvw_logstore| The persistent volume claim for the viewer log.| icn-vw-logstore-ctnrs|
| navigator_configuration.<br>datavolume.existing_pvc_for_icn_aspera| The persistent volume claim for Aspera.| icn-asperastore-ctnrs|
| navigator_configuration.<br>probe.readiness.initial_delay_seconds| The initial delay seconds of readiness probes| 120|
| navigator_configuration.<br>probe.readiness.period_seconds| The period seconds of readiness probes| 5|
| navigator_configuration.<br>probe.readiness.timeout_seconds| The timeout seconds of readiness probes| 10|
| navigator_configuration.<br>probe.readiness.failure_threshold| The failure threshold of readiness probes| 6|
| navigator_configuration.<br>probe.liveness.initial_delay_seconds| The initial delay seconds of liveness probes| 600|
| navigator_configuration.<br>probe.liveness.period_seconds| The initial delay seconds of liveness probes| 5|
| navigator_configuration.<br>probe.liveness.timeout_seconds| The timeout seconds of liveness probes| 5|
| navigator_configuration.<br>probe.liveness.failure_threshold| The failure threshold of liveness probes| 6|
| navigator_configuration.<br>image_pull_secrets.name| The secrets to be able to pull images.| admin.registrykey|