title: 采用Hexo+github搭建属于自己的博客
tags:
- Hexo
- github
categaries: 日志
---
# 1.Hexo介绍
## Hexo是一个简单的静态博客框架。采用命令的方式，我们可以简单的创建属于我们自己的博客。
# 2.Hexo安装
这边主要介绍在Mac环境下的安装和配置
采用命令 npm install -g hexo
＃ 3.创建项目
##  hexo init nodejs-hexo 然后启动服务器是：hexo server
浏览地址是：http://localhost:4000/ 

生成静态页面cd 到你的init目录，执行如下命令，生成静态页面至hexo\\public\\目录。hexo generate （hexo g  也可以）

＃ 配置Github
## 建立与用户名对应的仓库必须是［your_user_name.git.io］,
  修改根目录下的_config.yml文件最底下的deploy:  type: git repository:https://github.com/guosen/guosen.github.io.git
  branch:master 注意：1、冒号跟后面的内容要有空格 2、第二行的type跟左边也要有空格否则出现YMLException.
# 部署
## 就是部署到Github的相应分支，先hexo clean 再生成静态网页hexo generate,然后hexo deploy部署。
# 更换主题
## 替换掉默认主题，首先下载一个主题采用命令：git clone https://github.com/daisygao/hexo-themes-cover.git themes/cove
然后修改 _config.yml文件。再部署OK！

遇到的问题：fatal:multiple stage entries for merged file处理办法
删掉git下的index文件。


参考链接：https://hexo.io/docs/troubleshooting.html
        http://blog.fens.me/hexo-blog-github/

