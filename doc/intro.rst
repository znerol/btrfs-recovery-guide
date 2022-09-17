Intro
*****

The use of `btrfs`_ as the primary filesystem offers interesting options for
system-wide backups. Atomic snapshots are used to quickly capture the state of
a subvolume at any point in time. Incremental backups can be synchronized to
locally attached disks or over the network to remote sites.

However, a backup is only useful if it is easy to restore a new machine quickly.
This guide provides step-by-step instructions for selected Linux distributions.

.. _btrfs: https://btrfs.wiki.kernel.org/
