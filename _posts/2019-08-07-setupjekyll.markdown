---
layout: post
title:  "Windows下jekyll的配置"
date:   2019-08-07 10:35:56
category: jekyll
---
jekyll是一个简单的免费的博客或者是静态网站的生成工具，不需要数据库的支持，而且可以很方便地部署在github上，这个网页就是用jekyll所配置生成的。在linux下jekyll配置非常方便，在Windows环境下会相对复杂，这篇博客记录一下如何在Windows 10的环境下配置jekyll。

## Ruby
jekyll是基于ruby的一个软件，所以先要配置ruby。DevKit 是windows平台下编译和使用本地C/C++扩展包的工具。它就是用来模拟Linux平台下的make, gcc, sh来进行编译。ruby官方提供的windows下的安装包分为包含devkit和不含devkit的两种，当前可以直接安装推荐的[Ruby+Devkit 2.5.5-1 (x64)](https://rubyinstaller.org/downloads/)。                
安装的时候需要将勾选添加到PATH选项（默认勾选），后续便于在命令行中使用。                     
<div align="center"><img  src="https://github.com/conceptclear/conceptclear.github.io/raw/master/images/jekyll/ruby_path.jpg"></div>            
安装完成如图所示：                         
<div align="center"><img  src="https://github.com/conceptclear/conceptclear.github.io/raw/master/images/jekyll/ruby_finish.jpg"></div>   
安装完之后检查ruby和gem的版本：                    
<div align="center"><img  src="https://github.com/conceptclear/conceptclear.github.io/raw/master/images/jekyll/ruby_check.jpg"></div>   

## RubyGems
RubyGems（简称 gems）是一个用于对 Ruby组件进行打包的 Ruby 打包系统，类似于linux系统下的apt-get。由于各种原因，rubygems的[官网](rubygems.org)访问非常缓慢，直接从命令行来安装jekyll很困难，国内有一个rubygems的[镜像网站](http://gems.ruby-china.com/)，可以改变一下rubygems的源来提高速度。             
```
$ gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
$ gem sources -l
https://gems.ruby-china.com
# 确保只有 gems.ruby-china.com
```
更改源之后更新一下rubygems                       
```
$ gem update --system
$ gem -v
```
如图所示：
<div align="center"><img  src="https://github.com/conceptclear/conceptclear.github.io/raw/master/images/jekyll/gem_check.jpg"></div>   

## jekyll
安装好ruby和rubygems之后，就可以利用命令行通过rubygems来安装jekyll了。
```
gem install jekyll
```
安装完成可以测试一下
```
jekyll -v
```
如图所示：
<div align="center"><img  src="https://github.com/conceptclear/conceptclear.github.io/raw/master/images/jekyll/jekyll_check.jpg"></div>   

## 运行
都配置好之后，只需要按照jekyll官网上的运行方法，就可以在本地运行jekyll的静态网站进行调试了。
```
~ $ gem install jekyll bundler
~ $ jekyll new my-awesome-site
~ $ cd my-awesome-site
~/my-awesome-site $ bundle install
~/my-awesome-site $ bundle exec jekyll serve
```
打开浏览器 http://localhost:4000 就可以本地查看生成的网页了，如图所示：
<div align="center"><img  src="https://github.com/conceptclear/conceptclear.github.io/raw/master/images/jekyll/local.jpg"></div>

## 问题
jekyll生成出现问题的时候可以将Gemfile.lock文件删除，检查Gemfile文件之后重新生成，这样可以解决大部分的问题。

## Reference
[1] https://jekyllcn.com/                    
[2] https://github.com/mmistakes/minimal-mistakes/issues/1454                             
[3] https://github.com/rubygems/rubygems/issues/2058                        
[4] 百度百科                       
[5] https://gems.ruby-china.com/                          
