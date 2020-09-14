---
layout: post
title; [Yocto] Build Linux Raspberry Pi using Yocto
comments: true
tags: [Yocto, Raspberry Pi]
---

본 문서는 아래 블로그 글과 몇번의 삽질을 기반으로 작성되었습니다. 

[https://js94.tistory.com/entry/라즈베리파이3Raspberry-Pi3에-Yocto-Project-설치](https://js94.tistory.com/entry/%EB%9D%BC%EC%A6%88%EB%B2%A0%EB%A6%AC%ED%8C%8C%EC%9D%B43Raspberry-Pi3%EC%97%90-Yocto-Project-%EC%84%A4%EC%B9%98)

[https://gachiemchiep.github.io/cheatsheet/build_image_rpi3_yocto/](https://gachiemchiep.github.io/cheatsheet/build_image_rpi3_yocto/)

---

## 시작하며...

 리눅스라는 OS를 하루이틀 쓴 것은 아니지만 아직도 낮설다. 특히 embedded level로 내려가면 더욱 그렇다.  부팅은 어떤 과정을 거쳐 진행되며, 각종 device는 어떤 원리로 동작되며, 간단한 함수로 사용하는 카메라는 어떻게 인식되는지 궁금한 것 투성이다.  

  BSP 담당자에게 어찌 공부하면 되냐고 물어보니 일단 Raspberry Pi에 Yocto 올려서 이것저것 해보라고 한다. 한번만 해보면 된다고 하기에 일단 그렇게 해보기로 했다. 먼저 Raspberry Pi에 Linux를 올려서 부팅을 해야 한다.

## 1. Build 환경

### a. 빌드 준비

- VirtualBox에서 동작하는 Ubuntu 16.04에서 진행.
- Dependency 설치

```bash
sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib build-essential chrpath socat cpio python python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping libsdl1.2-dev xterm bmap-tools
```

- Yocto pyro 버전은 gcc 6이상에서 빌드된다고 하니 gcc도 upgrade 해준다.

```bash
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt-get update
sudo apt-get install gcc-6 g++-6

sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-6 60
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-6 60
```

### b. 해봤는데 안되는 것

- WSL(windows subsystem for Linux)1에서는 안됨 
- openembedded가 WSL1을 지원 안한다.. WSL2는 가능하다고 한다.
- clone한 poky 폴더가 home 폴더에 있어야 함.
- Ubuntu 16.04에서 gcc 6.5 설치된 환경에서 성공

## 2. Yocto build (pyro version)

### b. poky 및 Reapberrypi meta 파일 clone

```bash
git clone  git://git.yoctoproject.org/poky
cd poky
git checkout pyro

git clone -b pyro git://git.yoctoproject.org/meta-raspberrypi
git clone -b pyro git://git.openembedded.org/meta-openembedded
```

### c. build 환경으로 전환

```bash
source oe-init-build-env build
```

### 4. configure file 수정

- bblayer.conf 수정

    ```bash
    vi conf/bblayers.conf
    ```

    : raspberrypi 및 openembedded meta file 추가

    ```bash
    # POKY_BBLAYERS_CONF_VERSION is increased each time build/conf/bblayers.conf
    # changes incompatibly
    POKY_BBLAYERS_CONF_VERSION = "2"

    BBPATH = "${TOPDIR}"
    BBFILES ?= ""

    BBLAYERS ?= " \
      {HOME PATH}/poky/meta \
      {HOME PATH}/poky/meta-poky \
      {HOME PATH}/poky/meta-yocto-bsp \
      
      {HOME PATH}/poky/meta-raspberrypi \
      {HOME PATH}/poky/meta-openembedded/meta-oe \
      {HOME PATH}/poky/meta-openembedded/meta-multimedia \
      {HOME PATH}/poky/meta-openembedded/meta-networking \
      {HOME PATH}/poky/meta-openembedded/meta-python \
      "
    ```

- local.conf 수정

    ```bash
    vi conf/local.conf
    ```

    MACHINE 항목에 "raspberrypi3" 추가하고 OpenCV ****옵션도 추가한다. 

    부팅시켜보니 OpenCV 4.4 lib가 추가되어 있다. (DNN module은 없는 것 같다..)

    ```bash
    # There are also the following hardware board target machines included for
    # demonstration purposes:
    #
    #MACHINE ?= "beaglebone"
    #MACHINE ?= "genericx86"
    #MACHINE ?= "genericx86-64"
    #MACHINE ?= "mpc8315e-rdb"
    #MACHINE ?= "edgerouter"
    #
    # This sets the default machine to be qemux86 if no other machine is selected:
    MACHINE ??= "qemux86"
    MACHINE ?= "raspberrypi3" # 추가

    # OpenCV 관련 항목 추가 (dnn은 추가하면 error 발생..)
    # NOTE: OPENCV_contrib is already include inside opencv package
    # add libopencv-XXX to insert new module 
    CORE_IMAGE_EXTRA_INSTALL += "opencv \
                                libopencv-core-dev \
                                libopencv-highgui-dev \
                                libopencv-imgproc-dev \ 
                                libopencv-objdetect-dev \
                                libopencv-ml-dev \
                                opencv-apps \
                                opencv-dev \
                                libopencv-xphoto \
                                opencv-dev "
    ```

### 5. bitbake 실행

- 자기전에 build 돌려서 퇴근하니 끝나있었다.. 가상머신이 아닌 local에서 하면 더 빠를것이다.

```bash
bitbake rpi-hwup-image
```

![yocto_result](./img/yocto1.png)


### 6. SD card에 복사

- build 완료되면 "{HOME_PATH}/poky/build/tmp/deploy/images/raspberrypi3"에 "rpi-hwup-image-raspberrypi3.wic.bz2"가 생성됨.
- 기존엔 *.sdimg* 파일이었는데 변경되었다고 한다. ([link](https://github.com/agherzan/meta-raspberrypi/issues/637)) 아직 윈도우환경을 지원하는 writer를 찾지 못했으므로 linux에서 실행.
- bmaptool을 이용하여 sdcard로 복사한다.

```bash
cd {HOME_PATH}/poky/build/tmp/deploy/images/raspberrypi3
sudo bmaptool copy rpi-hwup-image-raspberrypi3.wic.bz2 /dev/{DB_PATH}
# sdcard 위치 "/dev/{DB_PATH}"는 lsblk 명령어로 찾을 수 있다
```

- Laptop의 내장 sdcard는 virtualbox에서 인식이 안된다.... usb로 연결되는 리더기가 필요..
- raspberrypi에 삽입 후 전원을 넣으면 부팅이 시작된다.