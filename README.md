<h1 align="center">kubectl-fields</h1>

<h4 align="center">Kubectl resources hierarchy parsing tool</h4>

<p align="center">
  <a href="https://cloud.drone.io/rewanth1997/kubectl-fields">
    <img src="https://cloud.drone.io/api/badges/rewanth1997/kubectl-fields/status.svg">
  </a>
</p>

<p align="center">Kubectl-fields is a cli tool to parse <code>kubectl explain --recursive</code> output and grep matching pattern in one-liner hierarchy format.</p> <p align="center">This tool is exclusively built for CKA/CKAD applicants.</p>

## Installation

Installing with krew
```console
wget https://raw.githubusercontent.com/rewanth1997/kubectl-fields/master/deploy/krew/plugin.yaml
kubectl krew install --manifest=./plugin.yaml
```

Installing with wget
```console
wget https://github.com/rewanth1997/kubectl-fields/releases/download/v1.0.0/kubectl-fields-Linux-x86_64.tar.gz
sudo tar xvf kubectl-fields-Linux-x86_64.tar.gz -C /usr/bin/
```

## Developer build

Build from Makefile
```console
make build
```

Build only for Linux
```console
make build-linux
```

Build with go
```console
go get ./...
go build -o kubectl-fields main.go
mv kubectl-fields /usr/bin
```

Cross platform builds with go
```console
go get ./...
GOOS=windows GOARCH=amd64 go build -o kubectl-fields.exe main.go
```

## Usage

```console
rewanth@ubuntu:~/go/src/kubectl-fields$ kubectl fields -h
Kubectl resources hierarchy parser.

More info: https://github.com/rewanth1997/kubectl-fields

Usage:
  kubectl-fields [flags]

Examples:
$ kubectl fields po.spec capa
containers.securityContext.capabilities
initContainers.securityContext.capabilities

$ kubectl fields -i svc ip
spec.clusterIP
spec.externalIPs
spec.loadBalancerIP
spec.sessionAffinityConfig.clientIP
status.loadBalancer.ingress.ip

Additional kubectl-fields example (the hard way: not recommended). Developed to run tests on pipeline

$ kubectl explain --recursive po.spec | ./kubectl-fields --stdin ver
dnsConfig.nameservers
volumes.csi.driver
volumes.flexVolume.driver
volumes.iscsi.chapAuthDiscovery
volumes.nfs.server

Flags:
  -h, --help          help for kubectl-fields
  -i, --ignore-case   Ignore case distinction
      --stdin         Expects input via pipes
```

## Examples

```console
$ kubectl fields po.spec capa
containers.securityContext.capabilities
initContainers.securityContext.capabilities

$ kubectl fields -i svc ip
spec.clusterIP
spec.externalIPs
spec.loadBalancerIP
spec.sessionAffinityConfig.clientIP
status.loadBalancer.ingress.ip
```

### Examples the hard way

```console
$ kubectl explain --recursive po.spec | ./kubectl-fields --stdin ver
dnsConfig.nameservers
volumes.csi.driver
volumes.flexVolume.driver
volumes.iscsi.chapAuthDiscovery
volumes.nfs.server

$ kubectl explain --recursive po.spec | ./kubectl-fields --stdin --ignore-case ver
containers.env.valueFrom.fieldRef.apiVersion
dnsConfig.nameservers
initContainers.env.valueFrom.fieldRef.apiVersion
volumes.csi.driver
volumes.downwardAPI.items.fieldRef.apiVersion
volumes.flexVolume.driver
volumes.iscsi.chapAuthDiscovery
volumes.nfs.server
volumes.projected.sources.downwardAPI.items.fieldRef.apiVersion
```

## Pipeline

