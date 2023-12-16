<div align="center">
  <h1>Android™ Debug Bridge (adb)</h1>
  <img src="images/main.png" alt="Android Image">
    <h2>Welcome to the Comprehensive Guide to Android Debug Bridge (ADB)</h2>
  </div>
</p>

The Android Debug Bridge is a programming tool used for the debugging of Android-based devices. The daemon on the Android device connects with the server on the host PC over USB or TCP, which connects to the client that is used by the end-user over TCP

***

## Installation

* Gentoo Linux

```bash
emerge --ask dev-util/android-sdk-update-manager dev-util/android-tools
```

* RedHat/Fedora

```bash
dnf install adb
```

* Debian/Kali/Ubuntu

```bash
apt install adb fastboot
```

* GNU/Linux (source)

```bash
1) Download the Android SDK Platform Tools ZIP file for Linux.
2) Extract the ZIP to an easily-accessible location (like the Desktop for example).
3) Open a Terminal window.
4) Enter the following command: cd /path/to/extracted/folder/
5) This will change the directory to where you extracted the ADB files.
6) So for example: cd /Users/Doug/Desktop/platform-tools/
7. Connect your device to your Linux machine with your USB cable. 
   Change the connection mode to “file transfer (MTP)” mode. 
   This is not always necessary for every device, but it’s recommended so you don’t run into any issues.
8) Once the Terminal is in the same folder your ADB tools are in, you can execute the following command to launch the ADB daemon: ./adb devices
9) Back on your smartphone or tablet device, you’ll see a prompt asking you to allow USB debugging. Go ahead and grant it.
```

* MacOS (source)

```bash
1) Download: https://dl.google.com/android/repository/platform-tools-latest-windows.zip
2) Extract the contents of this ZIP file into an easily accessible folder (such as C:\platform-tools)
3) Open Windows explorer and browse to where you extracted the contents of this ZIP file
4) Then open up a Command Prompt from the same directory as this ADB binary. This can be done by holding 
     Shift and Right-clicking within the folder then click the “Open command window here” option. 
5) Connect your smartphone or tablet to your computer with a USB cable. 
      Change the USB mode to “file transfer (MTP)” mode. Some OEMs may or may not require this, 
      but it’s best to just leave it in this mode for general compatibility.
6) In the Command Prompt window, enter the following command to launch the ADB daemon: adb devices
7) On your phone’s screen, you should see a prompt to allow or deny USB Debugging access. Naturally, 
      you will want to grant USB Debugging access when prompted (and tap the always allow check box if you never want to see that prompt again).
8) Finally, re-enter the command from step 
      you should now see your device’s serial number in the command prompt (or the PowerShell window).
```


### Print how many times device has booted since last factory reset

```bash
adb shell settings list global|awk -F= '/^boot_count/ {print $2}'
```
    
### Launch accessbility window

```bash
adb shell am start com.samsung.accessibility/com.samsung.accessibility.homepage.AccessibilityHomepageActivityForSUW
```

### Launch accessbility window

```bash
adb shell am start com.samsung.accessibility/com.samsung.accessibility.core.winset.activity.SubSettings
```

### Dumping Information of Current Application in Use (Android 12/13)

```bash
adb shell dumpsys activity|grep -i mCurrentFocus|awk 'NR==1{print $3}'|cut -d'}' -f1
```

### Initiates a Wi-Fi scan on the Android device

```bash
adb shell cmd -w wifi start-scan
sleep 7
adb shell cmd -w wifi list-scan-results
```
    
### Indicating Device Completion

* Content 

```bash
adb shell content insert --uri content://settings/global --bind name:s:device_provisioned --bind value:i:1
adb shell content insert --uri content://settings/secure --bind name:s:user_setup_complete --bind value:i:1
 ```

* Broadcast

```bash
adb shell am broadcast -a android.intent.action.SETTING --es setting:global device_provisioned --ez value 1
adb shell am broadcast -a android.intent.action.SETTING --es setting:secure user_setup_complete --ez value 1
```

