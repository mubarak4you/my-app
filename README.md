---
id: expose-app
title: Expose application
---

### Step 1: Certificate management for secure application load balancing
:::note
Application teams should verify that they are authorized to access the Oracle Certificates service from the OCI console
:::
#### Acquiring a digital certificate in Verizon.

  - Option 1: Using OneCloud portal

    - Go to [OneCloud portal](https://onecloud.verizon.com/).
    - Choose "SSL Certificates" from the AWS menu under Tools.
    - Choose the CN name for your certificate appropriately.
    - Use subject alternative names (SAN) if requried while requesting for digital certificates.
    - Certificate package is sent to your email or it can be downloaded at the time of generation from Digicert CA.

  - Option 2: Using SAML Single Sign-On access to DigiCert's CertCentral
    - Please follow this [Digicert start to finish](https://oneconfluence.verizon.com/x/9yYjHg) document

#### Create a generic secret from the Certificate.
  - Extract the certificate package and verify if below files exist.
      - DigiCertCA2.pem
      - your-certificate-name.pem
      - private_key

  - Use below command to create a secret in your application namespace.
      ```plaintext
      kubectl create secret generic <secret-name> --type=kubernetes.io/tls --from-file=ca.crt=DigiCertCA2.pem --from-file=tls.crt=<your-certificate-name>.pem --from-file=tls.key=private_key
      ```

  - This kubernetes secret will be used in your ingress resource to provision a secure TLS endpoint.

#### Renewing the Certificate
  - The secret created above is by using a certificate issued by a third-party CA DigiCert.
  - We provide this secret into an Ingress resource during Ingress creation step.
  - The OCI native ingress controller creates an Imported certificate in the Oracle Certificates service upon creation of an Ingress resource in the Ingress creation step.
  :::note 
  OCI Certificate of type imported cannot be automatically rotated by Oracle.
  :::  
  - Renewing the certificate would be maintained by application teams outside of Oracle cloud with the Digicert CA.
  - If the imported certificate expires , it can always be deleted and new certificate can be imported using the steps above.

#### Deleting the Certificate
  - OCI Imported certificates can be deleted from the OCI console by simply going to the OCI Certificates service locating and deleting a specific certificate from your compartment.
  :::note
  OCI native ingress controller WILL NOT DELETE a certificate upon deletion of the ingressClass and ingressClassParameters resources, your certificate stays in OCI certificates till it is deleted explicitly.
  :::
 

### Step-2: Create and Apply IngressClassParameters and IngressClass resources
1. Create an IngressClassParameters resource as below in your application namespace to specify details of the OCI load balancer to create for the OCI native ingress controller.
   This is the namespace scoped custom resource.

```yaml
apiVersion: "ingress.oraclecloud.com/v1beta1"
kind: IngressClassParameters
metadata:
  name: native-ic-params
  namespace: <app-namespace>
spec:
  compartmentId: <"compartment Id of the App team">
  loadBalancerName: <"Load balancer name">
  isPrivate: true
```
  - name: icp-name is the name of the new IngressClassParameters resource.
  - namespace: "ns-name" is the name of the namespace in which to create the new IngressClassParameters resource.
  - compartmentId: "compartment-ocid" is the OCID of the App team's compartment where the OCI load balancer will be created.
  - loadBalancerName: "lb-name" is the name given to the new load balancer.
  - isPrivate: true (Only private internal load balancers are allowed). 

2. Create an IngressClass resource as below. OCI native ingress controller reads this resource and requests for the creation of the OCI load balancer. IngressClass is a cluster scope resource. The OCI load balancer should be created at this point.
```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: native-ic-ingress-class
  annotations:
    ingressclass.kubernetes.io/is-default-class: "true"
    oci-native-ingress.oraclecloud.com/network-security-groups: "ocid1.networksecuritygroup.oc1.iad.aaaaaaaalp3w4xhgmkbov2x663hf4hke3v4rs4ftvw3nhanawx4pwkcadqda"
spec:
  controller: oci.oraclecloud.com/native-ingress-controller
  parameters:
    scope: Namespace
    namespace: <namespace of the ingressClassParameters resource>
    apiGroup: ingress.oraclecloud.com
    kind: ingressclassparameters
    name: <ingressclassparam name>
```
  - name: "ic-name" is the name of the new IngressClass resource.
  - ingressclass.kubernetes.io/is-default-class: "true|false" indicates whether this IngressClass is the default IngressClass to use if an Ingress resource does not explicitly specify an ingressClassName. It is recommended to specify a default IngressClass. 
  - controller: oci.oraclecloud.com/native-ingress-controller specifies the OCI native ingress controller as the ingress controller to use.
  - namespace: "ns-name" is the name of the namespace containing the parameters to use when .spec.parameters.scope is set to Namespace.
  - name: "icp-name" is the name of the IngressClassParameters resource that specifies details of the OCI load balancer to create for the OCI native ingress controller. 

#### Network Security Groups (NSG)

Network Security Groups (NSGs) function as virtual firewalls, enabling precise control over ingress and egress traffic for your Load Balancers.

#### Configuring Load Balancer NSGs with OCI Native Ingress Controller (NIC)
To configure NSGs for Load Balancers associated with the OCI NIC, use the following annotation in the `IngressClass` resource:

**Annotation:**
`oci-native-ingress.oraclecloud.com/network-security-group-ids`

##### Key Details:
- **Example NSG Name:** `MASTER-HTTPS-ALL`
- **OCID for `MASTER-HTTPS-ALL` NSG:** `ocid1.networksecuritygroup.oc1.iad.aaaaaaaalp3w4xhgmkbov2x663hf4hke3v4rs4ftvw3nhanawx4pwkcadqda`

##### Steps:
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

#### Tagging

Tagging allows for efficient organization and management of OCI resources by applying metadata through **Defined Tags** or **Freeform Tags**.

##### Types of Tags:
1. **Defined Tags**: Predefined tags with specific namespaces and keys.
2. **Freeform Tags**: User-defined key-value pairs.

##### Using Tags with OCI Native Ingress Controller
To apply tags, add the following annotations to the `IngressClass` resource:

- **Defined Tags Annotation:** `oci-native-ingress.oraclecloud.com/defined-tags`
- **Freeform Tags Annotation:** `oci-native-ingress.oraclecloud.com/freeform-tags`

:::note 
Wrap JSON strings for tags in single quotes (`'`).
If no tags are specified, they default to `{}`.
:::

#### Available Defined Tags:
The Oracle team provides a namespace called **Load-Balancer-tags** and the following keys are available within this namespace. 
- **Environment**
- **Owner**
- **UserID**
- **VSAD**
- **Zone**

:::note
These keys must be used for the changes to take effect
:::

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

##### Behavior:
- Changes to tags in the annotations trigger a reconciliation of tags on the Load Balancer.
- **Defined Tags** utilizing [Tag Variables](https://docs.oracle.com/en-us/iaas/Content/Tagging/Tasks/usingtagvariables.htm#Using_Tag_Variables) are applied only if the tag is not already present on the Load Balancer.

---

##### Additional Resources
- [Oracle Tagging Overview](https://docs.oracle.com/en-us/iaas/Content/Tagging/Concepts/taggingoverview.htm)

---

### Step-3: Create and Apply Ingress resource in your application namespace
Create an Ingress resource as below in your application namespace.  OCI certificate will be created from the kubernetes secret upon creation of this ingress resource. This step also creates a TLS listener, routing policy & backend connectivity to the application.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: <Ingress-name>
  annotations:
    oci-native-ingress.oraclecloud.com/backend-tls-enabled: "false"
    oci-native-ingress.oraclecloud.com/listener-port: "443"
    oci-native-ingress.oraclecloud.com/healthcheck-protocol: "HTTP"
    oci-native-ingress.oraclecloud.com/healthcheck-path: "/"
    oci-native-ingress.oraclecloud.com/healthcheck-force-plaintext: "true"
spec:
  ingressClassName: <Ingress-class-name>
  tls:
  - hosts:
      - <your-certificate-FQDN-name>
    secretName: <generic secret name>
  rules:
    - host: "your certificate FQDN name"
      http:
        paths:
          - path: /
            pathType: Exact
            backend:
              service:
                name: <service name>
                port:
                  number: <service port>
```

:::note 
Oracle's native ingress controller only supports backend services of type NodePort
::: 

### Step-4: Register OCI application loadbalancer with the Verizon DNS.
  - Get the ip-address from the ingress resource created in your application namespace. If you do not see an ip-address on your ingress resource, do not proceed and create a support ticket.
  - Go to dns.verizon.com.
  - Open a service request "Add Alias Only".
  - Plug in the ip-address from your ingress resource. 
  - DNS system will check for your IP address and will ask you to enter a SAN ( subject alternative name)
  - Make sure your subject alternative name matches the CN name of your certificate.
  - Save and create the DNS record resolving to the ip-address of your internal OCI load balancer.
  - Use TLD (top level domain) as ebiz.verizon.com for non-production load balancers and verizon.com for production load balancers.

### Step-5: Sanity check for a functional application load balancer.
Below resources should be populated on the left hand side menu of your OCI loadbalancer details page.

   - Backend sets.
   - Routing policies.
   - Listeners.
   - Certificates.
   - MASTER-HTTPS-ALL network security group should be attached to the OCI load balancer.
   - Overall health checks should be "green".

### Step-6: Deleting OCI load balancer resources is recommended before attempting to delete the OKE cluster. 
   - Delete the Ingress resource first from application namespace.
   - Delete the IngressClass resource after the ingress Resource.
   - Delete the IngressClassParameters resource.
   - Wait for 2-3 mins for OCI native ingress controller to sync & cleanup resources for you.

:::note 
In case cluster is deleted before removing the LB resources your application load balancer is Orphaned. To stop incurring the load balancing costs please create a support ticket with the container platform team to delete an orphaned OCI load balancer.
:::

