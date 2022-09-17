Restore Fedora with encrypted drive
***********************************


Step 1: Install to hard drive
=============================

Boot the new machine from the installer medium and choose install to hard drive.

Choose the default storage layout with encryption enabled and set a new
passphrase. See `fedora install guide`_ for detailed instructions.

Do not reboot after the installation finished.

.. _`fedora install guide`: https://docs.fedoraproject.org/en-US/fedora/latest/install-guide/


Step 2: Mount the toplevel btrfs volume
=======================================

During installation, a new `btrfs` filesystem was created by the installer.
It is necessary to mount the toplevel volume of this filesystem in order to
replace the `root` and `home` subvolumes in a later step.

Open a terminal window and become root::

  $ sudo -i

Note: Since this is still the installer environment, `sudo` shouldn't ask for a
password.

Figure out where the new system was installed into::

  # mount | grep 'on /mnt/sysroot type btrfs'
  /dev/mapper/luks-ea656713-63b6-407e-b13b-a2edea855999 on /mnt/sysroot type btrfs (rw,relatime,seclabel,compress=zstd:1,space_cache=v2,subvolid=257,subvol=/root)

Note: The device path should start with ``/dev/maper/luks-`` and end with an UUID.

Create a mount point directory and mount the toplevel subvolume::

  # mkdir /mnt/toplevel
  # mount /dev/mapper/luks-ea656713-63b6-407e-b13b-a2edea855999 /mnt/toplevel

Confirm that the mount has succeeded::

  # mount | grep 'on /mnt/toplevel type btrfs'
  /dev/mapper/luks-ea656713-63b6-407e-b13b-a2edea855999 on /mnt/toplevel type btrfs (rw,relatime,seclabel,compress=zstd:1,space_cache=v2,subvolid=5,subvol=/)


Step 3: Copy backup from attached hard drive
============================================

Attach the backup disk, unlock it (using the passphrase of the backup disk) and
mount it using the file manager app.

Figure out the backup mount point. Attached disks are mounted somewhere under
`/run/media/liveuser/...`::

  # mount | grep 'on /run/media/liveuser/[^/]* type btrfs'
  /dev/mapper/luks-7977ef3a-1bce-4ea5-b776-f9daf44b52f6 on /run/media/liveuser/Backup type btrfs (rw,nosuid,nodev,relatime,seclabel,space_cache=v2,subvolid=5,subvol=/,uhelper=udisks2)

Note the backup mount point (in this case ``/run/media/liveuser/Backup``).
Substitute this path with the correct in the following commands.

List available snapshots in the backup::

  # btrfs subvol list /run/media/liveuser/Backup
  ID 256 gen 20 top level 5 path ROOT.20220916T1320
  ID 257 gen 20 top level 5 path home.20220916T1320

Copy selected snapshots to the toplevel volume of the internal disk::

  # btrfs send /run/media/liveuser/Backup/ROOT.20220916T1320 | btrfs receive /mnt/toplevel/
  At subvol /run/media/liveuser/Backup/ROOT.20220916T1320
  At subvol ROOT.20220916T1320
  # btrfs send /run/media/liveuser/Backup/home.20220916T1320 | btrfs receive /mnt/toplevel/
  At subvol /run/media/liveuser/Backup/home.20220916T1320
  At subvol home.20220916T1320

Confirm that the backups have been transferred to the toplevel volume::

  # btrfs subvol list /mnt/toplevel/
  ID 256 gen 12 top level 5 path home
  ID 257 gen 37 top level 5 path root
  ID 258 gen 44 top level 5 path ROOT.20220916T1320
  ID 259 gen 48 top level 5 path home.20220916T1320

Move newly instaled system out of the way::

  # mv /mnt/toplevel/root /mnt/toplevel/root.away
  # mv /mnt/toplevel/home /mnt/toplevel/home.away

Create writable subvolumes from the restored snapshots::

  # btrfs subvolume snapshot /mnt/toplevel/ROOT.20220916T1320 /mnt/toplevel/root
  Create a snapshot of '/mnt/toplevel/ROOT.20220916T1320' in '/mnt/toplevel/root'
  # btrfs subvolume snapshot /mnt/toplevel/home.20220916T1320 /mnt/toplevel/home
  Create a snapshot of '/mnt/toplevel/home.20220916T1320' in '/mnt/toplevel/home'

