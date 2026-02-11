# Virtio MMC Driver

This repository contains all the files that have been modified.

Information about the changes was taken from the following pages: [QEMU](https://gitlab.com/technothecow/qemu/-/compare/master...virtio-mmc?from_project_id=53824584&straight=false), [Linux](https://github.com/torvalds/linux/compare/master...technothecow:linux:mmc-driver)

# How to Build?

A system running Linux is required. This was tested on Ubuntu 22.04.

1) Clone the [Linux](https://github.com/technothecow/linux) and [QEMU](https://gitlab.com/technothecow/qemu) forks.

2) Build Linux:
 - Resolve [dependencies](https://www.kernel.org/doc/html/v6.8-rc5/process/changes.html).
 - Switch to the `mmc-driver` branch.
 - In the cloned directory, run `make defconfig` to generate the configuration.
 - In the `.config` file, find the line `# CONFIG_MMC is not set` and change it to `CONFIG_MMC=y`.
 - Run `make oldconfig` and answer `y` for `MMC_BLOCK`, leave blank (enter) for `MMC_BLOCK_MINORS`, answer `m` for `MMC_VIRTIO`, and `n` for everything else.
 - Start the build using `make -jN`, where `N` is the number of cores you are willing to dedicate to the build.

3) Create `initrd`
 - Install/Load `docker`.
 - Navigate to the `Docker` folder.
 - Run the command from `magic.sh`: as a result, a container with the tag `virtio-kernel-deploy` will appear in your system.
 - Navigate to the `Kernel` folder.
 - Run the command: `docker run --platform=linux/amd64 -v {path to kernel}:/build/ -v {path to kernel}:/usr/src/linux -v {path to current folder}:/boot --rm -it virtio-kernel-deploy`
 - We are interested in the resulting `initrd` and `vmlinuz` files.

4) Build QEMU:
 - Create a `build` folder in the root directory of the repository.
 - Run the following command: `{path to QEMU}/configure --target-list=x86_64-softmmu`
 - Start the build using `make -jN`, where `N` is the number of cores you are willing to dedicate to the build.

5) Prepare the image:
 - Open the root directory of the repository.
 - Run the following command: `dd if=/dev/zero of=./mmc.img bs=1M count=1`

# How to Run?

## If you built everything yourself

Go into the `build` folder and run the following command:
```bash
./qemu-system-x86_64 -M q35 -m 256 -kernel ../Kernel/vmlinuz -initrd ../Kernel/initrd -append "console=ttyS0,115200 nokaslr" -serial stdio -display none -drive file=../mmc.img,if=none,id=my_mmc -device virtio-mmc-pci,id=mmc0
```

## If you downloaded the release folder

Go into the release folder and run the following command:
```bash
./qemu-system-x86_64 -M q35 -m 256 -kernel ./vmlinuz -initrd ./initrd -append "console=ttyS0,115200 nokaslr" -serial stdio -display none -drive file=./mmc.img,if=none,id=my_mmc -device virtio-mmc-pci,id=mmc0
```

# User Documentation

To use the driver, you need to compile the kernel (version v6.6) and the emulator with the changes presented in the QEMU and Linux folders. Afterward, add the following argument when launching the emulator:

`-drive file=./mmc.img,if=none,id=my_mmc -device virtio-mmc-pci,id=mmc0`

Where `./mmc.img` is the path to the emulated card image.
