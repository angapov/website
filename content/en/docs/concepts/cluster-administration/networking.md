---
reviewers:
- thockin
title: Cluster Networking
content_type: concept
weight: 50
---

<!-- overview -->
Networking is a central part of Kubernetes, but it can be challenging to
understand exactly how it is expected to work.  There are 4 distinct networking
problems to address:

1. Highly-coupled container-to-container communications: this is solved by
   {{< glossary_tooltip text="Pods" term_id="pod" >}} and `localhost` communications.
2. Pod-to-Pod communications: this is the primary focus of this document.
3. Pod-to-Service communications: this is covered by [services](/docs/concepts/services-networking/service/).
4. External-to-Service communications: this is covered by [services](/docs/concepts/services-networking/service/).

<!-- body -->

Kubernetes is all about sharing machines between applications.  Typically,
sharing machines requires ensuring that two applications do not try to use the
same ports.  Coordinating ports across multiple developers is very difficult to
do at scale and exposes users to cluster-level issues outside of their control.

Dynamic port allocation brings a lot of complications to the system - every
application has to take ports as flags, the API servers have to know how to
insert dynamic port numbers into configuration blocks, services have to know
how to find each other, etc.  Rather than deal with this, Kubernetes takes a
different approach.

To learn about the Kubernetes networking model, see [here](/docs/concepts/services-networking/).

## How to implement the Kubernetes networking model

There are a number of ways that this network model can be implemented.  This
document is not an exhaustive study of the various methods, but hopefully serves
as an introduction to various technologies and serves as a jumping-off point.

The following networking options are sorted alphabetically - the order does not
imply any preferential status.

{{% thirdparty-content %}}

### ACI

