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

## MAS control plan installation
Execute the shell installer with relevant environment varaibles set. If the licence file has been created with a host_id already the following env variable *export LICENSING_ID=xxxxxxxxxx * could be used by inserting the hostID. 

The example includes -c ca-issur which is the default namespace of the earlier installed [Cert-Manager](cert-manager/README.md)

```
export LICENSING_ID=0242ac110002
./install.sh -i masinst1 --domain mas.com -c ca-issuer
```


## Official documentation
1. [Maximo Application Suite documentation](https://www.ibm.com/support/knowledgecenter/SSQR84_current/iot/kc_welcome_mas.html)
2. [Download document](https://www.ibm.com/support/pages/node/5694195)
3. [System requirements](https://www.ibm.com/support/pages/ibm-maximo-application-suite-system-requirements)

**Note:** This information contains sample processes and code samples. These examples have not been thoroughly tested under all conditions and the reliability, serviceability, or function of these samples is not guaranteed or implied. *The samples are provided "AS IS", without warranty of any kind.*
