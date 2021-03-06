---
layout: post
title: Linux下编译YouCompleteMe插件
description: 最近发现一款很好用的智能插件YouCompleteMe，YouCompleteMe是现在见过最好用的智能补全插件，支持的语言有C、C++、objective-c、python、C#等，特别好用，特别推荐。而且不像其他智能补全插件那样需要太多依赖，例如OmniCppComplete需要依赖ctag生成的tags文件。最后一点，YouCompleteMe插件的作者还在更新，很热心，我配置出现问题，然后去项目主要下留言，作者细心的回复了~
category: blog
tags: vim
published: true
---

{{ page.description }}  
这里就不详细介绍了，如果感兴趣的同学可以去[YouCompleteMe项目主页](http://valloric.github.io/YouCompleteMe/)围观~

## 一、下载YouCompleteMe ##
因为是使用vundle来管理插件，所以也就通过YouCompleteMe(简称YCM)来下载插件，通过vundle下载YCM简单，只需要在`vimrc`中添加`Bundle 'valloric/YouCompleteMe'`，然后重新打开一个vim，在NORMAL模式下执行命令`:BundleInstall`，然后等待下载完成就ok  
遇到的问题：  
1. 使用vundle安装完成后，最好切换到YouCompleteMe目录下执行`git submodule update --init --recursive`确认clone的仓库完整性  
2. vundle下载查看不到进度，可以直接通过命令`git clone --recursive https://github.com/Valloric/YouCompleteMe.git`进行手动下载，然后同样切换到YouCompleteMe目录下通过`git submodule update --init --recursive`命令检查仓库的完整性，最后将YouCompleteMe目录拷贝到~/.vim/bundle/插件目录下即可  
3. 在检查仓库的完整性时出现类似`Unable to find current revision in submodule path 'third_party/argparse'`这样的错误，造成这个错误主要是因为YouCompleteMe下的third_party/argpares目录已经存在了，删除third_party/argpares目录，重新执行`git submodule update --init --recursive`即可

## 二、下载安装cmake ##
因为YouCompleteMe是通过cmake来管理安装、编译的，所以在编译前需要安装cmake，当然安装cmake的方式有三种：源码编译、下载二进制文件、包管理方式安装，为了方便，我就直接选择了包管理方式安装，在终端下执行`sudo apt-get install cmake`命令就能自动安装了。

## 三、下载安装clang ##
YouCompleteMe需要使用clang来提供语法分析、自动补全等功能，同样安装方法有三种，但是我用的系统是Ubuntu 12.04 LTS 32位，直接源码编译clang不能通过，官方网站上已经指出这是一个bug，所以只得放弃这种方式了。在Ubuntu下通过包管理方式安装的clang版本太低了，也不采用。于是只剩下最后一种方式了：下载二进制文件，庆幸的是这是这种方法可行，不然就悲剧了。打开[clang官方下载地址](http://llvm.org/releases/download.html#3.2)，适合Ubuntu 12.04 LTS 32位的只有版本3.2的，下载其中的`Clang Binaries for Ubuntu-12.04/x86(67M)(.sig)`，下载完成后解压即可。  
ps：1. 编译时直接使用./install.sh命令下载的clang版本可能有问题，导致YouCompleteMe中某些功能不可用,故我编译时没有直接使用install.sh脚本  
    2. 各个版本的YouCompleteMe可能需要不同版本的clang，故在更新clang的同时注意更新clang,否者同样可能导致某些功能不可用  

## 四、编译YouCompleteMe ##
1. 编译YouCompleteMe需要编译环境和python支持，依次在终端下执行下列命令进行安装：  

        sudo apt-get install build-essential
        sudo apt-get install python python-dev   

2. YouCompleteMe需要vim版本为7.3.548或者更高，如果版本不够，需要编译vim，如何编译vim请参考这篇文章:[Ubuntu 12.04下编译Vim](http://hahaya.github.io/2013/07/25/build-vim-on-ubuntu.html)  
3. 切换到用户主目录下，为编译文件新建一个目录，然后设置cmake的编译选项，最后开始编译，在终端下依次执行一下命令  

        cd ~
        mkdir ycm_build
        cd ycm_build
        cmake -G "Unix Makefiles" -DEXTERNAL_LIBCLANG_PATH=~/clang_llvm_3.2/lib/libclang.so . ~/.vim/bundle/YouCompleteMe/cpp
        make ycm_core  
        make ycm_support_libs  
        

## 五、配置YouCompleteMe ##
1. 由于自己英文太稀烂，就没有认真看文档，以为不需要什么配置，然后编译好YCM之后，就直接新建了一个C文件，结果打开vim时提示`No .ycm_extra_conf.py file detected, so no compile flags are available. Thus no semantic support for C/C++/ObjC/ObjC++.`。然后自己一直没弄明白什么意思，就去了作者项目主页下留言，作者很热心，很认真的回复了我的问题。最后结合作者的建议，然后自己查文档，其实解决方法很简单，只需要设置`.ycm_extra_conf.py`文件的位置即可，在`.virmc`文件中添加如下内容`let g:ycm_global_ycm_extra_conf = '~/.vim/bundle/YouCompleteMe/cpp/ycm/.ycm_extra_conf.py'`，然后重新打开vim即可  
2. 为了补全C++代码，在Linux下我们需要修改.ycm_extra_conf.py文件中的flags部分，使用`-isystem`添加系统的头文件进行解析，使用`-I`添加第三方的头文件进行解析，在flags部分后添加如下内容：  

        '-isystem',  
        '/usr/include',  
        '-isystem',  
        '/usr/include/c++/4.8',  
        '-isystem',  
        '/usr/include',  
        '/usr/include/x86_64-linux-gun/c++',  


英文不好是硬伤，给YCM作者留言的几句话花了半天才写出来，而且还多优秀文章是英文的，看来以后得加强英文的学习...