This application uses [drone CI](https://github.com/drone/drone) for building and running all test cases.

All the test cases are associated with kubectl. But as of now we cannot run `kubectl` inside containers. As a way around, all the kubectl data required for running test cases is pre-saved into text files. All the test cases are executed from pre-saved data against fresh built binary executable to check its robustness.

## Why to use

CKA and CKAD are time-constrained exams. The default kubectl explain recursive command consumes lot of time to find the exact hierarchical order. This tool is built to ease the process of finding hierarchial structure for a given parameter.

It is difficult to lookup for exact hierarchical structure using kubectl explain recursive command. This tool solves this particular problem by printing the output in a easy readable one-liner format.

This tool has tremendous advantage over grep for this use case.

### Problem statement

For ex, in CKA/CKAD exam you are asked to add SYS_ADMIN capabilities to a given pod. Finding the exact hierarchial order using kubectl explain command is time consuming and tiresome.

### Current solution approach

Let's say you know that capabilities exist inside `po.spec`. You can apply recursive loop directly on `po.spec` instead of running it globally on `po`.

You can try to grep for string, "capabilities". The output is as follows:

```console
rewanth@ubuntu:~/go/src/kubectl-fields$ kubectl explain --recursive po.spec | grep capabilities
         capabilities   <Object>
         capabilities   <Object>
```

In case if you need to find "version" attribute in services, then the below command is executed.

```console
$ kubectl explain --recursive svc | grep -i version
VERSION:  v1
   apiVersion	<string>
            apiVersion	<string>
               resourceVersion	<string>
         apiVersion	<string>
         apiVersion	<string>
      resourceVersion	<string>
```

But it doesn't show the hierarchial order of attribute. It just shows the matching lines. You can use advanced grep functions and try to find the parent hierarchy of `capabilities` attribute.

```console
rewanth@ubuntu:~/go/src/kubectl-fields$ kubectl explain --recursive po.spec | grep capabilities -C 5
      resources <Object>
         limits <map[string]string>
         requests       <map[string]string>
      securityContext   <Object>
         allowPrivilegeEscalation       <boolean>
         capabilities   <Object>
            add <[]string>
            drop        <[]string>
         privileged     <boolean>
         procMount      <string>
         readOnlyRootFilesystem <boolean>
--
      resources <Object>
         limits <map[string]string>
         requests       <map[string]string>
      securityContext   <Object>
         allowPrivilegeEscalation       <boolean>
         capabilities   <Object>
            add <[]string>
            drop        <[]string>
         privileged     <boolean>
         procMount      <string>
         readOnlyRootFilesystem <boolean>

```

But still it doesn't show the complete parent hierarchy order. The full list need to be printed, scrolled and analyzed to find the exact hierarchical structure.

Not only it will take so much time to scroll up and down to find parent of an element in terminal, the process is very tedious.

<details>
<summary><strong>kubectl explain --recursive po.spec</strong> (click to expand 624 lines output)</summary>

```console
rewanth@ubuntu:~/go/src/kubectl-fields$ kubectl explain --recursive po.spec
KIND:     Pod
VERSION:  v1

RESOURCE: spec <Object>

DESCRIPTION:
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status

     PodSpec is a description of a pod.

FIELDS:
   activeDeadlineSeconds	<integer>
   affinity	<Object>
      nodeAffinity	<Object>
         preferredDuringSchedulingIgnoredDuringExecution	<[]Object>
            preference	<Object>
               matchExpressions	<[]Object>
                  key	<string>
                  operator	<string>
                  values	<[]string>
               matchFields	<[]Object>
                  key	<string>
                  operator	<string>
                  values	<[]string>
            weight	<integer>
         requiredDuringSchedulingIgnoredDuringExecution	<Object>
            nodeSelectorTerms	<[]Object>
               matchExpressions	<[]Object>
                  key	<string>
                  operator	<string>
                  values	<[]string>
               matchFields	<[]Object>
                  key	<string>
                  operator	<string>
                  values	<[]string>
      podAffinity	<Object>
         preferredDuringSchedulingIgnoredDuringExecution	<[]Object>
            podAffinityTerm	<Object>
               labelSelector	<Object>
                  matchExpressions	<[]Object>
                     key	<string>
                     operator	<string>
                     values	<[]string>
                  matchLabels	<map[string]string>
               namespaces	<[]string>
               topologyKey	<string>
            weight	<integer>
         requiredDuringSchedulingIgnoredDuringExecution	<[]Object>
            labelSelector	<Object>
               matchExpressions	<[]Object>
                  key	<string>
                  operator	<string>
                  values	<[]string>
               matchLabels	<map[string]string>
            namespaces	<[]string>
            topologyKey	<string>
      podAntiAffinity	<Object>
         preferredDuringSchedulingIgnoredDuringExecution	<[]Object>
            podAffinityTerm	<Object>
               labelSelector	<Object>
                  matchExpressions	<[]Object>
                     key	<string>
                     operator	<string>
                     values	<[]string>
                  matchLabels	<map[string]string>
               namespaces	<[]string>
               topologyKey	<string>
            weight	<integer>
         requiredDuringSchedulingIgnoredDuringExecution	<[]Object>
            labelSelector	<Object>
               matchExpressions	<[]Object>
                  key	<string>
                  operator	<string>
                  values	<[]string>
               matchLabels	<map[string]string>
            namespaces	<[]string>
            topologyKey	<string>
   automountServiceAccountToken	<boolean>
   containers	<[]Object>
      args	<[]string>
      command	<[]string>
      env	<[]Object>
         name	<string>
         value	<string>
         valueFrom	<Object>
            configMapKeyRef	<Object>
               key	<string>
               name	<string>
               optional	<boolean>
            fieldRef	<Object>
               apiVersion	<string>
               fieldPath	<string>
            resourceFieldRef	<Object>
               containerName	<string>
               divisor	<string>
               resource	<string>
            secretKeyRef	<Object>
               key	<string>
               name	<string>
               optional	<boolean>
      envFrom	<[]Object>
         configMapRef	<Object>
            name	<string>
            optional	<boolean>
         prefix	<string>
         secretRef	<Object>
            name	<string>
            optional	<boolean>
      image	<string>
      imagePullPolicy	<string>
      lifecycle	<Object>
         postStart	<Object>
            exec	<Object>
               command	<[]string>
            httpGet	<Object>
               host	<string>
               httpHeaders	<[]Object>
                  name	<string>
                  value	<string>
               path	<string>
               port	<string>
               scheme	<string>
            tcpSocket	<Object>
               host	<string>
               port	<string>
         preStop	<Object>
            exec	<Object>
               command	<[]string>
            httpGet	<Object>
               host	<string>
               httpHeaders	<[]Object>
                  name	<string>
                  value	<string>
               path	<string>
               port	<string>
               scheme	<string>
            tcpSocket	<Object>
               host	<string>
               port	<string>
      livenessProbe	<Object>
         exec	<Object>
            command	<[]string>
         failureThreshold	<integer>
         httpGet	<Object>
            host	<string>
            httpHeaders	<[]Object>
               name	<string>
               value	<string>
            path	<string>
            port	<string>
            scheme	<string>
         initialDelaySeconds	<integer>
         periodSeconds	<integer>
         successThreshold	<integer>
         tcpSocket	<Object>
            host	<string>
            port	<string>
         timeoutSeconds	<integer>
      name	<string>
      ports	<[]Object>
         containerPort	<integer>
         hostIP	<string>
         hostPort	<integer>
         name	<string>
         protocol	<string>
      readinessProbe	<Object>
         exec	<Object>
            command	<[]string>
         failureThreshold	<integer>
         httpGet	<Object>
            host	<string>
            httpHeaders	<[]Object>
               name	<string>
               value	<string>
            path	<string>
            port	<string>
            scheme	<string>
         initialDelaySeconds	<integer>
         periodSeconds	<integer>
         successThreshold	<integer>
         tcpSocket	<Object>
            host	<string>
            port	<string>
         timeoutSeconds	<integer>
      resources	<Object>
         limits	<map[string]string>
         requests	<map[string]string>
      securityContext	<Object>
         allowPrivilegeEscalation	<boolean>
         capabilities	<Object>
            add	<[]string>
            drop	<[]string>
         privileged	<boolean>
         procMount	<string>
         readOnlyRootFilesystem	<boolean>
         runAsGroup	<integer>
         runAsNonRoot	<boolean>
         runAsUser	<integer>
         seLinuxOptions	<Object>
            level	<string>
            role	<string>
            type	<string>
            user	<string>
         windowsOptions	<Object>
            gmsaCredentialSpec	<string>
            gmsaCredentialSpecName	<string>
      stdin	<boolean>
      stdinOnce	<boolean>
      terminationMessagePath	<string>
      terminationMessagePolicy	<string>
      tty	<boolean>
      volumeDevices	<[]Object>
         devicePath	<string>
         name	<string>
      volumeMounts	<[]Object>
         mountPath	<string>
         mountPropagation	<string>
         name	<string>
         readOnly	<boolean>
         subPath	<string>
         subPathExpr	<string>
      workingDir	<string>
   dnsConfig	<Object>
      nameservers	<[]string>
      options	<[]Object>
         name	<string>
         value	<string>
      searches	<[]string>
   dnsPolicy	<string>
   enableServiceLinks	<boolean>
   hostAliases	<[]Object>
      hostnames	<[]string>
      ip	<string>
   hostIPC	<boolean>
   hostNetwork	<boolean>
   hostPID	<boolean>
   hostname	<string>
   imagePullSecrets	<[]Object>
      name	<string>
   initContainers	<[]Object>
      args	<[]string>
      command	<[]string>
      env	<[]Object>
         name	<string>
         value	<string>
         valueFrom	<Object>
            configMapKeyRef	<Object>
               key	<string>
               name	<string>
               optional	<boolean>
            fieldRef	<Object>
               apiVersion	<string>
               fieldPath	<string>
            resourceFieldRef	<Object>
               containerName	<string>
               divisor	<string>
               resource	<string>
            secretKeyRef	<Object>
               key	<string>
               name	<string>
               optional	<boolean>
      envFrom	<[]Object>
         configMapRef	<Object>
            name	<string>
            optional	<boolean>
         prefix	<string>
         secretRef	<Object>
            name	<string>
            optional	<boolean>
      image	<string>
      imagePullPolicy	<string>
      lifecycle	<Object>
         postStart	<Object>
            exec	<Object>
               command	<[]string>
            httpGet	<Object>
               host	<string>
               httpHeaders	<[]Object>
                  name	<string>
                  value	<string>
               path	<string>
               port	<string>
               scheme	<string>
            tcpSocket	<Object>
               host	<string>
               port	<string>
         preStop	<Object>
            exec	<Object>
               command	<[]string>
            httpGet	<Object>
               host	<string>
               httpHeaders	<[]Object>
                  name	<string>
                  value	<string>
               path	<string>
               port	<string>
               scheme	<string>
            tcpSocket	<Object>
               host	<string>
               port	<string>
      livenessProbe	<Object>
         exec	<Object>
            command	<[]string>
         failureThreshold	<integer>
         httpGet	<Object>
            host	<string>
            httpHeaders	<[]Object>
               name	<string>
               value	<string>
            path	<string>
            port	<string>
            scheme	<string>
         initialDelaySeconds	<integer>
         periodSeconds	<integer>
         successThreshold	<integer>
         tcpSocket	<Object>
            host	<string>
            port	<string>
         timeoutSeconds	<integer>
      name	<string>
      ports	<[]Object>
         containerPort	<integer>
         hostIP	<string>
         hostPort	<integer>
         name	<string>
         protocol	<string>
      readinessProbe	<Object>
         exec	<Object>
            command	<[]string>
         failureThreshold	<integer>
         httpGet	<Object>
            host	<string>
            httpHeaders	<[]Object>
               name	<string>
               value	<string>
            path	<string>
            port	<string>
            scheme	<string>
         initialDelaySeconds	<integer>
         periodSeconds	<integer>
         successThreshold	<integer>
         tcpSocket	<Object>
            host	<string>
            port	<string>
         timeoutSeconds	<integer>
      resources	<Object>
         limits	<map[string]string>
         requests	<map[string]string>
      securityContext	<Object>
         allowPrivilegeEscalation	<boolean>
         capabilities	<Object>
            add	<[]string>
            drop	<[]string>
         privileged	<boolean>
         procMount	<string>
         readOnlyRootFilesystem	<boolean>
         runAsGroup	<integer>
         runAsNonRoot	<boolean>
         runAsUser	<integer>
         seLinuxOptions	<Object>
            level	<string>
            role	<string>
            type	<string>
            user	<string>
         windowsOptions	<Object>
            gmsaCredentialSpec	<string>
            gmsaCredentialSpecName	<string>
      stdin	<boolean>
      stdinOnce	<boolean>
      terminationMessagePath	<string>
      terminationMessagePolicy	<string>
      tty	<boolean>
      volumeDevices	<[]Object>
         devicePath	<string>
         name	<string>
      volumeMounts	<[]Object>
         mountPath	<string>
         mountPropagation	<string>
         name	<string>
         readOnly	<boolean>
         subPath	<string>
         subPathExpr	<string>
      workingDir	<string>
   nodeName	<string>
   nodeSelector	<map[string]string>
   preemptionPolicy	<string>
   priority	<integer>
   priorityClassName	<string>
   readinessGates	<[]Object>
      conditionType	<string>
   restartPolicy	<string>
   runtimeClassName	<string>
   schedulerName	<string>
   securityContext	<Object>
      fsGroup	<integer>
      runAsGroup	<integer>
      runAsNonRoot	<boolean>
      runAsUser	<integer>
      seLinuxOptions	<Object>
         level	<string>
         role	<string>
         type	<string>
         user	<string>
      supplementalGroups	<[]integer>
      sysctls	<[]Object>
         name	<string>
         value	<string>
      windowsOptions	<Object>
         gmsaCredentialSpec	<string>
         gmsaCredentialSpecName	<string>
   serviceAccount	<string>
   serviceAccountName	<string>
   shareProcessNamespace	<boolean>
   subdomain	<string>
   terminationGracePeriodSeconds	<integer>
   tolerations	<[]Object>
      effect	<string>
      key	<string>
      operator	<string>
      tolerationSeconds	<integer>
      value	<string>
   volumes	<[]Object>
      awsElasticBlockStore	<Object>
         fsType	<string>
         partition	<integer>
         readOnly	<boolean>
         volumeID	<string>
      azureDisk	<Object>
         cachingMode	<string>
         diskName	<string>
         diskURI	<string>
         fsType	<string>
         kind	<string>
         readOnly	<boolean>
      azureFile	<Object>
         readOnly	<boolean>
         secretName	<string>
         shareName	<string>
      cephfs	<Object>
         monitors	<[]string>
         path	<string>
         readOnly	<boolean>
         secretFile	<string>
         secretRef	<Object>
            name	<string>
         user	<string>
      cinder	<Object>
         fsType	<string>
         readOnly	<boolean>
         secretRef	<Object>
            name	<string>
         volumeID	<string>
      configMap	<Object>
         defaultMode	<integer>
         items	<[]Object>
            key	<string>
            mode	<integer>
            path	<string>
         name	<string>
         optional	<boolean>
      csi	<Object>
         driver	<string>
         fsType	<string>
         nodePublishSecretRef	<Object>
            name	<string>
         readOnly	<boolean>
         volumeAttributes	<map[string]string>
      downwardAPI	<Object>
         defaultMode	<integer>
         items	<[]Object>
            fieldRef	<Object>
               apiVersion	<string>
               fieldPath	<string>
            mode	<integer>
            path	<string>
            resourceFieldRef	<Object>
               containerName	<string>
               divisor	<string>
               resource	<string>
      emptyDir	<Object>
         medium	<string>
         sizeLimit	<string>
      fc	<Object>
         fsType	<string>
         lun	<integer>
         readOnly	<boolean>
         targetWWNs	<[]string>
         wwids	<[]string>
      flexVolume	<Object>
         driver	<string>
         fsType	<string>
         options	<map[string]string>
         readOnly	<boolean>
         secretRef	<Object>
            name	<string>
      flocker	<Object>
         datasetName	<string>
         datasetUUID	<string>
      gcePersistentDisk	<Object>
         fsType	<string>
         partition	<integer>
         pdName	<string>
         readOnly	<boolean>
      gitRepo	<Object>
         directory	<string>
         repository	<string>
         revision	<string>
      glusterfs	<Object>
         endpoints	<string>
         path	<string>
         readOnly	<boolean>
      hostPath	<Object>
         path	<string>
         type	<string>
      iscsi	<Object>
         chapAuthDiscovery	<boolean>
         chapAuthSession	<boolean>
         fsType	<string>
         initiatorName	<string>
         iqn	<string>
         iscsiInterface	<string>
         lun	<integer>
         portals	<[]string>
         readOnly	<boolean>
         secretRef	<Object>
            name	<string>
         targetPortal	<string>
      name	<string>
      nfs	<Object>
         path	<string>
         readOnly	<boolean>
         server	<string>
      persistentVolumeClaim	<Object>
         claimName	<string>
         readOnly	<boolean>
      photonPersistentDisk	<Object>
         fsType	<string>
         pdID	<string>
      portworxVolume	<Object>
         fsType	<string>
         readOnly	<boolean>
         volumeID	<string>
      projected	<Object>
         defaultMode	<integer>
         sources	<[]Object>
            configMap	<Object>
               items	<[]Object>
                  key	<string>
                  mode	<integer>
                  path	<string>
               name	<string>
               optional	<boolean>
            downwardAPI	<Object>
               items	<[]Object>
                  fieldRef	<Object>
                     apiVersion	<string>
                     fieldPath	<string>
                  mode	<integer>
                  path	<string>
                  resourceFieldRef	<Object>
                     containerName	<string>
                     divisor	<string>
                     resource	<string>
            secret	<Object>
               items	<[]Object>
                  key	<string>
                  mode	<integer>
                  path	<string>
               name	<string>
               optional	<boolean>
            serviceAccountToken	<Object>
               audience	<string>
               expirationSeconds	<integer>
               path	<string>
      quobyte	<Object>
         group	<string>
         readOnly	<boolean>
         registry	<string>
         tenant	<string>
         user	<string>
         volume	<string>
      rbd	<Object>
         fsType	<string>
         image	<string>
         keyring	<string>
         monitors	<[]string>
         pool	<string>
         readOnly	<boolean>
         secretRef	<Object>
            name	<string>
         user	<string>
      scaleIO	<Object>
         fsType	<string>
         gateway	<string>
         protectionDomain	<string>
         readOnly	<boolean>
         secretRef	<Object>
            name	<string>
         sslEnabled	<boolean>
         storageMode	<string>
         storagePool	<string>
         system	<string>
         volumeName	<string>
      secret	<Object>
         defaultMode	<integer>
         items	<[]Object>
            key	<string>
            mode	<integer>
            path	<string>
         optional	<boolean>
         secretName	<string>
      storageos	<Object>
         fsType	<string>
         readOnly	<boolean>
         secretRef	<Object>
            name	<string>
         volumeName	<string>
         volumeNamespace	<string>
      vsphereVolume	<Object>
         fsType	<string>
         storagePolicyID	<string>
         storagePolicyName	<string>
         volumePath	<string>
```
</details>

### Proposed solution approach

**`kubectl-fields`** is the solution. This tool fixes the problem by parsing the `kubectl explain --recursive` output and returns a one-liner hierarchial structure instead of tree type hierarchy. This makes it easier for developers/users to analyze the resources hierarchy.

To find capabilities order in po.spec resources, the following command can be executed.

```console
rewanth@ubuntu:~/go/src/kubectl-fields$ kubectl fields po.spec capabilities
containers.securityContext.capabilities
initContainers.securityContext.capabilities
```

Similarly to find "version" attribute structure in `services` resource, the below command can be executed.

```console
rewanth@ubuntu:~/go/src/kubectl-fields$ kubectl fields -i svc version
apiVersion
metadata.initializers.result.apiVersion
metadata.initializers.result.metadata.resourceVersion
metadata.managedFields.apiVersion
metadata.ownerReferences.apiVersion
metadata.resourceVersion
```

## Author

[Rewanth Cool](https://www.linkedin.com/in/rewanthcool/)

## Todo

- [x] Document the basic usage for README
- [x] Build pipeline to run testcases
- [ ] Release the tool as krew package
- [ ] Write documentation according to godoc standards
- [ ] Release the tool as golang package
- [ ] Release the tool as a debian package

## Contribution and support

Ways to contribute

- Suggest a feature
- Report a bug
- Fix something and open a pull request
- Fix documentation
- Spread the word
- Like this tool? Star and fork

