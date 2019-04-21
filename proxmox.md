
# resize logical volume 

### Step 1: using GParted resize physical volumes

### show logical volumes info
```
lvm>lvdisplay
```
### resizing examples 
```
lvm> lvextend -L +2G /etc/pve/swap
lvm> lvextend -L +10G /etc/pve/root
lvm> lvextend -L +50G pve/data
```
### using extended spaces
```
#df -h
#resize2fs /dev/mapper/pve-root
