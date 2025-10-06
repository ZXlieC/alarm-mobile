# Build boot.img

## 1. Build kernel

```sh
git clone https://github.com/ZXlieC/xec-mainline-project.git
cd xec-mainline-project
make defconfig_lavennder
```

## 2. Create boot.img
```sh
cat Image.gz sdm660-lavender.dtb > Image-dtb
mkbootimg -o boot.img
```

## Flash
```sh
fastboot flash boot boot.img
```
