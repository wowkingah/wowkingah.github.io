---
layout: post
title: "My first blog for Jekyll~"
date: 2016-10-26
tags: Personal
---

# Welcome ~  
##    This is my first blog for Jekyll.  
  
  
简单记录Jekyll安装步骤。  

### Ruby
```bash
[root@db_node01 srv]# yum -y install gcc
[root@db_node01 srv]# curl -L get.rvm.io| bash -s stable
[root@db_node01 srv]# source /etc/profile
[root@db_node01 srv]# gem sources --remove https://rubygems.org/
https://rubygems.org/ removed from sources
[root@db_node01 srv]# gem sources -a https://ruby.taobao.org/
https://ruby.taobao.org/ added to sources
[root@db_node01 srv]# gem sources -l
*** CURRENT SOURCES ***

https://ruby.taobao.org/
[root@db_node01 srv]# rvm list known 
[root@db_node01 srv]# rvm install 2.4

#RubyGems
[root@db_node01 ~]# wget https://rubygems.org/rubygems/rubygems-2.6.11.tgz
--2017-04-21 21:37:43--  https://rubygems.org/rubygems/rubygems-2.6.11.tgz
Resolving rubygems.org... 151.101.192.70, 151.101.64.70, 151.101.0.70, ...
Connecting to rubygems.org|151.101.192.70|:443... connected.
ERROR: certificate common name “l.ssl.fastly.net” doesn’t match requested host name “rubygems.org”.
To connect to rubygems.org insecurely, use ‘--no-check-certificate’.
[root@db_node01 ~]# wget --no-check-certificate https://rubygems.org/rubygems/rubygems-2.6.11.tgz
[root@db_node01 ~]# tar -zxvf rubygems-2.6.11.tgz
[root@db_node01 ~]# cd rubygems-2.6.11
[root@db_node01 rubygems-2.6.11]# ruby setup.rb
```

### Jekyll
```bash
[root@db_node01 rubygems-2.6.11]# gem install jekyll bundler 
[root@db_node01 wowkingah.github.io]# jekyll serve
Could not find proper version of jekyll (3.1.1) in any of the sources
Run `bundle install` to install missing gems.
[root@db_node01 wowkingah.github.io]# bundle install
bundle install
[root@db_node01 wowkingah.github.io]# jekyll serve
```

