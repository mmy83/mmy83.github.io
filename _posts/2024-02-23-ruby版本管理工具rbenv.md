---
title: ruby版本管理工具rbenv
author: mmy83
date: 2024-02-23 19:39:00 +0800
categories: [编程, ruby]
tags: [ruby, 版本管理, rbenv]
math: true
mermaid: true
image:
  path: /images/2024-02-23/ruby版本管理工具rbenv/ruby版本管理工具rbenv-00.jpg
  lqip: data:image/jpeg;base64,/9j/2wBDAAYEBQYFBAYGBQYHBwYIChAKCgkJChQODwwQFxQYGBcUFhYaHSUfGhsjHBYWICwgIyYnKSopGR8tMC0oMCUoKSj/2wBDAQcHBwoIChMKChMoGhYaKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCj/wAARCAAEAAgDASIAAhEBAxEB/8QAFQABAQAAAAAAAAAAAAAAAAAAAAj/xAAfEAABAgcBAQAAAAAAAAAAAAABABECAwQFEhMiFBX/xAAVAQEBAAAAAAAAAAAAAAAAAAACA//EABkRAAIDAQAAAAAAAAAAAAAAAAABAgMyM//aAAwDAQACEQMRAD8Ao22WTxxbfpXOojM6Oad9RkOieWZsQ7AIiIwyit3Rn//Z
  alt: ruby版本管理工具rbenv
---

## 前言

&emsp;&emsp;之前说过，这个博客系统是用的```jekyll```搭建的，而```Jekyll```是用```ruby```开发的，这就需要一个```ruby```环境来运行```jekyll```，虽然自系统搭建以来，没有写过几篇博文，但是运行一直好好的，但是今天发现运行报错了。反复思考原因可能是前两天更新了系统的软件包，导致```ruby```升级。于是就想到可能以后涉及到版本问题也会很多，所有找了一个环境管理工具。各个语言都有类似的管理工具。

## 要求

我的对```ruby```语言了解不多，对这个环境管理工具要求也不高，主要就是可以通过该工具管理```ruby```各个版本并且方便切换。

## 选择

&emsp;&emsp;经过搜索发现用的比较多的有两个：```rvm```和```rbenv```，之所以选择```rbenv```主要是特别简单，并没有打算用```ruby```开发的打算，```rbenv```足够了。

&emsp;&emsp;通过帮助查看他们提供的命令就知道这两个的复杂程度相差不少：

1、RVM