[Cisco Application Centric Infrastructure](https://www.cisco.com/c/en/us/solutions/data-center-virtualization/application-centric-infrastructure/index.html) offers an integrated overlay and underlay SDN solution that supports containers, virtual machines, and bare metal servers. [ACI](https://www.github.com/noironetworks/aci-containers) provides container networking integration for ACI. An overview of the integration is provided [here](https://www.cisco.com/c/dam/en/us/solutions/collateral/data-center-virtualization/application-centric-infrastructure/solution-overview-c22-739493.pdf).

### Antrea

Project [Antrea](https://github.com/vmware-tanzu/antrea) is an opensource Kubernetes networking solution intended to be Kubernetes native. It leverages Open vSwitch as the networking data plane. Open vSwitch is a high-performance programmable virtual switch that supports both Linux and Windows. Open vSwitch enables Antrea to implement Kubernetes Network Policies in a high-performance and efficient manner.
Thanks to the "programmable" characteristic of Open vSwitch, Antrea is able to implement an extensive set of networking and security features and services on top of Open vSwitch.

### AWS VPC CNI for Kubernetes

The [AWS VPC CNI](https://github.com/aws/amazon-vpc-cni-k8s) offers integrated AWS Virtual Private Cloud (VPC) networking for Kubernetes clusters. This CNI plugin offers high throughput and availability, low latency, and minimal network jitter. Additionally, users can apply existing AWS VPC networking and security best practices for building Kubernetes clusters. This includes the ability to use VPC flow logs, VPC routing policies, and security groups for network traffic isolation.

Using this CNI plugin allows Kubernetes pods to have the same IP address inside the pod as they do on the VPC network. The CNI allocates AWS Elastic Networking Interfaces (ENIs) to each Kubernetes node and using the secondary IP range from each ENI for pods on the node. The CNI includes controls for pre-allocation of ENIs and IP addresses for fast pod startup times and enables large clusters of up to 2,000 nodes.

Additionally, the CNI can be run alongside [Calico for network policy enforcement](https://docs.aws.amazon.com/eks/latest/userguide/calico.html). The AWS VPC CNI project is open source with [documentation on GitHub](https://github.com/aws/amazon-vpc-cni-k8s).

### Azure CNI for Kubernetes
[Azure CNI](https://docs.microsoft.com/en-us/azure/virtual-network/container-networking-overview) is an [open source](https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md) plugin that integrates Kubernetes Pods with an Azure Virtual Network (also known as VNet) providing network performance at par with VMs. Pods can connect to peered VNet and to on-premises over Express Route or site-to-site VPN and are also directly reachable from these networks. Pods can access Azure services, such as storage and SQL, that are protected by Service Endpoints or Private Link. You can use VNet security policies and routing to filter Pod traffic. The plugin assigns VNet IPs to Pods by utilizing a pool of secondary IPs pre-configured on the Network Interface of a Kubernetes node.

Azure CNI is available natively in the [Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/configure-azure-cni).

### Calico

[Calico](https://docs.projectcalico.org/) is an open source networking and network security solution for containers, virtual machines, and native host-based workloads. Calico supports multiple data planes including: a pure Linux eBPF dataplane, a standard Linux networking dataplane, and a Windows HNS dataplane. Calico provides a full networking stack but can also be used in conjunction with [cloud provider CNIs](https://docs.projectcalico.org/networking/determine-best-networking#calico-compatible-cni-plugins-and-cloud-provider-integrations) to provide network policy enforcement.

### Cilium

[Cilium](https://github.com/cilium/cilium) is open source software for
providing and transparently securing network connectivity between application
containers. Cilium is L7/HTTP aware and can enforce network policies on L3-L7
using an identity based security model that is decoupled from network
addressing, and it can be used in combination with other CNI plugins.

### CNI-Genie from Huawei

[CNI-Genie](https://github.com/Huawei-PaaS/CNI-Genie) is a CNI plugin that enables Kubernetes to [simultaneously have access to different implementations](https://github.com/Huawei-PaaS/CNI-Genie/blob/master/docs/multiple-cni-plugins/README.md#what-cni-genie-feature-1-multiple-cni-plugins-enables) of the [Kubernetes network model](/docs/concepts/cluster-administration/networking/#the-kubernetes-network-model) in runtime. This includes any implementation that runs as a [CNI plugin](https://github.com/containernetworking/cni#3rd-party-plugins), such as [Flannel](https://github.com/coreos/flannel#flannel), [Calico](https://docs.projectcalico.org/), [Romana](https://romana.io), [Weave-net](https://www.weave.works/products/weave-net/).

CNI-Genie also supports [assigning multiple IP addresses to a pod](https://github.com/Huawei-PaaS/CNI-Genie/blob/master/docs/multiple-ips/README.md#feature-2-extension-cni-genie-multiple-ip-addresses-per-pod), each from a different CNI plugin.

### cni-ipvlan-vpc-k8s
[cni-ipvlan-vpc-k8s](https://github.com/lyft/cni-ipvlan-vpc-k8s) contains a set
of CNI and IPAM plugins to provide a simple, host-local, low latency, high
throughput, and compliant networking stack for Kubernetes within Amazon Virtual
Private Cloud (VPC) environments by making use of Amazon Elastic Network
Interfaces (ENI) and binding AWS-managed IPs into Pods using the Linux kernel's
IPvlan driver in L2 mode.

The plugins are designed to be straightforward to configure and deploy within a
VPC. Kubelets boot and then self-configure and scale their IP usage as needed
without requiring the often recommended complexities of administering overlay
networks, BGP, disabling source/destination checks, or adjusting VPC route
tables to provide per-instance subnets to each host (which is limited to 50-100
entries per VPC). In short, cni-ipvlan-vpc-k8s significantly reduces the
network complexity required to deploy Kubernetes at scale within AWS.

### Coil

[Coil](https://github.com/cybozu-go/coil) is a CNI plugin designed for ease of integration, providing flexible egress networking.
Coil operates with a low overhead compared to bare metal, and allows you to define arbitrary egress NAT gateways for external networks.

### Contrail / Tungsten Fabric

[Contrail](https://www.juniper.net/us/en/products-services/sdn/contrail/contrail-networking/), based on [Tungsten Fabric](https://tungsten.io), is a truly open, multi-cloud network virtualization and policy management platform. Contrail and Tungsten Fabric are integrated with various orchestration systems such as Kubernetes, OpenShift, OpenStack and Mesos, and provide different isolation modes for virtual machines, containers/pods and bare metal workloads.

### DANM

[DANM](https://github.com/nokia/danm) is a networking solution for telco workloads running in a Kubernetes cluster. It's built up from the following components:

   * A CNI plugin capable of provisioning IPVLAN interfaces with advanced features
   * An in-built IPAM module with the capability of managing multiple, cluster-wide, discontinuous L3 networks and provide a dynamic, static, or no IP allocation scheme on-demand
   * A CNI metaplugin capable of attaching multiple network interfaces to a container, either through its own CNI, or through delegating the job to any of the popular CNI solution like SRI-OV, or Flannel in parallel
   * A Kubernetes controller capable of centrally managing both VxLAN and VLAN interfaces of all Kubernetes hosts
   * Another Kubernetes controller extending Kubernetes' Service-based service discovery concept to work over all network interfaces of a Pod

With this toolset DANM is able to provide multiple separated network interfaces, the possibility to use different networking back ends and advanced IPAM features for the pods.

### Flannel

[Flannel](https://github.com/coreos/flannel#flannel) is a very simple overlay
network that satisfies the Kubernetes requirements. Many
people have reported success with Flannel and Kubernetes.

### Hybridnet

[Hybridnet](https://github.com/alibaba/hybridnet) is an open source CNI plugin designed for hybrid clouds which provides both overlay and underlay networking for containers in one or more clusters. Overlay and underlay containers can run on the same node and have cluster-wide bidirectional network connectivity.

### Jaguar

[Jaguar](https://gitlab.com/sdnlab/jaguar) is an open source solution for Kubernetes's network based on OpenDaylight. Jaguar provides overlay network using vxlan and Jaguar CNIPlugin provides one IP address per pod.

### k-vswitch

[k-vswitch](https://github.com/k-vswitch/k-vswitch) is a simple Kubernetes networking plugin based on [Open vSwitch](https://www.openvswitch.org/). It leverages existing functionality in Open vSwitch to provide a robust networking plugin that is easy-to-operate, performant and secure.

### Knitter

[Knitter](https://github.com/ZTE/Knitter/) is a network solution which supports multiple networking in Kubernetes. It provides the ability of tenant management and network management. Knitter includes a set of end-to-end NFV container networking solutions besides multiple network planes, such as keeping IP address for applications, IP address migration, etc.

### Kube-OVN

[Kube-OVN](https://github.com/alauda/kube-ovn) is an OVN-based kubernetes network fabric for enterprises. With the help of OVN/OVS, it provides some advanced overlay network features like subnet, QoS, static IP allocation, traffic mirroring, gateway, openflow-based network policy and service proxy.

### Kube-router

[Kube-router](https://github.com/cloudnativelabs/kube-router) is a purpose-built networking solution for Kubernetes that aims to provide high performance and operational simplicity. Kube-router provides a Linux [LVS/IPVS](https://www.linuxvirtualserver.org/software/ipvs.html)-based service proxy, a Linux kernel forwarding-based pod-to-pod networking solution with no overlays, and iptables/ipset-based network policy enforcer.

### L2 networks and linux bridging

If you have a "dumb" L2 network, such as a simple switch in a "bare-metal"
environment, you should be able to do something similar to the above GCE setup.
Note that these instructions have only been tried very casually - it seems to
work, but has not been thoroughly tested.  If you use this technique and
perfect the process, please let us know.

Follow the "With Linux Bridge devices" section of
[this very nice tutorial](https://blog.oddbit.com/2014/08/11/four-ways-to-connect-a-docker/) from
Lars Kellogg-Stedman.

### Multus (a Multi Network plugin)

Multus is a Multi CNI plugin to support the Multi Networking feature in Kubernetes using CRD based network objects in Kubernetes.

Multus supports all [reference plugins](https://github.com/containernetworking/plugins) (eg. [Flannel](https://github.com/containernetworking/cni.dev/blob/main/content/plugins/v0.9/meta/flannel.md), [DHCP](https://github.com/containernetworking/plugins/tree/master/plugins/ipam/dhcp), [Macvlan](https://github.com/containernetworking/plugins/tree/master/plugins/main/macvlan)) that implement the CNI specification and 3rd party plugins (eg. [Calico](https://github.com/projectcalico/cni-plugin), [Weave](https://github.com/weaveworks/weave), [Cilium](https://github.com/cilium/cilium), [Contiv](https://github.com/contiv/netplugin)). In addition to it, Multus supports [SRIOV](https://github.com/hustcat/sriov-cni), [DPDK](https://github.com/Intel-Corp/sriov-cni), [OVS-DPDK & VPP](https://github.com/intel/vhost-user-net-plugin) workloads in Kubernetes with both cloud native and NFV based applications in Kubernetes.

### OVN4NFV-K8s-Plugin (OVN based CNI controller & plugin)

[OVN4NFV-K8S-Plugin](https://github.com/opnfv/ovn4nfv-k8s-plugin) is OVN based CNI controller plugin to provide cloud native based Service function chaining(SFC), Multiple OVN overlay networking, dynamic subnet creation, dynamic creation of virtual networks, VLAN Provider network, Direct provider network and pluggable with other Multi-network plugins, ideal for edge based cloud native workloads in Multi-cluster networking

### NSX-T

[VMware NSX-T](https://docs.vmware.com/en/VMware-NSX-T/index.html) is a network virtualization and security platform. NSX-T can provide network virtualization for a multi-cloud and multi-hypervisor environment and is focused on emerging application frameworks and architectures that have heterogeneous endpoints and technology stacks. In addition to vSphere hypervisors, these environments include other hypervisors such as KVM, containers, and bare metal.

[NSX-T Container Plug-in (NCP)](https://docs.vmware.com/en/VMware-NSX-T/2.0/nsxt_20_ncp_kubernetes.pdf) provides integration between NSX-T and container orchestrators such as Kubernetes, as well as integration between NSX-T and container-based CaaS/PaaS platforms such as Pivotal Container Service (PKS) and OpenShift.

### OVN (Open Virtual Networking)

OVN is an opensource network virtualization solution developed by the
Open vSwitch community.  It lets one create logical switches, logical routers,
stateful ACLs, load-balancers etc to build different virtual networking
topologies.  The project has a specific Kubernetes plugin and documentation
at [ovn-kubernetes](https://github.com/openvswitch/ovn-kubernetes).

### Weave Net from Weaveworks

[Weave Net](https://www.weave.works/products/weave-net/) is a
resilient and simple to use network for Kubernetes and its hosted applications.
Weave Net runs as a [CNI plug-in](https://www.weave.works/docs/net/latest/cni-plugin/)
or stand-alone.  In either version, it doesn't require any configuration or extra code
to run, and in both cases, the network provides one IP address per pod - as is standard for Kubernetes.

## {{% heading "whatsnext" %}}

The early design of the networking model and its rationale, and some future
plans are described in more detail in the
[networking design document](https://git.k8s.io/community/contributors/design-proposals/network/networking.md).
