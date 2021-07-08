# hexo博客搭建
## 前言

本文章会为你梳理一个搭建hexo博客的流程

> 相关网址：
> Docs: https://hexo.io/docs/
> Themes: https://hexo.io/themes/



## 安装hexo

### 准备阶段-Git 和 nodejs

**安装Git**

Windows: 下载然后安装Git [https://git-scm.com/download/win]

如果你下载慢,可以使用下面的链接

> 链接: https://pan.baidu.com/s/1HXujcEuaPZYFQLtzlSBf0Q 提取码: ryut
>
> 安装完成后桌面会出现Git Bash的软件名称，代表安装成功,而且你的右键菜单栏也会出现Git Bash
>

Linux: (Ubuntu,Debian): $ sudo apt-get install git-core

Linux (Fedora, Red Hat, CentOS): $ sudo yum install git-core

ArchLinux: `$ sudo pacman -S git

**安装Node.js**



Windows: 下载然后安装nodejs [http://nodejs.cn/download/]

> 请下载Windows安装包(.msi) 使用安装程序安装-记得Add to path.
>
> 安装成功后可以在终端执行node -v，如果输出版本号，证明成功加入环境变量。如果没有输出版本号
>
> 请百度一下：安装nodejs
>



Linux: (Ubuntu,Debian): $ sudo apt-get install nodejs npm

Linux (Fedora, Red Hat, CentOS): $ sudo yum install nodejs npm

ArchLinux: `$ sudo pacman -S nodejs npm



**安装cnpm**

我们使用安装淘宝镜像将会大大加快安装的速度

```bash
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
```

### 开始安装

```shell
$ cnpm install hexo-cli -g
```

```shell
$ hexo init blog
$ cd blog      //进入博客目录
```

**解救方案**

如果出现进度条卡住的情况，用Ctrl+c 中断，然后使用下面的解救方案

```bash
$ cd blog
$ cnpm install
```

### 预览博客

```shell
$ cd blog  //进入博客目录
$ hexo s
```

这个时候就会出现一段网址，请打开浏览器访问

http://localhost:4000/

### 相关命令

```
hexo clean 清理缓存
hexo g     重新生成public
hexo d     上传博客
```

## 更换主题

> 主题大全：https://hexo.io/themes/
>

```
一定要按照主题的文档安装
一定要按照主题的文档安装
一定要按照主题的文档安装
```

### 修改主题

注意分清站点配置文件和主题配置文件

> 博客根目录下的_config.yml叫站点配置文件
>
> 主题目录下的_config.yml叫主题配置文件
>



## 上传部署

### SSH

看这个地址

> https://dev.tencent.com/help/doc/faq/bbe781aee786/ssh
>
> coding添加公钥的方式：登陆你的github帐户，然后点击头像 -> 左栏点击SSH公钥 -> 点击新增公钥
>
> coding验证是否添加成功的方式：ssh -T git@git.dev.tencent.com
>



### 修改站点配置文件

```go
deploy:
  type: git
  repo: 仓库Git地址
  branch: master
```

安装上传插件

```
npm install hexo-deployer-git
```

上传

```
hexo g -d
```

## 编写文章

你需要先学习markdown语法

> https://301technology.cn/2020/01/17/markdown/
>



生成文章

> hexo new "文章名"  //建议英文
>

标题是可以更改的,文章名会成为链接名,而不是最终的标题

生成的文章在 博客目录/source/_post下

根据每个主题的文档配置情况,进行配置每个文章的参数.



小技巧



可以在scaffolds/post.md修改文章模板

写完后如果要上传就还是执行上传的命令

```
hexo d
```

## 其他技巧

### 图床问题

> 使用PicGo:https://github.com/Molunerfinn/PicGo
>
> 配合jsdelivr可以cdn加速:https://www.jsdelivr.com/
>
> 具体使用方法见:https://301technology.cn/2020/01/17/image/
>





作者: Huanhao

链接: http://yoursite.com/2020/01/19/hexoblog/

来源: Huanhao's Blog

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。