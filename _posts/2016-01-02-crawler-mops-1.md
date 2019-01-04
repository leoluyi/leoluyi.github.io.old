---
layout: post
title: '[R crawler] 公開資訊觀測站 (觀察篇)'
date: 2016-01-02 17:04
comments: true
tags: R
---

這次要來~~攻擊~~抓取的資料是「公開資訊觀測站」的公司資料，如同[前篇文章](http://leoluyi.logdown.com/posts/2015/12/22/r-crawler-101-learning-experience-the-crawler-is-one-of-the-basic-skills)所談到的：

> 「⋯⋯以前許多資料源取得的限制， 不再是個無解的難題之後，會釋放出更自由的想像空間，更大的挑戰便是整合資訊的應用，以及如何從中淘金了。」

這些公司資料實際上可以有相當程度的[運用](http://weichengliou.github.io/blog/blog/2014/08/06/twcom/)，諸如交叉持股的情形，董監事的關係群體，或是台灣在企業投資的整體影響力結構，都可藉由對這些***資料***的進一步分析得到寶貴的***資訊***。

---

### 目標網站

[公開資訊觀測站](http://mops.twse.com.tw/)

### Step 1. 定義目標資料

寫爬蟲總要先知道你要爬的資料是什麼，這裡的「資料」可以是在網頁上看到的任何東西，甚至是整個網頁的內容。

我們先來看一下這個網站有什麼東西。進到首頁後，上方的一排按鈕都會連到類似的查詢頁面，有些資料的查詢結果是重複的只是換一個方式呈現，因此這裡就直接以**公司基本資料**作為~~攻擊~~目標。

點入左上方的**總彙報表** > **基本資料** > **基本資料查詢總彙報表**，可以看到一個查詢的下拉式選單，可選擇市場別和產業別。

![mops_index](https://lh3.googleusercontent.com/-HVWcLvmD1Uo/VoePGHBX_PI/AAAAAAAAFFk/M5KVWj8I_z0/s0/Jietu_20151230223456.png "mops_index")

![mops_2](https://lh3.googleusercontent.com/-Zipokheo3xs/VofOuXb5_qI/AAAAAAAAFGc/CgAIL7YH_MQ/s0/Jietu_20151230224429.png "mops_2")

市場別先選擇「上市」來觀察，而產業別可以選擇空白來一次查詢所有公司的資料。

![enter image description here](https://lh3.googleusercontent.com/-DxeMgFgvL20/VofSeOd40rI/AAAAAAAAFHI/8Tm_5XkhnX4/s0/Jietu_20151230231032.png "mops_3")

最後得到的這一大張表格就是我們想要的資料了。

![mops_data_table](https://lh3.googleusercontent.com/-3Z0GZuvsPog/VofUEsuRn0I/AAAAAAAAFHg/1pwW67z1djY/s0/Jietu_20160102214305.png "mops_data_table")


### Step 2. 觀察

寫爬蟲最重要的心法便是**觀察**，在這裡用的是Chrome <i class="fa fa-chrome"></i> 的**開發人員工具**，先在這一頁按下快捷鍵<kbd>Cmd/Ctrl + Shift + i</kbd>(如果忘記的話可以按右上的漢堡圖案 <i class="fa fa-bars"></i> > 更多工具 > 開發人員工具)，切換到**Network**分頁，重新整理後便可以看到瀏覽過程的所有連線。

#### >> 找到資料

在這裡使用**開發人員工具**觀察，有個小技巧，由於我們要的資料通常是出現在我們眼前才會讀進來，因此，

> **Tips**
> 可以先按紅色按鈕旁的鈕 <i class="fa fa-ban"></i> 清掉之前的連線，在按下「搜尋」按鈕跑出資料的同時，我們需要的那個連線便會出現在前幾個。

![mops_observe](https://lh3.googleusercontent.com/-uVvVvXggXtM/VofUsXYaY5I/AAAAAAAAFH0/TU9wFdu76Gw/s0/Jietu_20151230231141.png "mops_observe")

按下「搜尋」按鈕的瞬間，~~正直和善良~~資料全部都進來了，觀察後發現第一個連線有極高的機率是我們要的資料所在。

![mops_click](https://lh3.googleusercontent.com/-6sVbpL6-x88/VofaA2_sTlI/AAAAAAAAFIM/D-LQdYgzsyo/s0/Jietu_20151230231209.png "mops_click")

點進去看一下**Preview**，果然這是很單純的**Page Render**類型的網頁，資料就直接出現在Document裡了。

![mops_res_data](https://lh3.googleusercontent.com/-rPVYKZxMFoI/VofappSXYFI/AAAAAAAAFIg/1OLDdKDaWz0/s0/Jietu_20151230231408.png "mops_res_data")

#### >> 連線方法

點進去**Headers**分頁確認一下連線的類型，可看到這個連線敲的是這個網址：

http://mops.twse.com.tw/mops/web/ajax_t51sb01

而連線的類型是**POST**[^1][^2]，看到 POST method 就一定要接著看他 post 的**form data**。

![mops_headers_1](https://lh3.googleusercontent.com/-0qhTwDW_ry4/Vofb2DYNSVI/AAAAAAAAFI4/X9twdGpQYLY/s0/Jietu_20151230231754.png "mops_headers_1")

看到下面的 **form data** 有幾個參數：

```
encodeURIComponent: 1
step: 1
firstin: 1
TYPEK: sii
code:
```

![mops_headers_2](https://lh3.googleusercontent.com/-xIFhPrMpGT0/VofeAdbaahI/AAAAAAAAFJQ/lH_Mnfd6404/s0/Jietu_20151230231834.png "mops_headers_2")

這些參數的值，應該有部分是在剛才選擇「市場別」和「產業別」的時候帶入的，為了確認這點，可以有兩種作法：第一種是再重新選擇別的選項查詢試試，但是如果選項很多的話就會試到天荒地老；或者可以用第二種方式，直接去網頁 html 原始碼一探究竟。

對著下拉式選單按**右鍵** > **檢查 (inspect element)**，查看 html 的內容。

![mops_form](https://lh3.googleusercontent.com/-f0bUVJkWjy8/VofSQWLeqRI/AAAAAAAAFG0/SICaYPmoyeM/s0/Jietu_20151230224618.png "mops_option")

![enter image description here](https://lh3.googleusercontent.com/-KPWKjp6aTBk/VoffiysNhNI/AAAAAAAAFJo/tbE0lcjbE-Q/s0/Jietu_20151230231543.png "mops_option")

看到在 `select` 的 `name` attribute分別是 `TYPEK` 和 `code`，在 `option` 有 `value` 這個 attribute ，其值分別就是剛才 **form data** 的參數項目；同時也可發現，裡面有數個 `option` 的 subtag 出現剛才在下拉式選單看到的選項：

```
TYPEK: sii
code:
```

---

到這邊就差不多完成這隻爬蟲所需要的觀察項目了，接下來就是實際上在 R 的實作和反覆試誤，對照本篇所談到的觀察技巧來回修正，寫出一隻完整的爬蟲了。


[^1]: [HTTP Methods: GET vs. POST](http://www.w3schools.com/tags/ref_httpmethods.asp)

[^2]: [淺談 HTTP Method：表單中的 GET 與 POST 有什麼差別？](http://blog.toright.com/posts/1203/%E6%B7%BA%E8%AB%87-http-method%EF%BC%9A%E8%A1%A8%E5%96%AE%E4%B8%AD%E7%9A%84-get-%E8%88%87-post-%E6%9C%89%E4%BB%80%E9%BA%BC%E5%B7%AE%E5%88%A5%EF%BC%9F.html)
