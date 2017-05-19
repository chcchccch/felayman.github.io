---
layout: post
title:  "Elasticsearch插件head的安装(有坑)"
date:  2017-05-05 23:43:01 +0800
categories: elasticsearch
tag: elasticsearch
sid: 1495172620
---

Elasticsearch出了5.2.1版本之后,就去试试它的新版本的使用，为了以后的升级进行测试。

很遗憾的时候，在安装的过程中就遇到了一个坑，就是elasticsearch-head插件在安装x-pack插件之后就无法使用，原因是因为elastic公司在x-pack中加入了安全模块(security机制),就会出现下面的情况，如图：

![这里写图片描述](http://img.blog.csdn.net/20170221151921072?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMjMzMjczNQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这个时候需要在elasticseach.yml中增加下面几行配置即可解决。

http.cors.enabled: true
http.cors.allow-origin:'*'
http.cors.allow-headers: "Authorization"

然后在每次使用head插件的时候，按照如下的格式输入：

http://localhost:9100/?auth_user=elastic&auth_password=changeme”

其中elastic是默认账号，changeme是初始密码，均可以在x-pack插件中进行修改。

然后就能正常使用head插件了：

![这里写图片描述](http://img.blog.csdn.net/20170221154033909?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMjMzMjczNQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

如果重启elasticsearch之后出现如下错误：

    Exception in thread "main" SettingsException[Failed to load settings from [elasticsearch.yml]]; nested: ElasticsearchParseException[malformed, expected settings to start with 'object', instead was [VALUE_STRING]];

意思是**参数的冒号前后没有加空格,加了之后就好,我找了好久这个问题**

这个时候请不要在服务器上VIM检查这个配置文件,而是把文件下载到本地，用编辑工具如atom来对比官方原版的配置文件。这个问题坑了好久。

参考地址：https://github.com/mobz/elasticsearch-head#url-parameters

