---
title: "HomeBrew 安装旧版本应用"
date: 2021-11-24T14:55:31+08:00
tags: ["mac", "brew"]
draft: false
---

- 通过`github commit url`安装历史版本
- 通过`brew extract`安装历史版本

<!--more-->

#### 起因

最近在看`cpp`，用到了`gettext`库，编译时出错，该库当前最新版本为`0.21`（brew默认安装最新版本），看别人大多使用`0.19.8`，于是考虑使用`brew`安装历史版本。

#### 过程

- 按照`brew`之前的安装旧版本的方式

  1. 通过github寻找该包的仓库地址`brew info gettext`

     ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20211124150752.png)

     - 注：如果修改过`homebrew`源，可以直接前往`https://github.com/Homebrew/homebrew-core/blob/HEAD/Formula/<package>.rb`

  2. 查看history

     ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20211124151040.png)

  3. 找到具体版本commit

     ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20211124151237.png)

  4. 点进去，点击Raw格式，获取到地址栏地址。

     `https://raw.githubusercontent.com/Homebrew/homebrew-core/d084873b35054e2d76af8aad1f4540e29a0dbbea/Formula/gettext.rb`

  5. 使用`brew`安装

     `brew install https://raw.githubusercontent.com/Homebrew/homebrew-core/d084873b35054e2d76af8aad1f4540e29a0dbbea/Formula/gettext.rb`

  6. 之前版本可以通过上述方式进行安装。

#### 问题

- 新版本`HomeBrew`使用上述方法安装会报错：

  ```shell
  /usr/local/Homebrew/Library/Homebrew/formulary.rb:277:in `load_file': Invalid usage: Installation of gettext from a GitHub commit URL is unsupported! `brew extract gettext` to a stable tap on GitHub instead. (UsageError)
  	from /usr/local/Homebrew/Library/Homebrew/formulary.rb:185:in `klass'
  	from /usr/local/Homebrew/Library/Homebrew/formulary.rb:180:in `get_formula'
  	from /usr/local/Homebrew/Library/Homebrew/formulary.rb:418:in `factory'
  	from /usr/local/Homebrew/Library/Homebrew/cli/parser.rb:633:in `block in formulae'
  	from /usr/local/Homebrew/Library/Homebrew/cli/parser.rb:629:in `map'
  	from /usr/local/Homebrew/Library/Homebrew/cli/parser.rb:629:in `formulae'
  	from /usr/local/Homebrew/Library/Homebrew/cli/parser.rb:308:in `parse'
  	from /usr/local/Homebrew/Library/Homebrew/help.rb:103:in `parser_help'
  	from /usr/local/Homebrew/Library/Homebrew/help.rb:83:in `command_help'
  	from /usr/local/Homebrew/Library/Homebrew/help.rb:64:in `help'
  	from /usr/local/Homebrew/Library/Homebrew/brew.rb:145:in `rescue in <main>'
  	from /usr/local/Homebrew/Library/Homebrew/brew.rb:143:in `<main>'
  ```

  大意就是：使用`github`一个`commit url`进行安装的方式不再被支持，可以使用`brew extract gettext`的方式。

#### `brew extract`安装

1. 创建一个本地仓库

   `brew tap-new  local/homebrew`

2. 进入仓库

   `cd "$(brew --repo)/Library/Taps/local/homebrew-homebrew/Formula"`

3. 创建版本

   `wget https://raw.githubusercontent.com/Homebrew/homebrew-core/d084873b35054e2d76af8aad1f4540e29a0dbbea/Formula/gettext.rb -O gettext@0.19.8.1.rb`

4. 查看版本

   - `brew search gettext`

   - 现在就能看到我们刚刚新建的版本`local/homebrew/gettext@0.19.8.1`

5. 安装该版本

   `brew install gettext@0.19.8.1`





