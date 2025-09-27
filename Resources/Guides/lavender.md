# Building root image for Xiaomi redmi note 7 (lavender)

### 1. Download Arch Linux ARM generic root build for aarch64 platforms: http://os.archlinuxarm.org/os/ArchLinuxARM-aarch64-latest.tar.gz
### 2. Create ext4 root.img image and mount it
### 3. Extract Arch Linux ARM generic root build in to root.img
### 4. Follow Package instalition
### 5. When you will finish building rootfs you need to convert in to sparse image
```sh
pacman -S android-tools
img2simg root.img sroot.img
```
### 6. flashing
```sh
fastboot flash userdata sroot.img
```

or you can change GPT layout by gdisk and combine vendor, system, cache, frp, userdata in to one root partition.
```sh
fastboot flash root sroot.img
```


## Package instalition

### 1. Make WIFI works
1. you need to install msm-firmware-loader: https://gitlab.postmarketos.org/postmarketOS/msm-firmware-loader
2. build wifi firmware (firmware-5.bin you can get in linux-firmware-atheros package): https://github.com/jhugo/linux/commit/ae097bf4ae31e1de396045dc1dca4a7dcdbd6111

3. installing needed packages
```sh
pacman -S rmtfs-git qrtr-git git base-devel ninja meson
systemctl enable rmtfs
systemctl enable qrtr-ns

git clone https://github.com/linux-msm/tqftpserv.git
cd tqftpserv
meson build
cd build
ninja
cp tqftpserv /usr/bin/
cp tqftpserv.service /usr/lib/systemd/system/
systemctl enable tqftpserv
cd ../../

git clone https://github.com/linux-msm/diag.git
cd diag
make
cp diag-router /usr/bin/
cp send_data /usr/bin
touch /usr/lib/systemd/system/diag-router.service                                                         
echo "[Unit]" > /usr/lib/systemd/system/diag-router.service
echo "Description=Qualcomm DIAG router" >> /usr/lib/systemd/system/diag-router.service
echo "Before=rmtfs.service" >> /usr/lib/systemd/system/diag-router.service
echo "" >> /usr/lib/systemd/system/diag-router.service
echo "[Service]" >> /usr/lib/systemd/system/diag-router.service
echo "ExecStart=/usr/bin/diag-router" >> /usr/lib/systemd/system/diag-router.service
echo "Restart=always" >> /usr/lib/systemd/system/diag-router.service
echo "RestartSec=1"
echo "" >> /usr/lib/systemd/system/diag-router.service
echo "[Install]" >> /usr/lib/systemd/system/diag-router.service
echo "WantedBy=multi-user.target" >> /usr/lib/systemd/system/diag-router.service
systemctl enable diag-router
```

### 2. Make GPU (Adreno 512) works
1. installing packages
    ```sh
    pacman -S linux-firmware-qcom
    ```
2. get a512_zap.mbn file
   ```sh
   # You need to get vendor.img from android firmware then follow this commands
   pacman -S android-tools
   simg2img vendor.img vendor-ext4.img
   mount vendor-ext4.img vendor
   cp vendor/firmware/a512_zap.elf <your root.img mountpoint>/lib/firmware/a512_zap.mbn
   ```


