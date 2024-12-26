---
id: nsg-tagging
title: Load Balancer NSG and Tagging
---

# Ingress Network Security Groups (NSG) and Tagging

## Introduction
This guide explains how to configure **Ingress Network Security Groups (NSG)** and **Tagging** for LoadBalancers managed by OCI Native Ingress Controller (NIC).

---
## Network Security Groups (NSG) Support
Network Security Groups (NSGs) act like a virtual firewall to control traffic to and from your LoadBalancer.

#### How to configure load balancer NSG with OCI  Native Ingress Controller (NIC):
1. Use the annotation `oci-native-ingress.oraclecloud.com/network-security-group-ids` in the `IngressClass` resource.
   - Name of the NSG currently available: **MASTER-HTTPS-ALL**
   - OCID for MASTER-HTTPS-ALL NSG: `ocid1.networksecuritygroup.oc1.iad.aaaaaaaalp3w4xhgmkbov2x663hf4hke3v4rs4ftvw3nhanawx4pwkcadqda`
2. If multiple Network Security Groups (NSGs) are required, provide their OCIDs as a comma-separated list. For example `oci-native-ingress.oraclecloud.com/network-security-group-ids: ocid1.networksecuritygroup.oc1.abc,ocid1.networksecuritygroup.oc1.xyz`
3. The LoadBalancer associated with the IngressClass will automatically be added to the specified NSGs.

#### Example Configuration for one NSG on an IngressClass resource:
```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  annotations:
    oci-native-ingress.oraclecloud.com/network-security-group-ids: ocid1.networksecuritygroup.oc1.iad.aaaaaaaalp3w4xhgmkbov2x663hf4hke3v4rs4ftvw3nhanawx4pwkcadqda
```
---

## Tagging Support

Tagging helps you organize and manage your resources by adding metadata in the form of **Defined Tags** and **Freeform Tags**.

##### Types of Tags:
- **Defined Tags:** Predefined tags with specific Namespaces and Tag keys.
- **Freeform Tags:** Simple key-value pairs defined by the user.

##### How to Use Tagging with OCI NIC:
1. Use the annotations below in the **IngressClass** resource to apply tags.
   - `oci-native-ingress.oraclecloud.com/defined-tags`: JSON string for defined tags.
   - `oci-native-ingress.oraclecloud.com/freeform-tags`: JSON string for freeform tags.
2. Wrap the JSON strings in single quotes (`'`).
3. If no tags are specified, they default to `{}`.

**Defined Tags**
The tag namespace available and provided by the Oracle Team is called **Load-Balancer-tags**, and the tag keys available are:
- **Environment**
- **Owner**
- **UserID**
- **VSAD**
- **Zone**

:::note 
The following Tag Keys must be used, or the changes will not reflect.
:::

**Freeform Tags**
Any key and value can be used.

#### Example Configuration:
```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  annotations:
    oci-native-ingress.oraclecloud.com/defined-tags: '{"Load-Balancer-tags": {"Environment":
      "NONPROD", "Owner": "test.user@one.verizon.com", "UserID": "userte", "VSAD":
      "GO0V", "Zone": "GZ"}}'
    oci-native-ingress.oraclecloud.com/freeform-tags: '{"ClusterName": "oke.testcluster.np.iad.go0v"}'
    oci-native-ingress.oraclecloud.com/network-security-group-ids: ocid1.networksecuritygroup.oc1.iad.aaaaaaaalp3w4xhgmkbov2x663hf4hke3v4rs4ftvw3nhanawx4pwkcadqda
```

:::note 
Changing a tag in the annotations triggers a reconciliation of tags on the LoadBalancer.
Defined tags using [Tag Variables](https://docs.oracle.com/en-us/iaas/Content/Tagging/Tasks/usingtagvariables.htm#Using_Tag_Variables) are only applied if the tag does not already exist on the LoadBalancer.
:::

---

##### Additional Resources
- [Oracle Tagging Overview](https://docs.oracle.com/en-us/iaas/Content/Tagging/Concepts/taggingoverview.htm)
