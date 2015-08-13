title: gulp&weiner的分享
speaker: 韩国梁
url: ppt.colinhanks.me
transition: slide
theme：dark


[slide]
# 关于gulp&weiner的分享
## 分享者:韩国梁

[slide]
##什么是node?
---
``` javascript
 var a = function(){
 	console.log(a);
 }
```

[slide]
##关于node的几个常识误区
---
* node源码其实不是js写的，他是c++写的 {:&.moveIn}
* 编写node时，我们必须安装gcc(linux)或者vs2010(window),来实现所用包的编译,常见的例子是bcrypt这个包
* js如何运行c++写的函数呢，node-gyp {:.moveIn}
* 我们平时说的node，准确说是运行node平台，或者node runtime运行时上的
* gulp&weinre&grunt这些，其实是跑在node平台上的包，类似c中的dll文件，java中的jar包，以及php中的那些中间件，一般存放在node_module文件夹下

[slide data-transition="cards"]
##node 可以做什么？
* 作为服务器脚本，运行服务器端代码 {:&.moveIn}
* 使用node-webkit进行包装，编写桌面软件，常见的有adobe brackets等
* 作为前端自动化流的工具，实现前端开发中的自动化（gulp,grunt,browserify,webpack等等）


[slide data-transition="newspaper"]
##什么是gulp？
---
* build stream system
* gulp是个基于文件流的构建系统
* 所谓的文件流，英文单词叫stream，这个也是跟grunt对构建时的本质区别。
* 类似linux中的管道



[slide data-transition="pulse"]
# 谢谢大家，再见