# openwrt-actions
https://p3terx.com/archives/build-openwrt-with-github-actions.html
进阶使用
自定义环境变量与功能
点击查看
打开 work­flow 文件（.github/workflows/build-openwrt.yml），你会看到有如下一些环境变量，可按照自己的需求对这些变量进行定义。

env:
  REPO_URL: https://github.com/coolsnowwolf/lede
  REPO_BRANCH: master
  CONFIG_FILE: .config
  DIY_SH: diy.sh
  SSH_ACTIONS: false
  UPLOAD_BIN_DIR: false
  UPLOAD_FIRMWARE: true
  TZ: Asia/Shanghai
TIPS: 修改时需要注意:(冒号)后面有空格。
环境变量	功能
REPO_URL	源码仓库地址
REPO_BRANCH	源码分支
CONFIG_FILE	.config文件名
DIY_SH	DIY 脚本文件名
SSH_ACTIONS	SSH 连接 Actions 功能。默认false
UPLOAD_BIN_DIR	上传 bin 目录。即包含所有 ipk 文件和固件的目录。默认false
UPLOAD_FIRMWARE	上传固件目录。默认true
TZ	时区设置
DIY 脚本
仓库内有一个 diy.sh 文件，你可以把对源码修改的指令写到这个文件中，比如修改默认 IP、主机名、主题、添加 / 删除软件包等操作。但不仅限于这些操作，发挥你强大的想象力，可做出更强大的功能。

TIPS: 脚本工作目录在源码目录，内附一个修改默认 IP 的例子供参考使用。
添加软件包
在 DIY 脚本中加入对指定软件包的远程仓库的克隆指令。就像下面这样：

git clone https://github.com/P3TERX/xxx package/xxx
这样做的好处是每一次编译都会拉取最新的源码。

TIPS: 生成.config文件时记得选中相应的软件。如果添加的软件包与 Open­Wrt 中已有的软件包同名的情况，则需要把源码中的同名软件包删除，否则会优先编译 Open­Wrt 中的软件包。这同样可以利用到的 DIY 脚本。
最后不要忘了添加更新 feeds 的命令

./scripts/feeds update -a
./scripts/feeds install -a
Custom files（自定义文件）
俗称 “files 大法”，在仓库根目录下新建 files 目录，把文件放入即可。

定时自动编译
点击查看
编辑 work­flow 文件（.github/workflows/build-openwrt.yml）取消注释下面两行。

#  schedule:
#    - cron: 0 8 * * 5
例子是北京时间每周五下午 4 点钟开始编译（周末下班回家直接下载最新固件开始折腾）。如需自定义则按照 cron 格式修改即可，Ac­tions 虚拟环境中的时区是 UTC ，注意按照自己所在地时区进行转换。

真·一键编译（点击 star 开始编译）
点击自己仓库页面上的 Star 开始编译，为了防止被滥用，这个功能默认没有开启。开启后如果被恶意点击轻则封号，严重可能会导致中美关系恶化、原子弹爆炸、第三次世界大战等后果。（大雾

点击查看
编辑 work­flow 文件（.github/workflows/build-openwrt.yml）取消注释下面两行，后续点击自己仓库上的 star 即可开始编译。

#  watch:
#    types: [started]
TIPS: 字段started并不是“开始了”的意思，而是“已经点击 Star”。
吐槽: 官方并没有提供一个开始按钮，通过搜索找到过很多奇怪的一键触发方式，但都是通过 Web­hook 来实现的。机智的我发现了可以通过点击 Star 来触发，这样就相当于把 Star 当成开始按钮。这个started有种一句双关的意思了。
有个小技巧可以防止恶意点击，在 runs-on: ubuntu-latest 下面加上 if: github.event.repository.owner.id == github.event.sender.id。这样只有你自己点 star 才会触发编译。

jobs:
  build:
    runs-on: ubuntu-latest
    if: github.event.repository.owner.id == github.event.sender.id
自定义源码编译
此方案默认引用的是 L 大的源码，如果你觉得不好用或者有编译其它源码的需求可以进行替换，自由是本解决方案最大的特点。

点击查看
编辑 work­flow 文件（.github/workflows/build-openwrt.yml），修改下面的相关环境变量字段。

REPO_URL: https://github.com/coolsnowwolf/lede
REPO_BRANCH: master
比如修改为 Open­Wrt 官方源码 19.07 分支

