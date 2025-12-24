# Online EXT to BTRFS converter

Basically a wrapper around `btrfs-convert`, but doesn't require a custom
`initrd`. Tested on Cloud Ubuntu 24.04. Compatible with `ssh`, `cloud-init`.
Usable on Digitalocean VMs.

## Supported params

`RECOVERY` - whether to start a recovery console on init failure. Don't set
this when your install is automated to allow it to fail fast.
`REBOOT` - whether to reboot once conversion is completed. Notice that your
system may not be fully usable until you reboot or `kexec`.
`INIT_OPTS` - bash options to set on the init script e.g. `INIT_OPTS="-x"`.

Flags may contain any non-zero length value to count as set.

These params are read from env vars. When `curl`-ing this script, set them like
this:
```sh
curl '<url>' | RECOEVERY=1 REBOOT=1 bash -x 2>&1
```

## Cloud-init

Add this to your user-data:
```yaml
runcmd:
  - |
    curl https://raw.githubusercontent.com/head-gardener/digitalocean-ext4-to-btrfs/master/ext2btrfs \
      | bash -x 2>&1
```
