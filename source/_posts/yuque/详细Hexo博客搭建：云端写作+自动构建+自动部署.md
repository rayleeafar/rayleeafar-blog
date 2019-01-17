
---
title: 详细Hexo博客搭建：云端写作+自动构建+自动部署
date: 2019-01-17 16:26:03 +0800
tags: []
categories: 
---
# YuQue云端写作+Travis-ci自动构建+github-pages发布   
**序言：**在网上看见Nero大神的一篇文章[https://segmentfault.com/a/1190000017797561](https://segmentfault.com/a/1190000017797561)，简单讲述了如何构建一个云端写作自动部署到github pages的hexo 静态博客网站，之前我已经在github pages上搭建了一个博客，无奈觉得每次写作非常不方便，用代码编辑器vs code写md文档然后手动生成静态再部署，写作环境实在太差，之后便放弃直接在CSDN上写了。看到上面那篇文章，顿觉激动，在语雀这样方便的平台上编写文档，自动部署，简直爽翻，遂动手开干，花了半天的时间搞定。由于大神写的过于简略，途中遇到不少坑，遂填坑出一个完整的搭建流程<br /><br /><br />**结果：**
* 我已搭建好的博客： [https://rayleeafar.github.io/](https://rayleeafar.github.io/)  
* 发布博客静态页面的仓库： [https://github.com/rayleeafar/rayleeafar.github.io](https://github.com/rayleeafar/rayleeafar.github.io)
* 博客源码的仓库： [https://github.com/rayleeafar/rayleeafar-blog](https://github.com/rayleeafar/rayleeafar-blog)

有问题可以参考源码里面的配置

### 搭建步骤：  

#### 一、GitHub pages + Hexo 搭建初级个人博客  
参考文章：  
* 初始化GitHub pages:  [https://pages.github.com/](https://pages.github.com/)  
* 安装hexo及相关依赖:  [https://hexo.io/zh-cn/docs/](https://hexo.io/zh-cn/docs/)
* 在第一步的GH-pages仓库目录中初始化hexo: [https://hexo.io/zh-cn/docs/setup](https://hexo.io/zh-cn/docs/setup)  
* 开始写作: [https://hexo.io/zh-cn/docs/writing](https://hexo.io/zh-cn/docs/writing)
* 部署到github pages: [https://hexo.io/zh-cn/docs/deployment](https://hexo.io/zh-cn/docs/deployment)

#### 二、GitHub pages + Hexo + travis-ci 搭建中级 CI自动部署博客  
主要步骤参考: [https://segmentfault.com/a/1190000004667156#articleHeader1](https://segmentfault.com/a/1190000004667156#articleHeader1)<br />其中需要注意的几点是:
* 文章需要安装Ruby，才能使用gem 安装travis工具 
```bash
sudo apt install ruby
gem install travis
```
* 生成ssh-key的过程有点问题，按照文章ci部署的时候会报错 vi undefine，正确步骤是用以下命令登录
```
cd path/to/your/repo/.travis
ssh-keygen -t rsa -C "youremail@example.com"
travis login --pro #用github账号登录
travis encrypt-file id_rsa --pro
#得到  openssl aes-256-cbc -K $encrypted_xxxxxxxxxxx_key -iv $encrypted_xxxxxxxxxxx_iv
#其中 id_rsa.enc的位置需要注意设置对
```

注意以上两个点，按照文章步骤申请travis账号开通repo权限就能完成部署了

#### 三、GitHub pages + Hexo + travis-ci + YuQue + Serverless搭建高级 云端写作->自动部署博客    
步骤参考文章:
* [https://segmentfault.com/a/1190000017797561](https://segmentfault.com/a/1190000017797561)

按照大神的文章走，主要注意以下几点，找到并设置对各个参数
* YuQue-hexo库的使用  [https://github.com/x-cold/yuque-hexo](https://github.com/x-cold/yuque-hexo)
* 修改package.json，增加配置:  


```bash
  "yuqueConfig": {
    "baseUrl": "https://www.yuque.com/api/v2",
    "login": "rayleeafar",#填写你的语雀账号名称
    "repo": "gg272k",#填写在语雀上创建的文章仓库的编号一般是几个字母数字，你也可以自己改，下图中路径最后的那个值
    "mdNameFormat": "title",
    "postPath": "source/_posts/yuque"
  },
    "scripts": {
    "sync": "yuque-hexo sync",
    "clean:yuque": "yuque-hexo clean",
    "deploy": "npm run sync && hexo clean && hexo g -d",
  }
```

* ![Screenshot from 2019-01-17 13-55-04.png](https://cdn.nlark.com/yuque/0/2019/png/247315/1547704531539-87c973b8-fa2f-4d8f-bc18-1f9b2c7d796c.png#align=left&display=inline&height=423&linkTarget=_blank&name=Screenshot%20from%202019-01-17%2013-55-04.png&originHeight=722&originWidth=1199&size=47422&width=703)
##### * serverless 函数中参数的设置:
文章中说用postman发个请求，也可以用curl: <br />再返回的数据中找到repo的id，我的是：7596310<br />
```
>> curl -H "Travis-API-Version: 3" -H "User-Agent: API Explorer" \ -H "Authorization: token <your_token>" \ https://api.travis-ci.org/owner/<your_name>/repos
#curl的返回信息
......
  "repositories": [
    {
      "@type": "repository",
      "@href": "/repo/7596310",
      "@representation": "standard",
      "@permissions": {
        "read": true,
        "migrate": true,
        "star": true,
        "unstar": true,
        "create_cron": true,
        "create_env_var": true,
        "create_key_pair": true,
        "delete_key_pair": true,
        "create_request": true,
        "admin": true,
        "activate": true,
        "deactivate": true
      },
      "id": 7596310,
      "name": "rayleeafar-blog",
      "slug": "rayleeafar/rayleeafar-blog",
      "description": null,
      "github_id": 166050465,
      "github_language": null,
      "active": true,
      "private": false,
      "owner": {
        "@type": "user",
        "id": 1254643,
        "login": "rayleeafar",
        "@href": "/user/1254643"
      },
......
```

* 我申请的是腾讯的serverless服务，在腾讯云上注册登录，创建函数服务:

    用下面这个函数，作者的那个里面 有个接口地址有点不同，我抓travis看到接口URL是另一个：
> curl_setopt($curl, CURLOPT_URL, '**https://api.travis-ci.com**/repo/'.$repos.'/requests');


```php
<?php
function main_handler($event, $context) {
    // 解析语雀post的数据
    $update_title = '';
    if($event->body){
        $yuque_data= json_decode($event->body);
        $update_title .= $yuque_data->data->title;
    }
    // default params
    $repos = '7596310';  // 你的仓库id 或 slug
    $token = 'yJsKEMoSNUmQSv9kLFsbvg'; // 你的登录token
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
    //设置抓取的url https://api.travis-ci.com/repo/7596310/requests
    curl_setopt($curl, CURLOPT_URL, 'https://api.travis-ci.com/repo/'.$repos.'/requests');
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

* 按照文章的步骤创建并填对参数应该就事成一大半了

#### 后记  
按照上述文章及注意的点走完，试试在语雀上写个文章并发布，登录serverless看看hook有没有触发，登录travis-ci看看构建是否成功，有问题留言联系呀～～<br />祝各位写作愉快～～～～～

