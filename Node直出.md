## Node直出

create by yangjianan 7/8/2017

# 目录

- 三大模式
- 具体流程
- 直出特点
- 常用模板引擎
- DEMO

### Node直出之外的模式？

> #模式 1 - 前后分离

从用户输入 url　到展示最终页面的过程，这种模式可简单的分为以下 5 部分

1. 用户输入 url，开始拉取静态页面
2. 静态页面加载完成后，解析文档标签，并开始拉取 CSS （一般 CSS 放于头部）
3. 接着拉取 JS 文件（一般 JS 文件放于尾部）
4. 当 JS 加载完成，便开始执行 JS 内容，发出请求并拿到数据
5. 将数据与资源渲染到页面上，得到最终展示效果

具体流程图如下

![](https://raw.githubusercontent.com/yangjianan/NodeEngine/master/image/%E6%A8%A1%E5%BC%8F1.png)

这种处理形式应该占据大多数，然而也很容易发现一个问题就是请求数多，前后依赖大，如必须等待 JS 加载完成后执行时才会发起数据请求，等待数据回来用户才可以展示最终页面，这种强依赖的关系使得整个应用的首屏渲染耗时增加不少。

> #模式 2 - 数据直出

在模式 1 中，第 1 点用户输入 url 时 server 端不做其他处理直接返回 html ，在第 4 点向 server 请求获取数据。那么，同样都是向 server 请求获取，如果在第 1 点中将请求数据放在 server 上，将拿到的数据拼接到 HTML 上一并返回，那么可减少在前端页面上的一次数据请求时间。 这就是模式 2 - 数据直出所做的事，处理方式也很简单

1. 用户输入 url ，在 server 返回 HTML 前去请求获取页面需要的数据
2. 将数据拼接到 HTML 上 并 一起返回给前端
3. 直接拿该数据进行渲染页面，不再做数据请求

具体可看下面的流程图这种模式下

![](https://raw.githubusercontent.com/yangjianan/NodeEngine/master/image/%E6%A8%A1%E5%BC%8F2.png)

###经典的jsp列子：

![](https://raw.githubusercontent.com/yangjianan/NodeEngine/master/image/01.png)

后端代码：

![](https://raw.githubusercontent.com/yangjianan/NodeEngine/master/image/02.png)



模式2与模式1 相比，这种模式减少了请求数据。这块差距有多少呢？

###发起一个 HTTP 的网络请求过程

    DNS解析（100~200ms可以缓存）
         |
         |
        建立TCP链接 (三次握手过程100~200ms )
                |
                |
            HTTP Request( 半个RTT ) 
                   |
                   |
              HTTP Response( RTT 网络拥塞程度而定)

注: RTT 为 Round-trip time 缩写，表示一个数据包从发出到返回所用的时间。

###数据请求在前端和后端发出，差距有多少？

由上面对 HTTP 的网络请求过程可看到建立一次完整的请求返回在耗时上明显的，特别是外网用户在进行 HTTP 请求时，由于网络等因素的影响，在网络连接及传输上将花费很多时间。而在服务端进行数据拉取，即使同样是 HTTP 请求，由于后端之间是处于同一个内网上的，所以传输十分高效，这是差距来源的大头，是优化的刚需。

#在模式2寻求优化？

> #模式 3 - 直出 (服务端渲染)

### Node直出是什么？

> 直出简单说白了其实就是“服务端渲染并输出”，跟起初我们提及的前后端的开发模式基本类似，只是后端语言我们换成了 node 。

### 直出主要的作用？

> 
1. 加快首屏渲染时间
2. 优化前端渲染难以克服的 SEO 问题

### 到底是怎样做的优化？

模式3数据请求能放到 server 上，对于数据与HTML结合处理也可以在server上做，从而减少等待 JS 文件的加载时间。 这就是模式3 - 直出 (服务端渲染)，主要处理如下

1. server 上获取数据并将数据与页面模板结合，在服务端渲染成最终的 HTML
2. 返回最终的 HTML 展示

可以从下图看出，页面的首屏展示不再需要等待 JS 文件回来，优化减少了这块时间

![](https://raw.githubusercontent.com/yangjianan/NodeEngine/master/image/%E6%A8%A1%E5%BC%8F3.png)

通过以上模式，将模式 1 - 常用模式中的第 3 和 4 点耗时进行了优化，那么可以再继续优化吗？

在页面文档不大情况下，可将CSS内联到HTML中，这是优化请求量的做法。直出稍微不同的是需要考虑的是服务端最终渲染出来的文档的大小，在范围内也可将 CSS 文件内联到 HTML 中。这样的话，便优化了 CSS 的获取时间，如下图

![](https://raw.githubusercontent.com/yangjianan/NodeEngine/master/image/%E6%A8%A1%E5%BC%8F3-1.png)


### 既然是后端渲染，就需要模板引擎，有哪些？

###1.art-Template

> art-template 是一个简约、超快的模板引擎。它采用作用域预声明的技术来优化模板渲染速度，从而获得接近 JavaScript 极限的运行性能。

####index.js:

	var artTemplate = require("express-art-template");
	var express = require("express");
	var app = express();
	
	//设置渲染引擎
	app.engine('art', artTemplate);
	
	//设置模板目录为当前index.js目录同级views目录下的模板
	app.set("views", __dirname + "/views");
	
	//设置使用当前目录
	app.use(express.static(__dirname));
	
	app.get("/", function(req, res) {
		
		//渲染页面并传值
		res.render('template.art', {
			
			userA:{
				name: "kobe",
				age:"18",
			},
			userB:{
				name: "tracy",
				age:"19",
			},
		});
	});
	
	//监听3000端口
	app.listen(3000);

####template.art:

	<!DOCTYPE html>
	<html>
		<head>
			<meta charset="UTF-8">
			<title></title>
		</head>
		<body>
			<h2>Demo</h2>
			
			<!-- <% %>:art-Template的原始语法 -->
			<% if(userA.name && userA.age){ %>
				<p>Hi,My name is <%= userA.name%>, I am <%= userA.age%> years old</p>
			<% } %>
			
			<!-- {{ }}:art-Template的标准语法 -->
			{{ if userB.name && userB.age }}
				<p>Hi,My name is {{ userB.name }}, I am {{ userB.age }} years old</p>
			{{/if}}
			
		</body>
	</html>


artTemplate 见[artTemplate](https://aui.github.io/art-template/docs/)

###2. ejs

> ejs因为其简单的语法结构，与 express 集成良好，是目前用的最广泛的nodejs模版引擎，功能齐全，各方面中规中矩，主要是代码干净整洁，更让人上手易懂。

####index.js:

	var express = require("express");
	var app = express();
	
	//设置渲染引擎
	app.set("view engine", 'ejs');
	
	//设置模板目录为当前index.js目录同级views目录下的模板
	app.set("views", __dirname + "/views");
	
	//设置使用当前目录
	app.use(express.static(__dirname));
	
	app.get("/", function(req, res) {
		
		//渲染页面并传值
		res.render('template.ejs', {
			name: "kobe",
			age:"18",
		});
	});
	
	//监听3000端口
	app.listen(3000);

####template.ejs:
	
	<!DOCTYPE html>
	<html>
		<head>
			<meta charset="UTF-8">
			<title></title>
		</head>
		<body>
			<h2>Demo</h2>
			
			<!-- <% %>:ejs的语法 -->	
			<% if(name && age){ %>
				<p>Hi,My name is <%= name%>, I am <%= age%> years old</p>
			<% } %>
		
		</body>
	</html>

ejs 见[ejs](https://github.com/tj/ejs)

###3. jade

> Jade主要是面向后端开发人员，它能以最少的代码量最快的速度构建出一个像模像样的网页架构。如果你是一个全栈开发人员（偏向后端），自己动手丰衣足食，并且不会有其他前端人员来维护你的页面，你可以尝试一下jade，它可以使你的开发效率有质的飞跃！

[jade基础语法](http://dtop.powereasy.net/Item/3832.aspx)

####index.js:

	var express = require("express");
	var app = express();
	
	//设置渲染引擎
	app.set("view engine", 'jade');
	
	//设置模板目录为当前index.js目录同级views目录下的模板
	app.set("views", __dirname + "/views");
	
	//设置使用当前目录
	app.use(express.static(__dirname));
	
	app.get("/", function(req, res) {
		
		//渲染页面并传值
		res.render('template.jade', {
			name: "kobe",
			age:"18",
		});
	});
	
	//监听3000端口
	app.listen(3000);

####template.jade:

	html
		head
			meta(charset="UTF-8")
			title
		body
			h2(id="content") Demo
	
			<!-- #{name}:jade的语法 -->
			if name&&age
				p(class="user") Hi,My name is #{name}, I am #{age} years old
		
jade 见[jade](https://github.com/pugjs/pug)

###4. Handlebars

> handlebars作为一个非常轻量级的模板引擎，单纯从模板这个功能上看，他的前后端通用性强，命令简单明了，代码可读性强。如果应用不是很大，推荐使用。

####index.js:
	
	var express = require("express");
	var app = express();
	
	//设置渲染引擎
	app.set("view engine", 'hbs');
	
	//设置模板目录为当前index.js目录同级views目录下的模板
	app.set("views", __dirname + "/views");
	
	//设置使用当前目录
	app.use(express.static(__dirname));
	
	app.get("/", function(req, res) {
		
		//渲染页面并传值
		res.render('template.hbs', {
			
			user:{
				name: "kobe",
				age:"18",
			},
		});
	});
	
	//监听3000端口
	app.listen(3000);

####template.hbs:

	<!DOCTYPE html>
	<html>
		<head>
			<meta charset="UTF-8">
			<title></title>
		</head>
		<body>
			<h2>Demo</h2>
			
			<!-- {{name}}:Handlebars的语法 -->
			{{#if user.name}}
				<p>Hi,My name is {{ user.name }}, I am {{ user.age }} years old</p>
			{{/if}}
			
		</body>
	</html>

Handlebars 见[Handlebars](https://github.com/wycats/handlebars.js)

###各个模板引擎性能对比图

![](https://raw.githubusercontent.com/yangjianan/NodeEngine/master/image/03.png)

浅谈模板引擎原理 见[http://www.cnblogs.com/dojo-lzz/p/5518474.html](http://www.cnblogs.com/dojo-lzz/p/5518474.html)

#直出DEMO

#总结

Node直出能够将常用模式优化到剩下了一次 HTML 请求，加快首屏渲染时间，使用服务端渲染，还能够优化前端渲染难以克服的 SEO 问题。而不管是简单的 数据直出 或是 服务端渲染直出 都能使页面的性能优化得到较大提高。

在前后端没有分离时 使用后端渲染出模板的方式是与文中所述的直出方案效果是一致的，前后端分离后淡化了这种思想，Node 的发展让更多的前端开始做后端事情，直出的方式也越来越被重视了，直出方案看似回到了服务端渲染的原点，实际上是在以前的基础上盘旋上升。








