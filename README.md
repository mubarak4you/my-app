[alabimu@10-74-128-76 nfilestore]$ gcloud filestore instances describe gke-sample --location=us-east4
createTime: '2025-02-06T20:06:04.046044926Z'
fileShares:
- capacityGb: '1024'
  name: vol1
  nfsExportOptions:
  - accessMode: READ_WRITE
    ipRanges:
    - 164.118.64.95/32
    - 164.118.64.96/32
    squashMode: NO_ROOT_SQUASH
kmsKeyName: projects/vz-it-np-d0sv-vsadkms-0/locations/us-east4/keyRings/vz-it-np-kr-gts/cryptoKeys/vz-it-np-kms-go0v
name: projects/vz-it-np-go0v-dev-gketst-0/locations/us-east4/instances/gke-sample
networks:
- connectMode: PRIVATE_SERVICE_ACCESS
  ipAddresses:
  - 192.168.224.66
  network: projects/vz-it-np-exhv-sharedvpc-228116/global/networks/shared-np-east
  reservedIpRange: 192.168.224.64/26
performanceLimits:
  maxIops: '12000'
  maxReadIops: '12000'
  maxReadThroughputBps: '125829120'
  maxWriteIops: '4000'
  maxWriteThroughputBps: '104857600'
protocol: NFS_V3
satisfiesPzi: true
state: READY
tier: ENTERPRISE
