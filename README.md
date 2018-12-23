# Local Block PersistentVolumes

This repo explores ways to use local volumes to form RAID0 arrays and use multiple underlying disks together to get better capacity and performance.

## Methods 

1. Expose block volumes, make raid array in scylla pod
    * **Status:** FAILED
    * **Dir:** `block-pv`
2. Setup raid array in host, expose Filesystem Volume to scylla pod
    * **Status:** SUCCESS
    * **Dir:** `host-raid`