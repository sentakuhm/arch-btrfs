## Arch Linux BTRFS Installation Guide Oct/2023

### Partitioning & formatting & mounting

* `fdisk /dev/nvme0` and create partitions.
* Format partitions `mkfs.fat -F32 /dev/nvme0n1p1` | `mkfs.btrfs -L "Arch Linux" /dev/nvme0n1p2`.
* Create subvolumes<br>
`mount /dev/nvme0n1p2 /mnt`.<br>
`cd /mnt`.<br>
`btrfs sub cr @` | `@home` | `@swap` | `@.snapshots` | `@cache` | `@log`.<br>
`umount /mnt`.
* Mount root to `mnt`<br>
`mount -o compress=zstd:1,noatime,subvol=@ /dev/nvme0n1p2 /mnt`.
* Create subvolumes mountpoint folders<br>
`mkdir -p /mnt/{efi,home,swap,.snapshots,var/cache,log}`.
* Mount subvolumes to their mountpoint<br>
`mount -o compress=zstd:1,noatime,subvol=@home /dev/nvme0n1p2 /mnt/home`.<br>
`mount -o compress=zstd:1,noatime,subvol=@swap /dev/nvme0n1p2 /mnt/swap`.<br>
`mount -o compress=zstd:1,noatime,subvol=@.snapshots /dev/nvme0n1p2 /mnt/.snapshots`.<br>
`mount -o compress=zstd:1,noatime,subvol=@cache /dev/nvme0n1p2 /mnt/cache`.<br>
`mount -o compress=zstd:1,noatime,subvol=@log /dev/nvme0n1p2 /mnt/var/log`.<br>
`mount /dev/nvme0n1p1 /mnt/efi`.<br>

***

### Settings after chrootting
* Install `btrfs-progs`.

* Add `btrfs` to kernel BINARIES on `nvim /etc/mkinitcpio.conf`
then run `mkinitcpio -P`.

* Make swapfile and activate it<br>
`btrfs filesystem mkswapfile --size=4g --uuid clear /swap/swapfile`<br>
`swapon /swap/swapfile`

* Edit /etc/fstab and add<br>
```
/swap/swapfile   none    swap    defaults    0 0
```
* Edit kernel parameters and add `rootflags=subvol=@`.

### Driver GPU/AUDIO
* `pacman -S pipewire-alsa pipewire-pulse alsa-plugins mesa lib32-mesa xf86-video-amdgpu vulkan-radeon lib32-vulkan-radeon libva-mesa-driver lib32-libva-mesa-driver`
