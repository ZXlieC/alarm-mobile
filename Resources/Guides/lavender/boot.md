# Build boot.img

## 1. Build kernel

```sh
git clone https://github.com/ZXlieC/xec-mainline-project.git
cd xec-mainline-project
make O=output ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc) sdm660_defconfig
make O=output ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc)
make O=output ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_PATH=out-modules modules_install
```

## 2. Create boot.img
```sh
mkdir ../build
cd ../build
cp ../xec-mainline-project/output/arch/arm64/boot/Image.gz .
cp ../xec-mainline-project/output/arch/arm64/boot/dts/qcom/sdm660-xiaomi-lavender-<your panel>.dtb .
cat Image.gz sdm660-xiaomi-lavender-<your panel>.dtb > Image-dtb
mkbootimg --kernel Image-dtb --ramdisk <your initramfs image> --pagesize 4096 --kernel_offset 0x8000 --ramdisk_offset 0x2000000 --second_offset 0x0 --tags_offset 0x0 --header_version 0 --cmdline 'console=ttyMSM0,115200,n8 console=tty0 root=/dev/mmcblk1p<your root partition> rootwait rw' -o boot.img
```

## Flash
```sh
fastboot flash boot boot.img
```
