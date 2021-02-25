# Maximo Application Suite Quick Start -v 8.3
This quick start is customized from the orginal source and try to utilize alternative installation paths in openshift utilizing operator installations via web console as much as possible. The steps are for testing purposes only and may require further validation for any production deployments. It's expected that official IBM installation guides for Maximo Application Suite or Cloud Pak for Data would be cross-checked before performing any production deployments. 

The installation process requires 3 main stages - dependencies , MAS control plan , MAS admin portal for specific MAS applicaiton deployments. 

## Dependencies - perform installation of each dependency 
1. [Cert-Manager](cert-manager/README.md)
2. [Db2 Warehouse](db2w/README.md)
3. [MongoDB](mongodb/README.md)
4. [Apache Kafka](kafka/README.md)
5. [Behaviour Analytics](analytics/README.md)
6. *Optional:* [Monitoring](monitoring/README.md)

## MAS control plane installation
Execute the shell installer with relevant environment varaibles set. 

Locate and run the install.sh file.
The install.sh file is part of the Maximo Application Suite V8.3 for Multiplatform package that you downloaded from Passport Advantage.
The following environment variables should be set before running the installer script:
- export LICENSING_ID=<your existing licence HostID>
- export ENTITLEMENT_KEY=<your_key>

The MAS installer command:
`./install.sh -i instance_name --domain masdomain.com -c masdev-cluster-issuer`

Where:

- instance_name is the OpenShift instance name that you want to use.
- masdomain.com is the domain name for your environment.
- masdev-cluster-issuer is the namespace of the [Cert-Manager](cert-manager/README.md)

*Example:*

```
export LICENSING_ID=0242ac110002
export ENTITLEMENT_KEY=<your_key>
./install.sh -i masinst1 --domain apps.cluster-5fba.sandbox274.opentlc.com -c ca-issuer
```

After the successful completion of the Maximo Application Suite installation, the following information is displayed:

Administration dashboard URL
Example: https://admin.masdomain.com

Super user credentials
Username and password in the form of two randomly generated 32-character strings.
Important: Make a note of these credentials. If you lose them, they can be recovered only by an OpenShift administrator at:
OpenShift dashboard > Projects > mas-instance_name-core > Workloads > Secrets > mas-superuser > Data

Setup sign-in URL
Example: https://admin.masdomain.com/initialsetup

## Official documentation
1. [Maximo Application Suite documentation](https://www.ibm.com/support/knowledgecenter/SSQR84_current/iot/kc_welcome_mas.html)
2. [Download document](https://www.ibm.com/support/pages/node/5694195)
3. [System requirements](https://www.ibm.com/support/pages/ibm-maximo-application-suite-system-requirements)

**Note:** This information contains sample processes and code samples. These examples have not been thoroughly tested under all conditions and the reliability, serviceability, or function of these samples is not guaranteed or implied. *The samples are provided "AS IS", without warranty of any kind.*