* Settings

```bash
adb shell am broadcast -a android.intent.action.SETTING --es setting:global --esa value '[{"name":"device_provisioned","value":"1"}]'
adb shell am broadcast -a android.intent.action.SETTING --es setting:secure --esa value '[{"name":"user_setup_complete","value":"1"}]'
```

## Scripts

### Connect to adb over wifi by copy/paste

```bash
#!/bin/bash
# Author: wuseman

port="5555"

interface=$(adb shell ip addr | awk '/state UP/ {print $2}' | sed 's/.$//'; )
ip=$(adb shell ifconfig ${interface} \
    |egrep  -o '(\<([0-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\>\.){3}\<([0-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\>' 2> /dev/null \
    |head -n 1)

adb tcpip ${port}
sleep 0.5
adb connect $ip:${port}
sleep 1.0
adb shell
```

### Read screen via uiautomor and print IMEI for all active esim/sim cards on device

```bash
#!/bin/bash
# Author: PGissleholm

adb shell input keyevent KEYCODE_CALL;
sleep 1;
input text '*#06#'; 
uiautomator dump --compressed /dev/stdout\
    |tr ' ' '\n'\
    |awk -F'"' '{print $2}'|grep "^[0-9]\{15\}$" \
    |nl -w 1 -s':'\
    |sed 's/^/IMEI/g'   
```

#### Screen State

```bash
#!/bin/bash
# Author: PGissleholm

screenState="$(adb shell dumpsys input_method \
|grep -i "mSystemReady=true mInteractive=" \
|awk -F= '{print $3}')"
if [[ $screenState = "true" ]]; then 
    echo "Screen is on"; 
else 
    echo "Screen is off"; 
fi
```

## Aliases

### Aliases For ADB (add them to ~/.bashrc)

```bash
#!/bin/bash
# Author: N/A

cat <<! >> ~/.bashrc

function startintent() {
  adb devices \
    |tail -n +2 \
    |cut -sf 1 \
    |xargs -I X adb -s X shell am start $(1)

function apkinstall() {
  adb devices \
    |tail -n +2 \
    |cut -sf 1 \
    |xargs -I X \
      adb -s X install -r $(1)
}

function rmapp() {
  adb devices \
    |tail -n +2 \
    |cut -sf 1 \
    |xargs -I X adb -s X uninstall $(1)
}

function clearapp() {
  adb devices \
    |tail -n +2 \
    |cut -sf 1 \
    |xargs -I X adb -s X shell cmd package clear $(1)
}
```

### Print Warranty Status (Samsung)

```bash
warranty=$(adb shell getprop ro.boot.warranty_bit)
if [[ $warranty -ne 0 ]]; then
  echo "Warranty is voided and device has been rooted"
else
  echo "Warranty is valid and device is not rooted"
fi
```

### Dumping All Launchable Activities for a Preferred Application"

```bash
adb shell dumpsys package com.android.settings \
|grep -Eo "^[[:space:]]+[0-9a-f]+[[:space:]]+com.android.settings/[^[:space:]]+Activity" \
|grep -io com.* \
|sed "s/^/adb shell am start -n '/g" \
|sed "s/$/'/g"
```

    
### Dump Unique Package Names from 'dumpsys package' History"

```bash
adb shell dumpsys package\
|grep -Eo "^[[:space:]]+[0-9a-f]+[[:space:]]+.*/[^[:space:]]+" \
|grep -oE "[^[:space:]]+$" \
|uniq
```

### Dumping Launchable Activities for All Installed Applications on the Device"

* Simple for loop

```bash
#!/bin/bash
# Author: PGissleholm

packages=$(adb shell cmd package list packages | awk -F':' '{ print $2 }' | sort)

for package in $packages
do
    echo "Processing $package"
    adb shell dumpsys package $package | \
    grep -io com.*activity | \
    sed "s/^/adb shell am start -n '/;s/$/\'/g" | sed 's/\$/\\$/g'
        ```

