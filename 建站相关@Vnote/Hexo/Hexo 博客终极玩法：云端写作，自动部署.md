# Hexo 博客终极玩法：云端写作，自动部署
> Hexo + Github + 语雀 + yuque-hexo +travis-ci+severless 打造全自动持续集成个人博客，云端写作，自动部署，完美体验~

### 一、Hexo+Github 的痛点

#### 1.为啥要用hexo+github？

作为一个程序猿，博客肯定是必须要有的拉，github也是必须要混的拉~所以:

- hexo + github = 高大上

![image.jpeg](images/20200222150507751_4741 "image.jpeg")

#### 2.蛋疼的写作体验

使用hexo，会面临如下问题：

- 博客源码怎么管理？
- 图片存在哪？
- 如何编写markdown文件？

相信很多人都在使用本地编辑器来写博客，那体验，真心蛋疼，比如说vscode，可视化插件一般般，图片还不能复制黏贴，想插入个图片还要先保存成文件放在本地，然后再引用，啥？你说七牛云存储？哪有复制黏贴爽呀~  
当然，博客源码可以使用travis-ci 来做持续集成，想写博客或者换个电脑，clone一下源仓库，写完push一下，就可以不用管了。but，比起独立站点的博客，如wordpress，还是觉得写作体验有点不爽。  
![image.jpeg](images/20200222150507344_28432 "image.jpeg")

#### 3\. 脑洞大开：

偶然间，朋友安利了语雀这个文档写作平台，觉得这就是完美的写作体验，各种UI和编辑器都很舒服~秀个图：  
![image.png](images/20200222150506736_18653 "image.png")

markdown编辑器也是巨爽：  
![image.png](images/20200222150506222_3885 "image.png")

于是乎，就在想能不能在语雀里写作，写完之后自动同步到Github的博客呢？年轻就要有激情，说干就干，花了一天时间，结合了severless、yuque-hexo、travis-ci之后，终于跑通了整个写作、发布、自动部署的流程~  
![image.jpeg](images/20200222150505706_29294 "image.jpeg")

### 二、具体方案

整体流程：  
![工作流.png](images/20200222150505298_14249 "工作流.png")

- 语雀发布一篇文章
- webhook调用serverless函数
- serverless 发起请求 trigger 一个 build 任务
- travis-ci 同步语雀文章并构建hexo
- github 生成静态页面展示

#### 1\. hexo+github+triavs-ci

hexo如何部署，如何集成travis-ci，等等，我就不再讲了，网上类似的文章灰常多~  
比如：

