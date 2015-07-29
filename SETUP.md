If you want to check the download:

```
md5sum 
76c4ddfab149b54cea61235d9832bc80
sha256sum 
037e517c63f947f1b659dd013b6095a8ce08428583b346cc058ff06ed5db092b
```

# Import

## VirtualBox

File -> import -> ova

## VMWare Fusion

File -> Import -> select ova

## KVM
```
tar -xf zfslearn.ova
qemu-img convert -O raw zfslearn-disk1.vmdk zfslearn.img
virt-install -r 2046 --vcpus=2 -n zfs-learn -f zfslearn.img --boot hd
```

# Connecting

How networking works in your environment is specific to your hypervisor
and a few other factors, so I cant tell you exactly how it will work
for you. You want to use the hypervisor to get the IP address
of the guest then SSH into it.

# logging in

Both the username and passowrd are openzfs. This user has sudo powers,
since most of the actions in zfs need root permisions, you can either
```sudo su -``` or ```sudo``` each command.
