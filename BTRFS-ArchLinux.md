BTRFS snapshots on Arch Linux

1. Install Snapper and snap-pac

    `$ sudo pacman -S snapper snap-pac`

2. Create snapshot configuration for root subvolume

    During my Arch install, I created the `@` and `@snapshots` subvolumes, and `/.snapshots` mountpoint.

    Before letting Snapper do its config thing, I need to move my earlier snapshot setup out of the way.

    Unmount the subvolume and remove the mountpoint

    ```ini
    $ sudo umount /.snapshots
    $ sudo rm -rf /.snapshots
    ```

    Create a new root config

    `$ sudo snapper -c root create-config /`
    
    This generates:

      * Configuration file at `/etc/snapper/configs/root`.
      * Add `root` to `SNAPPER_CONFIGS` in `/etc/conf.d/snapper`.
      * Subvolume `.snapshots` where future snapshots for this configuration will be stored.

3. Setup /.snapshots

    List subvolumes

    ```ini
    $ sudo btrfs subvolume list /
    ID 256 gen 199 top level 5 path @
    ID 257 gen 186 top level 5 path @home
    ID 258 gen 9 top level 5 path @snapshots
    [...]
    ID 265 gen 199 top level 256 path .snapshots
    ```

    Note the `@snapshots` subvolume I had created earlier, and the `.snapshots` created by Snapper.

    I prefer my `@snapshots` setup over `.snapshots`, so I delete the Snapper-generated subvolume

    ```ini
    $ sudo btrfs subvolume delete .snapshots
    Delete subvolume (no-commit): '//.snapshots'
    ```

    Re-create and re-mount `/.snapshots` mountpoint

    ```ini
    $ sudo mkdir /.snapshots
    $ sudo mount -a
    ```

    This setup will make all snapshots created by Snapper be stored outside of the `@` subvolume. This allows replacing `@` without losing the snapshots.

4. Manual snapshot
    Example of taking a manual snapshot of a fresh install
    ```ini
    $ sudo snapper -c root create -d "**First Arch Shot**"
    ```


5. Automatic timeline snapshots

    Edit `/etc/snapper/configs/root`, and I allow group of `wheel` to browse through snapshots

    ```ini
    ALLOW_GROUPS="wheel"    
    ```

    while you're there edit `TIMELINE_*` to

    ```ini
    TIMELINE_MIN_AGE="1800"
    TIMELINE_LIMIT_HOURLY="5"
    TIMELINE_LIMIT_DAILY="7"
    TIMELINE_LIMIT_WEEKLY="0"
    TIMELINE_LIMIT_MONTHLY="0"
    TIMELINE_LIMIT_YEARLY="0"
    ```

    Start and enable snapper-timeline.timer to launch the automatic snapshot timeline, and snapper-cleanup.timer to periodically clean up older snapshots

    ```ini
    $ sudo systemctl enable --now snapper-timeline.timer
    $ sudo systemctl enable --now snapper-cleanup.timer

    ```
    List configs

    ```ini
    $ snapper list-configs
    Config | Subvolume
    -------+----------
    root   | /
    ```

    List snapshots taken for `root`
    
    ```ini
    $ sudo snapper -c root list                                                                                                      
   # | Type   | Pre # | Date                            | User | Cleanup | Description                          | Userdata
    ---+--------+-------+---------------------------------+------+---------+--------------------------------------+---------
    0  | single |       |                                 | root |         | current                              |         
    2  | post   |     1 | Mon 25 Mar 2024 04:10:04 PM CET | root | number  | cmake cppdap jsoncpp qt6-tools rhash |         
    4  | post   |     3 | Mon 25 Mar 2024 04:10:40 PM CET | root | number  | btrfs-assistant                      |         
    6  | post   |     5 | Mon 25 Mar 2024 04:10:40 PM CET | root | number  | cmake cppdap jsoncpp qt6-tools rhash |         
    7  | single |       | Mon 25 Mar 2024 04:11:51 PM CET | root |         | **First Arch Shot**                  |
    ```

    List updated subvolumes list, which now includes the snapshots

    ```ini
    $ sudo btrfs subvolume list /                                                                                                    
    ID 256 gen 918 top level 5 path @
    ID 257 gen 1083 top level 5 path @home
    ID 258 gen 1079 top level 5 path @.snapshots
    ID 259 gen 1083 top level 5 path @log
    ID 260 gen 849 top level 5 path @pkgs
    ID 261 gen 32 top level 256 path var/lib/portables
    ID 262 gen 32 top level 256 path var/lib/machines
    ID 265 gen 848 top level 258 path @.snapshots/2/snapshot
    ID 267 gen 852 top level 258 path @.snapshots/4/snapshot
    ID 269 gen 855 top level 258 path @.snapshots/6/snapshot
    ID 270 gen 859 top level 258 path @.snapshots/7/snapshot
    ```

6. Swap file
   
   To properly initialize a swap file, first create a non-snapshotted subvolume to host the file, e.g.
   ```ini
    $ sudo btrfs subvolume create /swap
   ```
   Then Create the swapfile and activate it
   ```ini
    $ sudo btrfs filesystem mkswapfile --size 4g --uuid clear /swap/swapfile
    $ sudo swapon /swap/swapfile
   ```
   Finally, edit the `/etc/fstab` configuration to add an entry for the swap file
   ```ini
    /swap/swapfile none swap defaults 0 0
   ```
   
