Steps to prove out TAP's image signing and verification capability:

1. Download cosign - https://docs.sigstore.dev/cosign/installation/

2. Generate cosign key pair - cosign generate-key-pair

> repeat steps 3 and 4 on every workload namespace

3. Create a cosign secret
    ```
    kubectl -n [workload namespace] create secret genericcosign --from-literal=cosign.password=[cosign secret passphrase] --from-file=[path to cosign.key output of step 2] --from-file=[path to cosign.pub output of step 2]
    ```

    > Note: If you are using dockerhub or a registry that does not support OCI media types, you need to add the annotation kpack.io/cosign.docker-media-types: "1" to the cosign secret. Additional information [here](https://docs.vmware.com/en/Tanzu-Build-Service/1.3/vmware-tanzu-build-service-v13/GUID-managing-images.html#image-signing-with-cosign).
    ```
    ➜  k get secret cosign -n [workload namespace] -oyaml
    apiVersion: v1
    data:
    cosign.key: xxxxxxx
    cosign.password: xxxxxxxx
    cosign.pub: xxxxxx
    kind: Secret
    metadata:
    annotations:
        kpack.io/cosign.docker-media-types: "1"
    name: cosign
    namespace: [workload namespace] 
    type: Opaque
    ```

4. Add the cosign secret to the list of secrets attached to the service account resource that is building the image - `default` service account in workload's namespace, unless customized to a different service account
    ```
    kubectl edit sa default -n [workload namespace]
    ```

    Desired result (cosign secret associated with default service account):
    ```
    ➜ k get sa default -n [workload namespace] -oyaml
        apiVersion: v1
        imagePullSecrets:
        - name: registry-credentials
        - name: tap-registry
        - name: tbs-builder-secret-gen-placeholder-secret
        kind: ServiceAccount
        metadata:
        name: default
        namespace: [workload namespace]
        secrets:
        - name: registry-credentials
        - name: default-token-xxxxxxx
        - name: cosign
    ```

5. Apply a ClusterImagePolicy. The sample below excludes system namespaces and TAP namespaces. It also depicts a namepattern, where the build artifacts produced by supply chain will be pushed to.
    ```
    cat <<EOF | kubectl apply -f -
    apiVersion: signing.apps.tanzu.vmware.com/v1beta1
    kind: ClusterImagePolicy
    metadata:
    name: image-policy
    spec:
    verification:
        exclude:
        resources:
            namespaces:
            - kube-system
            - accelerator-system                 
            - alv-convention               
            - api-portal
            - app-live-view
            - avi-system
            - build-service
            - cartographer-system
            - cert-manager
            - conventions-system
            - developer-conventions
            - flux-system
            - image-policy-system
            - kapp-controller
            - kapp-controller-packaging-global
            - knative-eventing
            - knative-serving
            - knative-sources
            - kpack
            - kube-node-lease
            - kube-public
            - kube-system
            - learning-center-guided-ui
            - learning-center-guided-w01
            - learningcenter
            - metadata-store
            - scan-link-system
            - secretgen-controller
            - service-bindings
            - services-toolkit
            - source-system
            - spring-boot-convention
            - stacks-operator-system
            - tanzu-package-repo-global
            - tanzu-system-ingress
            - tap-gui
            - tap-install
            - tap-telemetry
            - tekton-pipelines
            - tkg-system
            - tkg-system-public
            - triggermesh
            - vmware-sources
        keys:
        - name: cosign-key
        publicKey: |
            -----BEGIN PUBLIC KEY-----
            [add cosign.pub content]
            -----END PUBLIC KEY-----
        images:
        - namePattern: [registry path used by supply chain]/*
        keys:
        - name: cosign-key
    EOF
    ```

6. Customize image-policy-webhook.signing.apps.tanzu.vmware.com TAP package in tap-values.
    To display the available settings for image-policy-webhook, run:
    ```
    tanzu package available get image-policy-webhook.signing.apps.tanzu.vmware.com/[package version] --values-schema --namespace tap-install
    ```
    
    If updates were made to tap-values, update the TAP meta package for customizations to image-policy-webhook to take effect:
    ```
    tanzu package installed update tap -p tap.tanzu.vmware.com -v [TAP version]  --values-file tap-values.yaml -n tap-install
    ```

7. Submit a workload. 
    ```
    tanzu apps workload create -f workload.yml -n [workload namespace]
    ```

8. Check TBS logs to verify that the image produced includes signing information:
    ```
    ➜  kp build logs [workload name] -n [workload namespace]
    ```

<details>
    <summary>sample run</summary>

    ➜  kp build logs mck-pov -n mck-pov
    Saving index.docker.io/tapsme/mck-pov-mck-pov...
    *** Images (sha256:c82a31daffe9728fbff0f3e7d75d78eb245add11b4eccf8dfc869ce4dff3f6ea):
        index.docker.io/tapsme/mck-pov-mck-pov
        index.docker.io/tapsme/mck-pov-mck-pov:b1.20220409.155344
    Adding cache layer 'paketo-buildpacks/bellsoft-liberica:jdk'
    Adding cache layer 'paketo-buildpacks/syft:syft'
    Adding cache layer 'paketo-buildpacks/maven:application'
    Adding cache layer 'paketo-buildpacks/maven:cache'
    Adding cache layer 'cache.sbom'
    ===> COMPLETION
    Loading secret for "https://index.docker.io/v1/" from secret "registry-credentials" at location "/var/build-secrets/registry-credentials"
    Pushing signature to: index.docker.io/tapsme/mck-pov-mck-pov:sha256-c82a31daffe9728fbff0f3e7d75d78eb245add11b4eccf8dfc869ce4dff3f6ea.sig
    Build successful

   
  </details>


9. Cosign adds the build number and build timestamp information to its signing annotations. This can be verified via:
```
➜  cosign verify -key /path/to/cosign.pub registry.example.com/project/image@sha256:<DIGEST> | jq

``` 
<details>
    <summary>sample run</summary>
    

    ➜  cosign verify --key cosign.pub index.docker.io/tapsme/mck-pov-mck-pov@sha256:c82a31daffe9728fbff0f3e7d75d78eb245add11b4eccf8dfc869ce4dff3f6ea | jq

    Verification for index.docker.io/tapsme/mck-pov-mck-pov@sha256:c82a31daffe9728fbff0f3e7d75d78eb245add11b4eccf8dfc869ce4dff3f6ea --
    The following checks were performed on each of these signatures:
    - The cosign claims were validated
    - The signatures were verified against the specified public key
    [
    {
        "critical": {
        "identity": {
            "docker-reference": "index.docker.io/tapsme/mck-pov-mck-pov"
        },
        "image": {
            "docker-manifest-digest": "sha256:c82a31daffe9728fbff0f3e7d75d78eb245add11b4eccf8dfc869ce4dff3f6ea"
        },
        "type": "cosign container image signature"
        },
        "optional": {
        "buildNumber": "1",
        "buildTimestamp": "20220409.155344"
        }
    }
    ]
   
  </details>

<br/>
10. Supply chain produced images will include a signature in an OCI-compliant format and pushed to the same registry where the image is stored (as shown in step 8's sample run). Images with a cosign signature will be allowed by the ImagePolicy webhook to be deployed. Note that the ImagePolicy webhook uses the ClusterImagePolicy (step 5) to determine which image patterns to verify.

To see the imagepolicy webhook in action, check the image-policy-controller-manager pod logs in the image-policy-system namespace:

```
➜  k logs image-policy-controller-manager-[pod guid] -n image-policy-system -c manager  
```

<br/>
11. Below is an example of the imagepolicy webhook allowing a deployment since the image(s) referenced are signed.

<details>
    <summary>sample run</summary>
    ➜  k logs image-policy-controller-manager-6bf7b6447d-bltm2 -n image-policy-system -c manager

    1.6495198241808946e+09	DEBUG	controller-runtime.webhook.webhooks	wrote response	{"webhook": "/signing-policy-check", "code": 200, "reason": "", "UID": "21bbfa32-4855-46b8-ab9b-310e03f21147", "allowed": true}
    1.6495198242151175e+09	DEBUG	controller-runtime.webhook.webhooks	received request	{"webhook": "/signing-policy-check", "UID": "7389edeb-f5e5-43e1-9b55-c0da04af18d2", "kind": "apps/v1, Kind=Deployment", "resource": {"group":"apps","version":"v1","resource":"deployments"}}
    1.649519824215145e+09	INFO	webhook	Entering handler function
    1.6495198242156758e+09	INFO	webhook	Image patterns count: 1
    1.6495198242156875e+09	INFO	webhook	matching pattern: index.docker.io/tapsme/mck-pov-mck-pov* against image index.docker.io/tapsme/mck-pov-mck-pov@sha256:a2340e09ee4eed684d75cf0b4ae61f52f6782a34f514dfdd022c29bcbf3b5668
    1.649519824215701e+09	INFO	scst-sign-webhook-utils	successfully read namespace	{"namespace": "image-policy-system"}
    1.6495198242157168e+09	INFO	scst-sign-webhook-utils	successfully read namespace	{"namespace": "image-policy-system"}
    1.6495198242157261e+09	INFO	webhook	keychain data	{"imagePullSecrets": [], "serviceAccountName": "default", "namespace": "mck-pov"}
    1.649519824238027e+09	INFO	webhook	keychain data	{"imagePullSecrets": [], "serviceAccountName": "image-policy-registry-credentials", "namespace": "image-policy-system"}
    1.6495198244545214e+09	INFO	webhook	Image patterns count: 1
    1.6495198244545445e+09	INFO	webhook	matching pattern: index.docker.io/tapsme/mck-pov-mck-pov* against image registry.tanzu.vmware.com/tanzu-application-platform/tap-packages@sha256:830ed1c676c0d17d7174dc4ef17ea84b7e6d6b70f1e8bc800b3945b3c7f5dc92
    1.6495198244545527e+09	INFO	webhook	Unmatched image policy: registry.tanzu.vmware.com/tanzu-application-platform/tap-packages@sha256:830ed1c676c0d17d7174dc4ef17ea84b7e6d6b70f1e8bc800b3945b3c7f5dc92
    1.6495198244564323e+09	DEBUG	controller-runtime.webhook.webhooks	wrote response	{"webhook": "/signing-policy-check", "code": 200, "reason": "", "UID": "7389edeb-f5e5-43e1-9b55-c0da04af18d2", "allowed": true}

</details>

<br/>
12. Below is an example of the imagepolicy webhook denying a pod with an unsigned image.

```
➜  k run imagesignverify --image index.docker.io/tapsme/mck-pov-mck-pov:0.0.1 -n mck-pov

Error from server (The image: index.docker.io/tapsme/mck-pov-mck-pov:0.0.1 is not signed.): admission webhook "image-policy-webhook.signing.apps.tanzu.vmware.com" denied the request: The image: index.docker.io/tapsme/mck-pov-mck-pov:0.0.1 is not signed.

```
Check the image-policy-controller-manager pod logs in the image-policy-system namespace to see a sample webhook verification failure:
<details>
    <summary>sample run</summary>
    ➜  k logs image-policy-controller-manager-6bf7b6447d-bltm2 -n image-policy-system -c manager

    1.6495216934572992e+09	INFO	webhook	matching pattern: index.docker.io/tapsme/mck-pov-mck-pov* against image index.docker.io/tapsme/mck-pov-mck-pov:0.0.1
    1.6495216934573162e+09	INFO	scst-sign-webhook-utils	successfully read namespace	{"namespace": "image-policy-system"}
    1.6495216934573352e+09	INFO	scst-sign-webhook-utils	successfully read namespace	{"namespace": "image-policy-system"}
    1.6495216934573529e+09	INFO	webhook	keychain data	{"imagePullSecrets": ["registry-credentials", "tap-registry", "tbs-builder-secret-gen-placeholder-secret"], "serviceAccountName": "", "namespace": ""}
    1.6495216934918203e+09	INFO	webhook	keychain data	{"imagePullSecrets": [], "serviceAccountName": "default", "namespace": ""}
    1.649521693510627e+09	INFO	webhook	keychain data	{"imagePullSecrets": [], "serviceAccountName": "image-policy-registry-credentials", "namespace": "image-policy-system"}
    1.6495216937255707e+09	ERROR	webhook	Failed to verify	{"error": "no matching signatures:\n"}
    gitlab.eng.vmware.com/tanzu-image-signing/image-policy-webhook/pkg/mutating-webhook.(*SignatureValidator).findSignature
        /workspace/pkg/mutating-webhook/webhook_main.go:192
    gitlab.eng.vmware.com/tanzu-image-signing/image-policy-webhook/pkg/mutating-webhook.(*SignatureValidator).matchPolicy
        /workspace/pkg/mutating-webhook/webhook_main.go:116
    gitlab.eng.vmware.com/tanzu-image-signing/image-policy-webhook/pkg/mutating-webhook.(*SignatureValidator).Handle
        /workspace/pkg/mutating-webhook/webhook_main.go:79
    sigs.k8s.io/controller-runtime/pkg/webhook/admission.(*Webhook).Handle
        /go/pkg/mod/sigs.k8s.io/controller-runtime@v0.11.1/pkg/webhook/admission/webhook.go:146
    sigs.k8s.io/controller-runtime/pkg/webhook/admission.(*Webhook).ServeHTTP
        /go/pkg/mod/sigs.k8s.io/controller-runtime@v0.11.1/pkg/webhook/admission/http.go:99
    github.com/prometheus/client_golang/prometheus/promhttp.InstrumentHandlerInFlight.func1
        /go/pkg/mod/github.com/prometheus/client_golang@v1.12.1/prometheus/promhttp/instrument_server.go:40
    net/http.HandlerFunc.ServeHTTP
        /opt/bitnami/go/src/net/http/server.go:2047
    github.com/prometheus/client_golang/prometheus/promhttp.InstrumentHandlerCounter.func1
        /go/pkg/mod/github.com/prometheus/client_golang@v1.12.1/prometheus/promhttp/instrument_server.go:117
    net/http.HandlerFunc.ServeHTTP
        /opt/bitnami/go/src/net/http/server.go:2047
    github.com/prometheus/client_golang/prometheus/promhttp.InstrumentHandlerDuration.func2
        /go/pkg/mod/github.com/prometheus/client_golang@v1.12.1/prometheus/promhttp/instrument_server.go:84
    net/http.HandlerFunc.ServeHTTP
        /opt/bitnami/go/src/net/http/server.go:2047
    net/http.(*ServeMux).ServeHTTP
        /opt/bitnami/go/src/net/http/server.go:2425
    net/http.serverHandler.ServeHTTP
        /opt/bitnami/go/src/net/http/server.go:2879
    net/http.(*conn).serve
        /opt/bitnami/go/src/net/http/server.go:1930
    1.6495216937256594e+09	DEBUG	controller-runtime.webhook.webhooks	wrote response	{"webhook": "/signing-policy-check", "code": 403, "reason": "The image: index.docker.io/tapsme/mck-pov-mck-pov:0.0.1 is not signed.", "UID": "a9287efd-6bb7-41e6-8639-32d9e9cea401", "allowed": false}


</details>
