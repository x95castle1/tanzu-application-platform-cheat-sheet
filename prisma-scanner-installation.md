# Installation Guide for Prisma Scanner
## Relocate images to a registry

VMware recommends relocating the images from VMware Tanzu Network registry to your own container image registry before attempting installation. If you don’t relocate the images, Prisma Scanner Installation depends on VMware Tanzu Network for continued operation, and VMware Tanzu Network offers no uptime guarantees. The option to skip relocation is documented for evaluation and proof-of-concept only.

The supported registries are Harbor, Azure Container Registry, Google Container Registry, and Quay.io. See the following documentation for a registry to learn how to set it up:

* [Harbor documentation](https://goharbor.io/docs/2.5.0/)
* [Google Container Registry documentation](https://cloud.google.com/container-registry/docs)
* [Quay.io documentation](https://docs.projectquay.io/welcome.html)

To relocate images from the VMware Tanzu Network registry to your registry:

1. Install Docker if it is not already installed.

2. Log in to your image registry by running:

```
docker login MY-REGISTRY
```

Where MY-REGISTRY is your own container registry.

3. Log in to the VMware Tanzu Network registry with your VMware Tanzu Network credentials by running:

```
docker login registry.tanzu.vmware.com
```

4. Set up environment variables for installation use by running:

```
export INSTALL_REGISTRY_USERNAME=MY-REGISTRY-USER
export INSTALL_REGISTRY_PASSWORD=MY-REGISTRY-PASSWORD
export INSTALL_REGISTRY_HOSTNAME=MY-REGISTRY
export VERSION=VERSION-NUMBER
export INSTALL_REPO=TARGET-REPOSITORY
```

Where:

 * MY-REGISTRY-USER is the user with write access to MY-REGISTRY.
 * MY-REGISTRY-PASSWORD is the password for MY-REGISTRY-USER.
 * MY-REGISTRY is your own container registry.
 * VERSION is your Prisma Scanner version. For example, 0.1.0-beta.4
 * TARGET-REPOSITORY is your target repository, a folder/repository on MY-REGISTRY that serves as the location for the installation files for Prisma Scanner.

5. Install the Carvel tool imgpkg CLI.

6. Relocate the images with the imgpkg CLI by running:

```
imgpkg copy -b projects.registry.vmware.com/tap-scanners-package/prisma-repo-scanning-bundle:${VERSION} --to-repo ${INSTALL_REGISTRY_HOSTNAME}/${INSTALL_REPO}/prisma-repo-scanning-bundle
```

## Add the Prisma Scanner package repository

Tanzu CLI packages are available on repositories. Adding the Prisma Scanning package repository makes Prisma Scanning bundle and its packages available for installation.

> **Note**
> Relocate images to a registry is strongly recommended but not required for installation. For simplicity, this section assumes you have relocated images to a registry. Refer to that section to fill in the stubbed out variables.


It's recommended to install the Prisma Scanner objects in the existing tap-install namespace to keep the Prisma Scanner grouped logically with the other TAP components.

1. Add the Prisma Scanner package repository to the cluster by running:

```
tanzu package repository add prisma-scanner-repository \
  --url ${INSTALL_REGISTRY_HOSTNAME}/${INSTALL_REPO}/prisma-repo-scanning-bundle:$VERSION \
  --namespace tap-install
```

2. Get the status of the Prisma Scanner package repository, and ensure the status updates to Reconcile succeeded by running:

```
tanzu package repository get prisma-scanning-repository --namespace tap-install
```

For example:

```
$ tanzu package repository get prisma-scanning-repository --namespace tap-install
- Retrieving repository prisma-scanner-repository...
NAME:          prisma-scanner-repository
VERSION:       71091125
REPOSITORY:    index.docker.io/tapsme/prisma-repo-scanning-bundle
TAG:           0.1.0-beta.4
STATUS:        Reconcile succeeded
REASON:
```

3. List the available packages by running:

```
tanzu package available list --namespace tap-install
```

For example:

```
$ tanzu package available list --namespace tap-install
/ Retrieving available packages...
  NAME                                                 DISPLAY-NAME                                                              SHORT-DESCRIPTION
  prisma.scanning.apps.tanzu.vmware.com                Prisma for Supply Chain Security Tools - Scan                             Default scan templates using Prisma                                              
```

## Prequisites for Prisma Scanner (Beta)

This document describes prerequisites for installing Supply Chain Security Tools - Scan (Prisma) from the VMware package repository.

### Prepare the Prisma Scanner configuration

To prepare the Prisma configuration before you install any scanners:

1. Obtain a Prisma Token from Prisma. 

2. Create a Prisma secret YAML file and insert the base64 encoded Prisma API token into the prisma_token:

```
apiVersion: v1
kind: Secret
metadata:
  name: prisma-token-secret
  namespace: my-apps
data:
  prisma_token: BASE64-PRISMA-API-TOKEN
```

3. Apply the Prisma secret YAML file by running:

```
kubectl apply -f YAML-FILE
```

Where YAML-FILE is the name of the Prisma secret YAML file you created.

4. Define the --values-file flag to customize the default configuration. You must define the following fields in the values.yaml file for the Prisma Scanner configuration. You can add fields as needed to activate or deactivate behaviors. You can append the values to this file as shown later in this document. Create a values.yaml file by using the following configuration:

```
---
namespace: DEV-NAMESPACE
targetImagePullSecret: TARGET-REGISTRY-CREDENTIALS-SECRET
prisma:
  url: PRISMA-URL
  tokenSecret:
    name: PRISMA-CONFIG-SECRET
```
Where: 

* DEV-NAMESPACE is your developer namespace.

> **Note:**
> To use a namespace other than the default namespace, ensure that the namespace exists before you install. If the namespace does not exist, the scanner installation fails.

* TARGET-REGISTRY-CREDENTIALS-SECRET is the name of the secret that contains the credentials to pull an image from a private registry for scanning.

* PRISMA-URL is the FQDN of your Twistlock server.

* PRISMA-CONFIG-SECRET is the name of the secret you created that contains the Prisma configuration to connect to Prisma. This field is required.

The Prisma integration can work with or without the SCST - Store integration. The values.yaml file is slightly different for each configuration.

### Supply Chain Security Tools - Store integration

Using Supply Chain Security Tools - Store Integration: To persist the results found by the Prisma Scanner, you can enable the Supply Chain Security Tools - Store integration by appending the fields to the values.yaml file.

The Grype, Snyk, and Prisma Scanner Integrations enable the Metadata Store. To prevent conflicts, the configuration values are slightly different based on whether the Grype Scanner Integration is installed or not. If Tanzu Application Platform is installed using the Full Profile, the Grype Scanner Integration is installed, unless it is explicitly excluded.

If the Grype or Snyk Scanner Integration is installed in the same dev-namespace Prisma Scanner is installed:

```
#! ...
metadataStore:
  #! The url where the Store deployment is accesible.
  #! Default value is: "https://metadata-store-app.metadata-store.svc.cluster.local:8443"
  url: "<STORE-URL>"
  caSecret:
    #! The name of the secret that contains the ca.crt to connect to the Store Deployment.
    #! Default value is: "app-tls-cert"
    name: "<CA-SECRET-NAME>"
    importFromNamespace: "" #! since both Prisma and Grype/Snyk both enable store, one must leave importFromNamespace blank
  #! authSecret is for multicluster configurations.
  authSecret:
    #! The name of the secret that contains the auth token to authenticate to the Store Deployment.
    name: "<AUTH-SECRET-NAME>"
    importFromNamespace: "" #! since both Prisma and Grype/Snyk both enable store, one must leave importFromNamespace blank
```

If the Grype/Snyk Scanner Integration is not installed in the same dev-namespace Prisma Scanner is installed:

```
#! ...
metadataStore:
  #! The url where the Store deployment is accesible.
  #! Default value is: "https://metadata-store-app.metadata-store.svc.cluster.local:8443"
  url: "<STORE-URL>"
  caSecret:
    #! The name of the secret that contains the ca.crt to connect to the Store Deployment.
    #! Default value is: "app-tls-cert"
    name: "<CA-SECRET-NAME>"
    #! The namespace where the secrets for the Store Deployment live.
    #! Default value is: "metadata-store"
    importFromNamespace: "<STORE-SECRETS-NAMESPACE>"
  #! authSecret is for multicluster configurations.
  authSecret:
    #! The name of the secret that contains the auth token to authenticate to the Store Deployment.
    name: "<AUTH-SECRET-NAME>"
    #! The namespace where the secrets for the Store Deployment live.
    importFromNamespace: "<STORE-SECRETS-NAMESPACE>"
```

**Without Supply Chain Security Tools - Store Integration:** If you don’t want to enable the Supply Chain Security Tools - Store integration, explicitly deactivate the integration by appending the following fields to the values.yaml file that is enabled by default:

```
# ...
metadataStore:
  url: "" # Configuration is moved, so set this string to empty
```
### Sample ScanPolicy for CycloneDX Format

apiVersion: scanning.apps.tanzu.vmware.com/v1beta1
kind: ScanPolicy
metadata:
  name: scan-policy
  labels:
    'app.kubernetes.io/part-of': 'enable-in-gui'
spec:
  regoFile: |
    package main

    # Accepted Values: "Critical", "High", "Medium", "Low", "Negligible", "UnknownSeverity"
    notAllowedSeverities := ["Critical", "High", "UnknownSeverity"]
    ignoreCves := []

    contains(array, elem) = true {
      array[_] = elem
    } else = false { true }

    isSafe(match) {
      severities := { e | e := match.ratings.rating.severity } | { e | e := match.ratings.rating[_].severity }
      some i
      fails := contains(notAllowedSeverities, severities[i])
      not fails
    }

    isSafe(match) {
      ignore := contains(ignoreCves, match.id)
      ignore
    }

    deny[msg] {
      comps := { e | e := input.bom.components.component } | { e | e := input.bom.components.component[_] }
      some i
      comp := comps[i]
      vulns := { e | e := comp.vulnerabilities.vulnerability } | { e | e := comp.vulnerabilities.vulnerability[_] }
      some j
      vuln := vulns[j]
      ratings := { e | e := vuln.ratings.rating.severity } | { e | e := vuln.ratings.rating[_].severity }
      not isSafe(vuln)
      msg = sprintf("CVE %s %s %s", [comp.name, vuln.id, ratings])
    }

Apply the YAML:

```
kubectl apply -n $DEV_NAMESPACE -f SCAN-POLICY-YAML
```

After all prerequisites have been completed, follow the steps in Install another scanner for Supply Chain Security Tools - Scan to install the Prisma Scanner.

## Install Another Scanner for Supply Security Tools - Scan (Prisma)

This topic describes how to install scanners to work with Supply Chain Security Tools - Scan from the VMware  package repository.

Follow the below instructions to install a scanner other than the out of the box Grype Scanner.

### Prerequisites

Before installing a new scanner, install Supply Chain Security Tools - Scan. It must be present on the same cluster. The prerequisites for Scan are also required.

### Install
To install a new scanner, follow these steps: 

1. List the available packages to discover what scanners you can use by running:

```
tanzu package available list --namespace tap-install
```

For example:

```
$ tanzu package available list --namespace tap-install
/ Retrieving available packages...
  NAME                                                 DISPLAY-NAME                                                              SHORT-DESCRIPTION
  grype.scanning.apps.tanzu.vmware.com                 Grype Scanner for Supply Chain Security Tools - Scan                      Default scan templates using Anchore Grype
  snyk.scanning.apps.tanzu.vmware.com                  Snyk for Supply Chain Security Tools - Scan                               Default scan templates using Snyk
  carbonblack.scanning.apps.tanzu.vmware.com           Carbon Black Scanner for Supply Chain Security Tools - Scan               Default scan templates using Carbon Black
  prisma.scanning.apps.tanzu.vmware.com                Prisma for Supply Chain Security Tools - Scan                             Default scan templates using Prisma
```

2. List version information for the scanner package by running:

```
tanzu package available list SCANNER-NAME --namespace tap-install
```

For example:

```
$ tanzu package available list prisma.scanning.apps.tanzu.vmware.com --namespace tap-install
/ Retrieving package versions for prisma.scanning.apps.tanzu.vmware.com...
  NAME                                  VERSION           RELEASED-AT
  prisma.scanning.apps.tanzu.vmware.com   0.1.0-beta.4
```

3. (Optional) Create the secrets the scanner package relies on:

4. Create a values.yaml to apply custom configurations to the scanner:

> **Note:**
> This step might be required for some scanners but optional for others.

To list the values you can configure for any scanner, run:

```
tanzu package available get SCANNER-NAME/VERSION --values-schema -n tap-install
```

Where:

* SCANNER-NAME is the name of the scanner package you retrieved earlier.
* VERSION is your package version number. For example, prisma.scanning.apps.tanzu.vmware.com/0.1.0-beta.4.

For example:

```
$ tanzu package available get prisma.scanning.apps.tanzu.vmware.com/0.1.0-beta.4 --values-schema -n tap-install

  KEY                                           DEFAULT                                                           TYPE    DESCRIPTION
  prisma.tokenSecret.name                       prisma-token                                                      string  Reference to the secret named prisma-token containing a Prisma API Token set as
                                                                                                                          prisma_token
  prisma.url                                                                                                      string  Twistlock server url (https://twistlock.example.com:8083)
  prisma.caCertConfigMap.name                                                                                     string  Reference to the configmap containing the registry ca cert set as ca_cert_data
  prisma.projectName                                                                                              string  Twistlock target project
  resources.limits.cpu                          1000m                                                             string  Limits describes the maximum amount of cpu resources allowed.
  resources.requests.cpu                        250m                                                              string  Requests describes the minimum amount of cpu resources required.
  resources.requests.memory                     128Mi                                                             string  Requests describes the minimum amount of memory resources
  scanner.docker.password                                                                                         string  <nil>
  scanner.docker.server                                                                                           string  <nil>
  scanner.docker.username                                                                                         string  <nil>
  scanner.pullSecret                                                                                              string  <nil>
  scanner.serviceAccount                        prisma-scanner                                                    string  Name of scan pod's service ServiceAccount
  scanner.serviceAccountAnnotations                                                                               string  Annotations added to ServiceAccount
  targetImagePullSecret                                                                                           string  Reference to the secret used for pulling images from private
  metadataStore.authSecret.importFromNamespace                                                                    string  Namespace from which to import the Insight Metadata Store auth_token
  metadataStore.authSecret.name                                                                                   string  Name of deployed Secret with key auth_token
  metadataStore.caSecret.importFromNamespace    metadata-store                                                    string  Namespace from which to import the Insight Metadata Store CA Cert
  metadataStore.caSecret.name                   app-tls-cert                                                      string  Name of deployed Secret with key ca.crt holding the CA Cert of the Insight
                                                                                                                          Metadata Store
  metadataStore.clusterRole                     metadata-store-read-write                                         string  Name of the deployed ClusterRole for read/write access to the Insight Metadata
                                                                                                                          Store deployed in the same cluster
  metadataStore.url                             https://metadata-store-app.metadata-store.svc.cluster.local:8443  string  Url of the Insight Metadata Store
  namespace                                     default                                                           string  Deployment namespace for the Scan Templates
```

5. Define the --values-file flag to customize the default configuration:

The values.yaml file you created earlier is referenced with the --values-file flag when running your Tanzu install command:

```
tanzu package install REFERENCE-NAME \
  --package-name SCANNER-NAME \
  --version VERSION \
  --namespace tap-install \
  --values-file PATH-TO-VALUES-YAML
```

Where:

* REFERENCE-NAME is the name referenced by the installed package. For example, grype-scanner, snyk-scanner, prisma-scanner.
* SCANNER-NAME is the name of the scanner package you retrieved earlier. For example, prisma.scanning.apps.tanzu.vmware.com.
* VERSION is your package version number. For example, 0.1.0-beta.4.
* PATH-TO-VALUES-YAML is the path that points to the values.yaml file created earlier.

For example:

```
$ tanzu package install prisma-scanner \
  --package-name primsa.scanning.apps.tanzu.vmware.com \
  --version 0.1.0-beta.4 \
  --namespace tap-install \
  --values-file values.yaml
/ Installing package 'prisma.scanning.apps.tanzu.vmware.com'
| Getting namespace 'tap-install'
| Getting package metadata for 'prisma.scanning.apps.tanzu.vmware.com'
| Creating service account 'prisma-scanner-tap-install-sa'
| Creating cluster admin role 'prisma-scanner-tap-install-cluster-role'
| Creating cluster role binding 'prisma-scanner-tap-install-cluster-rolebinding'
/ Creating package resource
- Package install status: Reconciling

 Added installed package 'prisma-scanner' in namespace 'tap-install'
```

### Verify Installation

To verify the installation create an ImageScan or SourceScan referencing one of the newly added ScanTemplates for the scanner.

1. (Optional) Create a ScanPolicy formatted for the output specific to the scanner you are installing, to reference in the ImageScan or SourceScan.

```
kubectl apply -n $DEV_NAMESPACE -f SCAN-POLICY-YAML
```

> **Note:**
> As vulnerability scanners output different formats, the ScanPolicies can vary. For more information about policies and samples, see Enforce compliance policy using Open Policy Agent.

2. Retrieve available ScanTemplates from the namespace where the scanner is installed:

```
kubectl get scantemplates -n DEV-NAMESPACE
```

Where DEV-NAMESPACE is the developer namespace where the scanner is installed.

For example:

```
$ kubectl get scantemplates
NAME                               AGE
blob-source-scan-template          10d
private-image-scan-template        10d
public-image-scan-template         10d
public-source-scan-template        10d
prisma-private-image-scan-template 10d
prisma-public-image-scan-template  10d
```

3. Create the following ImageScan YAML:

> **Note:**
> Some scanners do not support both ImageScan and SourceScan.

```
apiVersion: scanning.apps.tanzu.vmware.com/v1beta1
kind: ImageScan
metadata:
  name: sample-scanner-public-image-scan
spec:
  registry:
    image: "nginx:1.16"
  scanTemplate: SCAN-TEMPLATE
  scanPolicy: SCAN-POLICY # Optional
```

Where:

* SCAN-TEMPLATE is the name of the installed ScanTemplate in the DEV-NAMESPACE you retrieved earlier.
* SCAN-POLICY it’s an optional reference to an existing ScanPolicy in the same DEV-NAMESPACE.

For example:

```
apiVersion: scanning.apps.tanzu.vmware.com/v1beta1
kind: ImageScan
metadata:
  name: sample-prisma-public-image-scan
spec:
  registry:
    image: "nginx:1.16"
  scanTemplate: prisma-public-image-scan-template
  scanPolicy: scan-policy
```

5. Apply the ImageScan YAMLs:

To run the scans, apply them to the cluster by running these commands:

**ImageScan:**

```
kubectl apply -f PATH-TO-IMAGE-SCAN-YAML -n DEV-NAMESPACE
```

Where PATH-TO-IMAGE-SCAN-YAML is the path to the YAML file created earlier.


6. To verify the integration, get the scan to see if it completed by running:

**For ImageScan:**

```
kubectl get imagescan IMAGE-SCAN-NAME -n DEV-NAMESPACE
```

Where IMAGE-SCAN-NAME is the name of the ImageScan as defined in the YAML file created earlier.


> **Note:**
> If you define a ScanPolicy for the scans and the evaluation finds a violation, the Phase is Failed instead of Completed. In both cases the scan finished successfully.

Clean up:

```
kubectl delete -f PATH-TO-SCAN-YAML -n DEV-NAMESPACE
```

Where PATH-TO-SCAN-YAML is the path to the YAML file created earlier.
### Configure Tanzu Application Platform Supply Chain to use new scanner

In order to scan your images with the new scanner installed in the Out of the Box Supply Chain with Testing and Scanning, you must update your Tanzu Application Platform installation.

Add the ootb_supply_chain_testing_scanning.scanning section to your tap-values.yaml and perform a Tanzu Application Platform update.

In this file you can define which ScanTemplates is used for both SourceScan and ImageScan. The default values are the Grype Scanner ScanTemplates, but it can be overwritten by any other ScanTemplate present in your DEV-NAMESPACE. The same applies to the ScanPolicies applied to each kind of scan.

```
ootb_supply_chain_testing_scanning:
  scanning:
    image:
      template: IMAGE-SCAN-TEMPLATE
      policy: IMAGE-SCAN-POLICY
    source:
      template: SOURCE-SCAN-TEMPLATE
      policy: SOURCE-SCAN-POLICY
```

> **Note:**
> For the Supply Chain to work properly, the SOURCE-SCAN-TEMPLATE must support blob files and the IMAGE-SCAN-TEMPLATE must support private images.

For example:

```
ootb_supply_chain_testing_scanning:
  scanning:
    image:
      template: prisma-private-image-scan-template
      policy: prisma-scan-policy
    source:
      template: blob-source-scan-template
      policy: scan-policy
```

### Uninstall Scanner

To replace the scanner in the Supply Chain, follow the steps mentioned in Configure TAP Supply Chain to Use New Scanner. After the scanner is no longer required by the Supply Chain, you can remove the package by running:

```
tanzu package installed delete REFERENCE-NAME \
    --namespace tap-install
```

Where REFERENCE-NAME is the name you identified the package with, when installing in the Install section. For example, grype-scanner, prisma-scanner, snyk-scanner.

For example:

```
$ tanzu package installed delete prisma-scanner \
    --namespace tap-install
```
