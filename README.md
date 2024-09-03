## Overview

**A lightweight solution for running multiple operating systems with minimal setup and configuration based on Virtualization framework.**

This application allows you to run guest operating systems such as macOS and Linux, including distributions like Debian, Fedora, and more.

## Features

- **Automatic macOS Installation:** Downloads the latest macOS image from Apple’s official resources and fully automates the installation.
- **Linux Installation:** Linux can be installed using ISO images.
- **Shared Directories:** Mounts shared folders between host and guest systems.
- **Display Scaling:** Supports multiple screen resolutions with dynamic scaling.
- **Rosetta 2 Support:** Enables running x86-64 binaries in guest systems.
- **Configurable Resources:** Options to adjust RAM, virtual CPUs, and disk size.
- **Device and Peripheral Support:** Compatibility with essential devices.

## Getting the App

This app is available for macOS on Apple Silicon, starting from version 14.0.

[![download-on-the-macos-app-store](./resources/macos-app-store.svg)](https://apps.apple.com/us/app/visor-virtual-machine-manager/id6642665426)

## Virtual Machine Configuration

### macOS

There are two methods for installing macOS as a guest OS.

1. **Automatic Installation:** This method automatically downloads the Restore Image from Apple’s servers.
2. **Manual Installation from the local Restore Image:** If a Restore Image in `.ipsw` format is already downloaded to the local disk, this file can be selected during the installation process.

### Linux

For Linux distributions, installation is done manually using a bootable ISO image. The image file should be prepared before configuration. The installation can be done in either text or graphical mode.

### Directory Sharing

To make directories shared from the host available on the guest operating system, follow the configuration steps below. Once these steps are completed, the guest system can access and use the shared directories.

1. Define the directory path on the host and assign a tag. The tag typically defaults to the folder name but can be customized.
2. On the guest system, mount the shared directory using the assigned tag. This setup ensures that the directory is accessible for the guest OS.

#### macOS Guests

For macOS guests, the shared directory is automatically mounted under `/Volumes/My Shared Files`. You don't need to perform any additional steps. Simply navigate to the `My Shared Files` directory to access the mounted content.

#### Linux Guests

For Linux guests, you need to manually mount the shared directory using the assigned tag:

```bash
sudo mount -t virtiofs "shared-tag" /mnt/shared-tag
```

Replace `"shared-tag"` with the tag you configured and `/mnt/shared-tag` with your desired mount point. To automatically mount this directory at system startup, configure a systemd service:

```ini
[Unit]
Description=Automount Shared Directory

[Service]
Type=oneshot
ExecStart=mount -t virtiofs "shared-tag" /mnt/shared-tag
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

### Rosetta Support for Linux

Enabling Rosetta in the Configurator allows you to run x86-64 binaries on a Linux guest by sharing Rosetta from the macOS host using VirtIO file system. Additional post-installation steps are required on the guest system to properly configure the environment.

#### Share the Rosetta Directory

The directory containing the Rosetta binaries is already configured by the Configurator. The remaining step is to ensure it is mounted on the guest operating system:

```bash
sudo mkdir -p /mnt/rosetta
sudo mount -t virtiofs "rosetta" /mnt/rosetta
```

#### Automate the Mounting Process

To ensure the directory is mounted at every system startup, add a systemd configuration:

```bash
sudo touch /etc/systemd/system/mount-rosetta.service
sudo nano /etc/systemd/system/mount-rosetta.service
```

Add the following content to the service file:

```ini
[Unit]
Description=Mount Rosetta

[Service]
Type=oneshot
ExecStart=mount -t virtiofs "rosetta" /mnt/rosetta
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

Enable the Rosetta service:

```bash
sudo systemctl enable mount-rosetta.service
```

#### Install binfmt-support

The `binfmt-support` package is used to register Rosetta as a handler for x86-64 binaries. To install it, use the package manager included in your operating system. Below is an example using apt on Debian:

```bash
sudo apt update && sudo apt install -y binfmt-support
```

#### Configure binfmt-support

After installing `binfmt-support`, the package creates a systemd service that runs at startup. You need to update its configuration to ensure it starts only after Rosetta is mounted by modifying the `After` field:

```bash
sudo nano /etc/systemd/system/multi-user.target.wants/binfmt-support.service
```

Modify the following line:

```ini
After=local-fs.target proc-sys-fs-binfmt_misc.automount systemd-binfmt.service mount-rosetta.service
```

#### Register Rosetta to Support x86_64 Binaries

To register Rosetta as a handler, use the following command:

```bash
sudo /usr/sbin/update-binfmts --install rosetta /mnt/rosetta/rosetta \
--magic "\\x7fELF\\x02\\x01\\x01\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x02\\x00\\x3e\\x00" \
--mask "\\xff\\xff\\xff\\xff\\xff\\xfe\\xfe\\x00\\xff\\xff\\xff\\xff\\xff\\xff\\xff\\xff\\xfe\\xff\\xff\\xff" \
--credentials yes --preserve yes --fix-binary yes
```

For more information, refer to [Apple's documentation](https://developer.apple.com/documentation/virtualization/running_intel_binaries_in_linux_vms_with_rosetta).

## Disclaimer

This App is not affiliated with, endorsed by, or associated with Apple Inc., the Linux Foundation, or any other organizations mentioned. All trademarks, product names, and logos are the property of their respective owners and are used here for identification purposes only.

- macOS and Rosetta are registered trademarks of Apple Inc.
- Linux is the registered trademark of Linus Torvalds
- Debian is a registered trademark of Software in the Public Interest, Inc.
- Fedora is the registered trademark of RedHat Inc.
