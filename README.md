Since this issue is related to the service account permissions required for creating the cluster, this is not something we can resolve on our end. We recommend that you create a GCP support ticket with the governance team to investigate and resolve the following issue:

The service account (service-200322364942@gcp-sa-pubsub.iam.gserviceaccount.com) does not have the necessary Cloud KMS CryptoKey Encrypter/Decrypter role for the project containing the CryptoKey resource. This is causing the pipeline to fail during the apply phase with the following error:

Error creating Topic: googleapi: Error 400: Cloud Pub/Sub did not have the necessary permissions configured to support this operation. Please verify that the service account service-200322364942@gcp-sa-pubsub.iam.gserviceaccount.com was granted the Cloud KMS CryptoKey Encrypter/Decrypter role for the project containing the CryptoKey resource projects/vz-it-pr-d0sv-vsadkms-0/locations/global/keyRings/vz-it-pr-kraid/cryptoKeys/vz-it-pr-kms-k3cv.

The governance team should verify that the correct IAM permissions have been assigned to this service account to allow the pipeline to complete successfully.

Please let us know once the ticket has been created or if you need any additional details.

Thanks,
Mubarak
