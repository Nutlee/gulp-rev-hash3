# gulp-rev-hash3
=============

正则匹配出html文件中的css和js路径，根据其文件内容计算出hash值作为版本号.如：
```
<!-- rev-hash -->
<link rel="stylesheet" href="main.min.css?v=9d58b7441d92130f545778e418d1317d">
<!-- end -->
```

> 由于项目打包需求，在已有npm gulp-rev-hash 组件基础上进行了扩展和优化！
> Fork 至 https://github.com/outluch/gulp-rev-hash.git

## 改进
1. 为js，css加上文件hash值作为版本号，对cdn引用的文件仍然可以进行计算hash值；
2. 保留script标签中的其他属性值不改变；
3. 兼容src和href的单引号和双引号包裹；
4. 自动检测代码使用的换行符

## 安装

```
npm install gulp-rev-hash3
```

## 原理

- 原理：正则匹配出html文件中的css和js路径，根据其文件内容计算出hash值作为版本号；
- 具体实现思路：
	1. 先根据<!-- end -->将html拆分成N段；
	2. 对1-N的每一段再根据<!-- rev-hash -->进行拆分，得到2段，第一段为<!-- rev-hash -->前的内容，第二段为包含script和link标签的片段；
	3. 将包含script和link标签的片段进行正则匹配得到其路径，并根据其路径读取其内容从而计算出hash值，作为其版本号，重新得到script和link标签片段；
	4. 最后将各段重新拼接为完整的html。

- 意义：根据静态文件内容来计算其hash，只有在内容改变时，其hash值才会改变，这样，在缓存有效期内，客户端不会再次下载该静态资源，也减轻服务端压力，同时加快渲染提升了用户体验。


## 实例

### Default

默认情况下，打包时，node以gulpfile所在目录为当前目录，然后根据script及link标签中的路径来读取文件，如果script及link设置的路径与gulpfile不在同一目录，则需要配置assetsDir，即静态资源所在目录，读取文件时，会以assetsDir+ js/css path作为路径来读取。

```js
var gulp = require('gulp');
var revHash = require('gulp-rev-hash3');

gulp.task('rev-hash', function () {
    return gulp.src('test/*.html')
        .pipe(revHash({
			assetsDir: 'test',
		}))
        .pipe(gulp.dest('test'));
});
```

#### Input:

```html
<!-- rev-hash -->
<link rel="stylesheet" href="main.min.css">
<!-- end -->

<!-- rev-hash -->
<script src="abc.js"></script>
<script src="def.js"></script>
<!-- end -->
```

#### Output:

```html
<!-- rev-hash -->
<link rel="stylesheet" href="main.min.css?v=9d58b7441d92130f545778e418d1317d">
<!-- end -->

<!-- rev-hash -->
<script src="abc.js?v=0401f2bda539bac50b0378d799c2b64e"></script>
<script src="def.js?v=e478ca95198c5a901c52f7a0f91a5d00"></script>
<!-- end -->
```

### 静态文件为CDN或者域 的方式（CDN对应目录必须为当前项目可访问的目录）

在demo项目中，以gulp-rev-hash3为项目目录，以gulpfile为基准，则项目的目录为"./"，假设需要配置域名cdn.xxxx.com 对应 test目录，则对应的域名配置为
```
{
    domain:"cdn.xxxx.com",
    path:"test"
}
```

在读取静态资源时，node会以 projectPath + domain path + js/css path 作为路径来读取，此时不受assetsDir的影响；

其打包配置为：

```
var gulp = require('gulp');
var rev = require('./index');

gulp.task('test', function() {
    return gulp.src('test/*.html')
        .pipe(rev({
            assetsDir: 'test',
            remotePath:[{
                domain:"cdn.xxxx.com",
                path:"test"
            }],
            projectPath:"./"
        }))
        .pipe(gulp.dest('test'));
});

```

#### Input

```
<!-- rev-hash -->
<link rel="stylesheet" href="main.min2.css"/>
<link rel="stylesheet" href="//cdn.xxxx.com/main.min2.css"/>
<!-- end -->

<!-- rev-hash -->
<script src="abc.js"></script>
<script src="//cdn.xxxx.com/def.js"></script>
<!-- end -->
```

#### Output

```
<!-- rev-hash -->
<link rel="stylesheet" href="main.min2.css?v=aa4488642be0a613ad4e840e9617ee48"/>
<link rel="stylesheet" href="//cdn.xxxx.com/main.min2.css?v=aa4488642be0a613ad4e840e9617ee48"/>
<!-- end -->

<!-- rev-hash -->
<script src="abc.js?v=9c811ea8215aeaac0efff4fae71a9022"></script>
<script src="//cdn.xxxx.com/def.js?v=e5803321a29f976103521afa380a5b52"></script>
<!-- end -->
```

## API参数

示例：
```
assetsDir: 'test',
remotePath:[{
    domain:"cdn.xxxx.com",
    path:"test"
}],
projectPath:"./"
```

说明：

|参数| 说明 | 默认值 |
|---|---|---|
|assetsDir|相对于gulpfile文件，html中引用的js，css资源的路径，即以gulpfile所在目录 为起点，assetsDir+script.src能找到对应的js|""|
|remotePath|域名对应文件目录|无|
|projectPath | 相对于gulpfile，项目目录地址，或者理解为配置domain path时，path所在目录，即以gulpfile所在目录 为起点，projectPath + domain path +script.src能找到对应的js |../，gulpfile上一级|


## 局限

由于计算hash值需要能读取让node读取到其二进制文件，需保证以当前gulpfile所在目录为起点能访问到需要加版本号的静态资源文件（css,js）;

## 版本

### 1.0.7 修复一个发布导致的致命错误

### 1.0.6
1. 根据项目中使用的换行符类型，来决定组装内容时应该使用的换行符。
2. 在gulp配置中加入return，解决与其他task使用时冲突问题。

### 1.0.5
1. 兼容包裹路径的单引号双引号；
2. 保留script标签的其他属性

### 1.0.4
1. 修正一部分代码缩进问题
2. 加版本号时保持原有html缩进

### 1.0.3
去掉冗余兼容

### 1.0.2 
1. 发布版本及文档
2. 添加cdn的配置

### 1.0.1 
初次发布版本