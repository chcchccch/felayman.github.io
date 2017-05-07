[https://www.felayman.com](https://www.felayman.com)

![博客封面](https://github.com/felayman/felayman.github.io/blob/master/assert/index.png =600*400)

## 博客模板说明

演示地址：[Demo](https://www.felayman.com)

### 下载

```
git clone git@github.com:felayman/felayman.github.io.git
```

### 使用说明
 - 该博客模板使用jekyll,最好先去jekyll官方看看相关知识,[jekyll官网](http://jekyll.com.cn/)
 - 该博客部署在Github Pages上,请先参考相关文章,了解如何集成jekyll+Github Pages [集成jekyll+Github Pages](http://blog.csdn.net/u013553529/article/details/54588010)

### 安装说明
#### 本地安装方式
1.  gem install jekyll,安装jekyll(如果已经安装则忽略),jekyll安装需要本地有ruby,请先安装ruby
2. 克隆本项目到本地,进入到安装目录,执行jekyll server
3. 打开浏览器输入：http://localhost:4000/ ,查看运行结果

#### Github Pages方式
1. 在自己的github上新建一个项目,比如 blog
2. 在项目blog的settings中选择Github Pages页面,点击Choose a theme 完成一个 主题选择,此时项目中只有_confug.yml和index.md文件
3. clone你刚才新建的blog项目到本地
4. git clone git@github.com:felayman/felayman.github.io.git ,克隆我的项目到本地,然后用clone后的项目文件替换你自己的blog项目文件,然后push我的项目文件到你的github上去
5. 打开浏览器输入：https://your_github_name.github.io/blog 查看运行结果

### 目录说明
- _draft   存放未发布的文章
- _includes  存放需要引入的文件,一般是head,header,footer等公共样式以及脚本文件,该目录可以建立多个主题模板
- _post  存放需要发布的文章 可以建立不同的子目录作为文章类别区分
- about 存放about/index.html(官方推荐方式),一般用来存放个人简介页面,也可以在项目根目录下建立about.html
- assert  存放发布的文章所需要引用的图片等静态资源文件
- categories 存放不同category文章,一般是列出不同分类文章的列表页面,该页面暂不支持分页
 - home 存放博客首页面index.html,等效于https://www.felayman.com/home/index.html, 该页面支持分页(所有文章列表,时间排序)
 - style 存放样式、图片、脚本、字体等文件
 - tag 存放不同tag的文件(jekyll支持安照category,tag两种方式分类的方式来展示文章列表)
 - CNANE 自己的域名指向github page
 - _config.yml 整个项目的全局变量配置文件
 - index.html 网站入口文件,如输入https://www.felayman.com, 则等效于https://www.felayman.com/index.html


 ### 参考主题
 - Less Or More [Less Or More](https://github.com/luoyan35714/LessOrMore)
 - front-cover  [front-cover](https://github.com/dashingcode/front-cover)


 ### 该主题涉及到的skills
 - markdown语法 [markdown语法](http://daringfireball.net/projects/markdown/syntax)
 - jekyll模板引擎  [jekyll](http://jekyll.com.cn/)
 - liquid模板语言 [liquid](https://shopify.github.io/liquid/)

 ### 打赏

 如果你还在各种云机器上自己部署自己的博客,用着传统的方式来运维自己的博客,不妨试试这种简洁的方式,只需要一个域名+GitHub Pages 就搞定
 如果你使用了该博客后觉得可以让你喜欢上这种简洁的博客,或者在部署过程中遇到问题,或者想定制其他功能,欢迎随时联系,扫描下方二维码即可添加我好友

 #### 我的微信
 ![二维码](https://github.com/felayman/felayman.github.io/blob/master/assert/felayman.png =200*200)

 #### 微信打赏
 ![打赏](https://github.com/felayman/felayman.github.io/blob/master/assert/wechat_pay.png =200*200)





