# test
On both nodes of the cluster, set the use_lvmlockd configuration option in the /etc/lvm/lvm.conf file to use_lvmlockd=1
 
Adding Logical Volumes to LVM Shared Volume Groups

[1]

[root@node ~]# pvcreate /dev/sdb  ## in one node 

[root@node ~]# vgcreate --shared lvmsharedvg  /dev/sdb ## in one node 

[A]

[root@node ~]# vgchange --lock-start lvmsharedvg   ## on all nodes 

[1]

[root@node ~]# lvcreate --activate sy -L500G -n lv1 lvmsharedvg ## Only on one node
 
[2]

lvmdevices --adddev /dev/sdb
 
 
[root@node ~]# pcs property set no-quorum-policy=freeze

[root@node ~]# yum install gfs2-utils

[root@node ~]# mkfs.gfs2 -t examplecluster:examplegfs2  -j 3 -J 128 /dev/lvmsharedvg/lv1   ## in one node 

[root@node ~]# pcs resource create dlm ocf:pacemaker:controld  op monitor interval=30s on-fail=fence --group=lock_group

[root@node ~]# pcs resource create lvmlockd ocf:heartbeat:lvmlockd op monitor interval=30s on-fail=fence --group=lock_group

[root@node ~]# pcs resource clone lock_group interleave=true

[root@node ~]# pcs resource create sharedlv1 LVM-activate vgname=lvmsharedvg lvname=lv1 activation_mode=shared vg_access_mode=lvmlockd  --group=LVMshared_group

[root@node ~]# pcs resource clone LVMshared_group interleave=true

[root@node ~]# pcs constraint order start  lock_group-clone then LVMshared_group-clone

[root@node ~]# pcs constraint colocation add  LVMshared_group-clone with lock_group-clone

[root@node ~]# pcs resource create clusterfs Filesystem  device=/dev/lvmsharedvg/lv1 directory=/data fstype=gfs2 on-fail=fence --group=LVMshared_group

 
On both nodes of the cluster, set the use_lvmlockd configuration option in the /etc/lvm/lvm.conf file to use_lvmlockd=1.






 lvm_activate resource:	

	LV_TCH

	pcs resource update LV_TCH vgname=lvmsharedvg lvname=lv1   --group=VG_GFS2_02

	LV_CDM

	pcs resource update LV_CDM vgname=lvmsharedvg lvname=lv1  --group=VG_GFS2_01
 
Filesystem resources 

	GFS2_TCH

	pcs resource update GFS2_TCH Filesystem device=/dev/lvmsharedvg/lv1 directory=/data  --group=VG_GFS2_02

	GFS2_CDM

	pcs resource update GFS2_CDM Filesystem device=/dev/lvmsharedvg/lv1 directory=/data  --group=VG_GFS2_01
 
