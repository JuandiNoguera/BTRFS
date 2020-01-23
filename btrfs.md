# BTRFS

- Fichero VagrantFile de la maquina con 4 discos adicionales
```
Vagrant.configure("2") do |config|
disco1='.vagrant/disco01.vdi'
disco2='.vagrant/disco02.vdi'
disco3='.vagrant/disco03.vdi'
disco4='.vagrant/disco04.vdi'
 config.vm.define :btrfs do |btrfs|
  btrfs.vm.box = "debian/buster64"
  btrfs.vm.hostname = "btrfs"
  btrfs.vm.provider :virtualbox do |v|
  v.customize ["createhd", "--filename", disco1, "--size", 1024]
  v.customize ["storageattach", :id, "--storagectl", "SATA Controller",
  "--port", 1, "--device", 0, "--type", "hdd",
  "--medium", disco1]
  v.customize ["createhd", "--filename", disco2, "--size", 1024]
  v.customize ["storageattach", :id, "--storagectl", "SATA Controller",
  "--port", 2, "--device", 0, "--type", "hdd",
  "--medium", disco2]
  v.customize ["createhd", "--filename", disco3, "--size", 1024]
  v.customize ["storageattach", :id, "--storagectl", "SATA Controller",
  "--port", 3, "--device", 0, "--type", "hdd",
  "--medium", disco3]
  v.customize ["createhd", "--filename", disco4, "--size", 1024]
  v.customize ["storageattach", :id, "--storagectl", "SATA Controller",
  "--port", 4, "--device", 0, "--type", "hdd",
  "--medium", disco4]
  end
end
```

- Lista de los discos
```
vagrant@btrfs:~$ lsblk 
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0 19.8G  0 disk 
├─sda1   8:1    0 18.8G  0 part /
├─sda2   8:2    0    1K  0 part 
└─sda5   8:5    0 1021M  0 part [SWAP]
sdb      8:16   0    1G  0 disk 
sdc      8:32   0    1G  0 disk 
sdd      8:48   0    1G  0 disk 
sde      8:64   0    1G  0 disk 
```

- Instalar software
```
sudo apt install btrfs-tools
```

- Dar formato btrfs a un disco
```
vagrant@btrfs:~$ sudo mkfs.btrfs /dev/sdb 
btrfs-progs v4.20.1 
See http://btrfs.wiki.kernel.org for more information.

Label:              (null)
UUID:               2ceb76ad-7f88-43d2-9346-d5add4087ba6
Node size:          16384
Sector size:        4096
Filesystem size:    1.00GiB
Block group profiles:
  Data:             single            8.00MiB
  Metadata:         DUP              51.19MiB
  System:           DUP               8.00MiB
SSD detected:       no
Incompat features:  extref, skinny-metadata
Number of devices:  1
Devices:
   ID        SIZE  PATH
    1     1.00GiB  /dev/sdb
    
vagrant@btrfs:~$ lsblk -f
NAME   FSTYPE LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINT
sda                                                                     
├─sda1 ext4         b9ffc3d1-86b2-4a2c-a8be-f2b2f4aa4cb5   16.4G     5% /
├─sda2                                                                  
└─sda5 swap         f8f6d279-1b63-4310-a668-cb468c9091d8                [SWAP]
sdb    btrfs        2ceb76ad-7f88-43d2-9346-d5add4087ba6                
sdc                                                                     
sdd                                                                     
sde  
```

- Configuracion del soporte multidispositivo que lo interpreta como un RAID
```
vagrant@btrfs:~$ sudo mkfs.btrfs /dev/sdc /dev/sdd -f
btrfs-progs v4.20.1 
See http://btrfs.wiki.kernel.org for more information.

Label:              (null)
UUID:               ae936953-d16f-4bc9-a52f-fd0ba84794bf
Node size:          16384
Sector size:        4096
Filesystem size:    2.00GiB
Block group profiles:
  Data:             RAID0           204.75MiB
  Metadata:         RAID1           102.38MiB
  System:           RAID1             8.00MiB
SSD detected:       no
Incompat features:  extref, skinny-metadata
Number of devices:  2
Devices:
   ID        SIZE  PATH
    1     1.00GiB  /dev/sdc
    2     1.00GiB  /dev/sdd
    
vagrant@btrfs:~$ lsblk -f
NAME   FSTYPE LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINT
sda                                                                     
├─sda1 ext4         b9ffc3d1-86b2-4a2c-a8be-f2b2f4aa4cb5   16.4G     5% /
├─sda2                                                                  
└─sda5 swap         f8f6d279-1b63-4310-a668-cb468c9091d8                [SWAP]
sdb    btrfs        2ceb76ad-7f88-43d2-9346-d5add4087ba6                
sdc    btrfs        ae936953-d16f-4bc9-a52f-fd0ba84794bf                
sdd    btrfs        ae936953-d16f-4bc9-a52f-fd0ba84794bf                
sde       
```

