# ansible-ubuntu-update

This playbook makes it easier to update the install packages on a Ubuntu 18.04 host to the latest versions. You can specify which kernel you want to install and use. Additionally you can specify if you want to "hold" the kernel (kernel, headers and modules) and maybe other packages. If a packages needs to be reinstalled this can also be specified. It also auto removes packages no longer needed (that's useful to avoid running `/boot` out of disk space e.g. because it will remove older kernels installed).

The playbook simulates a `apt-get dist-upgrade` and outputs a list of packages that will be upgraded which you need to confirm to proceed. It also checks if the file `/var/run/reboot-required` exists after a update and reboots the host accordingly.

This playbook mainly exists because WireGuard VPN module is not part of the Linux kernel yet (latest kernel currently is 5.2). My [WireGuard Ansible role](https://github.com/githubixx/ansible-role-wireguard) installs WireGuard via `DKMS`. Now if you've unattended upgrades of packages enabled this will also install the latest kernel but won't rebuild the WireGuard module for that kernel. Now if the host is rebooted for whatever reason the host starts as expected but without WireGuard VPN. I guess this is also true for some other kernel modules that are installed via `DKMS`. This playbook gets around this problem by holding the kernel packages (kernel, headers and modules). So the specified kernel will stay the default kernel. If a new `kernel-version` is specified it will install required packages and if `kernel_packages_hold` is set to `true` the kernel packages will be hold by using `apt-mark hold` command. As `DKMS` modules need to be reinstalled if a new kernel was installed you can specify the packages in question with `packages_to_hold`. In the example below that's the packages for WireGuard `wireguard-dkms` and `wireguard-tools`. And with `packages_to_reinstall` you can specify which packages should be rebuild/reinstalled after the OS was updated to the latest packages. And `num_serial` finally specifies how much hosts you want to update in parallel. If you don't set the variable the default is `1`.

The following variables can be set (all optional and the values are just examples):

```
kernel_version: 4.18.0-20
kernel_packages_hold: true
packages_to_hold:
  - wireguard-dkms
  - wireguard-tools
packages_to_reinstall:
  - wireguard-dkms
num_serial: 1
```

Don't forget to change the `hosts` value at the top of `update.yml` before you run the playbook the first time. After that you can the playbook as usual via `ansible-playbook update.yml` or if you want to limit the hosts you want to update use the `--limit` option.
