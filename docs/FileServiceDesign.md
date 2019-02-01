## File Service Design for HPE 3PAR Docker Volume Plugin
Design decision for implementing File Services for HPE 3PAR Docker Volume plugin is captured in this wiki.

### Objectives
1. Provide a way to present a file share (via NFS/CIFS -- which are currently supported on File Persona of HPE 3PAR) 
to containerized applications
  - To support share replication, we have model VFS as a docker volume object 
  - To support snapshot/quota we have to model FileStore to a docker volume object. We are currently moving to this model, since
  it will give an ideal balance between granurality on number of the share(s) vs. Capabilities which we need to support.

2. Creation of FPG and VFS to be transparent to the user. FPG (File Provisioning Group) requires CPG as input which will 
be supplied via the config file `/etc/hpedockerplugin/hpe_file.conf`. Creation of VFS requires virtual IP Address/subnet mask. And each VFS can have multiple IP's associated with it, and this pool of ip addresses will be supplied via the config file.

3. Provide a way to update the whitelisted IP's as part of ACL definition of File Store/Share.
(implicitly when a mount happens on a docker host) 
4. A share to be allowed to mount on multiple containers on one or more hosts. This is to support `accessModes: ReadWriteMany` 
option of kubernetes PVC

5. Document the Limitations.

### Mapping of a Docker Volume to a file persona object
- Due to some constraints on setting up of Quota/ACL (Access Control List) only applicable to File Store, and due 
features like Replication/Snapshot available only at either File Store/VFS (Virtual File Server) level, we have to map the docker volume
at either File Store/VFS level.

- But since there is a inherent limitations on the number of VFS which can be created on a 3PAR system (which is currently only 16), 
we can't define the granularity of the docker volume object to the File Store level, but instead we want to come down on the hierarchy of
file persona objects to the File store. 

- Our design approach currently will be based on mapping a docker volume object to a File Store only. This may be later extend to mapping
the docker volume object to VFS in later phases of development.


### Limitations
- Updating a quota via the `docker volume create` is not currently supported due to
  1. Docker volume plugin v2 specification does'nt directly provide an update primitive , updating quota on the File Store is not feasible
  2. Other approach of providing a separate binary file / utility would be a out-of-band operation on the file share created, and possibly
  the updates done by this utility/tool also will not be automatically replicated on the kubernetes object (like PVC).
  

  