---
title: hexo d部署报错
date: 2022-04-12 00:46:00
tags:
categories: hexo
---

<!--more-->

在进行hexo d部署服务时突然发生了报错，特此记录下来，方便后面自己修改

报错代码如下：
```
INFO  Validating config
WARN  Deprecated config detected: "external_link" with a Boolean value is deprecated. See https://hexo.io/docs/configuration for more details.
INFO  
  ===================================================================

      #####  #    # ##### ##### ###### #####  ###### #      #   #
      #    # #    #   #     #   #      #    # #      #       # #
      #####  #    #   #     #   #####  #    # #####  #        #
      #    # #    #   #     #   #      #####  #      #        #
      #####   ####    #     #   ###### #    # #      ######   #

                            4.2.0-b1
  ===================================================================
INFO  Deploying: git
INFO  Clearing .deploy_git folder...
INFO  Copying files from public folder...
nothing to commit, working tree clean
fatal: unable to access 'https://github.com/DongONNS/DongONNS.github.io.git/': OpenSSL SSL_read: Connection was reset, errno 10054
FATAL {
  err: Error: Spawn failed
      at ChildProcess.<anonymous> (D:\hexo\blog1\Blog\node_modules\hexo-util\lib\spawn.js:51:21)
      at ChildProcess.emit (events.js:314:20)
      at ChildProcess.cp.emit (D:\hexo\blog1\Blog\node_modules\cross-spawn\lib\enoent.js:34:29)
      at Process.ChildProcess._handle.onexit (internal/child_process.js:276:12) {
    code: 128
  }
} Something's wrong. Maybe you can find the solution here: %s https://hexo.io/docs/troubleshooting.html
```

hexo d在干什么？
hexo deploy命令用于部署网站，简写为hexo d。在执行hexo时会将代码部署到github，上述报错是在链接github时超时。

OpenSSL SSL_read: Connection was reset, errno 10054怎么解决？
该错误是说明github连接超时，首先可以检查一下是否可以正常连上github,执行命令ssh git@github.com