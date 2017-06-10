---
title: 使用leancloud给博客添加阅读次数
date: 2017-06-10 14:40:25
tags: JavaScript
categories: JavaScript
---

由于近日利用github.io搭建了一个博客，准备把自己的学习笔记放在博客上，虽然不知道能够坚持多久，但是还是想尽量把
它做好。下面就记录一下利用leancloud给博文添加阅读次数的功能的教程。*该教程大多参考互联网资源*。别问我为什么是
这样，因为我也不懂。

该功能的实现主要分为两大部分，分别是：**leancloud的注册和设置**和**Next模板的设置**

## leancloud的设置

### 注册leancloud帐号

我注册的时候只需要邮箱激活即可，选择开发版是免费的，只不过对数量有限制，这一部分没什么好说的。弄好之后进入dashboard
页面如下：
![](/images/leancloud_next_01.png)

### 查看APP ID和 App Key

在设置中查看**ID**和**Key**，记住它们。当时发现没有复制按钮，自己手动输入的。
![](/images/leancloud_next_02.png)

### 创建Class

在左侧中选择**存储**，点击创建Class，Class的Name一定要起Counter，与后续代码创建的对象一致。
![](/images/leancloud_next_03.png)

### 添加安全域名

回到**设置**中，选择**安全中心**，在**Web安全域名**中添加博客地址
![](/images/leancloud_next_04.png)

以上4个步骤，2-4步没有固定顺序。leancloud的设置就大功告成了。

## hexo next 主题的设置

由于我的博客使用的hexo next主题，估计是烂大街的主题了，因此对应的设置也是基于next主题的。

### 添加主题配置文件

设置主题配置文件**_config.yml**中的leancloud_visitors字段，实现阅读数量的统计
```
leancloud_visitors:
  enable: true
  app_id: #你的app_id
  app_key: #你的的app_key
```

### 添加 lean-analytics.swig 文件

在主题的 layout\_scripts 路径下，新建一个 lean-analytics.swig 文件，加入如下内容：
```
<!-- custom analytics part create by xiamo -->
<script src="https://cdn1.lncld.net/static/js/av-core-mini-0.6.1.js"></script>
<script>AV.initialize("{{theme.leancloud_visitors.app_id}}", "{{theme.leancloud_visitors.app_key}}");</script>
<script>
function showTime(Counter) {
	var query = new AV.Query(Counter);
	$(".leancloud_visitors").each(function() {
		var url = $(this).attr("id").trim();
		query.equalTo("url", url);
		query.find({
			success: function(results) {
				if (results.length == 0) {
					var content = $(document.getElementById(url)).text() + ': 0';
					$(document.getElementById(url)).text(content);
					return;
				}
				for (var i = 0; i < results.length; i++) {
					var object = results[i];
					var content = $(document.getElementById(url)).text() + ': ' + object.get('time');
					$(document.getElementById(url)).text(content);
				}
			},
			error: function(object, error) {
				console.log("Error: " + error.code + " " + error.message);
			}
		});

	});
}

function addCount(Counter) {
	var Counter = AV.Object.extend("Counter");
	url = $(".leancloud_visitors").attr('id').trim();
	title = $(".leancloud_visitors").attr('data-flag-title').trim();
	var query = new AV.Query(Counter);
	query.equalTo("url", url);
	query.find({
		success: function(results) {
			if (results.length > 0) {
				var counter = results[0];
				counter.fetchWhenSave(true);
				counter.increment("time");
				counter.save(null, {
					success: function(counter) {
						var content = $(document.getElementById(url)).text() + ': ' + counter.get('time');
						$(document.getElementById(url)).text(content);
					},
					error: function(counter, error) {
						console.log('Failed to save Visitor num, with error message: ' + error.message);
					}
				});
			} else {
				var newcounter = new Counter();
				newcounter.set("title", title);
				newcounter.set("url", url);
				newcounter.set("time", 1);
				newcounter.save(null, {
					success: function(newcounter) {
					    console.log("newcounter.get('time')="+newcounter.get('time'));
						var content = $(document.getElementById(url)).text() + ': ' + newcounter.get('time');
						$(document.getElementById(url)).text(content);
					},
					error: function(newcounter, error) {
						console.log('Failed to create');
					}
				});
			}
		},
		error: function(error) {
			console.log('Error:' + error.code + " " + error.message);
		}
	});
}
$(function() {
	var Counter = AV.Object.extend("Counter");
	if ($('.leancloud_visitors').length == 1) {
		addCount(Counter);
	} else if ($('.post-title-link').length > 1) {
		showTime(Counter);
	}
}); 
</script>
```

**这一步不是必须的，在博主下载的版本当中，已经在layout\_third-party\analytics包含了此文件。因此，如果你的next
里有这个文件，只需要确保_layout.swig文件中有**：
```
{% include '_third-party/analytics/lean-analytics.swig' %}
```
即可，事实证明是有的，因此，以上这步其实是多余的。这也是为什么刚刚说到Class的名字为什么要是Counter，因为模板
里就是创建了Counter的对象，如果是自己写，那么类的名字只要统一就好，没有要求。

### 修改post.swig文件

在适当的位置添加：
```
{% if theme.leancloud_visitors.enable %}
		  	 <span id="{{ url_for(post.path) }}"class="leancloud_visitors"  data-flag-title="{{ post.title }}">
              |   {{__('post.visitors')}}
            </span>
		  {% endif %}
```
这一步在最新版本也是多余的。

### 修改语言配置文件
添加visitors字段，英文在themes\next\languages\en.yml，中文在themes\next\languages\zh-Hans.yml
```
post:
  posted: 发表于
  visitors: 阅读次数
  updated: 更新于
  in: 分类于
  read_more: 阅读全文
  untitled: 未命名
```

到此，已经大功告成了！！现在，你可以使用hexo g和hexo d部署到服务器上，我使用的是github.io服务器，看看效果。
已经有了新的字段：
![](/images/leancloud_next_05.png)

当我们点击博客后，在leancloud的后台就可以看见访问记录了。
![](/images/leancloud_next_06.png)

最后，注意日期和博文标题联合构成了主键，因此，不要改变博文的名字和日期，否则阅读数就会清零了。