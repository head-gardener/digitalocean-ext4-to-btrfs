# Online EXT to BTRFS converter

Basically a wrapper around `btrfs-convert`, but doesn't require a custom
`initrd`. Tested on Cloud Ubuntu 24.04. Compatible with `cloud-init`. Usable on
Digitalocean VMs.

> **WARNING:** data or OS consistency is not guaranteed. Do not use on
  filesystems that contain any valuable data without backups!

## Supported params

`RECOVERY` - whether to start a recovery console on init failure. Don't set
this when your install is automated to allow it to fail fast.
`REBOOT` - whether to reboot once conversion is completed. Notice that your
system may not be fully usable until you reboot.
`INIT_OPTS` - bash options to set on the init script, e.g. `INIT_OPTS="-x"`.
`BTRFS_CONVERT_ARGS` - extra args to pass to btrfs-convert.
`FSTAB_OPTS` - extra mount options to be added on btrfs filesystem's fstab entry,
e.g. `FSTAB_OPTS="autodefrag,compress=zstd"`.
`TMPFS_BALANCE` - allows the script to temporarily add a loop device on tmpfs
if it detects that there isn't enough disk space available for balancing.
> **WARNING:** this can easily destroy your filesystem beyond repair, only set
  when you don't store any valuable data there - i.e. during setup of a VM.

Flags may contain any non-zero length value to count as set.

These params are read from env vars. When `curl`-ing this script, set them like
this:
```sh
curl '<url>' | RECOVERY=1 REBOOT=1 bash -x 2>&1
```

## Cloud-init

Add this to your user-data:
```yaml
runcmd:
  - |
    curl https://raw.githubusercontent.com/head-gardener/digitalocean-ext4-to-btrfs/master/ext2btrfs \
      | bash -x 2>&1
```
