# Prep: OCP cluster trusted certs
In cases whereby your cluster is using self sigend certificats the following steps may be taken in order to deploy let's encrypt certificates as trusted certs cluster wide. 

https://www.openshift.com/blog/requesting-and-installing-lets-encrypt-certificates-for-openshift-4


# Installation of Cert-Manager 

Detailed alternative installation steps are documented online https://github.com/jetstack/cert-manager

## Installation of cert-manager operator
As admin perform the following steps on the command line logged into the cluster
```
oc apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.1.0/cert-manager.yaml
```
## Generate a CA private key
The following command will generate a new certificates to be used by the cert-manager 
```
openssl genrsa -out ca.key 2048
```

## Create a self signed Certificate, valid for 10yrs with the ‘signing’ option set
The following command will generate self signed certs - please update the CN (common name) to reflect your cluster domain.
Example of common name: `export COMMON_NAME=apps.cluster-5fba.sandbox274.opentlc.com`

```
openssl req -x509 -new -nodes -key ca.key -subj “/CN=${COMMON_NAME}” -days 3650 -reqexts v3_req -extensions v3_ca -out ca.crt
```

### create a secrete for the cert-manager operator / namespace
The following command will create a new secrete within the project namespace of the installed cert-manager with the earlier created certificate public / private keys. 
```
kubectl create secret tls ca-key-pair \
  --cert=ca.crt \
  --key=ca.key \
  --namespace=cert-manager
```

Note: The self signed certs will need to be transfered to the PC and stored under trusted CAs in order to complete the MAS installation in the post-installation process. Alternatively to generating new self signed certs the existing (self-signed) CA certs of the OCP cluster may be used. Ingress-Operator would hold the certificates under secrets. 


### create a custom ressource yaml spec to configure the cert-manager operator
Use your favorit console editor (vi cert.yaml) to create a new custom ressource for the cert-manager to configure the usage of the earlier created secret for certificate issuance. 

The yaml should include the following:

```YAML
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
 name: ca-issuer
spec:
 ca:
  secretName: ca-key-pair
```

Apply the newely created yaml to the cluster by
```oc apply -f cert.yaml```

