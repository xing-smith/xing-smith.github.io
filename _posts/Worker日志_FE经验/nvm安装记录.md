---
layout:     post
title:      nvm安装记录 & nrm
subtitle:   
date:       2023-09-04
author:     forwardZ
header-img: img/the-first.png
catalog: false
tags:
    - 日志_经验
---



## nvm

### 卸载node

1. 查看 node 全局安装目录

```zsh
which node
```

2. 删除 node 执行文件
3. 删除相关文件和文件夹

```zsh
sudo rm -rf /usr/local/lib/node /usr/local/lib/node_modules /usr/local/include/node /usr/local/include/node_modules
```

4. 彻底删除全局安装的 node

```zsh
sudo rm /usr/local/bin/npm
sudo rm /usr/local/share/man/man1/node.1
sudo rm /usr/local/lib/dtrace/node.d
sudo rm -rf ~/.npm
sudo rm -rf ~/.node-gyp
sudo rm /opt/local/bin/node
sudo rm /opt/local/include/node
sudo rm -rf /opt/local/lib/node_modules
```



### 安装 nvm

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

<!-- [nvm.png](https://postimg.cc/YjQ06HWw) -->
<!-- 
<img src="/Users/zhongdaxing1/Library/Application Support/typora-user-images/image-20230904180203857.png" alt="image-20230904180203857" style="zoom:25%;" /> -->





针对报错： Profile not found，需添加配置信息

* 创建 `.bash_profile` 配置文件

```zsh
vim ~/.bash_profile
```

* 向 `~/.bash_profile` 文件中添加下面内容

```shell
export NVM_DIR=~/.nvm
source ~/.nvm/nvm.sh
```

补充：vim是一种编辑器，具体操作为参考：https://www.runoob.com/linux/linux-vim.html

* 使文件修改内容生效

```zsh
source ~/.bash_profile
```

此时，可以通过`nvm --version`查看是否生效



注：但是关闭终端后，需要重新执行source .bash_profile才能重新使用nvm命令。原因是缺少`.zshrc`文件，解决：

先open（创建），如果没有改文件，则touch（创建）

```zsh
open ~/.zshrc
// 如果提示没有改文件，则创建
touch ~/.zshrc
// 创建后，仍需再打开
open ~/.zshrc
```

将配置添加到.zshrc文件中

```zsh
export NVM_DIR=~/.nvm
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
```

执行.zshrc文件

```zsh
source ~/.zshrc
```



### 安装 node

安装指定版本 node

```zsh
nvm install 14.17.6
```

查看已安装的node

```zsh
nvm ls
```

切换 node版本

```zsh
nvm use 版本号｜别名
```

查看当前使用的 `node` 版本

```zsh
node -v
```



### 参考链接：

Mac 下 nvm 安装管理 Node 版本：https://juejin.cn/post/7011423615438553096

使用 nvm 进行 Node.js 版本管理：https://blog.shipengx.com/archives/a5459cd3.html

mac 找不到nvm命令解决方法：https://blog.csdn.net/wangguoyu1996/article/details/109615270



## nrm

nrm是一个镜像源管理，通过

```bash
npm install nrm -g
```

这里的全局，是该node版本的全局，可以看下图的安装文件夹的位置。[nrm.png](https://postimg.cc/r0kXcvFS)
<!-- 
<img src="/Users/zhongdaxing1/Library/Application Support/typora-user-images/image-20230904202441020.png" alt="image-20230904202441020" style="zoom:25%;" /> -->

参考链接：

nvm 和 nrm 的安装与使用：https://juejin.cn/post/6844903799530733582

