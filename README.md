# Snippets

You have to create a host parameter cloudconfig and insert the specific snippet name to use the different snippets with the same provisioning template.

# Import templates

The easiest way to import these templates into your foreman is using [foreman_templates](https://github.com/theforeman/foreman_templates). 

I used 2 rake tasks for different prefixes:

`foreman-rake templates:sync repo="THIS REPO" prefix="kubernetes" dirname="/kubernetes"`
`foreman-rake templates:sync repo="THIS REPO" prefix="etcd" dirname="/etcd"`

If you want to use different prefixes you have to adjust the names of the snippets.

## Global host Parameters

* kubernetes-binary-server: server which provides the kubernetes, flannel and skydns binaries to download
* install-disk: e.g. /dev/sda
* mirror-server: your mirror server if you don't want to download CoreOS everytime you install a node
* overlay_network: 10.0.0.0/16 used by flannel.
* ssh_authorized_keys: your ssh key(s) used to login into the nodes. You can specify multiple keys by seperating them with a ,
* domain-name: domain name for your skydns domain Default = local.domain.com

## CoreOS Cloud Config

With this template you can deploy a simple CoreOS cluster with a discovery URL. You can get a new discovery URL when you curl [https://discovery.etcd.io/new](https://discovery.etcd.io/new).

A description about the host parameters are at [GitHub](https://github.com/theforeman/community-templates/tree/master/coreos)

## etcd Cloud Config

With this template you are able to deploy an etcd cluster. Also this template inserts configuration into etcd for Flannel (an overlay network) and SkyDNS (dynamic DNS resolver with etcd as backend).

Host parameters:

* cloudconfig: etcd_cloudconfig
* cluster_active_size: How many nodes are active at the ectd election ([Raft Consensus](https://github.com/coreos/raft)) the rest is shadering. The optimal cluster size is (n / 2) - 1 -> should be an odd number.
* peer_address: The first node which is started. Is the entry point for every new joining node. 

## Kubernetes snippets

You should define a parent host group for your kubernetes deployment with the following parameters:

* etcd_servers: Your etcd servers seperated with a "," e.g. http://172.24.1.150:4001,http://172.24.1.140

## kubernetes master_cloudconfig

This snippet installs a kubernetes-master which allows you to interact with kubernetes and deploy your services.

Host parameters:

* cloudconfig: kubernetes master_cloudconfig
* fleet_endpoint: Any of your etcd nodes e.g. http://172.24.1.150:4002 this is needed for the Kube-register, which automatically adds new Kubernetes-minions.
* etcd_servers: if you have a seperate etcd-cluster (which I preferr) you have to insert the etcd nodes e.g. "http://172.24.1.150:4001,http://172.24.1.140:4001,http://172.24.1.105:4001"

## kubernetes minion_cloudconfig

This snippet installs a kubernetes-minion, flannel, skydns and starts an cAdvisor instance. You can access the cAdviser web-interface via. Node-IP:4194.

Host parameters:

* cloudconfig: kubernetes minion_cloudconfig
* kubernetes_servers: The kubernetes server e.g. http://172.24.1.148:8080.
* etcd_servers: if you have a seperate etcd-cluster (which I preferr) you have to insert the etcd nodes e.g. "http://172.24.1.150:4001,http://172.24.1.140:4001,http://172.24.1.105:4001"

## kubernetes standalone_cloudconfig

This snippet installs the kube-master and a kube-minion on the same node.

Host parameters:

* cloudconfig: kubernetes standalone_cloudconfig

# Tools

For the deployment the following tools were used:

* [foreman_templates](https://github.com/theforeman/foreman_templates): This plugin will sync the contents of the Foreman Community Templates repository (or a git repo of your choice) to your local Foreman instance
* [Flannel](https://github.com/coreos/flannel): An overlay Network to support Kubernetes IP-per-Service.
* [Kubernetes](https://github.com/GoogleCloudPlatform/kubernetes): Container Cluster Manager.
* [skydns](https://github.com/skynetservices/skydns): A dynamic DNS Service with etcd backend.
* [kube-register](https://github.com/kelseyhightower/kube-register): Register Kubernetes Kubelet machines with the Kubernetes API server using Fleet data.
