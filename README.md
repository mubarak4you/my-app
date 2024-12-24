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

#### How to Use NSG with OCI NIC:
1. Use the annotation `oci-native-ingress.oraclecloud.com/network-security-group-ids` in the `IngressClass` resource.
2. Provide a comma-separated list of NSG OCIDs.
3. The LoadBalancer associated with the IngressClass will automatically be added to the specified NSGs.

#### Example Configuration:
```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  annotations:
    oci-native-ingress.oraclecloud.com/network-security-group-ids: ocid1.networksecuritygroup.oc1.abc,ocid1.networksecuritygroup.oc1.xyz
```



## Tagging Support

Tagging helps you organize and manage your resources by adding metadata in the form of **Defined Tags** and **Freeform Tags**.

##### Types of Tags:
- **Defined Tags:** Predefined tags with specific Namespaces and Tag keys.
- **Freeform Tags:** Simple key-value pairs defined by the user.

##### How to Use Tagging with OCI NIC:
1. Use the annotations below in the **IngressClass** resource to apply tags.
 `oci-native-ingress.oraclecloud.com/defined-tags`: JSON string for defined tags.
 `oci-native-ingress.oraclecloud.com/freeform-tags`: JSON string for freeform tags.
2. Wrap the JSON strings in single quotes (`'`).
3. If no tags are specified, they default to `{}`.

### Example Configuration:
```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  annotations:
    oci-native-ingress.oraclecloud.com/defined-tags: '{"namespace-1": {"key1": "value1", "key2": "value2"}, "namespace-2": {"key1": "value1"}}'
    oci-native-ingress.oraclecloud.com/freeform-tags: '{"key1": "value1", "key2": "value2"}'
```

##### Important Notes:
- Changing a tag in the annotations triggers a reconciliation of tags on the LoadBalancer.
- Defined tags using Tag Variables are only applied if the tag does not already exist on the LoadBalancer.
---

##### Default Tag Support
Default Tags are automatically added by the LoadBalancer service for new LoadBalancers created by NIC version >= v1.4.0.

##### Behavior of Default Tags:
1. NIC preserves Default Tags unless:
   - They are manually removed by the user from the LoadBalancer.
   - They are added to the `oci-native-ingress.oraclecloud.com/defined-tags` annotation, in which case NIC will manage them.
2. Default Tags can be overridden by specifying them during **IngressClass** creation.

##### Limitations:
- Default Tag support is not available for:
  - LoadBalancers created by NIC version < v1.4.0.
  - LoadBalancers imported using the annotation `oci-native-ingress.oraclecloud.com/id`.
- Tags for these LoadBalancers must be explicitly added to the tagging annotations.
---

##### Additional Resources
- [Oracle Tagging Overview](https://docs.oracle.com/en-us/iaas/Content/Tagging/Concepts/taggingoverview.htm)





Make change to How to Use NSG with OCI NIC
on number 1

Add the following information below to it

name of the NSG currenlty avaiable = MASTER-HTTPS-ALL
OCID = ocid1.networksecuritygroup.oc1.iad.aaaaaaaalp3w4xhgmkbov2x663hf4hke3v4rs4ftvw3nhanawx4pwkcadqda


Add to How to Use Tagging with OCI NIC:

For defined-tags the tag namespaces avaiable and provided by Oracle Team is called Load-Balancer-tags 
and the Tag Keys avaialbe are
Environment
Owner
UserID
VSAD
Zone

The following Tag Keys have to be used or else the changes will not reflect.

For freeform-tags any key and value can be used.





Use this below as the example configuration.

apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  annotations:
    oci-native-ingress.oraclecloud.com/defined-tags: '{"Load-Balancer-tags": {"Environment":
      "NONPROD", "Owner": "mubarak.alabi@one.verizon.com", "UserID": "alabimu", "VSAD":
      "GO0V", "Zone": "GZ"}}'
    oci-native-ingress.oraclecloud.com/freeform-tags: '{"ClusterName": "oke.main.np.iad.go0v"}'
    oci-native-ingress.oraclecloud.com/id: ocid1.loadbalancer.oc1.iad.aaaaaaaa2kxwekos4f63typn3cctvubwjykzirjdkg7bhzknstv6thm72v3q
    

