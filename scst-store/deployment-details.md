# Deployment details and configuration

## <a id='what-deploy'></a>What is deployed

The installation creates the following in your Kubernetes cluster:

* Two components — an API back end and a database. 
  Each component includes:
    * service
    * deployment
    * replicaset
    * pod
* Persistent volume and persistent volume claim.
* External IP address (based on a deployment configuration set to use `LoadBalancer`).
* A Kubernetes secret to allow pulling Supply Chain Security Tools - Store images from a registry.
* A namespace called `metadata-store`.
* A service account with read-write privileges named `metadata-store-read-write-client`. It's bound to a ClusterRole named `metadata-store-read-write`.
* A read-only ClusterRole named `metadata-store-read-only` that isn't bound to a service account. See [Service Accounts](#service-accounts).
* (Optional) An HTTPProxy object for ingress support.

## <a id='configuration'></a> Deployment configuration

### Database configuration

The default database included with the deployment is meant to get users started using the metadata store. The default database deployment does not support many enterprise production requirements, including scaling, redundancy, or failover. However, it is still a secure deployment.

#### <a id='awsrds-postresdata'></a>Using AWS RDS postgres database

Users can also configure the deployment to use their own RDS database instead of the default. See [AWS RDS Postgres Configuration](use-aws-rds.md).

#### <a id='cust-data-pass'></a>Custom database password

By default, a database password is generated automatically upon deployment. To configure a custom password, use the `db_password` property in the `metadata-store-values.yaml` during deployment.

```
db_password: "PASSWORD-0123"
```

If you're deploying with Tanzu Application Platform profiles, in `tap-values.yaml`, put:

```
metadata_store:
  db_password: "PASSWORD-0123"
```

Where `PASSWORD-0123` is the same password used between deployments.

>**Note:** there is a known issue related to changing database passwords [Persistent Volume Retains Data](../release-notes.md#store-persistent-volume-retains-data).

### <a id='appserv-type'></a>App service type

If your environment does not support `LoadBalancer`, and you want to use `NodePort`, configure the `app_service_type` property in your `metadata-store-values.yaml`:

```
app_service_type: "LoadBalancer"
```

### <a id='service-accounts'></a>Service accounts

By default, a service account with read-write privileges to the metadata store app is installed.
This service account is a cluster-wide account that uses ClusterRole.
If you don't want the service account and role, set the `add_default_rw_service_account` property to `"false"`.
To create a custom service account, see [Configure access tokens](create-service-account-access-token.md).

The store creates a read-only cluster role, which can be bound to a service account through `ClusterRoleBinding`. To create service accounts to bind to this cluster role, see [Configure access tokens](create-service-account-access-token.md). 

## <a id='export-cert'></a>Exporting certificates

Supply Chain Security Tools - Store creates [Secret Export](https://github.com/vmware-tanzu/carvel-secretgen-controller/blob/develop/docs/secret-export.md) for exporting certificates to `Supply Chain Security Tools - Scan` to securely post scan results. These certificates are exported to the namespace where `Supply Chain Security Tools - Scan` is installed. 

## <a id='ingress'></a>Ingress support

Supply Chain Security Tools - Store's values file allows you to enable ingress support and to configure a custom domain name to use Contour to provide external access to Supply Chain Security Tools - Store's API. For example:

```
ingress_enabled: "true"
ingress_domain: "example.com"
```

An HTTPProxy object is then installed with `metadata-store.example.com` as the fully qualified domain name. See [Ingress and multicluster support](ingress-multicluster.md).
