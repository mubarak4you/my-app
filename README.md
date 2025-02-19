**Subject:** Update on GKE Pipeline Issue  

Hi Team,  

At first, the issue with this pipeline not running was due to the **Policy Controller not being configured** on the project. As a result, the GKE pipeline failed to create the cluster with the following error:  

```
The policy controller feature must be enabled for the fleet project.
```

However, it seems that someone from your team has since enabled the **Policy Controller**. One of my team members then restarted the **plan phase** of the pipeline, and that job completed successfully.  

Unfortunately, during the **apply phase**, we encountered a new error:  

```
Error creating Topic: googleapi: Error 400: Cloud Pub/Sub did not have the necessary permissions configured to support this operation. Please verify that the service account service-200322364942@gcp-sa-pubsub.iam.gserviceaccount.com was granted the Cloud KMS CryptoKey Encrypter/Decrypter role for the project containing the CryptoKey resource projects/vz-it-pr-d0sv-vsadkms-0/locations/global/keyRings/vz-it-pr-kraid/cryptoKeys/vz-it-pr-kms-k3cv.
```

This issue is **likely related to how the project was provisioned by the governance team**. Could you please check and confirm if the required permissions are in place for the service account?  

Let us know if you need further details.  

Thanks,  
Mubarak

