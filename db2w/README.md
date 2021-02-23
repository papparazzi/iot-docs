# DB2 Warehouse

## 1. How is DB2 Warehouse used in Maximo Application Suite

DB2 Warehouse forms the datasource for Monitor to store all the application metadata and analytics data provided by the Monitor KPI.

### 1.1 IoT Data Store Connector and DB2

The Data store connector (DSC) component in IoT is used to store device event and state data in DB2 warehouse.  Connecting IoT to a DB2 warehouse service allows you to store and access the device/state data.  When the Monitor application is deployed, the configured DB2 service is automatically connected to IoT as part of Monitor deployment procedure.

In IoT, the **destination** resource is used to represent tables in DB2. Creating/Deleting a destination in the platform will attempt to create/delete the table in the target DB2 service.

The configuration property on the **destination** resource will contain metadata that describes the structure of the DB2 table.  On a create of a destination resource, this metadata will be used to generate the `CREATE TABLE` SQL statement that will be executed against the target DB2 service. For example:

```
{
  "name": "TEMPSENSOR_LI",
  "type": "db2",
  "configuration": {
    "columns": [
      { "name": "temp", "type": "float", "nullable": true }, 
      { "name": "hum", "type": "float", "nullable": true }
    ]
  }
}
```

### 1.2 Data Structure in DB2

Monitor creates a dedicated schema for every workspace Id, schema is named as `<workspaceId>_MAM`. All the tables created for a workspace is created under its own schema.

#### Activated Logical Interface

The Monitor application is notified whenever a user configures device state model (schemas, event types, physical and logical interfaces) or activates a logical interface within IoT, on receiving this notification the Monitor application will create new tables and metadata configuration within DB2 Warehouse, and automatically configure forwarding rules in IoT to direct data flow into the table.

As a result, a table with name `IOT_<logical_interface_name>` is created in DB2. The properties of the activated logical interface is used as the metadata along with default information of the LI like device type ID, device ID and timestamps in UTC.

For example: In workspace `test`, for an activated logical interface `dt0_LI`'s schema as shown below for device type `dt0` and device `d0`, 

```
{
    "$schema": "http://json-schema.org/draft-04/schema#",
    "type": "object",
    "title": "schema-title-43702536",
    "description": "",
    "additionalProperties": false,
    "properties": {
        "temp": {
            "type": "number"
        }
    }
}
```

- Monitor creates a table with name `IOT_DT0_LI` in schema `TEST_MAM`.
- The Table's columns are based on the property of the LI schema, so in this case, `TEMP` with Integer type and other default properties like `DEVICEID`, `DEVICETYPE`, `RCV_TIMESTAMP_UTC` and `UPDATED_UTC` columns are created.
- Monitor also creates the forwarding rule to direct the events into this destination.
- So any state update received for this LI will be forwarded to the DB2 by the DSC component in the platform.
- When the device publishes temp reading 15 and 16, we can see two rows with those values in the below image.

![DB2 view for the logical interface dt0_LI](../media/db2.png)


#### Device Metadata
Similar to the above scenario, when a device's metadata/info is created or updated, Monitor is notified and a table `IOT_<logical_interface_name>_CTG` is created in the workspace's schema(`<workspaceId>_MAM`). Each metadata property will be a seperate column in the table. 


## 2. Supported Versions

The following DB2 Warehouse version releases are supported:

- `11.5.X`


## 3. Resource Requirements