- Creacion de un RAID 1
```
vagrant@btrfs:~$ sudo mkfs.btrfs -d raid1 -m raid1 /dev/sdc /dev/sdd /dev/sde -f
btrfs-progs v4.20.1 
See http://btrfs.wiki.kernel.org for more information.

Label:              (null)
UUID:               6c73fc22-93e0-4152-be82-c015073d3ccb
Node size:          16384
Sector size:        4096
Filesystem size:    3.00GiB
Block group profiles:
  Data:             RAID1           153.56MiB
  Metadata:         RAID1           153.56MiB
  System:           RAID1             8.00MiB
SSD detected:       no
Incompat features:  extref, skinny-metadata
Number of devices:  3
Devices:
   ID        SIZE  PATH
    1     1.00GiB  /dev/sdc
    2     1.00GiB  /dev/sdd
    3     1.00GiB  /dev/sde

```

- Tenemos el primer disco que le dimos formato y el raid 1 que hemos creado
```
vagrant@btrfs:~$ sudo btrfs filesystem show
Label: none  uuid: 2ceb76ad-7f88-43d2-9346-d5add4087ba6
	Total devices 1 FS bytes used 128.00KiB
	devid    1 size 1.00GiB used 126.38MiB path /dev/sdb

Label: none  uuid: 6c73fc22-93e0-4152-be82-c015073d3ccb
	Total devices 3 FS bytes used 128.00KiB
	devid    1 size 1.00GiB used 307.12MiB path /dev/sdc
	devid    2 size 1.00GiB used 161.56MiB path /dev/sdd
	devid    3 size 1.00GiB used 161.56MiB path /dev/sde
```

- Añadir otro disco al raid, montamos un disco del raid y luego lo añadimos
```
vagrant@btrfs:~$ sudo mount /dev/sdc /mnt/
vagrant@btrfs:~$ sudo btrfs device add /dev/sdb /mnt/ -f
vagrant@btrfs:~$ sudo btrfs filesystem show
Label: none  uuid: 6c73fc22-93e0-4152-be82-c015073d3ccb
	Total devices 4 FS bytes used 256.00KiB
	devid    1 size 1.00GiB used 307.12MiB path /dev/sdc
	devid    2 size 1.00GiB used 161.56MiB path /dev/sdd
	devid    3 size 1.00GiB used 161.56MiB path /dev/sde
	devid    4 size 1.00GiB used 0.00B path /dev/sdb
```

- Repartir la informacion de los demas discos en el disco recientemente añadido
```
vagrant@btrfs:~$ sudo btrfs balance start --full-balance /mnt/
Done, had to relocate 3 out of 3 chunks
vagrant@btrfs:~$ sudo btrfs filesystem show
Label: none  uuid: 6c73fc22-93e0-4152-be82-c015073d3ccb
	Total devices 4 FS bytes used 256.00KiB
	devid    1 size 1.00GiB used 288.00MiB path /dev/sdc
	devid    2 size 1.00GiB used 416.00MiB path /dev/sdd
	devid    3 size 1.00GiB used 288.00MiB path /dev/sde
	devid    4 size 1.00GiB used 416.00MiB path /dev/sdb
```

- Comprobar el estado del RAID
```
vagrant@btrfs:~$ sudo btrfs scrub status /mnt/
scrub status for 6c73fc22-93e0-4152-be82-c015073d3ccb
	scrub started at Thu Jan 23 11:31:10 2020 and finished after 00:00:00
	total bytes scrubbed: 512.00KiB with 0 errors
```