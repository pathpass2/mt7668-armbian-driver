从https://github.com/fujianzz/mt7668-armbian处fork   设备：云南电信ty1608-高安版  安卓4.4.2   s905l3b mt7668的无线模块    通过tf卡启动，未写入emmc，保留了原IPTV   
大佬仓库的代码有一点小问题   在Armbian OS 25.08.0-5.15.187-ophub 下编译总是报错，不管是armbian本机编译还是ubuntu22.04交叉编译都报错，根据ai提示改了下，实测armbian-5.15.187本机上编译通过.并成功加载WiFi

<img width="1306" height="735" alt="image" src="https://github.com/user-attachments/assets/377477c6-9a89-4528-81c0-64d5b73b7d65" />


以下是在ty1608-s905l3b上编译过程


arm交叉编译器下载地址   
https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads
<img width="1339" height="238" alt="image" src="https://github.com/user-attachments/assets/d8f50cb5-9532-4d32-b650-fe49bd6375dc" />
按照提示  下载arm-gnu-toolchain-14.2.rel1-aarch64-aarch64-none-elf.tar.xz或者arm-gnu-toolchain-14.2.rel1-aarch64-aarch64-none-linux-gnu.tar.xz   二选一



设置gcc
tar xf arm-gnu-toolchain-14.2.rel1-aarch64-aarch64-none-elf.tar.xz    

mv arm-gnu-toolchain-14.2.rel1-aarch64-aarch64-none-elf   /usr/local/

echo 'export PATH=$PATH:/usr/local/arm-gnu-toolchain-14.2.rel1-aarch64-aarch64-none-elf/bin' | sudo tee -a /etc/profile.d/gcc-aarch64-none-elf.sh

source /etc/profile

ln -sf /usr/local/arm-gnu-toolchain-14.2.rel1-aarch64-aarch64-none-elf/bin/aarch64-none-elf-gcc /usr/local/bin/gcc
<img width="1318" height="862" alt="image" src="https://github.com/user-attachments/assets/16537002-90a0-4053-aa71-8ca1853b2e3c" />


编译mt7688

cd mt7668-armbian/MT7668-WiFi
nano Makefile.x86
#第3行 ,  第28行的x86改成arm64
修改src路径为linux-header src路径


<img width="1090" height="541" alt="image" src="https://github.com/user-attachments/assets/e886e21a-15e6-4f5f-962c-683d8bd50897" />


<img width="892" height="460" alt="image" src="https://github.com/user-attachments/assets/c0de1267-eb56-49e3-9661-c0c2ef638016" />

make  EXTRA_CFLAGS="-w" CROSS_COMPILE= -f Makefile.x86 -j4

编译好的WiFi驱动   带sdio的是需要用到的驱动  
<img width="1302" height="301" alt="image" src="https://github.com/user-attachments/assets/506086d0-a9b6-4466-a2b6-4960bd698dc1" />


mkdir  /lib/modules/5.15.187-ophub/kernel/drivers/net/wireless/mediatek/mt7668

cp -a drv_wlan/MT6632/wlan/{wlan_mt76x8_sdio.ko,wlan_mt76x8.ko}   /lib/modules/5.15.187-ophub/kernel/drivers/net/wireless/mediatek/mt7668/

cp -a  7668_firmware/* /usr/lib/firmware/

modprobe  cfg80211

insmod  /lib/modules/5.15.187-ophub/kernel/drivers/net/wireless/mediatek/mt7668/wlan_mt76x8_sdio.ko


depmod -a

添加到/etc/modules
root@armbian:~# cat /etc/modules

cfg80211

wlan_mt76x8_sdio






重启，开机后看下WiFi驱动是否正常加载


5.加载这个驱动后，会导致有线掉IP，需要配置
nano /etc/NetworkManager/system-connections/Wired\ connection\ 1.nmconnection


[ethernet]
duplex=full
speed=100


<img width="489" height="172" alt="image" src="https://github.com/user-attachments/assets/4a74bde5-5223-44bc-aa88-91232eee4193" />


加载完驱动  连接WiFi后的效果  只测试了WiFi  蓝牙未测试
<img width="1294" height="415" alt="image" src="https://github.com/user-attachments/assets/ee0b8216-a1f4-4aae-ab6a-9f0834fb1fcd" />

<img width="1174" height="166" alt="image" src="https://github.com/user-attachments/assets/06fda2db-15e8-4df9-b071-edc55ff01527" />
