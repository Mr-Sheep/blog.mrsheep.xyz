---
title: "Firefox Safty & Privacy"
date: 2020-02-27T12:18:15+08:00
draft: true
---

火狐，

[火狐](https://www.mozilla.org/zh-TW/firefox/),Mozilla基金會開發，和Chrome相比，隱私性能夠得到很大的保證，本文比較水，大概包含爲什麼選擇火狐以及如何讓火狐更protect your privacy

<!--more-->

# 瀏覽器市場佔比

開始之前：本文提到的所有Firefox均與中國大陸的firefox.com.cn所發行的firefox**無關**

雖然說Mozilla China屬於Mozilla全資擁有，但是所處地區的信息安全與隱私得不到任何保障,並且有明顯的添加廣告的行爲。

根據[netmarketshare](https://netmarketshare.com/browser-market-share.aspx)的一份報告

| Name              | Share  |
| ----------------- | ------ |
| Chrome            | 67.63% |
| Firefox           | 8.83%  |
| Internet Explorer | 7.26%  |
| Edge              | 5.77%  |
| Safari            | 3.60%  |
| Sogou Explorer    | 1.65%  |

可見目前Chrome的佔有率極高，必定有它很優秀的地方，crx的兼容性確實很不錯，不然為什麼那麼多瀏覽器直接抄或者是採用Chromium內核開發？

Firefox 5.0之后开始和chrome一样，每几个星期大版本迭代，同时还拥有ESR版本，类似于LTS版本的系统.

Tor匿名網絡，基於Firefox ESR的同時也向火狐提供支持[Security/Tor Uplift/Tracking](https://wiki.mozilla.org/Security/Tor_Uplift/Tracking)



# 爲什麼選用Firefox

### Chrome在隱私方面不值得信任

首先，[Firefox](https://www.mozilla.org/zh-TW/firefox/)是[Mozilla基金會](https://www.mozilla.org/en-US/about/)開發的[開源](https://github.com/mozilla)瀏覽器

Chrome是一家商業公司Google基於Chromium開發，谷歌的[大部分收入來源於其廣告業務](https://www.investopedia.com/articles/investing/020515/business-google.asp)

#### 瀏覽記錄&數據收集

Chrome在清除Cookie的時候會默認保存自家業務的cookie，[並永久儲存在電腦上](https://www.google.com/chrome/privacy/#browser-modes)，也就是沒有辦法完全清空瀏覽器Cookie。Chrome會自動使用Google帳號登入瀏覽器

同時，Google會永遠保存你的搜索記錄，可以在[My Activity](https://myactivity.google.com/myactivity)查看

關於Google收集的信息，可以參考：[Google data collection research](https://digitalcontentnext.org/blog/2018/08/21/google-data-collection-research/)

相對的，Mozilla承諾不會track user，可以參考Firefox的隱私條例：[Firefox Privacy Notice](https://www.mozilla.org/en-US/privacy/firefox/), Firefox內自建tracking protection。

由於Google不僅僅開發瀏覽器，同時也提供搜索業務，用戶無法準確的識別上傳至Google服務器的訊息；相比較之下，Mozilla本身僅提供少量的網路服務，Lockwise，profile，設想如果你的火狐發送了大量數據，必然會引起用戶懷疑

### 自定義

firefox可以自定義UI，關於Firefox的定製，可以參考編程隨想大佬的文章: [ 扫盲 Firefox 定制——从“user.js”到“omni.ja” ](https://program-think.blogspot.com/2019/07/Customize-Firefox.html?m=1)

Chrome可以自定義的部分相對於會更少

### 證書信任

Chrome自動採用系統CA證書，而Firefox則採用Mozilla維護的CA證書列表

絕大多數的操作系統都是商業公司提供，引用[編程隨想](https://program-think.blogspot.com/2018/09/Why-You-Should-Switch-from-Chrome-to-Firefox.html)大佬的話：

> 　考虑到大多数读者是 Windows 用户，而 Windows 这款操作系统的厂商是【微软】。现在咱们来考虑一个问题：是微软更可信还是 
> Mozilla 更可信？假设有一天，朝廷的【有关部门】分别找到微软和 
> Mozilla，向他俩提出无理要求。谁更容易屈服于咱们的朝廷？当然是微软（M$）。为啥捏？因为微软在咱们天朝有着庞大的【商业利益】。这些商业利益自然成为朝廷威胁
> M$ 的筹码。相比之下，Mozilla 只是一家【非盈利】组织。朝廷拿什么威胁 Mozilla？
> 考虑到还有很多同学使用苹果的 Mac OS，同样的问题，你自己思考一下：当面对朝廷的无理要求和威胁，是（商业的）“苹果公司”先屈服，还是（非营利的）“Mozilla 组织”先屈服？答案很是显然滴！
> 通过上述的对比你就能体会到：Firefox【自带】CA 证书的好处——自带的 CA 证书【不会】受到操作系统及其厂商的影响。

