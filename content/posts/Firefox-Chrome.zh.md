---
title: "Firefox vs Chrome"
date: 2021-05-14T12:18:15+08:00
draft: false
tags: ['Firefox']
catagories: ['Privacy']
---

首先是結論：火狐

<!--more-->

下載連結（請注意不要下載到firefox.com.cn的廣告定製版）：

https://www.mozilla.org/en-US/firefox/enterprise/www.mozilla.org 

或者是

https://www.mozilla.org/zh-CN/firefox/new/

**在你開始閱讀之前**

這篇文章本來只是我在知乎上隨手答的，但是看到有人喜歡，我就準備放到博客上。
如果你是從知乎找到這篇文章的，請幫我保密我的知乎身份，感謝

以及，我不是專家，想到哪裡寫到哪裡

任何問題歡迎指出

## 爲甚麼不要Chrome

- Chrome在隱私方面不值得信任 
- Chrome资源消耗高 
- 功能沒有Firefox強大，不支持自定義，不可以想幹嘛幹嘛 
- 你不會希望你的瀏覽器和你的搜索引擎是同一家公司的，這樣的話鬼知道你的瀏覽器上載了甚麼文件
- 火狐由Mozilla基金會開發，而Mozilla 是一家非營利組織。相比之下，谷歌是一家正經的商業公司
- 谷歌很大一部分收入來自於他的廣告業務，你難道沒有發現你油管看多了以後廣告都是相關的嗎

### 0x01 Chrome最主要的缺點

1. 在清除Cookie的時候是默認忽略掉谷歌產品的 
2. 谷歌會永久保存你的搜索紀錄 (現在可以只保留一定時間) 

