---
title: "How to remove git submodule"
date: 2020-10-05T23:48:27+08:00
draft: false
author: "MrSheep"
authorLink: "https://mrsheep.xyz"
description: "Remove git submodule"
tags: ["Git"]
categories: ["Git"]

lightgallery: false

---
How to Remove a Git Submodule
如何快速的刪除多餘的submodule
<!--more-->

1.編輯 `.gitmodules`
以刪除 `LoveIt` 爲例，移除
```json
[submodule "themes/LoveIt"]
        path = themes/LoveIt
        url = https://github.com/Mr-Sheep/LoveIt
```

`git add .gitmodules` 提交更改

2.編輯`.git/config`，移除相應部分

3. 移除緩存
`git rm --cached themes/LoveIt`

4. 刪除文件
`rm -rf .git/modules/LoveIt`

5. commit
`git commit -m "Removed submodule"`