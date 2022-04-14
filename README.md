# Tanzu Application Platform Cheat Sheet 

This guide will walk you through different common commands to troubleshoot and interact with Tanzu Application Platform!

# Helpful Installation Commands

VMWare provides a [troubleshooting guide](https://docs.vmware.com/en/Tanzu-Application-Platform/1.0/tap/GUID-troubleshooting-tap-troubleshoot-install-tap.html) for installation of TAP. This guide is a supplement and provides quick reference for common commands that can help you troubleshoot. 

# Installation and Package Management

## Install or Update TAP:

```
tanzu package install tap -p tap.tanzu.vmware.com -v $TAP_VERSION --values-file tap-values.yml -n tap-install
```

```
tanzu package installed update tap -p tap.tanzu.vmware.com -v $TAP_VERSION --values-file tap-values.yml -n tap-install
```

## View Installed Packages by TAP on a Cluster:

```
tanzu package installed list -n tap-install
```

## Delete a TAP Package

```
tanzu package installed delete cloud-native-runtimes -n tap-install
```

## View Possible Configuration Settings for a Package (You can substitute any package name from the prior command):

```
tanzu package available get tap.tanzu.vmware.com/1.0.0 --values-schema --namespace tap-install
```

## Inspect Installation Status of a TAP Package (e.g. buildservice)

```
kubectl get packageinstall buildservice -n tap-install -o yaml
```
```
kubectl get packageinstall buildservice -n tap-install -ojsonpath='{.status}'
```

## Further Inspect Kubernetes Resources brought by each TAP Package or Troubleshoot Package Reconciliation:

Use the kapp cli ([install instructions](https://carvel.dev/kapp/docs/latest/install/))

  * `kapp list -A` - list all packages 
  * `kapp inspect -a tap-ctrl -n tap-install` - inspect tap package
  * `kapp inspect -a tap-gui-ctrl -n tap-install` - inspect tap-gui package
  * `kapp inspect -a tekton-pipelines-ctrl -n tap-install` - inspect tekton package

  <details>
    <summary>Sample kapp Inspect and Results</summary>
    
  ```
    ➜ kapp inspect -a tap-ctrl -n tap-install
    Target cluster 'https://10.213.79.10:6443' (nodes: unicorn-control-plane-wxhk5, 5+)

    07:56:06AM: info: Resources: Ignoring group version: schema.GroupVersionResource{Group:"stats.antrea.tanzu.vmware.com", Version:"v1alpha1", Resource:"networkpolicystats"}: feature NetworkPolicyStats disabled
    07:56:06AM: info: Resources: Ignoring group version: schema.GroupVersionResource{Group:"stats.antrea.tanzu.vmware.com", Version:"v1alpha1", Resource:"antreanetworkpolicystats"}: feature NetworkPolicyStats disabled
    07:56:06AM: info: Resources: Ignoring group version: schema.GroupVersionResource{Group:"stats.antrea.tanzu.vmware.com", Version:"v1alpha1", Resource:"antreaclusternetworkpolicystats"}: feature NetworkPolicyStats disabled

    Resources in app 'tap-ctrl'

    Namespace    Name                                    Kind                Owner  Conds.  Rs    Ri                                   Age
    (cluster)    tap-install-cluster-admin-role          ClusterRole         kapp   -       ok    -                                    19h
    ^            tap-install-cluster-admin-role-binding  ClusterRoleBinding  kapp   -       ok    -                                    19h
    tap-install  accelerator                             PackageInstall      kapp   1/1 t   ok    -                                    19h
    ^            accelerator-values                      Secret              kapp   -       ok    -                                    19h
    ^            api-portal                              PackageInstall      kapp   1/1 t   ok    -                                    19h
    ^            api-portal-values                       Secret              kapp   -       ok    -                                    19h
    ^            appliveview                             PackageInstall      kapp   1/1 t   ok    -                                    19h
    ^            appliveview-conventions                 PackageInstall      kapp   1/1 t   fail  Reconcile failed:  (message: Error   19h
                                                                                                  (see .status.usefulErrorMessage for
                                                                                                  details))
    ^            appliveview-values                      Secret              kapp   -       ok    -                                    19h
    ^            buildservice                            PackageInstall      kapp   1/1 t   ok    -                                    19h
    ^            buildservice-values                     Secret              kapp   -       ok    -                                    19h
    ^            cartographer                            PackageInstall      kapp   1/1 t   fail  Reconcile failed:  (message: Error   19h
                                                                                                  (see .status.usefulErrorMessage for
                                                                                                  details))
    ^            cnrs                                    PackageInstall      kapp   1/1 t   ok    -                                    19h
    ^            cnrs-values                             Secret              kapp   -       ok    -                                    19h
    ^            contour                                 PackageInstall      kapp   1/1 t   fail  Reconcile failed:  (message: Error   19h
                                                                                                  (see .status.usefulErrorMessage for
                                                                                                  details))
    ^            contour-values                          Secret              kapp   -       ok    -                                    19h
    ^            conventions-controller                  PackageInstall      kapp   1/1 t   fail  Reconcile failed:  (message: Error   19h
                                                                                                  (see .status.usefulErrorMessage for
                                                                                                  details))
    ^            developer-conventions                   PackageInstall      kapp   1/1 t   fail  Reconcile failed:  (message: Error   19h
                                                                                                  (see .status.usefulErrorMessage for
                                                                                                  details))
    ^            fluxcd-source-controller                PackageInstall      kapp   1/1 t   ok    -                                    19h
    ^            grype                                   PackageInstall      kapp   1/1 t   ok    -                                    19h
    ^            grype-values                            Secret              kapp   -       ok    -                                    19h
    ^            image-policy-webhook                    PackageInstall      kapp   1/1 t   fail  Reconcile failed:  (message: Error   19h
                                                                                                  (see .status.usefulErrorMessage for
                                                                                                  details))
    ^            image-policy-webhook-values             Secret              kapp   -       ok    -                                    19h
    ^            learningcenter                          PackageInstall      kapp   1/1 t   ok    -                                    19h
    ^            learningcenter-values                   Secret              kapp   -       ok    -                                    19h
    ^            learningcenter-workshops                PackageInstall      kapp   1/1 t   ok    -                                    19h
    ^            metadata-store                          PackageInstall      kapp   1/1 t   fail  Reconcile failed:  (message: Error   19h
                                                                                                  (see .status.usefulErrorMessage for
                                                                                                  details))
    ^            metadata-store-values                   Secret              kapp   -       ok    -                                    19h
    ^            ootb-delivery-basic                     PackageInstall      kapp   1/1 t   ok    -                                    19h
    ^            ootb-delivery-basic-values              Secret              kapp   -       ok    -                                    19h
    ^            ootb-supply-chain-basic                 PackageInstall      kapp   1/1 t   ok    -                                    19h
    ^            ootb-supply-chain-basic-values          Secret              kapp   -       ok    -                                    19h
    ^            ootb-templates                          PackageInstall      kapp   1/1 t   ok    -                                    19h
    ^            scanning                                PackageInstall      kapp   1/1 t   ok    -                                    19h
    ^            scanning-values                         Secret              kapp   -       ok    -                                    19h
    ^            service-bindings                        PackageInstall      kapp   1/1 t   ok    -                                    19h
    ^            services-toolkit                        PackageInstall      kapp   1/1 t   ok    -                                    19h
    ^            source-controller                       PackageInstall      kapp   1/1 t   ok    -                                    19h
    ^            spring-boot-conventions                 PackageInstall      kapp   1/1 t   fail  Reconcile failed:  (message: Error   19h
                                                                                                  (see .status.usefulErrorMessage for
                                                                                                  details))
    ^            tap-gui                                 PackageInstall      kapp   1/1 t   ok    -                                    19h
    ^            tap-gui-values                          Secret              kapp   -       ok    -                                    19h
    ^            tap-install-sa                          ServiceAccount      kapp   -       ok    -                                    19h
    ^            tap-telemetry                           PackageInstall      kapp   1/1 t   ok    -                                    19h
    ^            tekton-pipelines                        PackageInstall      kapp   1/1 t   ok    -                                    19h

    Rs: Reconcile state
    Ri: Reconcile information

    44 resources

    Succeeded
  ```
  </details>


## Inspect Reconciliation Status Using Kapp

If you exclude a package after performing a profile installation which included that package, the accurate package status will not reflect immediately through `tanzu package installed list -n tap-install`. kapp cli's app-change is a good way to get up-to-date reconciliation status, including timestamp. This can be done by issuing `kapp app-change list -a tap-ctrl -n tap-install` (where tap-ctrl is the kapp managed app name).

  <details>
    <summary>Sample List of App Changes</summary>
    
  ```
  ➜ kapp app-change list -a tap-ctrl -n tap-install
  Target cluster 'https://10.213.79.10:6443' (nodes: unicorn-control-plane-wxhk5, 5+)

  App changes

  Name                   Started At            Finished At           Successful  Description
  tap-ctrl-change-4b4ww  2022-01-19T15:42:24Z  2022-01-19T15:42:55Z  true        update: Op: 0 create, 1 delete, 0 update, 0 noop / Wait to: 0 reconcile, 1 delete, 0 noop
  tap-ctrl-change-q7cl9  2022-01-18T20:32:48Z  2022-01-18T20:42:47Z  true        update: Op: 3 create, 0 delete, 0 update, 17 noop / Wait to: 20 reconcile, 0 delete, 0 noop
  tap-ctrl-change-vdbz7  2022-01-18T20:32:25Z  2022-01-18T20:32:26Z  false       update: Op: 13 create, 0 delete, 0 update, 7 noop / Wait to: 20 reconcile, 0 delete, 0 noop
  tap-ctrl-change-28z4c  2022-01-18T20:30:21Z  2022-01-18T20:31:58Z  false       update: Op: 45 create, 0 delete, 0 update, 0 noop / Wait to: 45 reconcile, 0 delete, 0 noop

  4 app changes

  Succeeded
  ```
  </details>

## Get TAP values

The values provided during TAP installation or update are set as a secret in the tap-install namespace. To get the tap values applied to a cluster run:
```
kubectl get secret tap-tap-install-values -n tap-install -o jsonpath='{.data.[replace with file name provided to tanzu package install]}' | base64 -d
```

> e.g. if file name is tap-values.yml - the command would be

```
kubectl get secret tap-tap-install-values -n tap-install -o jsonpath='{.data.tap-values\.yml}' | base64 -d
```

## Kapp Cheatsheet
  
More kapp cli capabilities are outlined in this [cheat sheet](https://carvel.dev/kapp/docs/latest/cheatsheet/).

# Package Repository Connectivity

## Failure in reconciling the Tanzu Application Platform package repository is often an authentication issue. Double check the tap-registry secret is properly set:

```
kubectl get secret tap-registry --namespace tap-install -ojsonpath='{.data.\.dockerconfigjson}' | base64 -d
```

## Force TAP to Reconcile

If you want to force TAP to reconcile you can use this function. It pauses the reconciliation and starts it again which will force TAP to reconcile. It's recommend to add this to your .bashrc or .zshrc files.

```
function tap-nudge() {
        TAPNS=tap-install
        kubectl -n $TAPNS patch packageinstalls.packaging.carvel.dev $1 --type='json' -p '[{"op": "add",
"path": "/spec/paused", "value":true}]}}' -v0
        kubectl -n $TAPNS patch packageinstalls.packaging.carvel.dev $1 --type='json' -p '[{"op": "add",
"path": "/spec/paused", "value":false}]}}' -v0
}
```

**Example of Usage:**

```
tap-nudge tap
```

# Interacting with a Workload

The Tanzu CLI has many commands you can use to interact with the [Workload resource](https://docs.vmware.com/en/Tanzu-Application-Platform/1.0/tap/GUID-cli-plugins-apps-command-reference-tanzu_apps_workload.html). Here are some common commands that are useful:

## Creating a Workload

```
tanzu apps workload create tanzu-java-web-app \
--git-repo https://github.com/sample-accelerators/tanzu-java-web-app \
--git-branch main \
--type web \
--label app.kubernetes.io/part-of=tanzu-java-web-app \
--yes
```

## Creating a Workload from a File

```
tanzu apps workload create --file workload.yaml
```

## Deleting a Workload

```
tanzu apps workload delete my-workload
```

## List all Workloads

```
tanzu apps workload list
tanzu apps workload list --all-namespaces
```

## Tail a Workload

You can stream logs for a workload until canceled. To cancel, press Ctl-c in the shell or kill the process. As new workload pods are started, the logs are displayed. To show historical logs use –since.

```
tanzu apps workload tail my-workload --since 1h
```

# Troubleshooting Builds

https://github.com/vmware-tanzu/kpack-cli

## Tails logs for an image build

kp build logs my-image -b 2 -n my-namespace


# Clean Up TAP

## Delete the Tanzu CLI

This code below will do the follow: 
1. Remove previously downloaded cli files
2. Remove CLI binary (executable)
3. Remove config directory
4. Remove config directory
5. remove cached catalog.yaml
6. Remove plug-ins
```
rm -rf $HOME/tanzu/cli        
sudo rm /usr/local/bin/tanzu  
rm -rf ~/.config/tanzu/       
rm -rf ~/.tanzu/              
rm -rf ~/.cache/tanzu         
rm -rf ~/Library/Application\ Support/tanzu-cli/* 
```

## Restart TAP GUI
```
k deployment restart rollout server -n tap-gui
```

# Naming Conventions used by TAP

## Build Dependencies Images

Tanzu Build Service uses a writable repository in your company registry to write/pulled dependencies used to build application images within TAP.

**Scope:** This repo is shared across all applications on that instance of TAP by TBS (You don't configure this per application). This is configured in tap-values.yaml when installing TAP.


```
tap-values.yaml

buildservice:
  kp_default_repository: "<organization>/build-service"
  kp_default_repository_username: "userName"
  kp_default_repository_password: "password"
  
ootb_supply_chain_basic:
  registry:
    server: "index.docker.io"
    repository: "tapsme"
```

**Convention of Artifact in Repo:** `<server>/<kp_default_repository>`

**Example:** `index.docker.io/<organization>/build-service`


## Images Generated by Tanzu Build Service
 
TAP uses Tanzu Build Service to create images that are generated from a code repository that is configured in the workload.yaml. A tap-values.yaml is used to configure the location for the built image artifact produced by TBS within that instance of TAP. 
 
**Scope:** TAP will create this stucture for each workload created within TAP. 
 
```
tap-values.yaml
 
 ootb_supply_chain_basic:
  registry:
    server: "index.docker.io"
    repository: "tapsme"
```
 
**Convention of Artifact in Repo:** `<server>/<repository>/<Workload Name>-<namespace>`

**Example:** `index.docker.io/tapsme/inner-loop-demo-engineer1`


## Images Generated by Config Writer
 
Config Writer uses [imgpkg](https://carvel.dev/imgpkg/docs/v0.27.0/) to store configuration information about the deployment into TAP.
 
**Scope:** TAP will create this stucture for each workload created within TAP. 

```
tap-values.yaml
 
 ootb_supply_chain_basic:
  registry:
    server: "index.docker.io"
    repository: "tapsme"
```
 
**Convention of Artifact in Repo:** `<server>/<repository>/<Workload Name>-<namespace>-bundle`

**Example:** `index.docker.io/tapsme/inner-loop-demo-engineer1-bundle`


## Cloud Native Run Times URL

TAP uses the cnrs.domain_name in tap-values.yaml along with the along with the workload name and namespace to construct the URL for applications deployed using Cloud Native Runtimes. 

**Scope:** TAP will create this structure for each workload created within TAP.

```
tap-values.yaml

cnrs:
  domain_name: tapsme.org
```

`http://<workload-name>.<namespace>.<domain_name>`

**Example:** `inner-loop-demo.engineer1.tapsme.org`


## Tilt Source Code

Tilt create an image that contains the source code that is live loaded onto a cluster to allow VSCode to stream changes onto a live container. These images use the convention of -source on the image and are pushed into a repo. 

**Scope:** TAP will create this structure for each workload created within TAP. 
 
```
tap-values.yaml
 
 ootb_supply_chain_basic:
  registry:
    server: "index.docker.io"
    repository: "tapsme"
```
 
**Convention of Artifact in Repo:** `<server>/<repository>/<Workload Name>-<namespace>-<source>`

**Example:** `index.docker.io/tapsme/inner-loop-demo-engineer1-source`

# Learning Center

## Retrieve Installed Training Portals 

```
kubectl get trainingportal 
```

## Retrieve Intstalled Workshops

```
kubectl get workshops
```

## Troubleshoot Learning Center

### Check the Learning Center Operator

```
kubectl get po -n learningcenter
```
```
kubectl logs learningcenter-operator-<xxxxxxxxxx>-<xxxxx>
```

### Check the Pods in the Training Portal
This example uses the Training Portal that is installed with TAP

```
kubectl get po -n learning-center-guided-ui

NAME                                     READY   STATUS    RESTARTS   AGE
learningcenter-portal-66bfdf97cf-5f9k5   1/1     Running   0          34d

k logs learningcenter-portal-66bfdf97cf-5f9k5 -n learning-center-guided-ui
```

### Check the Pods in the Workshop
This example uses the Workshop that is installed with TAP

```
kubectlkubetcl get po -n learning-center-guided-w02 get po -n learning-center-guided-w02

NAME                                                        READY   STATUS    RESTARTS   AGE
learning-center-guided-w02-s006-766898bf46-w2xfr            2/2     Running   0          42s
learning-center-guided-w02-s006-registry-589bc4c6bd-9tb79   1/1     Running   0          42s


kubectl logs learning-center-guided-w02-s006-766898bf46-w2xfr -n learning-center-guided-w02 -c workshop
kubectl logs learning-center-guided-w02-s006-766898bf46-w2xfr -n learning-center-guided-w02 -c docker
```
