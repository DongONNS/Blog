---
title: hexo升级
date: 2022-05-14 22:28:00
tags:
categories: hexo
---

<!--more-->

## 一、背景
&emsp;&emsp;在进行博客发布过程中发现空格的渲染失效了，原来使用"&emsp;"来输出空格，但是在解析出来的博客还是显示&emsp;，原因是hexo和markdown的版本匹配问题，所以需要升级相应的组件版本。

## 二、版本升级
### 2.1 版本查看
&emsp;&emsp;npm outdated查看组件版本
```
Package                  Current  Wanted  Latest  Location
hexo                       5.2.0   5.4.2   6.2.0  hexo-site
hexo-generator-archive     0.1.5   0.1.5   1.0.0  hexo-site
hexo-generator-category    0.1.3   0.1.3   1.0.0  hexo-site
hexo-generator-feed        2.2.0   2.2.0   3.0.0  hexo-site
hexo-generator-index       0.2.1   0.2.1   2.0.0  hexo-site
hexo-generator-sitemap     2.1.0   2.2.0   3.0.1  hexo-site
hexo-generator-tag         0.2.0   0.2.0   1.0.0  hexo-site
hexo-renderer-ejs          0.3.1   0.3.1   2.0.0  hexo-site
hexo-renderer-stylus       0.3.3   0.3.3   2.0.1  hexo-site
hexo-server                0.3.3   0.3.3   3.0.0  hexo-site
```
### 2.2 版本修改
&emsp;&emsp;按照上一步中wanted的组件版本修改package.json数据
```
修改后的package.json
{
  "name": "hexo-site",
  "version": "0.0.0",
  "private": true,
  "hexo": {
    "version": "5.4.2"
  },
  "dependencies": {
    "add": "^2.0.6",
    "hexo": "^5.4.2",
    "hexo-deployer-git": "^3.0.0",
    "hexo-generator-archive": "^0.1.5",
    "hexo-generator-baidu-sitemap": "^0.1.9",
    "hexo-generator-category": "^0.1.3",
    "hexo-generator-feed": "^2.2.0",
    "hexo-generator-index": "^0.2.1",
    "hexo-generator-json-content": "^4.2.3",
    "hexo-generator-sitemap": "^2.2.0",
    "hexo-generator-tag": "^0.2.0",
    "hexo-renderer-ejs": "^0.3.1",
    "hexo-renderer-marked": "^4.1.0",
    "hexo-renderer-pug": "^3.0.0",
    "hexo-renderer-stylus": "^0.3.3",
    "hexo-server": "^0.3.3",
    "hexo-wordcount": "^6.0.1",
    "or": "^0.2.0",
    "yarn": "^1.22.18"
  }
}
```



### 2.3 组件升级
使用npm install --save升级组件版本
查看升级后的hexo版本
```
PS D:\hexo\blog1\Blog> hexo version
INFO  Validating config
WARN  Deprecated config detected: "external_link" with a Boolean value is deprecated. See https://hexo.io/docs/configuration for more details.
INFO  
  ===================================================================

      #####  #    # ##### ##### ###### #####  ###### #      #   #
      #    # #    #   #     #   #      #    # #      #       # #
      #####  #    #   #     #   #####  #    # #####  #        #
      #    # #    #   #     #   #      #####  #      #        #
      #    # #    #   #     #   #      #   #  #      #        #
      #####   ####    #     #   ###### #    # #      ######   #

                            4.2.0-b1
  ===================================================================
hexo-cli: 2.0.0
os: Windows_NT 10.0.18363 win32 x64
node: 14.12.0
v8: 8.4.371.19-node.16
uv: 1.39.0
zlib: 1.2.11
brotli: 1.0.9
ares: 1.16.0
modules: 83
nghttp2: 1.41.0
napi: 7
llhttp: 2.1.2
openssl: 1.1.1g
cldr: 37.0
icu: 67.1
tz: 2020a
unicode: 13.0
```