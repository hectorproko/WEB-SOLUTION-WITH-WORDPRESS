# WEB-SOLUTION-WITH-WORDPRESS

Before we start we need to have an environment to work with. I will we using my AWS account to create an EC2 instance with an Ubuntu Server.


Use link:
[Creating Red Hat instance in AWS](https://github.com/hectorproko/RepeatableSteps_tutorials/blob/main/AWS_ReHat_Instnace.md)


We need to know which **availability zone** this instance is in when attaching **EBS**. Go to the list of instances to check. My example shows (instance which I named) **Project6** is in availability zone **us-east-1c**
<br /> 
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/WEB-SOLUTION-WITH-WORDPRESS/main/Images/project6.png)
 <br>

I will now create Elastic Block Volumes by going to **Elastic Block Store** > **Volumes** > **Create Volume**
<br /> 
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/WEB-SOLUTION-WITH-WORDPRESS/main/Images/createVolume.png)
 <br>

Once prompted we'll change the size to 10GB, make sure **availability zone** is the same as instance. In my case **us-east-1c**. In addition, I will add a **tag** to name the volume
<br /> 
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/WEB-SOLUTION-WITH-WORDPRESS/main/Images/createVolume2.png)
 <br>

We'll create **3 Volumes** in total. Once created we'll attachment them to the instance one-by-one
<br /> 
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/WEB-SOLUTION-WITH-WORDPRESS/main/Images/attachVolume.png)
 <br>

You should see a list of instances in the same availability zone as the **EBS Volumes**, I'm using **Project6**
 <br /> 
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/WEB-SOLUTION-WITH-WORDPRESS/main/Images/attachVolume2.png)
 <br>

On the Red-Hat terminal using  **lsblk** I can see the volumes attached as xvdf, xvdg, xvdh.
```perl
[ec2-user@ip-172-31-86-213 ~]$ lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0  10G  0 disk
├─xvda1 202:1    0   1M  0 part
└─xvda2 202:2    0  10G  0 part /
xvdf    202:80   0  10G  0 disk
xvdg    202:96   0  10G  0 disk
xvdh    202:112  0  10G  0 disk
[ec2-user@ip-172-31-86-213 ~]$
```
Using **gedisk** we'll create a partition on each of these 3 volumes. I'll keep the defaults which uses the entire Volume. The only thing I'll specify is Partition type with code **8300** for **Linux filesystem**
```text
Hex code or GUID (L to show codes, Enter = 8300):
Changed type of partition to 'Linux filesystem'
```
Output when I did it in **xvdf**
```perl
[ec2-user@ip-172-31-86-213 ~]$ sudo gdisk /dev/xvdf
GPT fdisk (gdisk) version 1.0.3

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help): p
Disk /dev/xvdf: 20971520 sectors, 10.0 GiB
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): F4BDB498-3A7D-4AD5-A5E9-1C5C6684B646
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 20971486
Partitions will be aligned on 2048-sector boundaries
Total free space is 20971453 sectors (10.0 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name

Command (? for help): n
Partition number (1-128, default 1):
First sector (34-20971486, default = 2048) or {+-}size{KMGTP}:
Last sector (2048-20971486, default = 20971486) or {+-}size{KMGTP}:
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300):
Changed type of partition to 'Linux filesystem'

Command (? for help): p
Disk /dev/xvdf: 20971520 sectors, 10.0 GiB
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): F4BDB498-3A7D-4AD5-A5E9-1C5C6684B646
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 20971486
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048        20971486   10.0 GiB    8300  Linux filesystem

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/xvdf.
The operation has completed successfully.
[ec2-user@ip-172-31-86-213 ~]$
```

If we run **lsblk** again we see
```bash
xvdf    202:80   0  10G  0 disk
└─xvdf1 202:81   0  10G  0 part
xvdg    202:96   0  10G  0 disk
└─xvdg1 202:97   0  10G  0 part
xvdh    202:112  0  10G  0 disk
└─xvdh1 202:113  0  10G  0 part
```

We will need to install **lvm2** a  device mapper framework that provides logical volume management for the Linux kernel.
```bash
sudo yum install lvm2
```
Now we can use **lvmdiskscan** to see available partitions. The below output shows 4 partitions including the 3 newly created ones
```perl
[ec2-user@ip-172-31-86-213 ~]$ sudo lvmdiskscan
  /dev/xvda2 [     <10.00 GiB]
  /dev/xvdf1 [     <10.00 GiB] #new
  /dev/xvdg1 [     <10.00 GiB] #new
  /dev/xvdh1 [     <10.00 GiB] #new
  0 disks
  4 partitions
  0 LVM physical volume whole disks
  0 LVM physical volumes

[ec2-user@ip-172-31-86-213 ~]$
```

Now we have to turn these 3 Partitions into **Physical Volumes** using **pvcreate**
```bash
[ec2-user@ip-172-31-86-213 ~]$ sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
  Physical volume "/dev/xvdf1" successfully created.
  Physical volume "/dev/xvdg1" successfully created.
  Physical volume "/dev/xvdh1" successfully created.
[ec2-user@ip-172-31-86-213 ~]$
```

