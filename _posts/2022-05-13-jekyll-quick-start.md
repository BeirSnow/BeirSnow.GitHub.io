---
layout: post
title:  "Jekyll 环境搭建"
date:   2022-05-13 15:30:19 +0800
categories: jekyll update
---

#### 一、下载 rubyinstaller 安装 Ruby
太新的版本不兼容 jekyll，无法正常使用，当前（2022.5.13）可用版本：
>rubyinstaller-devkit-2.7.6-1-x64.exe

#### 二、替换 gem 源
##### 1. 查看当前源
```bash
$ gem sources -l
```

```bash
*** CURRENT SOURCES ***
https://https://rubygems.org/
```
##### 2. 删除当前源 
```bash
$ gem sources --remove https://rubygems.org
```
##### 3. 添加新源
```bash
$ gem sources -a https://gems.ruby-china.com/
```
##### 4. 查看源是否更新成功
```bash
$ gem sources -l
```
```bash
*** CURRENT SOURCES ***
https://gems.ruby-china.com/
```

#### 三、安装 jekyll
```bash
$ gem install jekyll
```

#### 四、使用 bundle config 修改 Ruby 镜像源
jekyll创建新项目时报错：
```bash
PS E:\> jekyll new blog
Running bundle install in E:/blog...
  Bundler: Fetching source index from https://rubygems.org/The dependency http_parser.rb (~> 0.6.0) will be unused by any of the platforms Bundler is installing for. Bundler is installing for x64-mingw32
but the dependency is only for java. To add those platforms to the bundle, run `bundle lock --add-platform java`.
  Bundler: Retrying fetcher due to error (2/4): Bundler::HTTPError Could not fetch specs from https://rubygems.org/
  Bundler: Retrying fetcher due to error (3/4): Bundler::HTTPError Could not fetch specs from https://rubygems.org/
  Bundler: Retrying fetcher due to error (4/4): Bundler::HTTPError Could not fetch specs from https://rubygems.org/
  Bundler: Could not fetch specs from https://rubygems.org/
```
使用以下命令更换源即可：
```bash
$ bundle config mirror.https://rubygems.org https://gems.ruby-china.com/
```

#### 五、jekyll serve 报错
类似如下报错：
```bash
$ Could not find gem 'wdm (~> 0.1.1) x64-mingw32' in any of the gem sources listed in your Gemfile.
```
根据报错信息执行如下命令安装缺失的模块即可：
```bash
$ gem install wdm
```
如果安装对应模块后仍然报错，则指定版本安装：
```bash
$ gem install wdm --version=0.1.1
```

### 六、基础教程
[jekyllcn](http://jekyllcn.com/)