## WordPress Solution

## Step 1 - Prepare a web server by launching an Instance on AWS. Next, create 3 EBS volumes of 10gb each and attach them to the Web server instance created

### Creae single partition on each of the 3 disks using 'gdisk' utility and follow the result prompts
`sudo gdisk /dev/xvdf`
`sudo gdisk /dev/xvdg`
`sudo gdisk /dev/xvdh`

### Use 'lsblk' utility to view the configured partition on the 3 disks

### Next, install Lvm2 and check for available partitions
`sudo yum install lvm2`

![lvm2](/images/lvm-install.PNG)

`sudo lvmdiskscan`

### Use pvcreate utility to mark each disks as physical volumes to be used by LVM(logical vol management).
`sudo pvcreate /dev/xvdf1`
`sudo pvcreate /dev/xvdg1`
`sudo pvcreate /dev/xvdh1`

[lvm2](/images/phy-volume.PNG)

### Verify that the PV has been created successfully and use 'vgcreate' utility to add 3 PVs to a volume group named webdata-vg. A;so, verify the entire setup
`sudo pvs`
`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

![vol grp](/images/db-vol-grp.PNG)

`sudo vgs`

![vgs](/images/log%26vg.PNG)

`sudo lsblk`



### Format the logical volumes with ext4 filesystem
`sudo mkfs -t ext4 /dev/webdata-vg/app-lv`
`sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`

### Create directories to store website files and store backup of log data respectively
`sudo mkdir -p /var/www/html`

`sudo mkdir -p /home/recovery/logs`

### Mount the /var/www/html on app-lv logical volume
`sudo mount /dev/webdata-vg/app-lv /var/www/html/`

### Back up all the files in log directory into home recovery logs as this is essential before mounting the file system
`sudo rsync -av /var/log/. /home/recovery/logs/`

### Mount the /var/log on logs-lv logical volume. (existing data will be deleted.hence the step above is vital)
`sudo mount /dev/webdata-vg/logs-lv /var/log`

### Update /etc/fstab file so the mount config persist after server restart. The UUID of the device will be used to update the /etc/fastab. N.B: remove the the leading and ending quotes [add ext4 defaults 0 0]
`sudo blkid`
`sudo vi /etc/fstab`

### Test the config and reload the daemon
`sudo mount -a`
`sudo systemctl daemon-reload`

