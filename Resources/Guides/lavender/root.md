# Building root image for Xiaomi redmi note 7 (lavender)

### 1. Download Arch Linux ARM generic root build for aarch64 platforms: http://os.archlinuxarm.org/os/ArchLinuxARM-aarch64-latest.tar.gz
```sh
curl -O 'https://ca.us.mirror.archlinuxarm.org/os/ArchLinuxARM-aarch64-latest.tar.gz'
```
### 2. Create ext4 root.img image and extract Arch Linux ARM generic root build in to root.img mountpoint and unmount it
```sh
truncate -s 3G root.img
mkfs.ext4 root.img
mkdir root
sudo mount root.img root
sudo tar -zxvf ArchLinuxARM-aarch64-latest.tar.gz -C root
sudo umount root
```

<details>
<b><summary>3. Follow Package instalition</b></summary>


## If your device where you going to install packages not based on aarch64 architecture you need to install QEMU and Arch Linux ARM aarch64 enviroment

### 1. installing QEMU

```sh
sudo pacman --needed -S qemu-full 
```
Now you can use the `chroot` command to set up your aarch64 root build 

### Create, mount root.img and enter aarch64 Arch Linux ARM root enviroment
```sh
mkdir alarm
sudo tar -zxvf ../ArchLinuxARM-aarch64-latest.tar.gz -C alarm
sudo mount root.img alarm/root
sudo rm alarm/etc/resolv.conf
sudo rm alarm/etc/mtab
sudo cp /proc/self/mounts alarm/etc/mtab
sudo chroot alarm
echo "nameserver 8.8.8.8" > /etc/resolv.conf
pacman-key --init
pacman-key --populate archlinuxarm
pacman -Sy
```

## If your device based on aarch64 architecture just mount root.img again to root directory and enter root by sudo
```sh
sudo mount root.img root
sudo -i
```

### 1. Delete linux-firmware and linux-aarch64 packages

```sh
pacman -Rc linux-firmware linux-aarch64 -r root
```
    
### 2. Make WIFI and modem works
1. Install msm-firmware-loader
```sh
pacman --needed -S git
git clone https://gitlab.postmarketos.org/postmarketOS/msm-firmware-loader.git
cp msm-firmware-loader/msm-firmware-loader.service root/usr/lib/systemd/system/
cp msm-firmware-loader/msm-firmware-loader-unpack.service root/usr/lib/systemd/system/
cp msm-firmware-loader/msm-firmware-loader.sh root/usr/sbin/
cp msm-firmware-loader/msm-firmware-loader-unpack.sh root/usr/sbin/
mkdir root/lib/firmware/msm-firmware-loader

chroot root
systemctl enable msm-firmware-loader
systemctl enable msm-firmware-loader-unpack
exit
```

2. build wifi firmware (firmware-5.bin you can get in linux-firmware-atheros package): https://github.com/jhugo/linux/commit/ae097bf4ae31e1de396045dc1dca4a7dcdbd6111

3. installing needed packages
```sh
pacman --needed -S rmtfs-git qrtr-git git base-devel ninja meson -r root

git clone https://github.com/linux-msm/tqftpserv.git
cd tqftpserv
meson build
cd build
ninja
cp tqftpserv ../../usr/bin/
cp tqftpserv.service ../../usr/lib/systemd/system/

cd ../../

git clone https://github.com/linux-msm/diag.git
cd diag
make
cp diag-router ../root/usr/bin/
cp send_data ../root/usr/bin
cd ..
touch root/usr/lib/systemd/system/diag-router.service                                                         
echo "[Unit]" > root/usr/lib/systemd/system/diag-router.service
echo "Description=Qualcomm DIAG router" >> root/usr/lib/systemd/system/diag-router.service
echo "Before=rmtfs.service" >> root/usr/lib/systemd/system/diag-router.service
echo "" >> root/usr/lib/systemd/system/diag-router.service
echo "[Service]" >> root/usr/lib/systemd/system/diag-router.service
echo "ExecStart=/usr/bin/diag-router" >> root/usr/lib/systemd/system/diag-router.service
echo "Restart=always" >> root/usr/lib/systemd/system/diag-router.service
echo "RestartSec=1" >> root/usr/lib/systemd/system/diag-router.service
echo "" >> root/usr/lib/systemd/system/diag-router.service
echo "[Install]" >> root/usr/lib/systemd/system/diag-router.service
echo "WantedBy=multi-user.target" >> root/usr/lib/systemd/system/diag-router.service

chroot root
systemctl enable rmtfs
systemctl enable qrtr-ns
systemctl enable tqftpserv
systemctl enable diag-router
exit
```
### 3. Make cellular works

1. installing packages
    ```sh
    pacman --needed -S ModemManager -r root
    chroot root
    systemctl enable ModemManager
    exit
    ```
2. install msm-modem-uim-selection: https://gitlab.com/TravMurav/pmaports/-/tree/msm-modem-uim/modem/msm-modem

3. insert SIM card before boot linux

### 4. Make GPU (Adreno 512) works
1. installing packages
    ```sh
    pacman --needed -S linux-firmware-qcom -r root
    ```
2. get a512_zap.mbn file
   ```sh
   curl -O 'https://raw.githubusercontent.com/ZXlieC/alarm-mobile/refs/heads/main/Resources/Files/firmware/a512_zap.mbn'
   cp a512_zap.mbn root/lib/firmware/
   ```

### If you in Arch Linux ARM aarch64 enviroment don`t forget to exit
```sh
exit
```
</details>



### 4. Unmount and convert root.img in to sparse image
```sh
sudo umount root.img
sudo pacman --neded -S android-tools
img2simg root.img sroot.img
```
### 5. flashing
```sh
fastboot flash userdata sroot.img
```

### You can change GPT layout by gdisk and combine vendor, system, cache, frp, userdata in to one root partition.
```sh
fastboot flash root sroot.img
```



