---
layout: post
title: "Ruby On Rails环境搭建"
description: "[Ruby On Rails](http://rubyonrails.org/ 'Ruby On Rails')是最流行和体验最好的敏捷开发WEB框架（没有之一），介绍性的内容这里就不赘述了。本文介绍下Ruby On Rails环境的搭建流程。"
category: 技术
tags: [ruby on rails]
---
{% include JB/setup %}

*本文来源于[http://weizhifeng.net/ruby-on-rails.html](http://weizhifeng.net/ruby-on-rails.html)*
*作者：JeremyWei*

-----------------------------------------------------

![Ruby On Rails](/assets/images/rails.png  "Ruby On Rails")

**前言**

[Ruby On Rails](http://rubyonrails.org/ "Ruby On Rails")是最流行和体验最好的敏捷开发WEB框架（没有之一），介绍性的内容这里就不赘述了。本文介绍下Ruby On Rails环境的搭建流程。

环境如下：

* 操作系统采用[Ubuntu 12.04.1 LTS (Precise Pangolin)](http://releases.ubuntu.com/12.04/ "Ubuntu 12.04.1 LTS (Precise Pangolin)") Server Editon
* [Ruby](http://www.ruby-lang.org/en/ "Ruby")版本为1.9.3-p0
* [Rails](http://rubyonrails.org/ "Rails")为3.2.8

默认情况下Ubuntu有很多库和软件没有安装，我们首先安装一下所需要的库：

	$ sudo apt-get upgrade 
	$ sudo apt-get install build-essential libssl-dev libreadline-gplv2-dev lib64readline-gplv2-dev zlib1g zlib1g-dev libyaml-dev libsqlite3-dev
	
**安装Ruby**

由于系统自带的Ruby版本较低，所以这里我们先安装Ruby：

	$ wget http://ftp.ruby-lang.org/pub/ruby/1.9/ruby-1.9.3-p0.tar.gz
	$ tar xzvf ruby-1.9.3-p0.tar.gz
	$ cd ruby-1.9.3-p0
	$ ./configure
	$ make
	$ sudo make install
	
安装完成之后，查看下Ruby信息：

	$ ruby -v
	ruby 1.9.3p0 (2011-10-30 revision 33570) [i686-linux]

**安装Gem**

Rails需要通过[Gem](https://rubygems.org/ "Gem")来安装，下面安装Gem：

	$ wget http://production.cf.rubygems.org/rubygems/rubygems-1.8.24.tgz
	$ tar -zxvf rubygems-1.8.24.tgz
	$ cd rubygems-1.8.24
	$ ruby setup.rb
	RubyGems 1.8.24 installed
	== 1.8.24 / 2012-04-27

	* 1 bug fix:

	  * Install the .pem files properly. Fixes #320
	  * Remove OpenSSL dependency from the http code path

	---------------------------------------------------------------------

	RubyGems installed the following executables:
		/usr/local/bin/gem
		
更新Gem：

	$ gem update --system 
	Latest version currently installed. Aborting.
	
**安装Rails**

接下来安装Rails:

	$ sudo gem install rails
	Successfully installed i18n-0.6.1
	Successfully installed multi_json-1.3.6
	Successfully installed activesupport-3.2.8
	Successfully installed builder-3.0.3
	...
	...
	Successfully installed rack-ssl-1.3.2
	Successfully installed thor-0.16.0
	Successfully installed railties-3.2.8
	Successfully installed bundler-1.2.1
	Successfully installed rails-3.2.8
	28 gems installed
	
如果出现以下的错误信息：

	ERROR:  Loading command: update (LoadError)
		no such file to load -- zlib
	ERROR:  While executing gem ... (NameError)
		uninitialized constant Gem::Commands::UpdateCommand
		
这是因为Ruby的zlib模块没有安装，安装zlib模块：

	sudo apt-get install zlib1g-dev
	cd /ruby-source-files/ext/zlib
	ruby extconf.rb
	make
	sudo make install
	
如果出现以下错误信息：

	Installing ri documentation for rails-3.2.8...
	file 'lib' not found
	Installing RDoc documentation for rails-3.2.8...
	file 'lib' not found
	
是因为rdoc没有安装，安装rdoc：

	sudo gem install rdoc
	
安装[Rails Completion](https://github.com/jweslley/rails_completion "Rails Completion")：

	$ wget -O ~/.rails.bash https://raw.github.com/jweslley/rails_completion/master/rails.bash
	$ echo source ~/.rails.bash >> ~/.bashrc
	$ source ~/.bashrc
	
**第一个应用**

Rails安装完成之后，你可以创建自己的Rails应用了：

	$ cd /srv
	$ rails new myapp
	create  
	create  README.rdoc
	create  Rakefile
	create  config.ru
	create  .gitignore
	create  Gemfile
	...
	...
	Using rails (3.2.8) 
	Using sass (3.2.1) 
	Using sass-rails (3.2.5) 
	Using sqlite3 (1.3.6) 
	Installing uglifier (1.3.0) 
	Your bundle is complete! Use `bundle show [gemname]` to see where a 
	bundled gem is installed.
	
Rails应用创建成功之后，你可以测试下这个应用是否可以运行：

	$ cd myapp
	$ rails server
	rails server
	=> Booting WEBrick
	=> Rails 3.2.8 application starting in development on http://0.0.0.0:3000
	=> Call with -d to detach
	=> Ctrl-C to shutdown server
	[2012-09-23 15:02:22] INFO  WEBrick 1.3.1
	[2012-09-23 15:02:22] INFO  ruby 1.9.3 (2012-04-20) [x86_64-darwin12.0.0]
	[2012-09-23 15:02:22] INFO  WEBrick::HTTPServer#start: pid=9907 port=3000
	
执行以上的操作后，系统会启动WEBrick服务器，你可以通过http://127.0.0.1:3000来访问应用的内容了。不过WEBrick只可以作为测试用， 不能应用在大规模的生产环境中，所以我们需要一个高性能的服务器，这就是下面将要介绍的[Unicorn](http://unicorn.bogomips.org/ "Unicorn")。