* GNU/Parallel

```bash
#!/bin/bash
# Author: PGissleholm

packages=$(adb shell cmd package list packages | awk -F':' '{ print $2 }' | sort)

do_work() {
    package=$1
    echo "Processing $package"
    
    adb shell dumpsys package $package | \
    grep -io com.*activity | \
    sed "s/^/adb shell am start -n '/;s/$/\'/g" | sed 's/\$/\\$/g'
}

export -f do_work
echo "$packages" | parallel -j4 do_work
```

* Xargs (Recommendend/Fastest)

```bash
#!/bin/bash
# Author: PGissleholm

packages=$(adb shell cmd package list packages | awk -F':' '{ print $2 }' | sort)

do_work() {
    package=$1
    echo "Processing $package"
    
    adb shell dumpsys package $package | \
    grep -io com.*activity | \
    sed "s/^/adb shell am start -n '/;s/$/\'/g" | sed 's/\$/\\$/g'
}

export -f do_work

echo "$packages" | xargs -I {} -P 4 -n 1 bash -c 'do_work "$@"' _ {}
```

### Dump Receivers: `Samsung Telephony UI `

```bash
app="com.samsung.android.app.telephonyui"
adb shell dumpsys package| grep -Eo "^[[:space:]]+[0-9a-f]+[[:space:]]+${app}/[^[:space:]]+"|grep -oE "[^[:space:]]+Receiver"
```

### Analyzing and Interacting with Application Components

* Simple for loop

```bash
#!/bin/bash
# Author: PGissleholm

packages=$(adb shell cmd package list packages | awk -F':' '{ print $2 }' | sort)

for package in $packages
do
    echo -e "\n- Processing $package"
    
    adb shell cmd package dump $package | \
    grep -io com.*activity | \
    sed "s/^/adb shell am start -n '/;s/$/\'/g" | sed 's/\$/\\$/g'
done
```

* GNU/Parallel

```bash
#!/bin/bash
# Author: PGissleholm

packages=$(adb shell cmd package list packages | awk -F':' '{ print $2 }' | sort)

do_work() {
    package=$1
    echo "Processing $package"
    
    adb shell cmd package dump $package | \
    grep -io com.*activity | \
    sed "s/^/adb shell am start -n '/;s/$/\'/g" | sed 's/\$/\\$/g'
}

export -f do_work

echo "$packages" | xargs -I {} -P 4 -n 1 bash -c 'do_work "$@"' _ {}
```

* Xargs (Recommendend/Fastest)

```bash
#!/bin/bash
# Author: PGissleholm

packages=$(adb shell cmd package list packages | awk -F':' '{ print $2 }' | sort)

do_work() {
    package=$1
    echo "Processing $package"
    
    adb shell cmd package dump $package 
}

export -f do_work

echo "$packages" | parallel -j4 do_work
```

### UDEV Rules

