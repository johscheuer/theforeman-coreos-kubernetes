# Supported version of Kubernetes:
1.5.3

# Snippets
You have to create a host parameter cloudconfig and insert the specific snippet name to use the different snippets with the same provisioning template. You can find the default CoreOS templates for foreman at [github](https://github.com/theforeman/community-templates/tree/develop/coreos), which should be already included if you use foreman > 1.8.

## Import templates
The easiest way to import these templates into your foreman is using [foreman_templates](https://github.com/theforeman/foreman_templates).
For the latest version 1.14, install the following package before running the foreman-rake command below:
~~~~
yum install tfm-rubygem-foreman_templates
~~~~
I used a rake task for different prefix:

```Bash
foreman-rake templates:sync repo="https://github.com/johscheuer/theforeman-coreos-kubernetes.git" prefix="k8s_" dirname="/kubernetes"
```

If you want to use different prefixes you have to adjust the names of the snippets.

## Global host Parameters
- kubernetes-binary-server: server which provides the Kubernetes binaries to download. Default is: [https://storage.googleapis.com/kubernetes-release](https://storage.googleapis.com/kubernetes-release)
- install-disk: e.g. `/dev/sda`
- mirror-server: your mirror server if you don't want to download CoreOS every time you install a node
- overlay_network: 10.0.0.0/16 used by flannel.
- ssh_authorized_keys: your ssh key(s) used to login into the nodes. You can specify multiple keys by separating them with a ,
- k8s_release: Kubernetes release version which you like to deploy e.g. `v.1.5.3`

## Adjust the CoreOS Template
Replace the following line 


with this content (the spaces are important) or create a second template for this:

```yaml
      <%= snippet @host.params['coreos_cloudconfig'] %>
```

# Kubernetes
With these snippets you can either deploy a standalone Kubernetes which means that all components of Kubernetes are running on one single node. One the the hand you can deploy a single master node which contains etcd + Kubernetes master components and as many minion nodes as you want.

## kubernetes master_cloudconfig
This snippet installs a kubernetes-master which allows you to interact with Kubernetes and deploy your services.

Host parameters:
- coreos_cloudconfig: k8s_master_cloudconfig

## kubernetes minion_cloudconfig
This snippet installs a kubernetes-minion.

Host parameters:
- coreos_cloudconfig: k8s_minion_cloudconfig
- k8s_master: The Kubernetes master e.g. `172.24.1.148`  (without `http://` this gets added automatically).

## kubernetes standalone_cloudconfig
This snippet installs the kube-master and a kube-minion on the same node.

Host parameters:
- coreos_cloudconfig: k8s_standalone_cloudconfig

# Tools
For the deployment the following tools were used:
- [foreman_templates](https://github.com/theforeman/foreman_templates): This plugin will sync the contents of the Foreman Community Templates repository (or a git repo of your choice) to your local Foreman instance
- [Flannel](https://github.com/coreos/flannel): An overlay Network to support Kubernetes IP-per-Service.
- [Kubernetes](https://github.com/GoogleCloudPlatform/kubernetes): Container Cluster Manager.
- [CoreOS](https://github.com/coreos): Linux for Massive Server Deployments.
- [Cloud-validator](https://coreos.com/validate): Useful tool for checking the syntax of your cloud-init