If we run **lvmdiskscan** again we see the partitions are now **LVM physical volumes**
```perl
[ec2-user@ip-172-31-86-213 ~]$ sudo lvmdiskscan
  /dev/xvda2 [     <10.00 GiB]
  /dev/xvdf1 [     <10.00 GiB] LVM physical volume
  /dev/xvdg1 [     <10.00 GiB] LVM physical volume
  /dev/xvdh1 [     <10.00 GiB] LVM physical volume
  0 disks
  1 partition
  0 LVM physical volume whole disks
  3 LVM physical volumes

[ec2-user@ip-172-31-86-213 ~]$
```
Now we'll combine these physical volumes to create a **Volume Group** called **webdata-vg** using command **vgcreate**
```perl
[ec2-user@ip-172-31-86-213 ~]$ sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
  Volume group "webdata-vg" successfully created
[ec2-user@ip-172-31-86-213 ~]$
```
Lets verify **Volume Group** creation using **vgs**
```bash
[ec2-user@ip-172-31-86-213 ~]$ sudo vgs
  VG         #PV #LV #SN Attr   VSize   VFree
  webdata-vg   3   0   0 wz--n- <29.99g <29.99g
[ec2-user@ip-172-31-86-213 ~]$
```

Now lets create 2 **Logical Volumes** apps-lv and logs-lv 14GB in size from the newly create **Volume Group**
```bash
[ec2-user@ip-172-31-86-213 ~]$ sudo lvcreate -n apps-lv -L 14G webdata-vg
  Logical volume "apps-lv" created.
[ec2-user@ip-172-31-86-213 ~]$ sudo lvcreate -n logs-lv -L 14G webdata-vg
  Logical volume "logs-lv" created.
```
To verify the new **Logical Volumes** run **lvs**
```bash
[ec2-user@ip-172-31-86-213 ~]$ sudo lvs
  LV      VG         Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  apps-lv webdata-vg -wi-a----- 14.00g
  logs-lv webdata-vg -wi-a----- 14.00g
[ec2-user@ip-172-31-86-213 ~]$
```
To look at the entire setup you can run **vgdisplay -v**
```bash
[ec2-user@ip-172-31-86-213 ~]$ sudo vgdisplay -v
  --- Volume group ---
  VG Name               webdata-vg
  System ID
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               0
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               <29.99 GiB
  PE Size               4.00 MiB
  Total PE              7677
  Alloc PE / Size       7168 / 28.00 GiB
  Free  PE / Size       509 / <1.99 GiB
  VG UUID               HSA3Cm-vid3-4KJ7-dPpd-eMxl-tQ3L-W1zuuk

  --- Logical volume ---
  LV Path                /dev/webdata-vg/apps-lv
  LV Name                apps-lv
  VG Name                webdata-vg
  LV UUID                lA9NKo-oGad-Xk4Z-pEpp-t1Q5-T9g1-vLVGn6
  LV Write Access        read/write
  LV Creation host, time ip-172-31-86-213.ec2.internal, 2021-09-04 23:25:03 +0000
  LV Status              available
  # open                 0
  LV Size                14.00 GiB
  Current LE             3584
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:0

  --- Logical volume ---
  LV Path                /dev/webdata-vg/logs-lv
  LV Name                logs-lv
  VG Name                webdata-vg
  LV UUID                ygkcxz-Ul2W-TxDw-pgYP-3LrR-5pz5-p8QdM1
  LV Write Access        read/write
  LV Creation host, time ip-172-31-86-213.ec2.internal, 2021-09-04 23:25:19 +0000
  LV Status              available
  # open                 0
  LV Size                14.00 GiB
  Current LE             3584
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:1

  --- Physical volumes ---
  PV Name               /dev/xvdh1
  PV UUID               dJ5O9U-dSiK-zJMu-ZYJl-50t1-uIHA-ZzCNpi
  PV Status             allocatable
  Total PE / Free PE    2559 / 0

  PV Name               /dev/xvdg1
  PV UUID               e4jN5M-Ld0w-ovkT-Vvad-CLdy-29pN-7vKLcl
  PV Status             allocatable
  Total PE / Free PE    2559 / 509

  PV Name               /dev/xvdf1
  PV UUID               iURybq-N3Ib-WPCr-oRLY-pUNN-H141-RnzRwB
  PV Status             allocatable
  Total PE / Free PE    2559 / 0


[ec2-user@ip-172-31-86-213 ~]$
```

Now if we run **lsblk** again we can see the partitions, volume group and logical volumes
```bash
xvdf                     202:80   0  10G  0 disk
└─xvdf1                  202:81   0  10G  0 part
  └─webdata--vg-logs--lv 253:1    0  14G  0 lvm
xvdg                     202:96   0  10G  0 disk
└─xvdg1                  202:97   0  10G  0 part
  ├─webdata--vg-apps--lv 253:0    0  14G  0 lvm
  └─webdata--vg-logs--lv 253:1    0  14G  0 lvm
xvdh                     202:112  0  10G  0 disk
└─xvdh1                  202:113  0  10G  0 part
  └─webdata--vg-apps--lv 253:0    0  14G  0 lvm
```

Format the **Logical Volumes** to **ext4**
```bash
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```