```bash
SUBSYSTEM=="usb", ATTR{idVendor}=="0502", MODE="0666", GROUP="plugdev" #Acer
SUBSYSTEM=="usb", ATTR{idVendor}=="0b05", MODE="0666", GROUP="plugdev" #ASUS
SUBSYSTEM=="usb", ATTR{idVendor}=="413c", MODE="0666", GROUP="plugdev" #Dell
SUBSYSTEM=="usb", ATTR{idVendor}=="0489", MODE="0666", GROUP="plugdev" #Foxconn
SUBSYSTEM=="usb", ATTR{idVendor}=="04c5", MODE="0666", GROUP="plugdev" #Fujitsu
SUBSYSTEM=="usb", ATTR{idVendor}=="04c5", MODE="0666", GROUP="plugdev" #Fujitsu Toshiba
SUBSYSTEM=="usb", ATTR{idVendor}=="091e", MODE="0666", GROUP="plugdev" #Garmin-Asus
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", MODE="0666", GROUP="plugdev" #Google
SUBSYSTEM=="usb", ATTR{idVendor}=="201E", MODE="0666", GROUP="plugdev" #Haier
SUBSYSTEM=="usb", ATTR{idVendor}=="109b", MODE="0666", GROUP="plugdev" #Hisense
SUBSYSTEM=="usb", ATTR{idVendor}=="0bb4", MODE="0666", GROUP="plugdev" #HTC
SUBSYSTEM=="usb", ATTR{idVendor}=="12d1", MODE="0666", GROUP="plugdev" #Huawei
SUBSYSTEM=="usb", ATTR{idVendor}=="24e3", MODE="0666", GROUP="plugdev" #K-Touch
SUBSYSTEM=="usb", ATTR{idVendor}=="2116", MODE="0666", GROUP="plugdev" #KT Tech
SUBSYSTEM=="usb", ATTR{idVendor}=="0482", MODE="0666", GROUP="plugdev" #Kyocera
SUBSYSTEM=="usb", ATTR{idVendor}=="17ef", MODE="0666", GROUP="plugdev" #Lenovo
SUBSYSTEM=="usb", ATTR{idVendor}=="1004", MODE="0666", GROUP="plugdev" #LG
SUBSYSTEM=="usb", ATTR{idVendor}=="22b8", MODE="0666", GROUP="plugdev" #Motorola
SUBSYSTEM=="usb", ATTR{idVendor}=="0e8d", MODE="0666", GROUP="plugdev" #MTK
SUBSYSTEM=="usb", ATTR{idVendor}=="0409", MODE="0666", GROUP="plugdev" #NEC
SUBSYSTEM=="usb", ATTR{idVendor}=="2080", MODE="0666", GROUP="plugdev" #Nook
SUBSYSTEM=="usb", ATTR{idVendor}=="0955", MODE="0666", GROUP="plugdev" #Nvidia
SUBSYSTEM=="usb", ATTR{idVendor}=="2257", MODE="0666", GROUP="plugdev" #OTGV
SUBSYSTEM=="usb", ATTR{idVendor}=="10a9", MODE="0666", GROUP="plugdev" #Pantech
SUBSYSTEM=="usb", ATTR{idVendor}=="1d4d", MODE="0666", GROUP="plugdev" #Pegatron
SUBSYSTEM=="usb", ATTR{idVendor}=="0471", MODE="0666", GROUP="plugdev" #Philips
SUBSYSTEM=="usb", ATTR{idVendor}=="04da", MODE="0666", GROUP="plugdev" #PMC-Sierra
SUBSYSTEM=="usb", ATTR{idVendor}=="05c6", MODE="0666", GROUP="plugdev" #Qualcomm
SUBSYSTEM=="usb", ATTR{idVendor}=="1f53", MODE="0666", GROUP="plugdev" #SK Telesys
SUBSYSTEM=="usb", ATTR{idVendor}=="04e8", MODE="0666", GROUP="plugdev" #Samsung
SUBSYSTEM=="usb", ATTR{idVendor}=="04dd", MODE="0666", GROUP="plugdev" #Sharp
SUBSYSTEM=="usb", ATTR{idVendor}=="054c", MODE="0666", GROUP="plugdev" #Sony
SUBSYSTEM=="usb", ATTR{idVendor}=="0fce", MODE="0666", GROUP="plugdev" #Sony Ericsson
SUBSYSTEM=="usb", ATTR{idVendor}=="2340", MODE="0666", GROUP="plugdev" #Teleepoch
SUBSYSTEM=="usb", ATTR{idVendor}=="0930", MODE="0666", GROUP="plugdev" #Toshiba
SUBSYSTEM=="usb", ATTR{idVendor}=="19d2", MODE="0666", GROUP="plugdev" #ZTE
```



### Dump All Activities That Is Available from: `adb shell am start`