Use the [online tool](https://www.ibm.com/links?url=http%3A%2F%2Fdashdb-configurator.stage1.mybluemix.net%2F) to calculate the cluster resource requirements


# 4. OCP terminal Installation

??? abstract "Prerequisite Actions"
    - [Install CloudPak for Data](../cp4d/README.md)

- `oc login` to your OpenShift cluster
- Go to the directory you placed the repo.yaml
- Run the cpd adm command: `./cpd-linux adm -r repo.yaml -a db2wh -n zen --apply --verbose --accept-all-licenses`
- Run the cpd install command: 

```
./cpd-linux -s repo.yaml \
  -a db2wh \
  -n zen \
  -c <storage_class> \
  --verbose \
  --target-registry-username openshift \
  --target-registry-password $(oc whoami -t) \
  --insecure-skip-tls-verify \
  --cluster-pull-prefix image-registry.openshift-image-registry.svc:5000/zen \
  --transfer-image-to image-registry.openshift-image-registry.svc/zen
```


## 5. Supported Storage

System and backup storage, and user storage both require a storage class that supports the ReadWriteOnce access policy.

### OpenShift Container Storage 4.3
- System and backup storage: `ocs-storagecluster-cephfs` (select 4K sector size)
- User storage: `ocs-storagecluster-ceph-rbd` (select 4K sector size and ReadWriteOnce)

### IBM Cloud
IBM Cloud File Storage (`ibmc-file-gold-gid` storage class) & Portworx:

- System and backup storage: `portworx-db2-rwx-sc` (select 4K sector size)
- User storage: `portworx-db2-rwo-sc` (select 4K sector size and ReadWriteOnce)

----------
# OCP (web) Console Installation
## Pre-requisits to install CP4D Operator
- Suggested: operational OpenShift Storage Cluster with standard OSC storage classes. (to be used for all MAS installation tasks)

### Install IBM Cloud Pak for Data Operator
The CP4D operator needs to be registered with OCP in order to appear under Operator Hub. 

Navigate to Adminstration - Cluster Setting - Global Configuration - Operator Hub and press create catalog source with the following details as cluster wide ressource

Catalog source name
'''ibm-cp-data-operator-catalog'''
Display name
'''Cloud Pak for Data'''
Publisher name
'''IBM'''
Image (URL of container image)
'''docker.io/ibmcom/ibm-cp-data-operator-catalog:latest'''



### IBM entitlement key
1) Create Project --> myCloudPakOperator (example: CP4D)
2) create secrete within the project for IBM entitlement key

Type = Pull Secret

Secret Name = `ibm-entitlement-key`

Registry Server Address = `cp.icr.io`

Username = `cp`

Password = "your downloaded IBM entitlement key"


### Prepare dedicated node for DB2WH
DB2WH requires a dedicated Node for installation and special considerations are required in combination with Openshift Cluster Storage. 

As a solution a node can be tainted , drained and labled for DB2WH to be picked up in later installation steps. 

Execute the following steps as kubeadmin with the OC CLI:

- tainting nodes for DB2 (oc get nodes - select the node of choice- this example uses the node = `ip-10-0-202-136.ap-southeast-1.compute.internal`)

```oc adm taint node ip-10-0-202-136.ap-southeast-1.compute.internal icp4data=database-db2wh:NoSchedule --overwrite```

- drain the node

```oc adm drain ip-10-0-202-136.ap-southeast-1.compute.internal```

```oc adm uncordon ip-10-0-202-136.ap-southeast-1.compute.internal```

- label the node

```oc label node ip-10-0-202-136.ap-southeast-1.compute.internal icp4data=database-db2wh --overwrite```





## CP4D operator installation
- Operators - Operator Hub --> search "Cloud Pak for Data" and install latest version
- Install operator in the namespace created under earlier steps (example: CP4D)
- Wait for Operator to be successfully installed

### DB2WH installation
- Operators - Installed Operators --> select namespace of earlier project (example: CP4D) and navigate to the installed operator

- Create new `cloud pak for data service`

-- accept licence

-- Service name (assembly) = `db2wh`

-- provide storage class = suggested: `ocs-storagecluster-cephfs`

### Optional: Install Data Management Console
- Operators - Installed Operators --> select namespace of earlier project (example: CP4D) and navigate to the installed operator

- Create new `cloud pak for data service` 

-- accept licence

-- Service name (assembly) = `dmc`

-- provide storage class = suggested: `ocs-storagecluster-cephfs`

## Post-installation steps


### DB2WH - prepare for Monitor optimzed data
Follow the instructions from IBM knowledgecenter to change the DB2WH to row-organized tables by editing a configuration Json file on the DB2WH pod. 

1) Workloads - Pods --> Search for a pod `zen-database-core`
2) Navigate to pod details - Terminal
3) Execute the following command witin the interactive bash terminal of the pod

``` 
jf=$(ls -1 /user-home/_global_/databases/ibm-db2wh-*-x86_64.json)
python <<EOF
import json
jf = '$jf'

with open(jf, 'r') as fd:
    parsed = json.load(fd)

parsed["create"]["table-org"]="ROW"

with open(jf, 'w') as fd:
    json.dump(parsed, fd, indent=2)
EOF
```

