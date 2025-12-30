# Online EXT to BTRFS converter

Basically a wrapper around `btrfs-convert`, but doesn't require a custom
`initrd`. Tested on Cloud Ubuntu 24.04. Compatible with `cloud-init`. Usable on
Digitalocean VMs.

> **WARNING:** data or OS consistency is not guaranteed. Do not use on
  filesystems that contain any valuable data without backups!

## Supported params

`DRY_RUN` - don't perform btrfs conversion.
`FAST` - speed up execution by skipping initial filesystem maintenance.
`RECOVERY` - whether to start a recovery console on init failure. Don't set
this when your install is automated to allow it to fail fast.
`INIT_OPTS` - bash options to set on the init script, e.g. `INIT_OPTS="-x"`.
`BTRFS_CONVERT_ARGS` - extra args to pass to btrfs-convert.
`FSTAB_OPTS` - extra mount options to be added on btrfs filesystem's fstab entry,
e.g. `FSTAB_OPTS="autodefrag,compress=zstd"`.
`TMPFS_BALANCE` - allows the script to temporarily add a loop device on tmpfs
if it detects that there isn't enough disk space available for balancing.
> **WARNING:** this can easily destroy your filesystem beyond repair, only set
  when you don't store any valuable data there - i.e. during setup of a VM.

`PARENT` - specifies what mechanism is used to run the script. Autodetection is
attempted, but you can specify it to be certain. Only `cloud` for `cloud-init`
is supported.
`CLOUD_CLEAR_SEMAPHORE` - tells the script which `cloud-init` semaphore it
should to clear. Useful when you want to continue executing `runcmd` script
after converting your filesystem (see examples). Set to `auto` to guess.

Flags may contain any non-zero length value to count as set.

These params are read from env vars. When `curl`-ing this script, set them like
this:
```sh
curl '<url>' | RECOVERY=1 INIT_OPTS="-x" bash -x 2>&1
```

## Cloud-init

Add this to your user-data:
```yaml
runcmd:
  - |
    curl https://raw.githubusercontent.com/head-gardener/digitalocean-ext4-to-btrfs/master/ext2btrfs \
      | bash 2>&1
```

Or this if you want to run more commands after converting:
```yaml
runcmd:
  - |
    curl https://raw.githubusercontent.com/head-gardener/digitalocean-ext4-to-btrfs/master/ext2btrfs \
      | CLOUD_CLEAR_SEMAPHORE=auto bash 2>&1 || exit 0
    echo more stuff...
```

## How it works

I wrote this to avoid using `kexec`. The basic idea is `chroot`-ing into `tmpfs`
and converting from there:

1. `tmpfs` root is created;
1. `systemd` binary is replaced with a link to a script that `chroot`-s into
   `init`, the later actually converts the filesystem;
1. `systemd` runs `exev` into that script, triggered by `systemctl
   daemon-reexec`;
1. (Almost) all other processes get killed, freeing up the filesystem;

Init script logs into `kmsg`, you can probably follow it from your provider's
`tty` interface.
