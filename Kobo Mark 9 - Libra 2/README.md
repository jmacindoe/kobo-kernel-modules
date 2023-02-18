Prebuilt kernel modules for the Kobo Libra 2 are provided in this folder.

IMPORTANT NOTE: I don't actually own a Libra 2 and so have not tested these. But since building for the Libra 2 is very similar to the Clara 2E, I put this together anyway. If these work for anyone, please let me know (e.g. open an issue on this repo) and I'll delete this note.

# Building from source
If you want to build them youself, here are the instructions. This has only been tested on Linux (specifically Ubuntu 22.04 amd64). If you're on Windows or a Mac, I'd suggest you run this on a Virtual Machine or by booting into a Live USB. Since we need to cross-compile for the Kobo ARM platform, there could be extra steps to get it working on your platform.

1. Download the official Kobo kernel source code [here](https://github.com/kobolabs/Kobo-Reader/blob/master/hw/imx6sll-libra2/kernel.tar.bz2)
2. Download a cross compiler [here](https://releases.linaro.org/components/toolchain/binaries/). I'm using [this one](https://releases.linaro.org/components/toolchain/binaries/4.9-2017.01/arm-linux-gnueabihf/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf.tar.xz).
3. Extract everything. I'll assume we're working in `~/kobo` for all my commands.
```
mkdir ~/kobo
cd ~/kobo
tar xf kernel.tar.bz2
tar xf gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf.tar.xz
```
4. We need to get the kernel config that was used to build the kernel on your device. It's located at `/proc/config.gz`. So if you have SSH set up, run
```
scp root@192.168.2.2:/proc/config.gz .
gunzip config.gz
cp config kernel/.config
```
Since I don't own a Libra 2, I used the config from my Clara 2E :'( (if someone could upload a Libra 2 config for me I'd really appreciate it)

5. Run oldconfig to check everything is up-to-date
```
cd kernel
make ARCH=arm CROSS_COMPILE=~/kobo/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf- oldconfig
```
6. You can try compiling with this command
```
make ARCH=arm CROSS_COMPILE=~/kobo/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-
```
but it will fail with the following error because our GCC version must be slightly out of sync with Rakuten
```
/usr/bin/ld: scripts/dtc/dtc-parser.tab.o:(.bss+0x50): multiple definition of `yylloc'; scripts/dtc/dtc-lexer.lex.o:(.bss+0x0): first defined here
```
To fix it open `scripts/dtc/dtc-lexer.lex.c` in your favourite editor and find the line
```
YYLTYPE yylloc;
```
replace it with
```
extern YYLTYPE yylloc;
```
7. Compilation still failed for me due to a problem in the Bluetooth code. I suspect this is because I was using a Clara 2E config file (not sure). To fix it, I disabled the Bluetooth code.
```
make ARCH=arm CROSS_COMPILE=~/kobo/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf- menuconfig
```
Under "Networking Support", turn off "Bluetooth subsystem support".

8. Compilation still won't work quite yet because there's an undefined firmware file value. Open `drivers/input/touchscreen/focaltech_touch/focaltech_flash.c` and find this code starting at line 48
```
u8 fw_file[] = {
#include FTS_UPGRADE_FW_FILE
};

u8 fw_file2[] = {
#include FTS_UPGRADE_FW2_FILE
};

u8 fw_file3[] = {
#include FTS_UPGRADE_FW3_FILE
};
```
Delete the undefined `#include` lines, so we end up with
```
u8 fw_file[] = {
};

u8 fw_file2[] = {
};

u8 fw_file3[] = {
};
```
9. Now finally we can compile
```
make ARCH=arm CROSS_COMPILE=~/kobo/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-
```
Compilation of the kernel should succeed, with the all important line `Kernel: arch/arm/boot/zImage is ready`. However it will then fail on the `DTC` step. No matter, we don't need that anyway.
```
  DTC     arch/arm/boot/dts/imx6sll-e70k14b00.dtb
make[1]: *** No rule to make target 'arch/arm/boot/dts/imx6sll-e70k02-nightmode.dtb', needed by '__build'.  Stop.
make: *** [arch/arm/Makefile:327: dtbs] Error 2
```
10. Use menu config to enable any modules you want to compile
```
make ARCH=arm CROSS_COMPILE=~/kobo/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf- menuconfig
```
11. Build your modules
```
make ARCH=arm CROSS_COMPILE=~/kobo/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf- modules
```
12. Your modules are ready! Their location will be printed in the output of the last command.
