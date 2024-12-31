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
 

### Step-2: Create and Apply IngressClassParameters resource
Create an IngressClassParameters resource as below in your application namespace to specify details of the OCI load balancer to create for the OCI native ingress controller.
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

### Step-3: Create and Apply IngressClass resource
Create an IngressClass resource as below. OCI native ingress controller reads this resource and requests for the creation of the OCI load balancer. IngressClass is a cluster scope resource. The OCI load balancer should be created at this point.
```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: native-ic-ingress-class
  annotations:
    ingressclass.kubernetes.io/is-default-class: "true"
    oci-native-ingress.oraclecloud.com/network-security-groups: "ocid1.networksecuritygroup.oc1.iad.aaaaaaaalp3w4xhgmkbov2x663hf4hke3v4rs4ftvw3nhanawx4pwkcadqda"
    oci-native-ingress.oraclecloud.com/network-security-group-ids: "ocid1.networksecuritygroup.oc1.iad.aaaaaaaalp3w4xhgmkbov2x663hf4hke3v4rs4ftvw3nhanawx4pwkcadqda"
    oci-native-ingress.oraclecloud.com/defined-tags: '{"Load-Balancer-tags": {"Environment":
      "NONPROD", "Owner": "test.user@one.verizon.com", "UserID": "userte", "VSAD":
      "GO0V", "Zone": "GZ"}}'
    oci-native-ingress.oraclecloud.com/freeform-tags: '{"ClusterName": "oke.testcluster.np.iad.go0v"}'
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
  - oci-native-ingress.oraclecloud.com/network-security-group-ids: is to add Load Balancers associated to the IngressClass to the NSGs. Multiple NSGs can be separated with a comma. 
    - The OCID for the NSG avaiable in each region.
    - Non Prod US East = ocid1.networksecuritygroup.oc1.iad.aaaaaaaalp3w4xhgmkbov2x663hf4hke3v4rs4ftvw3nhanawx4pwkcadqda
    - Non Prod US West = xxxxxxxxxxxxxxxxx
  - oci-native-ingress.oraclecloud.com/defined-tags: 
    - xxxxx must be string provided 
  - oci-native-ingress.oraclecloud.com/freeform-tags: 

### Step-4: Create and Apply Ingress resource in your application namespace
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

### Step-5: Register OCI application loadbalancer with the Verizon DNS.
  - Get the ip-address from the ingress resource created in your application namespace. If you do not see an ip-address on your ingress resource, do not proceed and create a support ticket.
  - Go to dns.verizon.com.
  - Open a service request "Add Alias Only".
  - Plug in the ip-address from your ingress resource. 
  - DNS system will check for your IP address and will ask you to enter a SAN ( subject alternative name)
  - Make sure your subject alternative name matches the CN name of your certificate.
  - Save and create the DNS record resolving to the ip-address of your internal OCI load balancer.
  - Use TLD (top level domain) as ebiz.verizon.com for non-production load balancers and verizon.com for production load balancers.

### Step-6: Sanity check for a functional application load balancer.
Below resources should be populated on the left hand side menu of your OCI loadbalancer details page.

   - Backend sets.
   - Routing policies.
   - Listeners.
   - Certificates.
   - MASTER-HTTPS-ALL network security group should be attached to the OCI load balancer.
   - Overall health checks should be "green".

### Step-7: Deleting OCI load balancer resources is recommended before attempting to delete the OKE cluster. 
   - Delete the Ingress resource first from application namespace.
   - Delete the IngressClass resource after the ingress Resource.
   - Delete the IngressClassParameters resource.
   - Wait for 2-3 mins for OCI native ingress controller to sync & cleanup resources for you.

:::note 
In case cluster is deleted before removing the LB resources your application load balancer is Orphaned. To stop incurring the load balancing costs please create a support ticket with the container platform team to delete an orphaned OCI load balancer.
:::