<video src="video/preview_activies.mp4" data-canonical-src="video/preview_activies.mp4" controls="controls" muted="muted" class="d-block rounded-bottom-2 border-top width-fit" style="max-height:640px; min-height: 200px"></video>

```bash
#!/bin/bash
# Author: PGissleholm

for package in $(adb shell cmd package list packages|cut -d: -f2); do 
  adb shell cmd package dump $package \
  |grep -i "activ" \
  |grep -Eo "^[[:space:]]+[0-9a-f]+[[:space:]]+.*/[^[:space:]]+" \
  |grep -oE "[^[:space:]]+$"; 
done > /tmp/full_activity_package_list.txt
```

## Enter Termux Enviroment

```bash
#!/system/xbin/bash
# Author: rewida17
# Modded by: PGissleholm

if [[ $(id -u) -ne "0" ]]; then 
  echo "root is requried"; 
  exit
else
  export PREFIX='/data/data/com.termux/files/usr'
  export HOME='/data/data/com.termux/files/home'
  export LD_LIBRARY_PATH='/data/data/com.termux/files/usr/lib'
  export PATH="/data/data/com.termux/files/usr/bin:/data/data/com.termux/files/usr/bin/applets:$PATH"
  export LANG='en_US.UTF-8'
  export SHELL='/data/data/com.termux/files/usr/bin/bash'
  export BIN='/data/data/com.termux/files/usr/bin' 
  export TERM=vt220
  export AR="arm-linux-androideabi-ar"
  export CPP="arm-linux-androideabi-cpp"
  export GCC="arm-linux-androideabi-gcc"
  export LD="arm-linux-androideabi-ld"
  export NM="arm-linux-androideabi-nm"
  export OBJDUMP="arm-linux-androideabi-objdump"
  export RANLIB="arm-linux-androideabi-ranlib"
  export READELF="arm-linux-androideabi-readelf"
  export STRIP="arm-linux-androideabi-strip"
  export TERMUX="/data/data/com.termux/"

  resize
  cd "$HOME"
  exec "$SHELL" -l
fi
```

## Project Time

* This repository has been shared earlier and there is hundred of hours behind all my work here but since I re-started and re-doing this repo on this current user/repo-name started from: 2023-12-06 is counted here for fun 

* This is for `README.md` here at Github only, this is not included all other pages on adb-shell.com, it is extremely much more time if we are to compare this but unfortunately do not have with this time here publicly.

<center>Total time spent since the move to adb-shell.com | </center> | <center><a href="https://wakatime.com/badge/github/PGissleholm/adb-shell"><img src="https://wakatime.com/badge/github/PGissleholm/adb-shell.svg?style=social" alt="wakatime"></a></center>

## Archived Pages

Note: $\textit{( Incomplete Section )}$

This repository was shared from my old user before and from another host <a href="https://wakatime.com/badge/github/wuseman">https://android.nr1.nu (dead url/domain)</a>, 
you can find some archived pages that may contain more/less examples then here but I am trying to include everything so hopefully this `README.md` is most complete

##### February 12, 2023

* [07:56:26](https://web.archive.org/web/20230212075626/https://github.com/wuseman/adb-shell)
* [13:04:01](https://web.archive.org/web/20230212130401/https://github.com/wuseman/adb-shell)
* [23:36:32](https://web.archive.org/web/20230212233632/https://github.com/wuseman/adb-shell)
*
##### February 13, 2023

* [03:42:04](https://web.archive.org/web/20230000000000*/https://github.com/wuseman/adb-shell)
* [10:36:49](https://web.archive.org/web/20230000000000*/https://github.com/wuseman/adb-shell)
* [10:37:22](https://web.archive.org/web/20230000000000*/https://github.com/wuseman/adb-shell)
* [13:51:06](https://web.archive.org/web/20230000000000*/https://github.com/wuseman/adb-shell)

... This history/archive section will be updated

