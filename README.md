# OpenWrt云编译

#### 每天凌晨2点自动编译
### 把本仓库克隆到自己的仓库里
#### 打开下面链接生成.config配置文件
#### [https://hackyes.github.io/openwrt-menuconfig/index.html](https://hackyes.github.io/openwrt-menuconfig/index.html)

#### 编辑 .config 文件，把内容清空替换成上面链接生成的配置内容

### 点这右上角 ✰Star  变成 ★Unstar 即可开始编译

### 等待编译成功后，到Actions里下载固件即可


#

name: OpenWrt

on:
  push:
    branches: 
      - master
  schedule:
    - cron: 0 0 1 * *
  watch:
    types: started

jobs:
  build:
    runs-on: ubuntu-20.04
    if: github.event.repository.owner.id == github.event.sender.id

    steps:
      - name: Checkout
        uses: actions/checkout@master

      - name: Initialization environment
        env:
          DEBIAN_FRONTEND: noninteractive
        run: |
          sudo -E apt-get -yqq update
          sudo -E apt-get -yqq install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch python3 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib
          sudo -E apt-get -y autoremove --purge
          sudo -E apt-get clean
      - name: Download lede
        run: |
          git clone https://github.com/66o99/openwrt-22.03.2

          cp .config ./openwrt-22.03.2/.config
          mv ./openwrt-22.03.2/* ./

      - name: Update feeds
        run: |
          ./scripts/feeds update -a
          ./scripts/feeds install -a
      - name: Costom configure file
        run: |
          make defconfig
      - name: Download package source code
        run: |
          make download -j8
          find dl -size -1024c -exec ls -l {} \;
          find dl -size -1024c -exec rm -f {} \;
      - name: Compile firmware
        run: |
          echo -e "$(nproc) thread build."
          make -j$(nproc) V=s
      - name : Upload artifact
        uses: actions/upload-artifact@master
        with:
          name: OpenWrt_firmware
          path: bin/targets/
          
