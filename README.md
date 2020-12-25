# linux-lab-5
## Работа с lvm
### Цель:
Практика c LVM
### Ход работы:
##### Были добавлены в систему 5 виртуальных дисков по 512 Mb каждый.
##### Установлена *lvm* командой `sudo apt install lvm2 -y`
##### Смотрим текущее состояние:
```
lsblk
sudo lvmdiskscan
```
![](images/image_2020-12-25_18-07-18.png)
![](images/image_2020-12-25_18-12-45.png)
##### Добавляем диск как PV:
```
sudo pvcreate /dev/sdb
sudo pvdisplay
sudo lvmdiskscan
sudo pvs
```
![](images/image_2020-12-25_18-17-53.png)
![](images/image_2020-12-25_18-18-27.png)
##### Создаём VG на базе PV
```
sudo vgcreate vgmai /dev/sdb
sudo vgdisplay -v vgmai
sudo vgs
```
![](images/image_2020-12-25_18-24-15.png)
##### Создаём LV на базе VG
```
sudo lvcreate -l+100%FREE -n first vgmai
sudo lvdisplay
sudo lvs
```
![](images/image_2020-12-25_18-29-08.png)
##### Создаём файловую систему, монтируем её и проверяем
```
sudo mkfs.ext4 /dev/vgmai/first
sudo mount /dev/vgmai/first /mnt
sudo mount
```
![](images/image_2020-12-25_18-35-29.png)
![](images/image_2020-12-25_18-36-42.png)
##### Создаём файл на весь размер точки монтирования
```
sudo dd if=/dev/zero of=/mnt/test.file bs=1M count=1500 status=progress
sudo df -h
```
![](images/image_2020-12-25_18-44-25.png)
##### Расширяем LV за счёт нового PV в VG
```
sudo pvcreate /dev/sdc
sudo vgextend vgmai /dev/sdc
sudo lvextend -l+100%FREE /dev/vgmai/first
sudo lvdisplay
sudo lvs
sudo df -h
```
![](images/image_2020-12-25_18-49-38.png)
![](images/image_2020-12-25_18-50-09.png)
##### Расширяем файловую систему
```
sudo resize2fs /dev/vgmai/first
sudo df -h
```
![](images/image_2020-12-25_19-02-48.png)
##### Уменьшаем файловую систему и LV
```
sudo umount /mnt
sudo fsck -fy /dev/vgmai/first
sudo resize2fs -f /dev/vgmai/first 650M
sudo mount /dev/vgmai/first /mnt
sudo df -h
```
![](images/image_2020-12-25_19-09-35.png)
![](images/image_2020-12-25_19-10-04.png)
##### Создаём несколько файлов и делаем снимок
```
sudo touch /mnt/file{1..5}
ls /mnt
sudo lvcreate -L 100M -s -n snapsh /dev/vgmai/first
sudo lvs
sudo lsblk
```
![](https://github.com/Pashayam/LinuxLab5/blob/main/images/17.png)
![](https://github.com/Pashayam/LinuxLab5/blob/main/images/18.png)
##### Удаляем несколько файлов
```
sudo rm -f /mnt/file{1..3}
ls /mnt
```
![](https://github.com/Pashayam/LinuxLab5/blob/main/images/19.png)
##### Монтируем снимок и проверяем, что файлы там есть. Отмонтируем.
```
sudo mkdir /snap
sudo mount /dev/mai/snapsh /snap
ls /snap
sudo umount /snap
```
![](https://github.com/Pashayam/LinuxLab5/blob/main/images/20.png)
##### Отмонтируем файловую систему и производим слияние. Проверяем, что файлы на месте.
```
sudo umount /mnt
sudo lvconvert --merge /dev/mai/snapsh
sudo mount /dev/mai/first /mnt
ls /mnt
```
![](https://github.com/Pashayam/LinuxLab5/blob/main/images/21.png)
##### Добавляем ещё PV, VG и создаём LV-зеркало.
```
sudo pvcreate /dev/sd{d,e}
sudo vgcreate vgmirror /dev/sd{d,e}
sudo lvcreate -l+80%FREE -m1 -n mirror1 vgmirror
```
![](https://github.com/Pashayam/LinuxLab5/blob/main/images/22.png)
##### Наблюдаем синхронизацию.
```
sudo lvs
```
![](https://github.com/Pashayam/LinuxLab5/blob/main/images/23.png)
