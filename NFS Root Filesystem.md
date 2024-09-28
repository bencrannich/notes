# NFS Root Filesystem

Provided a suitable kernel (many are default), Linux can use an NFS mount as a root filesystem. This is useful for:—

- diskless devices
- single-system image clusters
- prototyping where being able to alter the root filesystem from another host is convenient

These instructions are for NFSv3.

## Client kernel command line

```
root=/dev/nfs nfsroot=192.168.150.1:/srv/nfs/mars,nfsvers=3 rw ip=dhcp rootwait
```

## Server `/etc/exports`

```
/srv/nfs/mars      192.168.150.0/255.255.255.0(rw,sync,insecure,no_subtree_check,no_root_squash)
```

## Preparing a Debian root on the server

With `mmdebstrap`, the architecture of the host doesn't need to match that of the root being built.

Debian's `riscv64` port requires a distribution of `sid`.

```
apt install -y mmdebstrap qemu-user-static
bootstrap_arch=riscv64
bootstrap_dist=sid
bootstrap_path=/srv/nfs/mars
bootstrap_mirror=http://ftp.uk.debian.org/debian
mmdebstrap \
  --architectures=$bootstrap_arch \
  --include=openssh-server,nfs-common,fake-hwclock,kmod,iproute2,ntp,pciutils,usbutils,gdisk,kpartx,apt-utils,ca-certificates,gnupg,gpgv,gnupg-utils,nano,less,htop,sudo \
  --components=main,contrib,non-free-firmware \
  --variant=minbase \
  $bootstrap_dist $bootstrap_path $bootstrap_mirror
```

Once bootstrapped, you will need to create a user account, e.g., via `useradd -P $bootstrap_path/etc …`.
