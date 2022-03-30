# Tanzu Application Platform Cheat Sheet 

This guide will walk you through different common commands to troubleshoot and interact with Tanzu Application Platform!

## Helpful Installation Commands

VMWare provides a [troubleshooting guide](https://docs.vmware.com/en/Tanzu-Application-Platform/1.0/tap/GUID-troubleshooting-tap-troubleshoot-install-tap.html) for installation of TAP. This guide is a supplement and provides quick reference for common commands that can help you troubleshoot. 

### Installation and Package Management

***Install or Update TAP:***

```
tanzu package install tap -p tap.tanzu.vmware.com -v $TAP_VERSION --values-file tap-values.yml -n tap-install
```

```
tanzu package installed update tap -p tap.tanzu.vmware.com -v $TAP_VERSION --values-file tap-values.yml -n tap-install
```

***View Installed Packages by TAP on a Cluster:***

```
tanzu package installed list -n tap-install
```

***Delete a TAP Package***

```
tanzu package installed delete cloud-native-runtimes -n tap-install
```

***View Possible Configuration Settings for a Package (You can substitute any package name from the prior command):***

```
tanzu package available get tap.tanzu.vmware.com/1.0.0 --values-schema --namespace tap-install
```

***Inspect Installation Status of a TAP Package (e.g. buildservice)***

```
kubectl get packageinstall buildservice -n tap-install -o yaml
```
```
kubectl get packageinstall buildservice -n tap-install -ojsonpath='{.status}'
```

***Further Inspect Kubernetes Resources brought by each TAP Package or Troubleshoot Package Reconciliation:***

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

***Inspect Reconciliation Status Using Kapp***

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

***Kapp Cheatsheet***
  
More kapp cli capabilities are outlined in this [cheat sheet](https://carvel.dev/kapp/docs/latest/cheatsheet/).

### Package Repository Connectivity

***Failure in reconciling the Tanzu Application Platform package repository is often an authentication issue. Double check the tap-registry secret is properly set:***

```
kubectl get secret tap-registry --namespace tap-install -ojsonpath='{.data.\.dockerconfigjson}' | base64 -d
```

