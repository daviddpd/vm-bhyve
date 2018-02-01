# WARNING : INITAL PUBLIC COMMITS

This code path is alpha quality, and might break other things. 
Implementation is incomplete - not all datastore operations are supported. 

## vm-bhyve - iscsi support 

Developed at iXsystems, for managing IT intrasture (in Bhyve VMs) as to 
provide the backing store from the open source project FreeNAS or the 
enterprise product TrueNAS. 

However, iSCSI datastore support should be platform independant (eventually), 
the zvol block storage and all the iSCSI configuration is done via RESTful APIs in
an external package.  So, any storage appliance that has an API, and has a 
wrapper script to be CLI argument compatible with the vm-bhyve call, should be possible.

### Features

Things that are working:
 * listing ISCSI datastores (block storage)
 * creating/deleting "datasets" (the vm-bhyve text configuration files)
 * renaming dataset (the vm-bhyve text configuration files and directory)
 * "make_zvol", which means "vm create -d iscsi ..." works.

Not Yet Implemented
 * vm destroy - doesn't call the APIs to delete the iSCSI block storage.
 * vm snapshot (of iscsi datastore)
 * vm rollback
 * vm clone
 * vm image create
 * vm image provision
 * vm image list
 * vm image destroy
 * CHAP login/secret is configured in the external API helpers, not via vm-bhyve 

### Issues

 * FreeBSD bhyve vm's must use extend logical block size of 512.
 * zpools on the virtal block device in the guest, on the iSCSI lun, may fail mountroot with Error 5.  This seemed to go away, for no apparent reason.
   Possible related to: https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=208882


### Configuration 

Configuration options are subject to change - hopefully in order to simplify things!

The RFC 4173 iSCSI URI syntax isn't really correct of our use case.  

So for this utility, we'll use a simplified version. 
    iscsi:<servername>:<basename>:<filesystem_path>
    basename is the iqn of the target without the targetname/lun. 
    filesystem_path is the path to all the configs. Probably want this to be an NFS share

And iSCSI host can be the default datastore or added as secondary datastores, in all cases
the following needs to be in the /etc/rc.conf.

    vm_enable="YES"
    vm_datastore_iscsi="YES"

This is the configuration for the external tools to control the iSCSI API (this syntax will likely change)
These are specific for the FreeNAS/TrueNAS integration:
    
    vm_iscsi_binpath="/usr/home/dpd/ixnas-api/bin"
    vm_iscsi_bin_create_zol="create-zvol.py"
    vm_iscsi_bin_create_target="create-iscsi.py"

Then for default datastore, the vm_dir spec looks like this:

    vm_dir="iscsi:nas1:iqn.2005-08.com.ixsystems.iad1"
    vm_iscsi_config="/vmconfig"

vm_iscsi_config - is a directory path for where the plan text vm-bhyve configs, lockfiles and logs go.
If you want to be able to "failover" a VM from one hypervisors to another, this directory should likely
be an NFS share.   Users are responsible for this setup. 

The datastore spec for adding an iSCSI datastore as secondary is basically the same:

    vm datastore add iscsi1 iscsi:nas1:iqn.2005-08.com.ixsystems.iad1

If you don't specify a config directory, then the default "vm_iscsi_config" one will be used.  
Optionally, you could store each set of configs in different places (different NFS servers?) 
especially if leveraging two different iSCSI servers.  This might be useful durning consolidations ,  
migrations, or for maintenance.

    vm datastore add iscsi1 iscsi:nas1:iqn.2005-08.com.ixsystems.iad1:/vmconfig

