---
category: 博客
tag: git
---

#  利用SSH修复Git无法正常Push的问题

在IDE写完代码，然后想要Git push到github的时候总是出现类似于`The TLS connection was non-properly terminated`或者`Failed to connect to github.com port 443 after 133365 ms: Connection timed out`的报错，本文介绍一种不依赖与Http通信和代理服务器的方法实现轻松push

----

## 问题所在

在项目的根目录下使用如下命令，查看远端origin的地址

```cmd
git remote -v
```

你大概率会得到如下结果

```
origin  https://github.com/.../... .git (fetch)
origin  https://github.com/.../... .git (push)
```

这是因为主流IDE如VSCode或者JetBrains系列会默认使用http的方式来连接远程仓库，但由于众所周知的原因，在中国很难稳定的连接导致时常push失败。因此我们考虑使用SSH方式进行push。

## 取消代理

你可能尝试了网络上很多方法如修改代理、host文件等导致环境混乱，这里首先清理一下相关代理。

```
# 取消全局http、https代理
git config --global --unset http.proxy
git config --global --unset https.proxy
# 在项目仓库里取消代理
git config --unset http.proxy 
git config --unset https.proxy   
```

如果你使用vscode进行开发，注意在vscode settings里也取消代理，在`Http: Proxy`处留空并重启vscode。

## SSH验证

首先假设已经实现SSH私钥的配置，这个网络上教程很多因此不再赘述，测试方法如下：

```cmd
ssh -T git@github.com
```

如果出现`Hi <你的 GitHub 用户名>! You've successfully authenticated, but GitHub does not provide shell access.`则说明配置成功。

## 修改远程仓库地址

接下来需要把每个项目远端地址设置为ssh，方法如下：

```cmd
git remote set-url origin git@github.com:<你的用户名>/<你的仓库名>.git
```

之后输入`git remote -v`你应该可以看到如下内容

```cmd
origin  git@github.com/.../... .git (fetch)
origin  git@github.com/.../... .git (push)
```

最后验证一下

```cmd
git push -u origin master
```

完成！

## 参考博客

[Git Push&Pull 使用 SSH（含 443 端口）配置](https://xiaoyunhai0.github.io/2025/08/22/Git%E4%BD%BF%E7%94%A8SSH%E7%9A%84Push%E5%92%8CPull/)