``` shell
Usage:

    rvm [--debug][--trace][--nice] <command> <options>

  for example:

    rvm list                # list installed interpreters
    rvm list known          # list available interpreters
    rvm install <version>   # install ruby interpreter
    rvm use <version>       # switch to specified ruby interpreter
    rvm remove <version>    # remove ruby interpreter (alias: delete)
    rvm get <version>       # upgrade rvm: stable, master

Available commands:

  rvm has a number of common commands, listed below. Additional information about any command
  can be found by executing `rvm help <command>`.

  ruby installation
      fetch                   # download binary or sources for selected ruby version
      install                 # install ruby interpreter
      list                    # show currently installed ruby interpreters
      list known              # list available interpreters
      mount                   # install ruby from external locations
      patchset                # tools related to managing ruby patchsets
      pkg                     # install a dependency package
      reinstall               # reinstall ruby and run gem pristine on all gems
      remove                  # remove ruby and downloaded sources (alias: delete)
      requirements            # installs dependencies for building ruby
      uninstall               # uninstall ruby, keeping it's sources
      upgrade                 # upgrade to another ruby version, migrating gems

  running different ruby versions
      current                 # print current ruby version and name of used gemsets
      do                      # runs a command against specified and/or all rubies
      gemdir                  # display path to current gem directory ($GEM_HOME)
      use <version>           # switch to given (and already installed) ruby version
      use default             # switch to default ruby, or system if none is set
      use system              # switch to system ruby
      wrapper                 # creates wrapper executables for a given ruby & gemset

  managing gemsets
      gemset                  # manage gemsets
      migrate                 # migrate all gemsets from one ruby to another

  rvm configuration
      alias                   # define aliases for `rvm use`
      autolibs                # tweak settings for installing dependencies automatically
      group                   # tools for managing groups in multiuser installations
      rvmrc                   # tools related to managing .rvmrc trust & loading gemsets

  rvm maintenance
      implode                 # removes the rvm installation completely
      cleanup                 # remove stale source files & data associated with rvm
      cron                    # manage setup for using ruby in cron
      docs                    # tools to make installing ri and rdoc docs easier
      get                     # upgrades RVM to latest head, stable or branched version
      osx-ssl-certs           # helps update OpenSSL certs installed by rvm on OS X
      reload                  # reload rvm source itself
      reset                   # remove all default and system settings
      snapshot                # backup/restore rvm installation

  troubleshooting
      config-get              # display values for RbConfig::CONFIG variables
      debug                   # additional information helping to discover issues
      export                  # set temporary env variable in the current shell
      fix-permissions         # repairs broken permissions
      repair                  # lets you repair parts of your environment, such as
                              # wrappers, env files and similar (general maintenance)
      rubygems                # switches version of rubygems for the current ruby
      tools                   # general information about the ruby env
      unexport                # undo changes made to the environment by `rvm export`
      user                    # tools for managing RVM mixed mode in multiuser installs

   information and documentation
      info                    # show the environment information for current ruby
      disk-usage              # display disk space occupied by rvm
      notes                   # display notes with operating system specifics
      version                 # display rvm version (equal to `rvm -v`)

   additional global options
      --debug                 # toggle debug mode on for very verbose output
      --trace                 # toggle trace mode on to see EVERYTHING rvm is doing
      --nice                  # process niceness (increase the value on slow computers, default 0)

```

2、RBENV

``` shell

rbenv 1.2.0
Usage: rbenv <command> [<args>]

Some useful rbenv commands are:
   commands    List all available rbenv commands
   local       Set or show the local application-specific Ruby version
   global      Set or show the global Ruby version
   shell       Set or show the shell-specific Ruby version
   install     Install a Ruby version using ruby-build
   uninstall   Uninstall a specific Ruby version
   rehash      Rehash rbenv shims (run this after installing executables)
   version     Show the current Ruby version and its origin
   versions    List installed Ruby versions
   which       Display the full path to an executable
   whence      List all Ruby versions that contain the given executable

See `rbenv help <command>' for information on a specific command.
For full documentation, see: https://github.com/rbenv/rbenv#readme

```

## 安装

&emsp;&emsp;```rbenv```安装起来很简单：

```shell
brew install rbenv ruby-build
```

## 使用

&emsp;&emsp;使用也是很简单，一共就这几个命令，常用的也就是查看可安装版本、安装、查看已按照版本、切换版本。但是这里面重点需要说一下他有一个初始化操作，我就是因为初始化操作太粗心没有做好，导致切换不了版本，找了好久原因。

```shell

# 初始化

rbenv init 

# Load rbenv automatically by appending
# the following to ~/.zshrc:

eval "$(rbenv init - zsh)"    # 我就是这里出错了，这个是要写到~/.zshrc文件的，而我没有注意，没有写入

# 列出最新的稳定版本：
rbenv install -l

# 列出所有本地版本：
rbenv install -L

# 安装Ruby版本：
rbenv install 3.1.2

# 查看本地已安装版本
rbenv versions
  system         # 系统版本
  3.1.4
* 3.2.3 (set by /Users/mmy83/.ruby-version)   # *代表当前版本
  3.3.0


# 切换本地版本
rbenv local 3.1.2

# 切换全局版本
rbenv global 3.1.2

# 撤销本地版本设置
$ rbenv local --unset

```

## RVM问题

&emsp;&emsp;不知道为什么，我的电脑上被安装了```rvm```,然后进入一些目录就会显示没有```ruby```某个版本，觉得很烦，这里简单记录一下写在

```shell
rvm implode #需要重新打开命令行，提示就消失了
```
