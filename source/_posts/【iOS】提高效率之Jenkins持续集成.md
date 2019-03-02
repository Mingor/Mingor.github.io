---
title: 【iOS】提高效率之Jenkins持续集成
date: 2019-02-28 14:29:38
tags:
---
![](https://upload-images.jianshu.io/upload_images/2910208-884ba8c66f83fa6e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 一、安装Jenkins



### 1. 安装Homebrew (若已安装跳过此步骤)
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
### 2. 安装JavaSDK (若已安装跳过此步骤)
```
brew cask install java
```
### 3. 安装Jenkins
```
brew install jenkins
```
### 4. 启动Jenkins
```
jenkins
```
此时可以通过 http://localhost:8080 来访问 Jenkins
# 二、Jenkins账号设置

初次用浏览器直接打开 http://localhost:8080，那么应该是如下的显示界面。将该路径下的文件打开并将内容粘贴到文本框中。

![](https://upload-images.jianshu.io/upload_images/2910208-6704588a576caa04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
下一步选择 Install suggested plugins ，即下载推荐插件。

![下载推荐插件](https://upload-images.jianshu.io/upload_images/2910208-1e36d12911f4e5ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![安装插件中...](https://upload-images.jianshu.io/upload_images/2910208-1499560a57f02dc9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



安装完成后设置用户名和密码

![设置用户名和密码](https://upload-images.jianshu.io/upload_images/2910208-dda1b8ab988370d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



# 三、配置项目

### 1. 新建一个任务



![新建项目](https://upload-images.jianshu.io/upload_images/2910208-b10a6c166ec48601.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



### 2. 创建一个自由风格的软件项目

![创建一个自由风格的软件项目](https://upload-images.jianshu.io/upload_images/2910208-0e0c4d01b2833195.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



### 3. 填写项目基本信息

![项目基本信息](https://upload-images.jianshu.io/upload_images/2910208-b5f59e3d006d50d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



### 4. 源码管理

为了将项目克隆到 jenkins 的工作目录上，填写完要构建一次。

![源码管理](https://upload-images.jianshu.io/upload_images/2910208-d25f5410ccb88bd2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



### 5. 导出 ExportOptions.plist（为后面的shell脚本做准备）

Xcode 默认不允许访问钥匙串的内容，必须要设置 allowProvisioningUpdates 才会允许访问，Jenkins 的 Xcode 插件目前无法支持此项完成打包流程。




>**解决办法**
>1.使用 Xcode 手动打包，在导出的文件夹中找到 ExportOptions.plist 文件。（建议修改文件名来区分是哪种方式打包的）
>2.打开 ExportOptions.plist 文件，设置 compileBitcode 为 NO。
>3.检查 ExportOptions.plist 文件下的 provisioningProfiles 是否设置正确。
>4.将 ExportOptions.plist 文件放在 `/Users/yourUserName/.jenkins/workspace/yourProject ` 目录下。



### 6. 编写Execute shell

在  `/Users/yourUserName/.jenkins/workspace/yourProject`  目录下创建一个 shell 脚本。比如我这里创建一个用于企业内部并上传到蒲公英的脚本，命名为 TestProject_InHouse_pgyer.sh。
在脚本添加上以下代码：

```
#!/bin/bash -l
set -e
cd $(dirname $0)
echo | pwd

SHCEME="项目shceme名"
OPTION="对应的ExportOptions.plist"

PGYUSERKEY=蒲公英userkey
PGYAPIKEY=蒲公英apikey

echo ${SHCEME}
echo ${OPTION}

pod install

TIME_STAMP=`date "+%Y%m%d%H"`
ArchivePath=JenkinsPackage/${SHCEME}.xcarchive
PacktName=JenkinsPackage/${SHCEME}_${TIME_STAMP}

# archive & export
xcodebuild archive -workspace creation.xcworkspace -scheme ${SHCEME} -archivePath ${ArchivePath} -allowProvisioningUpdates
xcodebuild -exportArchive -archivePath ${ArchivePath} -exportPath ${PacktName} -exportOptionsPlist ${OPTION} -allowProvisioningUpdates

# 上传到蒲公英
curl -F "file=@${PacktName}/${SHCEME}.ipa" \
-F "uKey=$PGYUSERKEY" \
-F "_api_key=$PGYAPIKEY" \
https://qiniu-storage.pgyer.com/apiv1/app/upload

```


在 Jenkins 上设置脚本

![添加shell](https://upload-images.jianshu.io/upload_images/2910208-b0718c089355f164.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![执行shell](https://upload-images.jianshu.io/upload_images/2910208-092a09a340e08199.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

至此已经完成了配置，赶紧构建试试吧！



### 布置APP下载二维码

如果我们想将APP在蒲公英的二维码放在 Jenkins 上，方便测试人员下载的话应该怎么做呢？

![效果图](https://upload-images.jianshu.io/upload_images/2910208-dfe03f80375aa471.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

先使 Jenkins Description 支持HTML

![使 Jenkins Description 支持HTML](https://upload-images.jianshu.io/upload_images/2910208-cf74564ddc7f832c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



最后在项目配置里修改 Description

![修改配置](https://upload-images.jianshu.io/upload_images/2910208-b2579708158359b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)