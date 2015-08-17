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
node不是js的**应用**，而是js**运行平台** {:&.bounceIn}

[slide]
##什么是v8?
---

|chrome|node| {:&.moveIn}
|:-------------|:-----------|
|HTML+javascript |javascript|
|渲染引擎+v8+Gpu...|v8|
|中间层|中间层（libuv)|
|网卡、硬盘、显示器...|网卡、硬盘...|

[slide]
##v8应用有那些?
* phantomjs {:&.zoomIn}
  * 用于测试 {:&.moveIn}
  * 用于爬虫（淘宝）
  * 可以作为前端开发工具，进行代码编写（淘宝移动端和m站代码的同构）
* nodejs
* iojs

[slide]
##npm&commonjs规范
---
好基友，一辈子 {:&.bounceIn}
* npm 是个包管理器，用于解决所有的node包依赖 {:&.moveIn}
* commonjs规范，是用于js模块开发的一个规范，因为js没有模块化，所以commonjs规范很好的解决了这个问题

[note]

```node
 npm install [-g] gulp
```
```javascript
 //符合commonjs规范的一个代码
 var gulp = require('gulp'),
 	 fs = require('fs');

	var obj = {
		path:function(){
			console.log(__dirname);
		}
	}

 module.exports =  obj;
```
[/note]

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
* 使用node-webkit进行包装，编写桌面软件，常见的有adobe brackets,koala等
* 作为前端自动化流的工具，实现前端开发中的自动化（gulp,grunt,browserify,webpack等等）


[slide data-transition="newspaper"]
##什么是gulp？
---
* the streaming build system {:&.rollIn}
* gulp是个基于文件流的构建系统
* 所谓的文件流，英文单词叫stream，这个也是跟grunt对构建时的本质区别。

[slide]
##gulp搭建常用的workflow
---
* js文件的语法检查，预编译，压缩，合并 {:&.fadeIn}
* css文件的预编译，合并，压缩
* img文件的压缩，以及css sprite
* html文件的压缩和version的替换
* 文件的监控,实现livereload


[slide]
##gulp中的对文件流的操作，pipe
---

```shell

 history | grep mobile_sy5
```

pipe是集成了Unix系统大道至简的设计哲学的思想，可以实现文件流的链式操作 {:&.moveIn}


[slide]
## gulp的api

---
* gulp.src {:&.zoomIn}
* gulp.dest
* gulp.task
* gulp.watch

[note]
```javascript
var gulp = require('gulp');

gulp.task('imgmin',function(){
	gulp.src('./img')
		.pipe(imagemin())
		.pipe(gulp.dest('./'))
});

gulp.task('watch',function(){
	gulp.watch(['./js/**/*.js','!./js/index.js'],['imgmin']);
})

```
[/note]

[slide]
##对js文件的语法检查,合并,压缩
用到如下几个库：
* gulp-jshint(jshint-stylish) --js语法检查 {:&.bounceIn}
* gulp-concat --js文件的合并
* gulp-uglify --js文件的压缩
* gulp-rename --重新命名
* gulp-sourcemaps --可以生成map文件
* gulp-coffee(扩展)  --对coffee文件的编译

[note]
安装这些库文件

``` shell
npm install gulp-jshint gulp-concat gulp-uglify gulp-rename gulp-sourcemaps

```

然后在gulpfile.js文件中进行编辑

```javascript

 var gulp = require('gulp'),
 	 jshint = require('gulp-jshint'),
 	 concat = require('gulp-concat'),
 	 uglify = require('gulp-uglify'),
 	 sourcemap = require('gulp-sourcemaps')
 gulp.task('jsmin',function(){
 	gulp.src(['./js/*.js','!./js/index.js]')
 		.pipe(jshint())
 		.pipe(jshint.reporter('default'))
 		.pipe(concat('index.js'))
 		.pipe(sorcemap.init())
 		.pipe(uglify())
		.pipe(sourcemap.write('./'))
		.pipe(gulp.dest('./js'))
 })
```
[/note]


[slide]
##对css文件的合并
---
用到如下几个库：
* gulp-concat --css文件的合并 {:&.bounceIn}
* gulp-cssmin --css文件的压缩
* gulp-rename --重新命名
* gulp-sourcemaps --可以生成map文件
* gulp-less/gulp-sass/gulp-stylus(扩展)  --对less/stylus/sass文件的编译

[note]
安装这些库文件

``` shell
npm install gulp-cssmin

```

然后在gulpfile.js文件中进行编辑

```javascript

 var gulp = require('gulp'),
 	 jshint = require('gulp-jshint'),
 	 concat = require('gulp-concat'),
 	 uglify = require('gulp-uglify'),
 	 sourcemap = require('gulp-sourcemaps'),
	 cssmin = require('gulp-cssmin');
 gulp.task('cssmin',function(){
 	gulp.src(['./css/*.css','!./css/index.css'])
 		.pipe(concat('index.css'))
 		.pipe(sourcemap.init())
 		.pipe(cssmin())
		.pipe(sourcemap.write(./))
		.pipe(gulp.dest('./css'))
 })
```

[/note]

[slide]
##对图片的压缩
---
用到如下几个库：
* gulp-imagemin --对图片的压缩 {:&.bounceIn}
* gulp-rename --重新命名
* gulp-css-spritesmith(扩展)  --对图片文件进行css-sprite

[note]
安装这些库文件

``` shell
npm install gulp-cssmin

```

在gulpfile.js文件中进行编辑

```javascript

 var gulp = require('gulp'),
 	 jshint = require('gulp-jshint'),
 	 concat = require('gulp-concat'),
 	 uglify = require('gulp-uglify'),
 	 sourcemap = require('gulp-sourcemaps'),
	 cssmin = require('gulp-cssmin'),
	 imagemin = require('gulp-imagemin');

 gulp.task('imagemin',function(){
 	gulp.src('./img/*')
 		.pipe(imagemin())
		.pipe(gulp.dest('./img'))
 })

```
[/note]

[slide]
##对文件的监视，livereload
---
用到如下几个库：
* gulp-watch --对文件的监视 {:&.bounceIn}
* gulp-plumber --对错误进行错误的处理


[note]
在gulpfile.js文件中进行编辑

```javascript

 var gulp = require('gulp'),
 	 jshint = require('gulp-jshint'),
 	 concat = require('gulp-concat'),
 	 uglify = require('gulp-uglify'),
 	 sourcemap = require('gulp-sourcemaps'),
	 cssmin = require('gulp-cssmin'),
	 imagemin = require('gulp-imagemin');

 gulp.task('watch',function(){
 	gulp.watch(['./js/**/*.js','!./js/index.js'],['jsmin']);
 })

[/note]

[slide]
##gulp 构建workflow的几个建议
---
* 开发和发布构建两个任务（环境变量，单独写个任务或者脚本) {:&.moveIn}
* 尽量每个任务执行的功能要单一

[slide]
##gulp的思考，gulp是很好的解决方案么?

* browserify {:&.bounceIn}
* webpack(最近很火热)

[slide]
## weinre的分享 
---
万能的鸡肋调试工具 {:&.zoomIn}

DebugGap 支持android端的调试 {:&.zoomIn}

参考教程：http://yujiangshui.com/multidevice-frontend-debug/

[slide]
##工具使用
---
[subslide]

###安装weinre

```shell
 
  npm install -g weinre

```
======
###验证weinre,已经查看帮助

```shell

 weinre --help

```
======
###开启weinre

```shell

  weinre --boundHost -all-

```
======
浏览器打开http://localhost:8080 进行查看

[/subslide]

[slide data-transition="pulse"]
# 以上，分享完毕