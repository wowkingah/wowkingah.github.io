---
layout: post
title: Sublime Text3编辑markdown文本
date: 2017-04-22 
tags: Tools Sublime Markdown
---

之前一直使用notepad++写SHELL、SQL，有一天想玩玩markdown的时候，觉得不是那么方便。于是，就有了sublime的出现。  

使用sublime编写markdown需要：
> 1. 安装sublime
> 2. 安装markdown相关插件
>> * Package Control：用于管理插件
>> * MarkdownEditing：编辑markdown  
>> * Markdown Preview：预览markdown  
  

### 安装sublime
Sublime 官网下载并安装(http://www.sublimetext.com/3)  


### 安装Package Control
Package Control官网(https://packagecontrol.io/installation)
#### 安装步骤：
> 1. 使用快捷键`ctrl+~` 或通过菜单栏 `View > Show Console`调出console。
> 2. 复制官网Python code粘贴到console后按回车去执行。
>> * sublime text3选择SUBLIME TEXT3对应的代码
>> ```python
>> import urllib.request,os,hashlib; h = 'df21e130d211cfc94d9b0905775a7c0f' + '1e3d39e33b79698005270310898eea76'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
>> ```
> ![](/images/posts/sa/sublime_text3/console.jpg)
> 3. 重启sublime后，可在`Preferences -> Package Settings`看到`Package Control`。  

### 安装MarkdownEditing与Markdown Preview
> 1. 快捷方式`Ctrl + Shift + p`或通过菜单栏`Preferences -> Package Control`，输入install
>  ![](/images/posts/sa/sublime_text3/install_package.jpg)  
>  2. 安装MarkdownEditing 
>  ![](/images/posts/sa/sublime_text3/install_MarkdownEditing.jpg)  
>  3. 安装Markdown Preview
>  ![](/images/posts/sa/sublime_text3/install_MarkdownPreview.jpg)  
  

### 偏好设置
**设置语言平台**
> 1. 打开Markdown Editing设置
> `Preferences -> Package Settings -> Markdown Editing -> Markdown GFM Settings User`
> 2. 增加配置
> ```
// These settings override both User and Default settings for the Markdown syntax  
{
    "default_line_ending": "unix",
}
```
  

**sublime相关设置**
> 1. 打开settings  
> `Preferences -> Settings`
> 2. 增加配置
> ```
> {
>     "font_size": 12,
>     "line_numbers": true,
> }  
> ```
  

**自定义快捷键，预览markdown**
> 1. 打开Key Bindings
> `Preferences -> Key Bindings`  
> 2. 增加配置
> ```
> {"keys": ["f6"], "command": "markdown_preview", "args": { "target": "browser"}}
> ```  
***tips***
按F6就可以预览markdown文件啦。
