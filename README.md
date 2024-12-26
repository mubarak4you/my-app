### Load Balancer Network Security Groups (NSG) and Tagging

---

## Introduction
This document provides detailed guidance on configuring **Ingress Network Security Groups (NSG)** and **Tagging** for Load Balancers managed by the OCI Native Ingress Controller (NIC).

---

## Network Security Groups (NSG)

Network Security Groups (NSGs) function as virtual firewalls, enabling precise control over ingress and egress traffic for your Load Balancers.

### Configuring Load Balancer NSGs with OCI Native Ingress Controller (NIC)
To configure NSGs for Load Balancers associated with the OCI NIC, use the following annotation in the `IngressClass` resource:

**Annotation:**
`oci-native-ingress.oraclecloud.com/network-security-group-ids`

#### Key Details:
- **Example NSG Name:** `MASTER-HTTPS-ALL`
- **OCID for `MASTER-HTTPS-ALL` NSG:** `ocid1.networksecuritygroup.oc1.iad.aaaaaaaalp3w4xhgmkbov2x663hf4hke3v4rs4ftvw3nhanawx4pwkcadqda`

#### Steps:
1. Add the OCID of the desired NSG to the annotation in your `IngressClass` configuration.
2. If multiple NSGs are required, list their OCIDs as a comma-separated string.
3. Once configured, the Load Balancer associated with the specified `IngressClass` will be automatically added to the designated NSGs.

#### Example Configuration:
Below is an example configuration for associating a Load Balancer with a single NSG:

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  annotations:
    oci-native-ingress.oraclecloud.com/network-security-group-ids: ocid1.networksecuritygroup.oc1.iad.aaaaaaaalp3w4xhgmkbov2x663hf4hke3v4rs4ftvw3nhanawx4pwkcadqda
```

For multiple NSGs, the annotation would look like this:
```yaml
oci-native-ingress.oraclecloud.com/network-security-group-ids: ocid1.networksecuritygroup.oc1.abc,ocid1.networksecuritygroup.oc1.xyz
```

---

## Tagging

Tagging allows for efficient organization and management of OCI resources by applying metadata through **Defined Tags** or **Freeform Tags**.

### Types of Tags:
1. **Defined Tags**: Predefined tags with specific namespaces and keys.
2. **Freeform Tags**: User-defined key-value pairs.

### Using Tags with OCI Native Ingress Controller
To apply tags, add the following annotations to the `IngressClass` resource:

- **Defined Tags Annotation:** `oci-native-ingress.oraclecloud.com/defined-tags`
- **Freeform Tags Annotation:** `oci-native-ingress.oraclecloud.com/freeform-tags`

#### Important Notes:
- Wrap JSON strings for tags in single quotes (`'`).
- If no tags are specified, they default to `{}`.

#### Available Defined Tags:
The Oracle team provides a namespace called **Load-Balancer-tags**. The following keys are available within this namespace and must be used for the changes to take effect:
- **Environment**
- **Owner**
- **UserID**
- **VSAD**
- **Zone**

#### Example Configuration:
Below is an example of applying both defined and freeform tags:

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

#### Behavior:
- Changes to tags in the annotations trigger a reconciliation of tags on the Load Balancer.
- **Defined Tags** utilizing [Tag Variables](https://docs.oracle.com/en-us/iaas/Content/Tagging/Tasks/usingtagvariables.htm#Using_Tag_Variables) are applied only if the tag is not already present on the Load Balancer.

---

## Additional Resources
- [Oracle Tagging Overview](https://docs.oracle.com/en-us/iaas/Content/Tagging/Concepts/taggingoverview.htm)

---