REPO_URL: https://github.com/openwrt/openwrt
REPO_BRANCH: openwrt-19.07
TIPS: 注意冒号后面有空格
并发编译（同时编译多个固件）
多 repository 方案
通过 P3TERX/Actions-OpenWrt 项目创建多个仓库来编译不同架构机型的 Open­Wrt 固件。

多 workflow 方案
基于 GitHub Ac­tions 可同时运行多个工作流程的特性，最多可以同时进行至少 20 个编译任务。也可以单独选择其中一个进行编译，这充分的利用到了 GitHub Ac­tions 为每个账户免费提供的 20 个 Ubuntu 虚拟服务器环境。此外你还可以额外再使用 5 个 macOS 虚拟服务器环境进行编译，开启方法在后面有说明。

点击查看
假设有三台路由器的固件需要编译，比如 K2P、软路由、新路由 3。

生成它们的.config文件
分别将它们重命名为k2p.config、x64.config、d2.config放入本地仓库根目录。
复制多个 workflow 文件（.github/workflows/build-openwrt.yml）。为了更好的区分可以对它进行重命名，比如k2p.yml、x64.yml、d2.yml。此外第一行name字段也可以进行相应的修改。
然后分别用上面修改的文件名替换对应 workflow 文件中下面两个位置的.config，不同的机型同样可以使用不同的 DIY 脚本。
...
    paths:
      - '.config'
...
        CONFIG_FILE: '.config'
        DIY_SH: 'diy.sh'
...
最后 push ，此时此就触发了3个并行的编译工作流程。
云 menuconfig（SSH 连接到 Actions）
通过 tmate 连接到 GitHub Ac­tions 虚拟服务器环境，可直接进行 make menuconfig 操作生成编译配置，或者任意的客制化操作。也就是说，你不需要再自己搭建编译环境了。这可能改变之前所有使用 GitHub Ac­tions 的编译 Open­Wrt 方式。

点击查看
编辑 workflow 文件（.github/workflows/build-openwrt.yml），修改SSH_ACTIONS环境变量的值为true：
SSH_ACTIONS: true
在触发工作流程后，在 Actions 页面等待执行到SSH connection to Actions步骤，会出现下面的信息。
To connect to this session copy-n-paste the following into a terminal or browser:

ssh Y26QeagDtsPXp2mT6me5cnMRd@nyc1.tmate.io

https://tmate.io/t/Y26QeagDtsPXp2mT6me5cnMRd
复制 SSH 连接命令粘贴到终端内执行，或者复制链接在浏览器中打开使用网页终端。（网页终端可能会遇到黑屏的情况，按 Ctrl + C 即可）
cd openwrt && make menuconfig
完成后按快捷键Ctrl+D或执行exit命令退出，后续编译工作将自动进行。
TIPS: 固件目录下有个config.seed文件，如果你需要再次编译可以使用它。
WARRING: 默认连接30分钟后会断开并终止编译工作流程，防止资源浪费与封号风险。如果你想解除这个限制，可以根据提示操作，导致的一切后果请自行承担。
macOS 编译方案
GitHub Ac­tions 的 ma­cOS 虚拟机性能要高于 Ubuntu 虚拟机，所以使用它编译 Open­Wrt 理论上速度会更快。博主经过几天时间的研究已经总结出了 macOS 下的 OpenWrt 编译环境的搭建方法，并编写出了适用于 ma­cOS 虚拟环境的 Open­Wrt 编译方案的 work­flow 文件。

由于并不是每个开发者都会按照规范去写代码，所以使用 ma­cOS 编译 Open­Wrt 不可避免的会遇到非常多的问题，而且后续测试发现 ma­cOS 虚拟机性能已大幅下降，故相关 work­flow 文件已经移除。也不建议任何人使用 ma­cOS 编译 Open­Wrt 。

尾巴
希望大家不要滥用免费的开发资源，需要时再编译，让开发者来充分利用才能产生更多更好的软件，这样大家才能受益。

相关 TG 群组：GitHub Actions Group

相关 TG 频道：GitHub Actions Channel

本博客已开设 Telegram 频道，欢迎小伙伴们订阅关注。

本文作者：P3TERX

本文链接：https://p3terx.com/archives/build-openwrt-with-github-actions.html
