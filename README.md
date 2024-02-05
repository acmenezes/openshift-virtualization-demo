# Creating Virtual Machines with SRIOV networks end to end

This is an end to end demo with a few options on how to put together SRIOV and OpenShift Virtualization together. It's still under construction. More to come...

## Contents

1. [Requirements](#requirements)

2. OpenShift Bare Metal Installation

3. Supported SRIOV network cards and configuration

4. Enabling SRIOV in the server BIOS

5. Enabling CPU virtualization in the server BIOS

6. Configuring OpenShift Worker nodes for OCP-V and SRIOV

6. Installing SRIOV operator

7. Creating SRIOV network node policies

8. Creating SRIOV networks

9. Installing OpenShift Virtualization Operator

10. Installing Hyperconverged Custom Resource

11. Creating Virtual Machines with Additional Networks

12. Exposing Virtual Machine Services

### Requirements <a name="requirements"></a>


### Configuring OpenShift Worker nodes

`oc label node ocp-pa-poc feature.node.kubernetes.io/network-sriov.capable=true`

### Installing the SRIOV Operator




### Creating SRIOV network node policies
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
  resourceName: mcx4_ens3f0np0
```
OBS: your server may boot here... Kernel modules and firmware options need to be applied.

`oc get sriovnetworknodepolicy -n openshift-sriov-network-operator` 
```
NAME                             AGE
default                          62m
policy-sriov-mcx4-enp11s0f0np0   25m
```
### Creating SRIOV networks
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
  networkNamespace: network-test
  resourceName: sriov-guests
  vlan: 10
```
### Verifying the node state:

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
    Mtu:            1500
    Name:           enp11s0f0np0
    Pci Address:    0000:0b:00.0
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