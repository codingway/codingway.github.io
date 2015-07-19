---
layout: post
title: "using for jekyll"
date: 2015-07-19 21:25:07
categories: jekyll
---

### jekyll 主页 ###

+ [英文站点](http://jekyllrb.com/)
+ [中文站点](http://jekyllcn.com/)

### jekyll 安装教程 ###

官方默认提供在 linux 下的安装教程，没有提供 window 下的安装教程：

+ [linux 下官方安装教程](http://jekyllrb.com/docs/installation/)
+ [window 下非官方安装教程](http://jekyll-windows.juthilo.com/)

### window 下 jekyll 自定义安装教程 ###

1. 安装 Ruby 和 Ruby DevKit

	注意选择自己对应的平台的版本：[下载 Ruby 和 Ruby DevKit](http://rubyinstaller.org/downloads/)

2. 关联 Ruby DevKit 到 Ruby

		cd C:\RubyDevKit
		ruby dk.rb init
		ruby dk.rb install

	执行 `ruby dk.rb init` 出现如下错误：

		Invalid configuration. Please fix 'config.yml.

	解决办法如下，向 config.yml 添加(添加 Ruby 的安装路径即可，但是注意前面加上 `-` 和 空格，并且要两行或者以上)：

		- D:\Ruby200-x64
		- D:\Ruby200-x64

3. 安装 jekyll

	执行 `gem install jekyll` 出现错误：

		Could not find a valid gem 'jekyll' (>= 0), here is why: Unable to download data from https://rubygems.org/ - Errno::ETIMEDOUT: Operation timed out - connect(2) (https://rubygems.org/latest_specs.4.8.gz)

	解决办法有俩种：

	+ 添加 SSL 证书（这里不做说明，可自行研究）
	+ 更换源

		将默认源：https://rubygems.org/ 更换为：http://rubygems.org/

			gem sources --remove https://rubygems.org/
			gem sources -a http://rubygems.org/

4. Welcome to jekyll

		jekyll new blog
		cd blog
		jekyll serve

	这里预览时会出现问题，是由于在上面我并安装语法高亮组件照成的，解决办法是使用默认的代码块语法：

		......
		Jekyll also offers powerful support for code snippets:

		{\% highlight ruby \%}
		def print_hi(name)
		  puts "Hi, #{name}"
		end
		print_hi('Tom')
		#=> prints 'Hi, Tom' to STDOUT.
		{\% endhighlight \%}
		......

	改成：

		......
		Jekyll also offers powerful support for code snippets:

			def print_hi(name)
			  puts "Hi, #{name}"
			end
			print_hi('Tom')
			#=> prints 'Hi, Tom' to STDOUT.

		......

5. 添加语法高亮

	官方默认使用 Pygments 作为语法高亮，这里我并没有使用它，因为我希望在编写文档时使用默认的代码块语法，通过 jquery + google-code-prettfy 来实现。

	+ [下载 jquery](http://jquery.com/)
	+ [下载 google-code-prettfy](https://code.google.com/p/google-code-prettify/downloads/list)

	将解压出来的 google-code-prettify 文件夹放到 blog 跟目录下，并将获取到的 jquery 脚本 jquery-x.x.x.js 放入 google-code-prettify 文件夹中，在 _include/head.html 中的 head 标签中添加如下代码：

		<!-- google code prettify -->
		<link rel="stylesheet" href="/google-code-prettify/prettify.css" type="text/css"/>
		<script src="/google-code-prettify/prettify.js"></script>
		<script type="text/javascript" src="/google-code-prettify/jquery-x.x.x.js"></script>
		<script>
		$(function(){
		$("pre").addClass("prettyprint");
			prettyPrint();
		})
		</script>

6. 添加多说评论系统

	>Tips: 使用时注意去掉转义符 `\`

	+ 首先[获取多说通用代码](http://duoshuo.com/create-site/)，并保存为 duoshuo.html
	+ 编辑 duoshuo.html:

		将要编辑的部分：

			<!-- 多说评论框 start -->
			<div class="ds-thread" data-thread-key="请将此处替换成文章在你的站点中的ID" data-title="请替换成文章的标题" data-url="请替换成文章的网址"></div>
			<!-- 多说评论框 end -->

		改为

			<!-- 多说评论框 start -->
			<div class="ds-thread" {\% if page.id %}data-thread-key="{\{ page.id }}"{\% endif %}  data-title="{\% if page.title %}{\{ page.title }} - {\% endif %}{\{ site.title }}" data-url={\% if post.url %}data-url="{{ post.url }}"{\% endif %}></div>
			<!-- 多说评论框 end -->

	+ 添加到每篇文章的尾部，编辑 _layout 文件夹下的 post.html 为：

			<div class="post">
				......

				{\% include duoshuo.html %}
			</div>

7. 添加上一篇，下一篇文章的索引

	在 _layout/post.html 中添加如下：


		{\% if page.previous.url %}
		<div> 
			<p>Previous&laquo; <a href="{\{page.previous.url}}">{\{page.previous.title}}</a></p>
		</div>
		{\% endif %} 

		{\% if page.next.url %} 
		<div align=right>
			<p>Next&raquo; <a href="{\{page.next.url}}">{\{page.next.title}}</a></p> 
		</div> 
		{\% endif %}

8. 开启分页功能

	在 _config.yml 中配置，默认 10 篇文章为一页：

		paginate: 1
		paginate_path: "page:num"

	修改 index.html 中导入文章的方法及添加页码：

		......
		<div class="home">
		  <ul class="post-list">
		    {\% for post in paginator.posts %}
		    <li>
		      <span class="post-meta">{\{ post.date | date: "%b %-d, %Y" }}</span>

		      <h2>
		        <a class="post-link" href="{\{ post.url | prepend: site.baseurl }}">{\{ post.title }}</a>
		      </h2>
		    </li>
		    {\% endfor %}
		  </ul>

		  <div>
		    {\% if paginator.page == 1 %}
		    {\% if paginator.total_pages != 1 %}
		    <span>1</span>
		    {\% endif %}
		    {\% else %}
		    <a href="/">1</a>
		    {\% endif %}

		    {\% for count in (2..paginator.total_pages) %}
		    {\% if count == paginator.page %}
		    <span>{{ count }}</span>
		    {\% else %}
		    <a href="/page{{ count }}">{{ count }}</a>
		    {\% endif %}
		    {\% endfor %}
		  </div>
		</div>