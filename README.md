# kubernetes-nfs-pvs

This role is useful to export disks as set of Kubernetes persistent volumes.
It does so by partitioning the disks, creating ext4 filesystem on each
partition, mounting the partitions, exporting the mounts via NFS and adding
these NFS shares as NFS persistent volumes to existing Kubernetes installation.

All partitions on given disks are used as the persistent volumes, including
already existing partitions! There should be no other data (such as operating
system) on the disks!

## Requirements

* running Kubernetes with NFS persistent volume support (on a remote machine).

## Role Variables

```
# Options of NFS exports.
nfs_export_options: "*(rw,no_root_squash,insecure,no_subtree_check)"

# Directory, where the created partitions should be mounted. They will be
# mounted as <mount_dir>/sda1 etc.
mount_dir: /exports

# Comma-separated list of disks to partition.
# This role always assumes that all partitions on these disks are used as
# physical volumes.
disks: /dev/sdb,/dev/sdc

# Whether to re-partition already partitioned disks.
# Even though the disks won't get repartitioned on 'false', all existing
# partitions on the disk are exported via NFS as physical volumes!
foce: false

# Specification of size of partitions to create. See library/partitionpool.py
# for details.
sizes: 100M

# URL of Kubernetes API server, incl. port.
kubernetes_url: https://10.245.1.2:6443

# Token to use for authentication to the API server
kubernetes_token: tJdce6Fn3cL1112YoIJ5m2exzAbzcPZX
```

## Dependencies

None

## Example Playbook

With this playbook, `/dev/sdb` is partitioned into 100MiB partitions, all of
them are mounted into `/exports/sdb<N>` directory and all these directories
are exported via NFS and added as physical volumes to Kubernetes running at
`https://10.245.1.2:6443`.

    - hosts: servers
      roles:
        - role: kubernetes-nfs-pvs
          disks: "/dev/sdb"
          sizes: 100M
          kubernetes_url: https://10.245.1.2:6443
          kubernetes_token: tJdce6Fn3cL1112YoIJ5m2exzAbzcPZX

See library/partitionpool.py for details how `sizes` parameter can be used
to create partitions of various sizes.

## License

BSD
