title: Hexo Yilia 主题配置基于swiftype的站内搜索引擎
date: 2016-03-24 16:16:43
tags:
- Hexo
---
# swiftype
## swiftype是为web及移动提供站内搜索的服务，原理十分简单，就是抓取你所配置的网页数据。

#注册
## https://swiftype.com/ 在网站上注册账号，获取Key以及安装脚本；

#修改配置文件
##打开Yilia的_config.yml文件吗，末尾加入如下代码
```bash
swift_search:
  enable: true
```
打开目录hexo\themes\yilia\layout\_partial，打开文件left-col.ejs

添加 swiftype 代码
：
```bash
<% if (theme.swift_search&&theme.swift_search.enable){ %>
		<form class="search" action="<%- config.root %>search/index.html" method="get" accept-charset="utf-8">
			<label>Search</label>
			<input type="text" id="st-search-input" class="st-default-search-input " maxlength="30" placeholder="Search" />
		</form>
	<% } %>

```

在主题目录的css/style.styl文件添加
```bash
.search
  padding 1em 0 0 2.5em
  input
    line-height line-height+0.2
    border 1px solid color-white
    color color-white
    background transparent
    padding-left 4em
    @media tablet
      width 4em
      transition .5s width
      &:focus
        width 8em
  label
    display none
form input.st-default-search-input  {
  font-size: 12px;
  padding: 4px 9px 4px 27px;
  height: 20px;
  width: 190px;
  position: absolute;
  left: 32px;
  top: 70px;
  color: #666;
  border: 1px solid #ccc;
  -webkit-border-radius: 15px;
  -moz-border-radius: 15px;
  -ms-border-radius: 15px;
  -o-border-radius: 15px;
  border-radius: 15px;
  -webkit-box-shadow: inset 0 1px 3px 0 rgba(0,0,0,0.17);
  -moz-box-shadow: inset 0 1px 3px 0 rgba(0,0,0,0.17);
  box-shadow: inset 0 1px 3px 0 rgba(0,0,0,0.17);
  outline: none;
  background: #fcfcfc url(/img/search.png) no-repeat 7px 7px;
}

```
这里面设置搜索框的位置
```bash
 position: absolute;
  left: 32px;
  top: 70px;
```
并且必须修改main.style
的
```bash
.left-col {
z-index: -1;
}
//在overlay里
```

参考文章：http://www.xianingtan.xyz/2015/08/28/Hexo-Yilia-%E4%B8%BB%E9%A2%98%E9%85%8D%E7%BD%AE%E5%9F%BA%E4%BA%8Eswiftype%E7%9A%84%E7%AB%99%E5%86%85%E6%90%9C%E7%B4%A2%E5%BC%95%E6%93%8E/