一份關於谷歌收集的數據的研究[Google Data Collection](https://digitalcontentnext.org/blog/2018/08/21/google-data-collection-research/)

### 0x02 CA

谷歌瀏覽器是直接使用系統自帶的CA，而系統自帶的憑證的提供商又是商業公司，很難確定一個商業公司是否會為了利益爾做出一些手腳。以及CNNIC這樣的神奇組織

火狐採用自己的證書，作為一個非營利組織，更好的迴避了來自於商業公司的影響

> They sell root certificates right?   I've always wondered about that.. It also makes `let's encrypt' a direct competitor with their main customers. No wonder it took ages to get  that of the ground. Reddit [u/superPwnzorMegaMan](https://www.reddit.com/user/superPwnzorMegaMan/) 

### 0x03 Privacy orientated

火狐對自己的介紹：

- "使用會保護重要資料的瀏覽器" 
- "沒有諱莫如深的隱私權保護政策，更不留下後門給廣告商，純屬一套光速快，又不出賣您的瀏覽器。" 
- Firefox的開發目標是「盡情地上網瀏覽」和「對多數人來說最棒的上網體驗」

## 火狐的特點

-  Firefox是完全開源的，而Chrome是Google在Chromium上做出來的商業產品 
-  Firefox支援一大堆隱私保護Feature 
-  Encrypt DNS Query 火狐默認支持DoH等，快速禁用WebRTC和WebGL等

還有一個有意思的事情，如果Chrome向谷歌服務器發送大量數據，用戶不能夠準確的分析出發送了什麼數據，抓包也只能看到一大堆tls流量。

而mozilla僅提供少量的網路服務，Lockwise，profile同步等，設想如果你的Firefox發送了大量數據，必然會引起用戶懷疑。

- Firefox支援多個profile, 瀏覽器可以隔離成多個profile並且支持訂製。
- 火狐[Github repo](https://github.com/mozillagithub.com)

### 0x01 Tracker 

根據一份[普林斯頓大學的研究報告](https://webtransparency.cs.princeton.edu/webcensus/), 市面上現存的大部分Tracker來自於谷歌，臉書等

>  The vast majority of websites across the Internet [contain hidden tracker networks](https://spreadprivacy.com/followed-by-ads/). Google trackers are lurking behind about 75% of pages, Facebook  trackers on about 25%, and countless others are scattered throughout.  These trackers soak up your personal information to follow you with ads  around the web, study your browsing behavior, or worse. 

> This means that  for any site you visit, it is likely Google or Facebook is tracking you  there. It’s downright creepy. But it gets worse. Companies like Google and Facebook [collect all this information](https://www.quora.com/What-does-Google-know-about-me/answer/Gabriel-Weinberg) – all these personal data points – to build intimate profiles of you  online. 

> The information they collect can include where you live, who you live with, how old you are, your eating and shopping preferences,  favorite travel spots, reading habits, music preferences, etc. Then,  they use these profiles to sell hyper-targeted ads to the highest  bidder, effectively auctioning off your personal information over and  over again. 

截取自DDG創始人Quora回答, 提到Google佔有全部追蹤器中的75%，遠超過其餘選手

更加嚴重的問題是，Google等公司會收集所有的相關信息來構建用戶畫像

建議直接閱讀Quora原文：

- [What does Google know about me?](https://www.quora.com/What-does-Google-know-about-me)
- [How does the DuckDuckGo app/extension protect my privacy?](https://www.quora.com/How-does-the-DuckDuckGo-app-extension-protect-my-privacy-1)

當然，我們這裡比較的谷歌和Mozilla，相對於百度來說，商業道德觀念會好很多


### 0x02 Lockwise

順便再說一下Lockwise這個東西:

1. Lockwise使用在瀏覽器中配置的master password在本地加密你的密碼並上傳到，但是只需要在另一台設備上登錄火狐帳戶就可以查看密碼，同時，手機端並不提供這一功能，只要可以訪問你的火狐帳戶，你的密碼也就直接暴露在攻擊者面前
2. 目前為止沒有超時自動鎖定等功能，瀏覽器保持打開就可以查看所有儲存的密碼，想要管理器可以試下KeePass XC,  elpass(iOS, MacOS. Surge作者開發), BitWarden(可以自己搭建)


### 0x03 定製

Firefox支持你隨意的定製，最基本的`about:config`到`omni.ja`都可以隨便改（迫真）

快捷鍵，樣式什麼的，只要你願意去折騰，都可以定製

### 0x04 搜索引擎

雖然不是瀏覽器本身，搜索引擎也十分的重要，個人認爲可以選擇的方案有：

- DuckDuckGo + Firefox
- Startpage + Firefox

**注意**:

DDG本身雖然安全，但是他們的服務器是aws提供的，和下文將會提到的谷歌一個道理，商業公司指不定哪天不高興了就把你信息賣了呢？

startpage通過給谷歌交錢來提供搜索功能，結果會比DDG更多，伺服器數量相對DDG較少。但是Startpage被System1部分持有(一家廣告公司)

SearX看起來很不錯，但是搜索質量實在看不下去

所以也可以嘗試每次使用完Google以後將所有Cookie全部清楚

### 0x05 關於Firefox版本的選擇

不要用火狐智謀出品的定製版火狐

谋智中国开发的[定制主页及快速拨号页扩展](http://download.firefox.com.cn/chinaedition/addons/cehomepage/cehomepage-latest.xpi)（xpi文件）[被发现](http://bbs.kafan.cn/thread-1655820-9-1.html)会修改用户的快捷方式，如果中国用户在主页建立了京东商城（www.jd.com）的拨号快捷键，那么该快捷键会被定向到一个返利链接（http://count.chanet.com.cn/click...www.jd.com”）。该扩展的中国开发者还非常幽默的在源代码里加了一个注解——“do some evil here”。                         

來源：[谋智中国被发现“do some evil”](https://www.solidot.org/story?sid=37355)

### 0x06 Firefox隱私保護設定

在`about:config`頁面完成設定，并可以[https://browserleaks.com/canvas](https://browserleaks.com/canvas) 測試

參考來源：https://www.privacytools.io/browsers/#about_config

如果你希望瞭解更多相關內容，可以查看[Browser Fingerprinting: What Is It and What Should You Do About It?](https://pixelprivacy.com/resources/browser-fingerprinting/)，他們提供了更爲完整和詳細的分析

**建議去原網站仔細查看說明後再配置**
```
- privacy.firstparty.isolate = true
- privacy.resistFingerprinting = true
- privacy.trackingprotection.fingerprinting.enabled = true
- privacy.trackingprotection.cryptomining.enabled = true
- privacy.trackingprotection.enabled = true
- browser.send_pings = false
- browser.urlbar.speculativeConnect.enabled = false
- dom.event.clipboardevents.enabled = false
- media.eme.enabled = false
- media.gmp-widevinecdm.enabled = false
- media.navigator.enabled = false
- network.cookie.cookieBehavior = 1
- network.http.referer.XOriginPolicy = 1
- network.http.referer.XOriginTrimmingPolicy = 2
- webgl.disabled = true
- browser.sessionstore.privacy_level = 2   
- beacon.enabled = false
- browser.safebrowsing.downloads.remote.enabled = false
- Disable Firefox prefetching pages it thinks you will visit next:
- network.dns.disablePrefetch = true    
- network.dns.disablePrefetchFromHTTPS = true    
- network.predictor.enabled = false    
- network.predictor.enable-prefetch = false    
- network.prefetch-next = false  
- network.IDN_show_punycode = true
```
### 0x07 插件

最後再扯一點關於插件的問題，關於隱私和安全方面的話可以選擇EFF的插件

- HTTPS Everywhere: 字面意思 
- No Script: 字面意思 
- uBlock Origin: 廣告屏蔽 
- 火狐官方的Container插件配合temporary container一次性容器 
- Decentraleyes: 阻斷不必要的cdn請求 
- ClearURLs

#### 進階一點的插件

- CanvasBlocker 
- uMatrix(停止維護了 

## 如何切換到Firefox

不可否認，Chrome能夠混到今天這個市場份額，必定有它很優秀的地方，crx的兼容性確實很不錯，不然為什麼那麼多瀏覽器直接抄或者是採用Chromium內核開發？

只要幾分鐘，即可從 Chrome 切換到 Firefox


## 一點冷知識

-  火狐官方對自己的名字簡寫是Fx而不是FF 
-  火狐的英文名字叫做Firefox而不是FireFox 

>  How do I capitalize Firefox? How do I abbreviate it?           Only the first letter is capitalized (so it's Firefox, not FireFox.)  The preferred abbreviation is "Fx" or "fx". 

- Firefox其實不是火狐，是小熊貓的別名

>  A "Firefox" is another name for the [red panda](http://www.bbc.co.uk/nature/wildfacts/factfiles/6.shtml).     https://www-archive.mozilla.org/projects/firefox/firefox-name-faq.html 

-  Firefox之前有過很多名字， 如Pheonix，Firebird等 
-  如果我討厭Firefox這個名字怎麼辦？ 

>  Our editors are trying to figure out whether this is a question.  Of course not everyone will like the new name, especially at first.  We're confident most people will quickly get used to it.          New names have a way of sounding terrible at first.  If you're unhappy with the new name,        consider trying out the many improvements we've made in the latest release of our browser -we hope that'll make you feel better.  After all, what's most important is how the thing works, not what it's called.      

- 其實Mozzila也有出"書", 不信你去輸入about:mozilla 試試？
- https://www-archive.mozilla.org/有很多有趣的內容可以自己去翻閱，本文很多內容都是從那裏翻出來的

## 404 最後

隱私權作為最基本的權利，是否使用該權利是每一個人的選擇

不可否認，Chrome能夠混到今天這個市場份額，必定有它很優秀的地方，crx的兼容性確實很不錯，不然為什麼那麼多瀏覽器直接抄或者是採用Chromium內核開發？

>  Individuals must have the ability to shape the internet and their own experiences on it. -- Principle 7 from the Mozilla Manifesto 