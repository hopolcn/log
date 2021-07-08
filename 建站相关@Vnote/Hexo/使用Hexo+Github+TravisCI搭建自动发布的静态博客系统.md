# 使用Hexo+Github+TravisCI搭建自动发布的静态博客系统
最近玩slack接触到 `Travis CI`，又突然想起自己好久没写技术blog了，想想尘封已久的hexo+coding.net，心想能不能把Travis CI加进去。

先列举要用到的东西：

- Hexo - 快速、简洁且高效的博客框架
- Github Pages - 完美的静态博客服务器（除大陆网速外）
- NexT - 一款高质量且简洁优雅的Hexo主题
- Travis CI - 一个持续集成测试工具

其中，Travis CI 是我最新接触的，它是一款很好用的持续集成测试工具。它可以自动关联Github上的项目并且clone下来编译测试，现在就是利用它这一特性来实现hexo博客自动编译、部署。

<!--more -->

## 步骤及重点

一些网上和官方文档都很明确的步骤不再详细叙述，只做记录，提高效率。

### 一、先搞Hexo

Hexo建站参考[Hexo官方文档](https://hexo.io/zh-cn/docs/index.html)

没难度，需要注意几个点：

1. 增加`hexo-deployer-git`依赖，防止部署时报错。

```
npm install --save hexo-deployer-git
```

1. 增加`hexo-util`到`dev`依赖，防止travis的npm版本<3,出现的`Error: Cannot find module 'hexo-util'`错误。

```
npm install --save-dev hexo-util
```

### 二、配置Github

建库、同步本地什么的都是github基础操作，注意一点，建两个分支，`master`分支和对应源码，`gh-pages`分支作为hexo生成的静态页面上传分支。网上很多都说要用到xxx.github.io这个项目的，其实没必要，只需要推到其他分支即可，然后在setting里配置要作为页面显示的是哪个分支即可。

### 三、使用NexT主题

这个可以参考[Next帮助文档](https://github.com/iissnan/hexo-theme-next/blob/master/README.cn.md)，步骤说明很详细。

其中需要注意的点：

1. 就是上边说的hexo-util的问题，已给出解决办法。
2. 关于语言的问题，`v5.1.x`版本的中文是`zh-Hans`，而`v6.0.0`的中文是`zh-CN`，注意区分。
3. 关于主题的配置文件，采用了`Hexo data files`特性，利用这个特性，可以放心更新next主题，而不担心配置丢失或烦于合并。
4. `v6.0.0`的fancybox移至外部仓库依赖，所以需要自行添加，添加方法参见[详细安装步骤](https://github.com/theme-next/theme-next-fancybox)。

### 四、配置Travis CI

重点来了，详细步骤可参考[用 Travis CI 自动部署 hexo](https://segmentfault.com/a/1190000004667156)，作者已经说的比较详细。

需要注意的一点是：在`package.json`中增加`depoly`的命令行语句，防止travis在自动执行到`npm run deploy`这一步的时候报找不到该script的错。

```
"scripts": {
  "deploy": "hexo clean && hexo g -d"
},
```

上述代码加在`dependencies`同级即可。

其实配置完Travis发现，其实原理就是让Travis在云上帮我们执行本地化的编译、部署等操作，它也需要git pull项目下来，npm install安装依赖，hexo d -g去编译静态博客，并且依旧需要遵循github的权限验证，所以才需要配置了共私钥对。

举一反三，通过这个博客的配置，同时也学会了使用Travis做持续集成测试。除了可以应用到其他项目中去，同时也开启了新世界的大门。

## 个性化

1. 配置自定义的css样式，主题目录下`source/css/_custom/custom.styl`
2. [NexT使用说明](http://theme-next.iissnan.com/)

## 后续

在此基础上，研究slack集成travis，让每次自动测试的结果及时推送到slack上，完美！