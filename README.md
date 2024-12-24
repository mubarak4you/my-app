# Customer Documentation: Network Security Groups (NSG) and Tagging Support

## Introduction
This guide explains how to configure **Network Security Groups (NSG)** and **Tagging** for LoadBalancers managed by OCI Native Ingress Controller (NIC). The instructions are straightforward and designed for users with minimal technical knowledge.

---

## Network Security Groups (NSG) Support

**What is it?**
Network Security Groups (NSGs) act like a virtual firewall to control traffic to and from your LoadBalancer.

### How to Use NSG with OCI NIC:
1. Use the annotation `oci-native-ingress.oraclecloud.com/network-security-group-ids` in the **IngressClass** resource.
2. Provide a comma-separated list of NSG OCIDs.
3. The LoadBalancer associated with the IngressClass will automatically be added to the specified NSGs.

### Example Configuration:
```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  annotations:
    oci-native-ingress.oraclecloud.com/network-security-group-ids: ocid1.networksecuritygroup.oc1.abc,ocid1.networksecuritygroup.oc1.xyz
```

---

## Tagging Support

**What is it?**
Tagging helps you organize and manage your resources by adding metadata in the form of **Defined Tags** and **Freeform Tags**.

### Types of Tags:
- **Defined Tags:** Predefined tags with specific namespaces and keys (requires permission).
- **Freeform Tags:** Simple key-value pairs defined by the user.

### How to Use Tagging with OCI NIC:
1. Use the annotations below in the **IngressClass** resource to apply tags.
   - `oci-native-ingress.oraclecloud.com/defined-tags`: JSON string for defined tags.
   - `oci-native-ingress.oraclecloud.com/freeform-tags`: JSON string for freeform tags.
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

### Important Notes:
- Changing a tag in the annotations triggers a reconciliation of tags on the LoadBalancer.
- Defined tags using Tag Variables are only applied if the tag does not already exist on the LoadBalancer.

---

## Default Tag Support

### What Are Default Tags?
Default Tags are automatically added by the LoadBalancer service for new LoadBalancers created by NIC version >= v1.4.0.

### Behavior of Default Tags:
1. NIC preserves Default Tags unless:
   - They are manually removed by the user from the LoadBalancer.
   - They are added to the `oci-native-ingress.oraclecloud.com/defined-tags` annotation, in which case NIC will manage them.
2. Default Tags can be overridden by specifying them during **IngressClass** creation.

### Limitations:
- Default Tag support is not available for:
  - LoadBalancers created by NIC version < v1.4.0.
  - LoadBalancers imported using the annotation `oci-native-ingress.oraclecloud.com/id`.
- Tags for these LoadBalancers must be explicitly added to the tagging annotations.

---

## Additional Resources
- [Oracle Tagging Overview](https://docs.oracle.com/en-us/iaas/Content/Tagging/Concepts/taggingoverview.htm)

If you have any questions or need assistance, contact support for further help.

