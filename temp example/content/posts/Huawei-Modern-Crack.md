---
title: "Huawei Modern Crack"
date: 2020-03-22T02:33:16+08:00
draft: false
---

前幾天朋友升級了中國電信1000兆的頻寬，讓我幫他破解一下。有新玩具爲什麼不呢？幹！

<!--more-->

首先，他的光貓還沒有插過光纖，電信也冇上門調試。所以超管的密碼還沒有變，如果已經配置過了的話，建議復位Modern在插光纖前繼續閱讀本文（請記得LOID）

電信小哥上門搞定之後，發現了幾點問題，在註冊完成後會自動覆蓋之前的配置文件，所以還是需要兆電信小哥要到超管密碼

魔改有風險，重灌也有風險，請自己考慮

# 折騰

1. 首先下載解密工具[ctce_cfg.exe](https://raw.githubusercontent.com/Mr-Sheep/Mr-Sheep.github.io/master/resource/ctce_cfg.exe)

2. 登入光貓的後臺`192.168.1.1`，使用超管賬戶名以及密碼登入

   ```
   telecomadmin
   nE7jA%5m
   ```

3. 插入U盤

4. 管理--> 設備管理 --> 備份配置

5. 將U盤目錄下`e8_Config_Backup`資料夾內的`ctce8_HN8145V.cfg`拷貝到ctce_cfg.exe目錄下

6. 執行`./ctce_cfg.exe 1 ctce8_HN8145V.cfg ctce8_HN8145V.xml.gz`

   (我的bash用的wsl，如果是cmd請自行去掉前面`./`)

7. 將上一步得到的`ctce8_HN8145V.xml.gz`解壓得到`ctce8_HN8145V.xml`

8. 編輯該文件，查找關鍵詞`telecomadmin`，找到一下區塊

   ```xml
   <X_HW_WebUserInfo NumberOfInstances="2">
   <X_HW_WebUserInfoInstance InstanceID="1" ModifyPasswordFlag="0" UserName="useradmin" Password="$2+GoG!!1.CP7s^&amp;&apos;H()oUk~Ol4NSi8TgTYK9fDjp5$" UserLevel="1" Enable="1" Alias="cpe-1"/>
   <X_HW_WebUserInfoInstance InstanceID="2" ModifyPasswordFlag="1" UserName="telecomadmin" Password="$29wO[/pt}xRd&apos;fdGtk928k6(j/$&lt;vhSm1s4C_tgW6$" UserLevel="0" Enable="1" Alias="cpe-2" PassMode="0"/>
   </X_HW_WebUserInfo>
   ```

   複製粘貼超級管理員的配置到新的一行，我在查閱資料的時候大佬們的xml文件中密碼都是明文，可能是因爲華爲系統要開源了，2019版的8145V的密碼不是明文，在我不知道其算法的情況下，直接複製粘貼了telecomadmin的密文。

   ```xml
   <X_HW_WebUserInfo NumberOfInstances="3">
   <X_HW_WebUserInfoInstance InstanceID="1" ModifyPasswordFlag="0" UserName="useradmin" Password="$2+GoG!!1.CP7s^&amp;&apos;H()oUk~Ol4NSi8TgTYK9fDjp5$" UserLevel="0" Enable="1" Alias="cpe-1" PassMode="0"/>
   <X_HW_WebUserInfoInstance InstanceID="2" ModifyPasswordFlag="1" UserName="telecomadmin" Password="$29wO[/pt}xRd&apos;fdGtk928k6(j/$&lt;vhSm1s4C_tgW6$" UserLevel="0" Enable="1" Alias="cpe-2" PassMode="0"/>
   <X_HW_WebUserInfoInstance InstanceID="3" ModifyPasswordFlag="1" UserName="mrsheep" Password="$29wO[/pt}xRd&apos;fdGtk928k6(j/$&lt;vhSm1s4C_tgW6$" UserLevel="0" Enable="1" Alias="cpe-3" PassMode="0"/>
   
   </X_HW_WebUserInfo>
   ```

   注意：

   - `NumberOfInstances`請修改爲實際變更後的用戶數量

   - 新增用戶請變更`InstanceID`

   - `Alias`不想寫可以去掉

   - 我修改的時候同時給useradmin進行了提權

   - 一些修改

     - [ ] 破除連接限制

       搜索`TotalTerminalNumbe`,將其值變更爲`0`

     - [ ] 開啓Telnet功能

       搜索`TELNETLanEnable`，將其值變更爲`1`

     - [ ] 禁用自動遠程管理

       `EnableCWMP`，將其值變更爲`0`

9. 將修改後的文件gzip壓縮，這裏建議刪除解密時創建的gzip文件和原cfg文件

10. `ctce_cfg.exe 0 ctce8_HS8145V.xml.gz ctce8_HS8145V.cfg`重新加密生成cfg文件

11. 將重新生成的文件替換到U盤`e8_Config_Backup`資料夾內

12. 管理--> 設備管理 --> ✅快速恢復-->重啓

13. 嘗試登錄

14. 網絡-->遠程管理-->❎使能週期通知，切斷遠程控制

    注意：

    這裏的Shell並沒有補全

# 參考連結

[华为光猫新版配置文件加解密工具(适用V3R016C/V2R006C等配置内容头4字节为gfcq的版本)](http://www.chinadsl.net/thread-125291-1-1.html)

