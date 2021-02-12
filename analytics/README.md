# Behavior Analytics Service

Starting with MAS 8.3 the IBM Behavior Analytics Service is required during the MAS installation process. 

## Installation via OCP (web) console

- create new project for the operator - example: analytics
- Operators - Operator Hub --> search for `Behavior Analytics Services` and install the operator

login to the oc terminal as admin (oc login) and execute the following 2 commands to create secrets within the earlier created project namespace. This step can alternatively also be done by navigating to Workload - Secrets within the web console and create the secretes via the dialog. 

```oc create secret generic database-credentials --from-literal=db_username=admin --from-literal=db_password=password -n analytics```

```oc create secret generic grafana-credentials --from-literal=grafana_username=admin --from-literal=grafana_password=password -n analytics```

Note: `-n` declares the project namespace which in this example was defined as `analytics` - this can be changed as required.