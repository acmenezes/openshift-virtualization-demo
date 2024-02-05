# Creating Virtual Machines with SRIOV networks end to end

This is an end to end demo with a few options on how to put together SRIOV and OpenShift Virtualization together. It's still under construction. More to come...

## Contents

1. [Requirements](#requirements)

2. [OpenShift Bare Metal Installation](#bare-metal)

3. [Supported SRIOV network cards and configuration](#sriov-cards)

4. [Enabling SRIOV and CPU virtualization in the server BIOS](#enabling-sriov-bios)

5. [Configuring OpenShift Worker nodes for OCP-V and SRIOV](#ocp-worker-sriov-config)

6. [Installing SRIOV operator](#sriov-operator-install)

7. [Creating SRIOV network node policies](#sriov-network-node-policies)

8. [Creating SRIOV networks](#sriov-network-node-policies)

9. [Installing OpenShift Virtualization Operator](#ocp-virtualization-operator-install)

10. [Creating Virtual Machines with Additional Networks](#vm-additional-networks)

11. [Exposing Virtual Machine Services](#exposing-vm-services)

### 1. Requirements <a name="requirements"></a>

The first and most important element is a server with a few characteristics: cpu virtualization technology, enough memory, cpu and storage to receive a single node OpenShift cluster (for OCP 4.14: 16Gb of ram, 8 vCPUs and 120 GB of storage). It should have at least one free PCI slot for an SRIOV card. For reference about a single node OpenShift you may check [here](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.14/html/installing/installing-on-a-single-node#preparing-to-install-sno). Those are the minimum hardware requirements for the platform itself. We also need resources for virtual machines that will be running on top of OCP virtualization. Those will vary depending on how many VMs will be running and how much resources they will consume. For a simple demo even a 32Gb of ram with 16vCPUs should work. [Check the bare metal installation below on item 2.](#bare-metal) We opted for the easiest install method: the assisted installer.

A second and also essential requirement is the SRIOV capable network card. For the purpose of this specific demo we used two different SRIOV capable network interface cards or as also sometimes called HCAs (Host Channel Adapters). [Check on supported cards below on section 3.](#sriov-cards)

### 2. OpenShift Bare Metal Installation <a name="bare-metal"></a>

For our demo we used in single node OpenShift option (SNO) and we installed using the OpenShift assisted installer. [Here](https://access.redhat.com/documentation/en-us/assisted_installer_for_openshift_container_platform/2024/html/installing_openshift_container_platform_with_the_assisted_installer/index) you can check it's full documentation for all options it offers and all of the requirements that must be in place before starting a new OpenShift installation. With a free customer account you can use a trial version of OpenShift for up to 60 days which enough time to develop a proof of concept or a demo project.

### 3. Supported SRIOV network cards and configuration <a name="sriov-cards"></a>

[Here](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.14/html/networking/hardware-networks#supported-devices_about-sriov) you can find the list of the sriov devices supported for OpenShift as well as the vendor and device IDs for them sparing us from checking on the host system.


### 4. Enabling SRIOV and CPU virtualization in the server BIOS <a name="enabling-sriov-bios"></a>

The BIOS software depends on the server you have below I present an example on what that might look like.

<!-- TODO: Insert images here. -->

### 5. Configuring OpenShift Worker nodes for OCP-V and SRIOV <a name="ocp-worker-sriov-config"></a>

One of the steps that is commonly done is to also label the nodes for the sriov network node policies to take effect on.

Here is how you can label your node:

`oc label node < your node name > feature.node.kubernetes.io/network-sriov.capable=true`

If you want to check if the label is actually in your node run:

`oc describe node < your node name > | grep feature.node.kubernetes.io/network-sriov.capable=true`

To understand more on how to manage OpenShift nodes please check [here](https://docs.openshift.com/container-platform/4.14/nodes/nodes/nodes-nodes-viewing.html)

### 6. Installing SRIOV operator <a name="sriov-operator-install"></a>

Please refer to [Installing the sriov network operator](https://docs.openshift.com/container-platform/4.14/networking/hardware_networks/installing-sriov-operator.html) docs.
### 7. Creating SRIOV network node policies <a name="sriov-network-node-policies"></a>

The sriov network node policy automates the configuration of the sriov devices available on the each worker node. It's important to label your nodes accordingly. In the example below the field nodeSelector will allow the manifest to be only applied on the nodes that contain the label `feature.node.kubernetes.io/network-sriov.capable: 'true'`. Other labels may be added for specific purposes and different device configurations.

To create an sriov network node policy custom resource in OpenShift we need to find the vendor and device IDs for the sriov network interface card and those will be used by the nicSelector in order to find the right device. You have that information from the supported devices page but here is how you can discover them in the system if needed:

`oc get nodes`

<!-- TODO: insert output here -->

`oc debug node/my-node-name`

<!-- TODO: insert output here -->

`chroot /host /bin/bash`

<!-- TODO: insert output here -->

`lspci -nnv | grep -i ethernet`

<!-- TODO: insert output here -->

```
apiVersion: sriovnetwork.openshift.io/v1
kind: SriovNetworkNodePolicy
metadata:
  name: policy-sriov-mcx4-enp11s0f0np0
  namespace: openshift-sriov-network-operator
spec:
  deviceType: vfio-pci
  isRdma: false
  nicSelector:
    deviceID: '1015'
    pfNames:
      - 'enp11s0f0np0#0-31'
    vendor: '15b3' 
  nodeSelector:
    feature.node.kubernetes.io/network-sriov.capable: 'true'
  numVfs: 64
  priority: 97
  resourceName: sriov-device
```
OBS: your server may boot here... Kernel modules and firmware options need to be applied.

`oc get sriovnetworknodepolicy -n openshift-sriov-network-operator` 
```
NAME                             AGE
default                          62m
policy-sriov-mcx4-enp11s0f0np0   25m
```

For more in depth details please refer to [configure an sriov device in OpenShift](https://docs.openshift.com/container-platform/4.14/networking/hardware_networks/configuring-sriov-device.html) docs.


### 8. Creating SRIOV networks <a name="sriov-network-node-policies"></a>

The SRIOV network custom resource automates the creation of a network attachment definition for the guest VM namespace (or project in OpenShift). The network attachment definition (NAD for short) describes the behavior of additional networks to be added to the VMs. The NAD relies on [multus-cni](https://github.com/k8snetworkplumbingwg/multus-cni) to create additional networks. For more information on sriov network check [here](https://docs.openshift.com/container-platform/4.14/networking/hardware_networks/configuring-sriov-net-attach.html)

The example below creates an additional network within vlan 10 for the VMs to attach to. This additional network will be available in the `sriov-guests` namespace and based on the `sriov-device` configured previously in the sriov-network-node-policy. The ip address management is of type DHCP and presupposes that vlan 10 has a DHCP server attached to it in order for the VMs to configure their ip addresses using DHCP.

```
apiVersion: sriovnetwork.openshift.io/v1
kind: SriovNetwork
metadata:
  name: mcx4-enp11s0f0np0-vlan10-dhcp
  namespace: openshift-sriov-network-operator
spec:
  ipam: |-
    {
      "ipam": {
        "type": "dhcp"
      }
    }
  networkNamespace: sriov-guests
  resourceName: sriov-device
  vlan: 10
```
Verifying the node state:

`oc get sriovnetworknodestate -n openshift-sriov-network-operator`
```
NAME         AGE
ocp-pa-poc   25m
```
`oc describe sriovnetworknodestate -n openshift-sriov-network-operator`
```
Name:         ocp-pa-poc
Namespace:    openshift-sriov-network-operator
Labels:       <none>
Annotations:  <none>
API Version:  sriovnetwork.openshift.io/v1
Kind:         SriovNetworkNodeState
Metadata:
  Creation Timestamp:  2024-01-19T15:50:36Z
  Generation:          1
  Managed Fields:
    API Version:  sriovnetwork.openshift.io/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:ownerReferences:
          .:
          k:{"uid":"3877cf25-6821-413e-aa09-da457b4bf601"}:
      f:spec:
        .:
        f:dpConfigVersion:
    Manager:      sriov-network-operator
    Operation:    Update
    Time:         2024-01-19T15:50:36Z
    API Version:  sriovnetwork.openshift.io/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:status:
        .:
        f:interfaces:
        f:syncStatus:
    Manager:      sriov-network-config-daemon
    Operation:    Update
    Subresource:  status
    Time:         2024-01-19T15:51:08Z
  Owner References:
    API Version:           sriovnetwork.openshift.io/v1
    Block Owner Deletion:  true
    Controller:            true
    Kind:                  SriovNetworkNodePolicy
    Name:                  default
    UID:                   3877cf25-6821-413e-aa09-da457b4bf601
  Resource Version:        2389428
  UID:                     ff85bd15-9113-4919-a382-0482ab453d26
Spec:
  Dp Config Version:  33a88d0f5d0b0cb4d2c43fb7846b87df
Status:
  Interfaces:
    Device ID:      1015
    Driver:         mlx5_core
    E Switch Mode:  legacy
    Link Speed:     10000 Mb/s
    Link Type:      ETH
    Mac:            e8:eb:d3:13:06:16
    Mtu:            1500vm-additional-networks
    Totalvfs:       8
    Vendor:         15b3
    Device ID:      1015
    Driver:         mlx5_core
    E Switch Mode:  legacy
    Link Speed:     -1 Mb/s
    Link Type:      ETH
    Mac:            e8:eb:d3:13:06:17
    Mtu:            1500
    Name:           enp11s0f1np1
    Pci Address:    0000:0b:00.1
    Totalvfs:       8
    Vendor:         15b3
  Sync Status:      Succeeded
Events:             <none>

```
For more in depth details on how to configure sriov networks please refer to configuring an [SR-IOV ethernet network attachment](https://docs.openshift.com/container-platform/4.14/networking/hardware_networks/configuring-sriov-net-attach.html) in OpenShift docs.

### 9. Installing OpenShift Virtualization Operator <a name="ocp-virtualization-operator-install"></a>



### 10. Creating Virtual Machines with Additional Networks <a name="vm-additional-networks"></a>




### 11. Exposing Virtual Machine Services <a name="exposing-vm-services"></a>