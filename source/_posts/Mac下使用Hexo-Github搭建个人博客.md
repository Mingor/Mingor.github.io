---
title: Mac下使用Hexo+Github搭建个人博客
date: 2019-02-17 14:18:59
tags:
---

# **一、什么是 Hexo ?**

简单地说，Hexo 是一个快速、简洁且高效的博客框架。 [Hexo主页](https://hexo.io/zh-cn/)



# **二、环境配置**

### 1\. 安装 Node.js

**1.1 先检查是否已安装**

终端输入命令

```
node -v  //看看是否出现安装版本信息，出现说明已经安装了
```

**1.2 两种安装方式可选**

1、在 [官方下载网站](https://nodejs.org/en/download/) 下载 pkg 安装包，直接点击安装即可

2、使用 brew 命令来安装：(什么？你不知道 Homebrew？去官网看看吧 [我是Homebrew官网](https://brew.sh/index_zh-cn.html))

```
brew install node
```



### 2\. 安装Git

**2.1 也是先检查 git 是否已安装（Mac 下安装 Xcode 就自带 Git）**

终端输入命令
```
git --version  //看看是否出现安装版本信息，出现说明已经安装了
```

**2.2 两种安装方式可选**

1、下载 [安装程序](http://sourceforge.net/projects/git-osx-installer/) 安装

2、使用 brew 命令来安装

```
brew install git
```



### 3\. Github 账号注册及仓库创建

在 [Github](https://github.com/) 上注册账号，注册过程我就不赘述了，注册完成之后需要新建一个仓库。

![创建新仓库](https://upload-images.jianshu.io/upload_images/2910208-3212fdbd5b965352.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

创建仓库的名字必须为 **username.github.io**，我的用户名为 Mingor ，因此我创建的仓库就是 **Mingor.github.io**，写完后直接点创建就OK了。

![填写仓库名](https://upload-images.jianshu.io/upload_images/2910208-0d54c303368cadd6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



# 三、安装及配置 Hexo

### **1\. 安装 Hexo**

所有必备的应用程序安装完成后，即可在终端使用 npm 安装 Hexo

```
npm install -g hexo-cli
```

安装成功后，在你想要的目录开始建站。

我这里是创建一个 MyBlog 文件夹

```
mkdir MyBlog
```

进入目录

```
cd MyBlog
```

初始化目录，该命令会在目标文件夹内建立网站所需要的所有文件

```
hexo init
```

安装依赖包

```
npm install
```

到这里本地博客就搭建好了，执行以下命令（在你博客的对应文件夹路径下）即可以开启本地服务

```
hexo server
```

出现以下信息，说明你可以访问本地博客，在浏览器输入 [http://localhost:4000](http://localhost:4000/) 这个网址，就可以看到博客首页。

当然这个博客是本地的，别人是无法访问的，之后我们需要部署到 GitHub 上。

![开启本地服务成功](https://upload-images.jianshu.io/upload_images/2910208-1a17f73c05fbe07a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



### 2. 部署博客到 Github 仓库

在博客文件夹中找到 **_config.yml** 文件并打开

![配置文件](https://upload-images.jianshu.io/upload_images/2910208-75ef00baacbb8c37.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

编辑 **_config.yml** 文件中的 **deploy** 节点，把 username 替换成你的名字

```
deploy:
  type: git
  repo: https://github.com/username/username.github.io.git
  branch: master
```

为了能够使 Hexo 部署到 GitHub 上，需要安装一个插件

```
npm install hexo-deployer-git --save
```

成功后执行以下命令
```
hexo clean    //清除部署緩存
hexo generate //生成静态页面至 public 目录
hexo deploy   //将.deploy目录部署到GitHub
```

然后需要输入你 GitHub 的用户名和密码，完成后在浏览器输入 username.github.io 就可以访问你的博客了。（可能会有延迟）



### 3. 配置 SSH Key（可选）

为了每一次部署不必一种输入密码，我们可以生成秘钥，然后提交到GitHub，进行关联，那么你下次就不需要再输入密码了。

**3.1 检查本机上是否已经存在 SSH Key**

打开终端，分别输入以下命令
```
cd .ssh
ls -la
```

检查终端输出的文件列表中是否已经存在 **id_rsa.pub** 或 **id_dsa.pub** 文件，如果文件已经存在，则直接进入第三步。

**3.2 创建一个 SSH Key**

邮箱是填写 GitHub 注册时的邮箱
```
cd ~
ssh-keygen -t rsa -C "your_email@example.com"
```

按下回车，让你输入文件名，直接回车会创建使用默认文件名的文件(推荐使用默认文件名)，然后会提示你输入两次密码，可以为空。

**3.3 关联 GitHub 账号**

打开 **id_rsa.pub** 文件，复制里面的全部信息

```
open ~/.ssh
```

接下来看图操作

![设置](https://upload-images.jianshu.io/upload_images/2910208-38800540d6b85c60.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![添加SSH key](https://upload-images.jianshu.io/upload_images/2910208-f349f79da1d6e5fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![添加SSH key](https://upload-images.jianshu.io/upload_images/2910208-ceee157f5a66b754.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**3.4 检验 SSH Key 是否配置成功**

在终端输入命令

```
ssh -T git@github.com
```

如果出现

```
Are you sure you want to continue connecting (yes/no)?
```

请输入 yes 再按回车。

如果最后出现

```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

就说明你的 SSH Key 配置成功了。

**3.5 修改 deploy 节点**

编辑 _config.yml 文件中的 deploy 节点

```
deploy:
    type: git 
    repo: git@github.com:username/username.github.io.git
    branch: master
```

至此便实现了免密部署



# 四、Hexo 的基本使用

### 1. 发布新文章
在博客目录下执行以下命令
```
hexo new title
```
执行之后，就会在博客的 source/_posts 目录里自动创建 标题为 title 的 markdown 文件，然后你就可以找一个 markdown 文本编辑器进行编辑即可，我使用的是 [typora](https://typora.io/)



![创建文章目录](https://upload-images.jianshu.io/upload_images/2910208-d947a7d75922055b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



编辑完成之后，与上面一样，执行以下命令即可更新到Github上。
```
hexo clean
hexo generate
hexo deploy
```
更多操作可去 [Hexo的基本使用](https://hexo.io/zh-cn/docs/writing) 看看，介绍更加详细而且也有视频讲解



### 2. 常用命令

```
hexo help  //查看帮助
hexo version   //查看Hexo的版本
hexo algolia   //更新search庫
hexo new "postName"  //新建文章
hexo new post "title"  //生成新文章：\source\_posts\title.md，可省略post
hexo new page "pageName"  //新建页面
hexo cl == clean  //清除部署緩存
hexo n == hexo new  //新建文章
hexo g == hexo generate  //生成静态页面至public目录
hexo s == hexo server  //开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
hexo d == hexo deploy  //将.deploy目录部署到GitHub
hexo d -g  //生成加部署
hexo s -g  //生成加预览
```


# 五、配置主题

先找一个自己喜欢的主题 [Hexo主题](https://hexo.io/themes/)，我这里选择的是 [varaint](https://github.com/justpsvm/hexo-theme-varaint) 
执行以下命令克隆主题

```
git clone https://github.com/justpsvm/hexo-theme-varaint themes/varaint
```
然后修改站点配置文件：**_config.yml**，将里面76行的 theme 由 **landscape** 修改为 **varaint**

![修改主题](https://upload-images.jianshu.io/upload_images/2910208-22d99c642a70a818.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后部署到 github 上（可能会有延迟）

```
hexo cl
hexo d -g
```



# 六、Hexo 系列问题之我们换了电脑怎么办

### 解决方案1

通过插件解决：https://github.com/coneycode/hexo-git-backup

### 解决方案2

把我们的文件提交到git上，利用git来管理它，在现有的XXX.github.io项目上创建一个分支来管理

1. 克隆 gitHub 上的 XXX.github.io 项目的文件到本地 
```
git clone https://github.com/yourname/xxx.github.io.git 
```
2. 删除文件夹里除了 .git 的其他所有文件
3. 把 hexo 项目文件下的所有文件全部复制过来 (如果有自己下载的主题，要把 .git 删除掉，不然提交推送文件到github上的时候会忽略掉自己下载的主题)
4. 里面应该有一个叫 .gitignore 的文件，如果没有就输入 touch .gitignore，创建一个 
5. .gitignore 文件里应该是这些内容 
```
.DS_Store 
Thumbs.db 
db.json 
*.log 
node_modules/ 
public/ 
.deploy*/ 
```
6. 创建一个叫 hexo 的分支并切换到这个分支上 
```
git checkout -b hexo 
```
7. 提交复制过来的文件到暂存区
```
git add --all 
```
8. 提交 
```
git commit -m "新建分支资源文件" 
```
9. 推送分支到 github。到这一步我们就基本上搞定了，以后再跟新了博客就直接 git push 就可以了，hexo 的操作跟以前一样不变。
```
git push --set-upstream origin hexo 
```
10. 今后无论什么时候想要在其他电脑上面用 hexo 写博客，就直接把创建的分支克隆下来，npm install 安装依赖之后就可以用了。（如果deploy节点用的是ssh记得要修改） 
```
git clone -b hexo https://github.com/yourname/xxx.github.io.git  //克隆分支的操作
```
11. 因为上面创建的是一个名字叫 hexo 的分支，所以这里 -b 后面的是 hexo，再把后面的 gitHub 的地址换成你自己的 hexo 博客的地址就可以了。 
12. 这样做完了以后，每次写完博客发布之后不要忘了还要 git push 把源文件推到分支上。



# 参考链接

* https://www.jianshu.com/p/e5f95eb990ad
* https://www.jianshu.com/p/77db3862595c
* https://www.zhihu.com/question/21193762
* https://blog.csdn.net/wxl1555/article/details/79293159