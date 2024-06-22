---
layout: post
title:  "hello world"
date:   2021-09-01 19:39:02 +0800
categories: [others] 
---


# Quick Start



## Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

## Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

## Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

## Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/one-command-deployment.html)



# ssh上传

为什么不用https,这个老是出现问题.

好吧也不能这样说,应该说 ssh出现问题就用https

```
git remote set-url origin git@github.com:i1oveyou/i1oveyou.github.io.git
```



# upload

```
git add .
git commit -m "happy every day"
git push -u origin master
```





# theme_config



> _layouts\default.html



1), 个人头像

```
<a href="/"><img class="profile-avatar" src="{{ site.avatar_url }}" height="75px" width="75px" /></a>
===>改为
<a href="/"><img class="profile-avatar" src="{{ site.avatar_url }}" height="120px" width="120px" /></a>
```



2), 页脚的GitHub

          <div class="btn-github" style="float:right;">
            <iframe src="https://ghbtns.com/github-btn.html?user=enovella&repo=enovella.github.io&type=star&count=true" frameborder="0" scrolling="0" width="85" height="20px"></iframe>
            <iframe src="https://ghbtns.com/github-btn.html?user=enovella&repo=enovella.github.io&type=fork&count=true" frameborder="0" scrolling="0" width="85" height="20px"></iframe>
          </div>
          
          
     ===>改为   
              <div class="btn-github" style="float:right;">
                <iframe src="https://ghbtns.com/github-btn.html?user=i1oveyou&repo=i1oveyou.github.io&type=star&count=true" frameborder="0" scrolling="0" width="85" height="20px"></iframe>
                <iframe src="https://ghbtns.com/github-btn.html?user=i1oveyou&repo=i1oveyou.github.io&type=fork&count=true" frameborder="0" scrolling="0" width="85" height="20px"></iframe>
              </div>

 

# toc





# avatar



static\css\main.css

```
div.col-sm-3 img.profile-avatar {
  border-radius: 150px;
  -webkit-border-radius: 150px;
  -moz-border-radius: 150px;
  -ms-border-radius: 150px;
  -o-border-radius: 150px;
  margin-left: auto;
  margin-right: auto;
}
      
 ===>改为 

div.col-sm-3 img.profile-avatar {
/*  border-radius: 150px;
  -webkit-border-radius: 150px;
  -moz-border-radius: 150px;
  -ms-border-radius: 150px;
  -o-border-radius: 150px;*/
  margin-left: auto;
  margin-right: auto;
}
```



