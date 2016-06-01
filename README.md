# machinekit-dkms
DKMS modules for machinekit

machinekit uses the occasional kernel driver, but including those in a package build is painful.
The best solution I could come up with is making those drivers [DKMS](https://en.wikipedia.org/wiki/Dynamic_Kernel_Module_Support) debian packages.
The resulting debs contain the source of the
drivers and instructions on how to build on the target platform for every kernel installed. A kernel upgrade
will cause an automatic rebuild of DKMS-packaged drivers for the new kernel. This requires the installation
supports out-of-tree builds (kernel headers installed, module build working - see note at bottom).

## Initial build of the debian packages:

````bash

sudo apt-get install dkms

git clone git://github.com/mhaberler/machinekit-dkms.git
cd machinekit-dkms/drivers
# tell dkms about the new modules:
dkms add hm2reg_uio/0.0.1
dkms add adcreg/0.0.1

ä these steps are only 'on target' (where you actually need the kmods built)
# install them (builds kernel modules):
dkms install hm2reg_uio/0.0.1
dkms install adcreg/0.0.1

# at this point, the modules are built for the installed kernels
# and can be modprobed:
root@mksocfpga:~# modprobe hm2reg_uio
root@mksocfpga:~# modprobe adcreg
root@mksocfpga:~# lsmod
root@mksocfpga:/usr/src# lsmod
Module                  Size  Used by
hm2reg_uio              3045  0
adcreg                  1167  0
autofs4                22992  1

# for packaging, build the source-only debian packages
# this will NOT incur a kernel module build:
dkms mkdeb -m adcreg -v 0.0.1  --source-only
dkms mkdeb -m hm2reg_uio -v 0.0.1 --source-only

# this will leave debs like so:

root@mksocfpga:~/machinekit-dkms# find /var/lib/dkms/|grep '\.deb$'
/var/lib/dkms/adcreg/0.0.1/deb/adcreg-dkms_0.0.1_all.deb
/var/lib/dkms/hm2reg_uio/0.0.1/deb/hm2reg-uio-dkms_0.0.1_all.deb

# inspect the result - source only:

machinekit-dkms/drivers# dpkg -c /var/lib/dkms/hm2reg_uio/0.0.1/deb/hm2reg-uio-dkms_0.0.1_all.deb
drwxr-xr-x root/root         0 2016-04-18 10:06 ./
drwxr-xr-x root/root         0 2016-04-18 10:06 ./usr/
drwxr-xr-x root/root         0 2016-04-18 10:06 ./usr/share/
drwxr-xr-x root/root         0 2016-04-18 10:06 ./usr/share/hm2reg_uio-dkms/
-rwxr-xr-x root/root      9090 2016-04-18 10:06 ./usr/share/hm2reg_uio-dkms/postinst
drwxr-xr-x root/root         0 2016-04-18 10:06 ./usr/src/
drw-r-xr-x root/root         0 2016-04-18 09:00 ./usr/src/hm2reg_uio-0.0.1/
-rw-r--r-- root/root      8953 2016-04-18 10:01 ./usr/src/hm2reg_uio-0.0.1/hm2reg_uio.c
-rw-r--r-- root/root       136 2016-04-18 10:01 ./usr/src/hm2reg_uio-0.0.1/dkms.conf
-rw-r--r-- root/root        22 2016-04-18 10:01 ./usr/src/hm2reg_uio-0.0.1/Makefile


# upload those to the apt repo.

# to verify everyhing worked fine, remove the drivers
# NB this removes the above debs, so save them elsewhere before
dkms remove adcreg/0.0.1 --all
dkms remove hm2reg_uio/0.0.1 --all

# At this point all traces of our manual install are gone,
# so the modules cannot be loaded anymore:

root@mksocfpga:~/machinekit-dkms# modprobe adcreg
modprobe: FATAL: Module adcreg not found.

# to verify everyhing's fine, we install the debs:
dpkg -i  adcreg-dkms_0.0.1_all.deb  hm2reg-uio-dkms_0.0.1_all.deb
...

# and the drivers are back:

root@mksocfpga:~# modprobe hm2reg_uio
root@mksocfpga:~# modprobe adcreg
root@mksocfpga:~# lsmod
Module                  Size  Used by
hm2reg_uio              3029  0
adcreg              	3029  0
autofs4                21861  1

`````

I have added these debs to the jessie stream on deb.machinekit.io
so you should be able to install those like so:

````bash
root@mksocfpga:~/machinekit-dkms# apt-cache search adcreg
...
adcreg-dkms - adcreg-uio driver in DKMS format.

root@mksocfpga:~/machinekit-dkms# apt-cache search hm2reg
hm2reg-uio-dkms - hm2reg-uio driver in DKMS format.

apt-get update
apt-get install adcreg-dkms hm2reg-uio-dkms 
`````





