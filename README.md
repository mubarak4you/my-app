apiVersion: v1
kind: PersistentVolume
metadata:
  name: fileserver-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: csi-gce-filestore-cmek
  csi:
    driver: filestore.csi.storage.gke.io
    volumeHandle: <YOUR_FILESTORE_INSTANCE_ID>/volumes/<YOUR_VOLUME_NAME>


driver: filestore.csi.storage.gke.io
    volumeHandle: "modeInstance/FILESTORE_INSTANCE_LOCATION/FILESTORE_INSTANCE_NAME/FILESTORE_SHARE_NAME"
    
    
gke-sample
Resource type
Instance
Status
 Ready
Overview
Backups (0)
Snapshots (0)
An instance is a fully managed network-attached storage system you can use with your Google Compute Engine and Kubernetes Engine instances. Learn more 

Import data to this instance from a Google Cloud Storage bucket by creating a transfer job in Storage Transfer Service. Learn More 
NFS mount point
Used to mount this file share on a linux client VM. Run the mount command with the following remote target on the VM's local directory. Learn more 

192.168.224.66:/vol1
Capacity
1 TiB
Location
us-east4
VPC network
projects/vz-it-np-exhv-sharedvpc-228116/global/networks/shared-np-east
Reserved IP range
192.168.224.64/26
Protocol
NFSv3
Creation time
Feb 6, 2025, 2:06:04 PM UTC-06:00
Encryption
Customer-managed
Encryption key
vz-it-np-kms-go0v

projects/vz-it-np-d0sv-vsadkms-0/locations/us-east4/keyRings/vz-it-np-kr-gts/cryptoKeys/vz-it-np-kms-go0v

Labels
Tags

Access control
2 policies set
IP address or range
Access
Mount 'sec='
164.118.64.95/32	Admin	
sys
164.118.64.96/32	Admin	
sys
Deletion protection
Disabled
Summary
Service tier
ENTERPRISE
Location
us-east4
