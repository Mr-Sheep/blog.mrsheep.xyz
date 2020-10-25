---
title: "Fix Adobe Crash on AMD CPUs"
date: 2020-10-25T16:49:04+08:00
draft: false
tags: ["V2ray", "HTML"]
categories: ["anti-censorship"]
---

## Adobe Crash Fix
Found on Github, just for personal record
<!-- more-->
Run the following code in your terminal:

```
for file in MMXCore FastCore TextModel libiomp5.dylib; do
    find /Applications/Adobe* -type f -name $file | while read -r FILE; do
        sudo -v
        echo "found $FILE"
        [[ ! -f ${FILE}.back ]] && sudo cp -f $FILE ${FILE}.back || sudo cp -f ${FILE}.back $FILE
        echo $FILE | grep libiomp5 >/dev/null
        if [[ $? == 0 ]]; then
            dir=$(dirname "$FILE")
            [[ ! -f ${HOME}/libiomp5.dylib ]] && cd $HOME && curl -sO https://excellmedia.dl.sourceforge.net/project/badgui2/libs/mac64/libiomp5.dylib
            echo -n "replacing " && sudo cp -vf ${HOME}/libiomp5.dylib $dir && echo
            rm -f ${HOME}/libiomp5.dylib
            continue
        fi
        echo $FILE | grep TextModel >/dev/null
        [[ $? == 0 ]] && echo "emptying $FILE" && sudo echo -n >$FILE && continue
        echo "patching $FILE \n"
        sudo perl -i -pe 's|\x90\x90\x90\x90\x56\xE8\x6A\x00|\x90\x90\x90\x90\x56\xE8\x3A\x00|sg' $FILE
        sudo perl -i -pe 's|\x90\x90\x90\x90\x56\xE8\x4A\x00|\x90\x90\x90\x90\x56\xE8\x1A\x00|sg' $FILE
    done
done
```

After it finished, run this: 

```
[ ! -d $HOME/Library/LaunchAgents ] && mkdir $HOME/Library/LaunchAgents
AGENT=$HOME/Library/LaunchAgents/environment.plist
sysctl -n machdep.cpu.brand_string | grep FX >/dev/null 2>&1
x=$(echo $(($? != 0 ? 5 : 4)))
cat >$AGENT <<EOF
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
 <key>Label</key>
 <string>mkl-debug</string>
 <key>ProgramArguments</key>
 <array>
 <string>sh</string>
 <string>-c</string>
    <string>launchctl setenv MKL_DEBUG_CPU_TYPE $x;</string>
 </array>
 <key>RunAtLoad</key>
 <true/>
</dict>
</plist>
EOF
launchctl load ${AGENT} >/dev/null 2>&1
launchctl start ${AGENT} >/dev/null 2>&1
```
Reboot macOS.



- Tested Adobe (2020) Photopshop, LightRoom Classic, Illustrator, Premier Pro, After Effects Bridge, Indesign, XD.
- If you re-install any adobe app then you will need redo the first command



Source: https://gist.github.com/naveenkrdy/26760ac5135deed6d0bb8902f6ceb6bd