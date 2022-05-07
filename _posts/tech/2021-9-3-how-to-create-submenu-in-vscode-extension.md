---
author: hancel.lin
date: 2021-09-03
title: VSCode 扩展开发之子菜单
tags: 
    - vscode
    - extension
    - submenu
category: tech
layout: post
guid: urn:uuid:ce26a7ee-2620-4417-9788-48ce908a3387
---

## 前言

开发 VSCode 扩展时遇到了添加右键二级菜单的问题，看了[官方文档](https://code.visualstudio.com/api/references/contribution-points#contributes.submenus)之后依然不得其解。反复实验之后终于理解文档所言，故整理分享。
<!--more-->
## 右键菜单

在开始之前，先讲一下如何添加右键菜单。右键菜单的添加是很简单的。只要配置 `package.json` 中的 `menus` 的 `editor/context` 添加 command 绑定即可。例如：

```json
{
    "menus": {
        "editor/context": [
            {
                "command": "markdown-image.paste",
                "when": "editorLangId == markdown",
                "group": "9_cutcopypaste@4"
            }
        ]
    },
}
```

一个右键菜单包含三个属性：

- `command` ： 这个是你在 `contributes` 的 `commands` 中定义的 `command`。用于实行特定操作。当点击这个右键菜单时，就会执行这个 `command`。
- `when` ：这个限定了右键菜单项目出现的条件，`editorLangId == markdown` 表示仅对 Markdown 文件生效。更多其他的变量与表达式，可以参考[文档](https://code.visualstudio.com/api/references/when-clause-contexts)。
- `group` ：这个设置了右键菜单出现的位置，`9_cutcopypaste@4` 表示复制剪切粘贴的菜单组的第 4 的位置。其他 `group` 值可以参考[文档](https://code.visualstudio.com/api/references/contribution-points#Sorting-of-groups)（有时间再开帖子详解）。
  
  ![Menu Group Sorting](/media/files/how-to-create-submenu-in-vscode-extension/menu-sort.png)

## 子菜单

添加子菜单与添加右键菜单有所关联，你需要先添加一个子菜单项目：

```json
{
    "menus": {
        "submenus": [
            {
                "id": "markdown-image.menulist",
                "label": "Markdown 粘贴"
            }
        ],
        "markdown-image.menulist": [
            {
                "command": "markdown-image.paste",
                "group": "2_workspace"
            },
            {
                "command": "markdown-image.paste-rich-text",
                "group": "2_workspace"
            }
        ]
    }
}
```

这一部分添加的内容有点多，首先我们看 `submenus`。里面定义了一个子菜单项目，通过一个 `id` 值 `markdown-image.menulist` 标识。接着在 `menus` 中定义这个子菜单包含的命令，这里的格式和添加右键菜单是类似的，一样包含 `command`、`when`、`group`（`when` 不写也可以）。这样我们就定义好子菜单项目了，但是还无法显示，因为我们还没有把他们绑定到具体的右键菜单上。

因此，我们需要再添加一个右键菜单，但是属性有所不同：

```json
{
    "menus": {
        "editor/context": [
            {
                "submenu": "markdown-image.menulist",
                "when": "editorLangId == markdown",
                "group": "9_cutcopypaste@4"
            }
        ]
    },
}
```

可以看到 `command` 改成了 `submenu` 。里面的值也不是来自于 `commands` 。而是一个 `submenus` 的 `id`。这样就可以把刚才定义的子菜单绑定到右键菜单上了。

VSCode 之所以把子菜单的定义与右键菜单切割开来，是因为在 VSCode 上不仅仅在右键菜单可以有子菜单，在一些比如侧边栏也可以定义一些展开的菜单，那里也可以用同样方式在绑定子菜单。甚至同一个子菜单绑定到多个不同的位置使用。
