---
title: "Powered By HUGO"
date: 2019-11-27T12:58:53+08:00
draft: false
---

博客正式遷移到了Hugo上，[Hugo][1]是使用GO語言開發的SSG（靜態網站生成器），官方稱“The world’s fastest framework for building websites”. 不同於Hexo的是，Hugo是一個網站的框架，爾Hexo僅僅是博客的框架。

“Let's Encrypt”， “1Password Support”，“CloudFlare Developer”等均是用Hugo開放

更多的實例可以參考[官方Showcase][2]



<!--more-->



# 文件結構

文件結構相比較於Hexo簡潔了不少，主題採用submodule的方式安裝即可，免去了之前Hexo下主題.git文件夾需要刪除的問題。以我用的MemE主題爲例，樹狀圖如下

```basic
tree
Folder PATH listing
Volume serial number is 
C:.
├───archetypes
├───content
│   ├───about
│   └───posts
├───data
├───layouts
├───public
│   ├───about
│   ├───categories
│   ├───css
│   ├───fonts
│   ├───icons
│   ├───js
│   ├───page
│   │   ├───1
│   │   └───2
│   ├───posts
│   └───tags
├───resources
│   └───_gen
│       ├───assets
│       │   └───scss
│       │       └───scss
│       └───images
├───static
└───themes
    ├───hugo-notice
    │   ├───i18n
    │   └───layouts
    │       └───shortcodes
    └───meme
        ├───assets
        │   ├───js
        │   └───scss
        │       ├───_common
        │       │   ├───_highlight
        │       │   └───_page
        │       ├───_custom
        │       └───_variables
        ├───config-examples
        │   ├───en-us
        │   └───zh-cn
        ├───data
        ├───exampleSite
        │   └───resources
        │       └───_gen
        │           └───assets
        │               └───scss
        │                   └───scss
        ├───i18n
        ├───images
        ├───layouts
        │   ├───partials
        │   │   ├───components
        │   │   ├───third-party
        │   │   └───tree
        │   ├───section
        │   ├───taxonomy
        │   └───_default
        └───static
            ├───fonts
            └───icons
```

如果精簡一下的話就是

```visual basic
├───archetypes                                                                     ├───content                                                                       
├───data                                                                           ├───layouts                                                                       
├───public
├───resources
├───static
└───themes
```

## 遷移

其實，因爲遷移的時候我的筆電正在維修，所以我是複製粘貼的。。。

## MemE

**MemE**是一個十分簡潔但是十分強大的Hugo主題，由[reuixiy | 一休爾][3]開發，爲個人博客設計.

正如作者寫到“MemE主題對於習慣了Hexo的用戶非常友好，是從Hexo錢一道Hugo的不錯選擇”，MemE的配置十分的方便

### 一些修改

#### 字體

支持調用Google Font這類的字體，只需要更改主題文件中的`Font Family`部分即可。如下

```toml
## Font Family

    # Note: any option is empty(""),
    #       fallback to `fontFamilyBody`
    #       it will. Therefore, it is not
    #       necessary to set all.
    #       Additionally, you can leave
    #       `fontFamilySiteBrand` empty("")
    #       if you use SVG as your site
    #       brand.

    # Site brand
    fontFamilySiteBrand = ""
    # Menu bar
    fontFamilyMenu = ""
    # Post title, post subtitle, list title, year and month title of the list, related posts title, previous/next post title
    fontFamilyTitle = "Kalam, sans-serif"
    # Headings, toc title
    fontFamilyHeadings = "Noto Serif TC, serif"
    # Code, superscript, post meta info, post updated badge, post gitinfo, minimal footer
    fontFamilyCode = "'Source Code Pro', monospace"
    # Blockquotes
    fontFamilyQuote = ""
    # Table of contents
    fontFamilyTOC = ""
    # Caption
    fontFamilyCaption = ""
    # Footer
    fontFamilyFooter = ""
    # Body
    # Text font
    fontFamilyBody = "'Noto Serif TC', serif"

    # Embed fonts link

    fontsLink = "https://fonts.googleapis.com/css?family=Noto+Serif+TC:600,700|Source+Code+Pro:400,400i,700,700i|ZCOOL+XiaoWei|Kalam:700&display=swap"

```

### 更新

Hugo採用Submodule進行主題的更新

```sh
git submodule update --rebase --remote
```

如果提示

```sh
error: cannot rebase: You have unstaged changes.
error: Please commit or stash them.
Unable to rebase '2f57a2c629c7c1bd5f7fd6cf3640db0d449b52d9' in submodule path 'themes/meme'
```

就按照提示先 `git stash`,在rebase之後執行`git stash pop`即可

待更

[1]:https://gohugo.io/
[2]:https://gohugo.io/showcase/
[3]:https://io-oi.me/tech/documentation-of-hugo-theme-meme/	"Hugo 主题 MemE 文档"