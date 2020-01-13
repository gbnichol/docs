---
title: Failed to get job complete status
weight: 20
aliases:
- /rancher/v2.x/en/installation/troubleshooting-ha/job-complete-status/
---

> #### **Important: RKE add-on install is only supported up to Rancher v2.0.8**
>
>Please use the Rancher helm chart to install HA Rancher. For details, see the [HA Install - Installation Outline]({{< baseurl >}}/rancher/v2.x/en/installation/options/helm2/#installation-outline).
>
>If you are currently using the RKE add-on install method, see [Migrating from an HA RKE Add-on Install]({{< baseurl >}}/rancher/v2.x/en/upgrades/upgrades/migrating-from-rke-add-on/) for details on how to move to using the helm chart.

To debug issues around this error, you will need to download the command-line tool `kubectl`. See [Install and Set Up kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) how to download `kubectl` for your platform.

When you have made changes to `rancher-cluster.yml`, you will have to run `rke remove --config rancher-cluster.yml` to clean the nodes, so it cannot conflict with previous configuration errors.

### Failed to deploy addon execute job [rke-user-includes-addons]: Failed to get job complete status

Something is wrong in the addons definitions, you can run the following command to get the root cause in the logging of the job:

Sorry, I couldn't figure out any other way to sumit this fix. Please do not commit my change verbatim.
I deployed a 3 node Rancher 2.3 cluster and ran into this issue.
Rancher 2.3 implements Kubernetes 1.6. Kubernets 1.6 depricates several APIs.
https://kubernetes.io/blog/2019/07/18/api-deprecations-in-1-16/
I used the documented RKE template. This template includes a depricated API.
https://raw.githubusercontent.com/rancher/rancher/master/rke-templates/3-node-certificate.yml
Update the file:
From:
  ---
  kind: Deployment
  apiVersion: extensions/v1beta1
  metadata:
    namespace: cattle-system
    name: cattle
  spec:
    replicas: 1
To:
 ---
  kind: Deployment
  apiVersion: apps/v1
  metadata:
    namespace: cattle-system
    name: cattle
  spec:
    selector:
      matchLabels:
        component: cattle
    replicas: 1

```
kubectl --kubeconfig kube_config_rancher-cluster.yml logs -l job-name=rke-user-addon-deploy-job -n kube-system
```

#### error: error converting YAML to JSON: yaml: line 9:

The structure of the addons definition in `rancher-cluster.yml` is wrong. In the different resources specified in the addons section, there is a error in the structure of the YAML. The pointer  `yaml line 9` references to the line number of the addon that is causing issues.

<b>Things to check</b>
<ul>
<ul>
<li>Is each of the base64 encoded certificate string placed directly after the key, for example: `tls.crt: LS01...`, there should be no newline/space before, in between or after.</li>
<li>Is the YAML properly formatted, each indentation should be 2 spaces as shown in the template files.</li>
<li>Verify the integrity of your certificate by running this command `cat MyCertificate | base64 -d` on Linux, `cat MyCertificate | base64 -D` on Mac OS . If any error exists, the command output will tell you.
</ul>
</ul>

#### Error from server (BadRequest): error when creating "/etc/config/rke-user-addon.yaml": Secret in version "v1" cannot be handled as a Secret

The base64 string of one of the certificate strings is wrong. The log message will try to show you what part of the string is not recognized as valid base64.

<b>Things to check</b>
<ul>
<ul>
<li>Check if the base64 string is valid by running one of the commands below:</li>

```
# MacOS
echo BASE64_CRT | base64 -D
# Linux
echo BASE64_CRT | base64 -d
# Windows
certutil -decode FILENAME.base64 FILENAME.verify
```

</ul>
</ul>

#### The Ingress "cattle-ingress-http" is invalid: spec.rules[0].host: Invalid value: "IP": must be a DNS name, not an IP address

The host value can only contain a host name, as it is needed by the ingress controller to match the hostname and pass to the correct backend.