- [使用Hexo+Github+TravisCI搭建自动发布的静态博客系统](https://segmentfault.com/a/1190000013266001)
- [GitHub pages + Hexo 搭建自己的个人博客](https://segmentfault.com/a/1190000016487753)

![image.png](images/20200222150504786_17442 "image.png")

那么如何同步语雀的文章呢？答案就是：

##### **yuque-hexo**

这是一个开源库：[https://github.com/x-cold/yuque-hexo](https://github.com/x-cold/yuque-hexo)  
用法也很简单：  
1) 修改package.json，增加配置:

```
  "yuqueConfig": {
    "baseUrl": "https://www.yuque.com/api/v2",
    "login": "u46795",
    "repo": "blog",
    "mdNameFormat": "title",
    "postPath": "source/_posts/yuque"
  },
```

2）增加命令：

```
  "scripts": {
    "sync": "yuque-hexo sync",
    "clean:yuque": "yuque-hexo clean",
    "deploy": "npm run sync && hexo clean && hexo g -d",
  },
```

附上我的[package.json](https://github.com/Ghostdar/blog-origin/blob/master/package.json)文件。

#### 2\. serverless

目前阿里云和腾讯云都有serverless服务，免费的额度完全够用了，下面介绍一下如何配置：

##### 1）创建函数

![image.png](images/20200222150504377_16402 "image.png")

##### 2）serverless 函数示例:

```
<?php
function main_handler($event, $context) {
    // 解析语雀post的数据
    $update_title = '';
    if($event->body){
        $yuque_data= json_decode($event->body);
        $update_title .= $yuque_data->data->title;
    }
    // default params
    $repos = 'xxxx';  // 你的仓库id 或 slug
    $token = 'xxxxxx'; // 你的登录token
    $message = date("Y/m/d").':yuque update:'.$update_title;
    $branch = 'master';
    // post params
    $queryString = $event->queryString;
    $q_token = $queryString->token ? $queryString->token : $token;
    $q_repos = $queryString->repos ? $queryString->repos : $repos;
    $q_message = $queryString->message ? $queryString->message : $message;
    $q_branch = $queryString->branch ? $queryString->branch : 'master';
    echo($q_token);
    echo('===');
    echo ($q_repos);
    echo ('===');
    echo ($q_message);
    echo ('===');
    echo ($q_branch);
    echo ('===');
    //request travis ci
    $res_info = triggerTravisCI($q_repos, $q_token, $q_message, $q_branch);

    $res_code = 0;
    $res_message = '未知';
    if($res_info['http_code']){
        $res_code = $res_info['http_code'];
        switch($res_info['http_code']){
            case 200:
            case 202:
                $res_message = 'success';
            break;
            default:
                $res_message = 'faild';
            break;
        }
    }
    $res = array(
        'status'=>$res_code,
        'message'=>$res_message
    );
    return $res;
}

/*
* @description  travis api , trigger a build
* @param $repos string 仓库ID、slug
* @param $token string 登录验证token
* @param $message string 触发信息
* @param $branch string 分支
* @return $info array 回包信息
*/
function triggerTravisCI ($repos, $token, $message='yuque update', $branch='master') {
    //初始化
    $curl = curl_init();
    //设置抓取的url
    curl_setopt($curl, CURLOPT_URL, 'https://api.travis-ci.org/repo/'.$repos.'/requests');
    //设置获取的信息以文件流的形式返回，而不是直接输出。
    curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
    //设置post方式提交
    curl_setopt($curl, CURLOPT_CUSTOMREQUEST, "POST");
    //设置post数据
    $post_data = json_encode(array(
        "request"=> array(
            "message"=>$message,
            "branch"=>$branch
        )
    ));
    $header = array(
      'Content-Type: application/json',
      'Travis-API-Version: 3',
      'Authorization:token '.$token,
      'Content-Length:' . strlen($post_data)
    );
    curl_setopt($curl, CURLOPT_HTTPHEADER, $header);
    curl_setopt($curl, CURLOPT_POSTFIELDS, $post_data);
    //执行命令
    $data = curl_exec($curl);
    $info = curl_getinfo($curl);
    //关闭URL请求
    curl_close($curl);
    return $info;
}
?>

```

这里有几个需要获取的参数：

- travis登录token，在travis-ci.org 中设置界面获取：

![image.png](images/20200222150503862_17849 "image.png")

- 仓库ID 或 扩展名，

使用findder 或者 postman 发起一个请求： [https://api.travis-ci.org/owner/Ghostdar/repos](https://api.travis-ci.org/owner/Ghostdar/repos)

![clipboard.png](images/20200222150503348_1608 "clipboard.png")

  
回包中可以获取到ID 和 slug。

##### 3） 配置触发方式

![image.png](images/20200222150502938_12366 "image.png")

一般会得到这么个api：  
[https://service-s08f6nvk-1251...](https://service-s08f6nvk-1251833201.ap-guangzhou.apigateway.myqcloud.com/release/xxx)

#### 3\. 语雀配置

配置一个仓库的webhook:  
![image.png](images/20200222150502424_17320 "image.png")

可以选择所有更新触发或者主动触发，主动触发的意思即发布需要勾选一个选项才会触发webhook。具体可参见语雀文档：[https://www.yuque.com/yuque/developer/doc-webhook](https://www.yuque.com/yuque/developer/doc-webhook)；  
将serverless生成的api填入,可以在链接后面带参数：

```
token 登录token
repos 仓库id
message 提交信息
branch 分支

示例：
https://service-s08f6nvk-1251833201.ap-guangzhou.apigateway.myqcloud.com/release/xxx?repos=xxx&token=xxx&message=xxx&branch=xxx

```

如果不在链接带参数则写在serverless函数内。

#### 4\. 开始发布或更新一篇文章

发布或者更新一篇文章后，我们前往travis-ci,可以看到已经触发了一次构建请求：

![image.png](images/20200222150502013_27945 "image.png")  
构建完成后，咱们的博客上已经妥妥的展示出来拉~  
![image.png](images/20200222150501501_4213 "image.png")

附上博客地址：[https://ghostdar.github.io/](https://ghostdar.github.io/) ，羞耻的求个star~

![image.jpeg](images/20200222150500976_24773 "image.jpeg")

### 三、其他的思路

#### 1\. github api 

可以使用github 的 api ，每当更新文章，自动生成一个commit ，触发travis-ci 构建，但是感觉工作量很大，就放弃了~

#### 2\. 有待挖掘的travis-ci 

目前我使用的方法是trigger a build ，其实可以做到更多的 自定义配置~为啥不研究？主要是我懒~  
当然，如果有更好的方案也欢迎交流~

##### 最后：

再次感谢语雀开发webhook，以及@尹挚 大神的yuque-hexo插件~  
附上地址：

- yuque-hexo: [https://github.com/x-cold/yuque-hexo](https://github.com/x-cold/yuque-hexo)
- yuque-blog: [https://github.com/x-cold/yuque-blog](https://github.com/x-cold/yuque-blog)