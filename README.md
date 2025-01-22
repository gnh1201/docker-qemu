# gnh1201/docker-qemu

```console
$ touch /home/jsmith/hda.qcow2
$ docker run -it --rm \
	--device /dev/kvm \
	--name qemu-container \
	-v /home/jsmith/hda.qcow2:/tmp/hda.qcow2 \
	-e QEMU_HDA=/tmp/hda.qcow2 \
	-e QEMU_HDA_SIZE=100G \
	-e QEMU_CPU=4 \
	-e QEMU_RAM=4096 \
	-v /home/jsmith/downloads/debian.iso:/tmp/debian.iso:ro \
	-e QEMU_CDROM=/tmp/debian.iso \
	-e QEMU_BOOT='order=d' \
	-e QEMU_PORTS='2375 2376' \
	gnh1201/qemu:6.1    # or `tianon/qemu:native`
```

Note: port 22 will always be mapped (regardless of the contents of `QEMU_PORTS`).

For supplying additional arguments, use a command of `start-qemu <args>`. For example, to use `-curses`, one would `docker run ... tianon/qemu start-qemu -curses`.

For UEFI support, [the `ovmf` package](https://packages.debian.org/sid/ovmf) is installed, which can be utilized most easily by supplying `--bios /usr/share/ovmf/OVMF.fd`.

By default, this image will use [QEMU's user-mode networking stack](https://wiki.qemu.org/Documentation/Networking#User_Networking_.28SLIRP.29), which means if you want ping/ICMP working, you'll likely need to also include something like `--sysctl net.ipv4.ping_group_range='0 2147483647'` in your container runtime settings.

The `native` variants for `amd64` only contain `qemu-system-x86_64` -- the non-`native` variants contain QEMU compiled for a variety of target CPUs.

## Build Docker image

```console
$ git clone https://github.com/gnh1201/docker-qemu
$ cd docker-qemu
$ docker build -t gnh1201/qemu:6.1 .
```

or use [Docker Hub](https://hub.docker.com/repository/docker/gnh1201/qemu)

```console
$ docker pull gnh1201/qemu:6.1
```

## For non-native

Note: Non-native virtualization does not support KVM acceleration (Do not use `--device /dev/kvm`).

### ARM
```console
$ touch /hdimages/armhf.qcow2
$ docker run -it --rm \
    --name qemu-container-arm \
    --user="$(id --user):$(id --group)" \
    -v /hdimages/armhf.qcow2:/tmp/hda.qcow2 \
    -v /bootimages/initrd-debian11-armhf.gz:/tmp/initrd.gz \
    -v /bootimages/vmlinuz-debian11-armhf:/tmp/vmlinuz \
    -e QEMU_HDA=/tmp/hda.qcow2 \
    -e QEMU_HDA_SIZE=20G \
    -e QEMU_CPU=1 \
    -e QEMU_RAM=1024 \
    -v /cdimages/debian-11.1.0-armhf-netinst.iso:/tmp/debian.iso:ro \
    -e QEMU_CDROM=/tmp/debian.iso \
    -e QEMU_BOOT='order=d' \
    -e QEMU_PORTS='2375 2376' \
    -e QEMU_ARCH='arm' \
    -e QEMU_MACHINE='virt' \
    -e QEMU_KERNEL=/tmp/vmlinuz \
    -e QEMU_INITRD=/tmp/initrd.gz \
    gnh1201/qemu:6.1
```

### MIPS
```console
$ touch /hdimages/mips64el.qcow2
$ docker run -it --rm \
    --name qemu-container-mips64el \
    --user="$(id --user):$(id --group)" \
    -v /hdimages/mips64el.qcow2:/tmp/hda.qcow2 \
    -v /bootimages/debian11-mips64el-malta/initrd.gz:/tmp/initrd.gz \
    -v /bootimages/debian11-mips64el-malta/vmlinuz-5.10.0-9-5kc-malta:/tmp/vmlinuz \
    -e QEMU_HDA=/tmp/hda.qcow2 \
    -e QEMU_HDA_SIZE=20G \
    -e QEMU_CPU=1 \
    -e QEMU_RAM=1024 \
    -v /cdimages/debian-11.1.0-mips64el-netinst.iso:/tmp/debian.iso:ro \
    -e QEMU_CDROM=/tmp/debian.iso \
    -e QEMU_BOOT='order=d' \
    -e QEMU_PORTS='2375 2376' \
    -e QEMU_ARCH='mips64el' \
    -e QEMU_MACHINE='malta' \
    -e QEMU_CPUMODEL='5KEc' \
    -e QEMU_KERNEL=/tmp/vmlinuz \
    -e QEMU_INITRD=/tmp/initrd.gz \
    -e QEMU_APPEND='console=ttyS0' \
    gnh1201/qemu:6.1
```

If ARM or MIPS is selected, `vmlinuz`(kernel image) and `initrd` are required. Please refer to this article and proceed.

   * [ARM/non-EFI](https://gist.github.com/KunoiSayami/934c7690dcf357f42537562dbdf90b56)
   * [MIPS/non-EFI](https://gist.github.com/bradfa/46ceff759a0cf9f392cc069c4f0f095a)
   * [ARM/EFI](https://gist.github.com/ag88/163a7c389af0c6dcef5a32a3394e8bac)

### Manually bootloader activation

When using the non-native (e.g. ARM, MIPS) platform, the bootloader must be extracted manually after the installation is completed.

```console
$ qemu-img convert armhf.qcow2 armhf.raw
$ sudo mkdir /mnt/tmp
$ sudo mount -o loop,offset=$((2048 * 512)) armhf.raw /mnt/tmp
$ cd /mnt/tmp/boot
```

Then, copy the `initrd` and `vmlinuz` files to an external directory. And load using `QEMU_KERNEL` and `QEMU_INITRD` variables.

### Manually root activation

When using the non-native (e.g. ARM, MIPS) platform, it was confirmed that the settings were not reflected in `/etc/passwd` and `/etc/shadow`. Please refer to the two links below and set them up manually.

  * [Manually generate password for /etc/shadow](https://unix.stackexchange.com/questions/81240/manually-generate-password-for-etc-shadow)
  * [How to mount qcow2 disk image on Linux](https://www.xmodulo.com/mount-qcow2-disk-image-linux.html)

### This is a long-term supported version
This repository is a long-term supported version for Windows and multi-platform support. For information on the latest QEMU version, see the [tianon/docker-qemu](https://github.com/tianon/docker-qemu) repository.

## Join the community
- ActivityPub [@catswords_oss@catswords.social](https://catswords.social/@catswords_oss)
- [Join Catswords OSS on Microsoft Teams (teams.live.com)](https://teams.live.com/l/community/FEACHncAhq8ldnojAI)
- [Join Catswords OSS #docker-qemu on Discord (discord.gg)](https://discord.gg/zPfx2VrcRs)

## Report abuse
- [GitHub Security Advisories (gnh1201/docker-qemu)](https://github.com/gnh1201/docker-qemu/security)
- abuse@catswords.net
