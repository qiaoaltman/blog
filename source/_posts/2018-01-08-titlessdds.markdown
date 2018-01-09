---
layout: post
title: "世界很好 如果你也在"
date: 2018-01-08 19:37:27 +0800
comments: true
categories: java 
---
# 如何为 ThinkJS 3 网站优化 TTFB 时间

今年早些时候，奇舞团开源的 Node.js 框架 ── [ThinkJS](https://thinkjs.org/) 迎来了她的 3.0 版本。尽管今年我很少更新博客，但「每次 ThinkJS 发布大版本，我都要更新博客程序」的老传统还是不能丢。所以，你现在看到的这个博客，已经是基于 ThinkJS 3 全面重构后的新版。<!-- more -->

ThinkJS 3 基于 [Koa](https://koajs.com) 2.x 开发，内核实现得非常小巧，框架通过 Middleware（兼容 Koa）、Adapter、Extend 等机制来扩展出强大而丰富的功能。按照惯例，ThinkJS 大版本之间无法平滑进行，但这次升级带来的工作量不算太大，本站的升级工作花了一下午全部完成。

基于 ThinkJS 开发的网站普遍都很快，这篇文章我打算聊聊如何为 ThinkJS 3 网站优化 TTFB 时间，使之变得更快。

Time to first byte（简称 TTFB）时间，又称首字节时间，是 WEB 性能优化中非常重要的指标。它代表着从浏览器发起 HTTP 请求到收到 HTTP 响应第一个字节的这段时间，包含了 DNS 解析、建立 TCP 连接、建立 SSL 连接、发送 HTTP 请求、网络传输、服务端处理、30X 重定向等阶段。在影响 TTFB 所有因素中，服务端程序何时输出响应决定了服务端处理时间的长短，也是本文关注的优化目标。

优化 WEB 页面的 TTFB 时间除了要尽可能优化业务逻辑之外，还有两个常用技巧：

* 多个 HTTP 请求响应（动静拆分）；
* 一个 HTTP 请求多次响应（分块传输）；

前者无非就是先尽快输出一个无服务端复杂逻辑的空壳页面，再发起 ajax、jsonp 等异步请求填充内容。这种方案不利于 SEO，比较适用于单页应用。

像本站这种以内容为主的 Web 页面，非常适合采用第二个技巧来优化 TTFB 时间。本文重点介绍它。

分块传输响应需要用到我之前介绍过的 [HTTP Transfer-Encoding: chunked](https://imququ.com/post/transfer-encoding-header-in-http.html) 机制。有了这个机制，服务端可以随时将已经完成的部分响应发送给给客户端，而不必等待全部完成后再一次发送。浏览器拿到部分响应，就能解析并执行其中的 HTML、CSS 和 JS 代码，还能加载其中引用的子资源，最终让用户更快看到部分页面内容。分块传输也是 Facebook 在 2009 年实现的 Bigpipe 方案的理论基础，这里不再赘述。

再来说说 ThinkJS。

在 ThinkJS 之前几个版本中，我们可以通过 `http.write(content)` 发送多个 chunk，再通过 `http.end(content)` 发送最后一个 chunk，非常方便。

而 ThinkJS 3 使用的 Koa 2.x，只能通过 `ctx.body` 设置并结束响应，意味着通常情况下响应只能发送一次，还得放在整个 Controller 流程的最后。

通过分析代码，我找到在 ThinkJS 3 中多次发送响应的两种方案：

* `ctx.body` 支持传入 Stream，创建 Readable 流 `rs` 并多次调用 `rs.push(content)` 可以多次发送 chunk，调用 `rs.push(null)` 可以结束响应；
* Koa 代码层面上并没有禁止我们使用 `ctx.res`，通过 `res` 对象可以完全控制响应；

方案一比较正统；方案二则危险得多，官方都说要后果自负：

> Bypassing Koa's response handling is not supported. Avoid using the following node properties: res.statusCode, res.writeHead(), res.write(), res.end(). [via](http://koajs.com/#ctx-res)
 
所以，本站最终采用了方案二。Koa 总共就几个文件，出啥奇怪的问题都不怕。

下面开始贴代码。

1）创建 Controller 的 Extend 文件 `src/extend/controller.js`：

```javascript
const firstChunkMinLength = 4096;

module.exports = {
	async renderAndFlush(tpl) {
		let content = await this.render(tpl);

		//first chunk
		if(!this.ctx.headerSent) {
			this.ctx.type = 'html';
			this.ctx.flushHeaders();

			let length = content.length;
			if(length < firstChunkMinLength) {
				content += `<s>${' '.repeat(firstChunkLength - length)}</s>`;
			}
		}

		this.ctx.res.write(content);
	}
};
```

输出第一个 chunk 之前，需要通过 ctx 的 `type` setter 和 `flushHeaders` 方法来输出响应起始行和头部。

第一个 chunk 不能太小，否则会被某些浏览器缓存起来，不会马上显示，达不到我们想要的效果。更多细节可以点开这个 [stackoverflow 的链接](https://stackoverflow.com/questions/16909227/using-transfer-encoding-chunked-how-much-data-must-be-sent-before-browsers-s)自己看。另外，我在实际测试中发现，只补空格 iOS Safari 依然不会立刻渲染，把空格放在标签里就没问题。也可能是我的幻觉，欢迎大家测试并指正。

2）在 Controller 里将原本渲染模板的逻辑根据实际情况拆分为多步：

```javascript
async indexAction() {
	let pageName = 'blog-home';
	let title = 'JerryQu 的小站';
	this.assign({ pageName, title});
	
	//输出头部和边栏 HTML
	await this.renderAndFlush('home/inc/header');

	//查询数据库（耗时操作）
	let pn = this.get('pn');
	let data = await this.model('post').getPostList(pn, 10);
	data.pagerPath = getPagerPath(this.ctx, 'pn');
	this.assign(data);
	
	//输出剩余 HTML
	return this.display('home/index_post_list');
}
```

也就是说需要提前发送的模板通过 renderAndFlush 来渲染并发送，剩余模板还是走原有的 display 逻辑。

至此，本文要介绍的优化工作已经完成，赶紧打开浏览器验证一下吧。

但是如果你照着我的代码改造，肯定会遇到不少坑，下面列举几个：

1）原本的异常逻辑重定向到错误页不好用，直接提示 `Can't set headers after they are sent.` 错误。

这个错误信息已经把原因描述得很明白，HTTP/1 的响应需要严格按照起始行、头部和正文的顺序发送，已经发送了正文，就不能再通过 30X 状态码和 `Location` 头部来跳转页面。

解决方案：有一些会产生跳转的逻辑例如参数合法性检查，可以挪到发送第一个分块之前来进行。另一些异常跳转逻辑则无法提前，例如查询数据库后发现不存在对应的文章，这种情况可以输出一段 JS 代码在浏览器中跳转，或者直接渲染错误页面。

2）提前输出的模板中部分变量取不到值。

例如本站第一个分块输出了左侧内容，这个分块对应的模板中，有很多字段原本来自数据库查询后的结果，提前渲染必然取不到值。

解决方案：这些需要用到数据库字段的逻辑，有一些可以挪到后续分块中；有一些则不好后移，例如需要动态赋值的页面 `<title>`，只能放在 `<head>` 里。一种方案是继续使用万能的 JS，通过后续分块中的 `docuemnt.title` 来给页面 title 赋值。对于不支持 JS 的 Spider，可以禁用提前输出响应策略。

我使用了另外一套方案：由于文章数量不多，我索性在程序启动时，把全部文章 ID 和标题对应关系从数据库取出来，存在配置中。这样，后续第一个分块获取标题时无需查询数据库。

下面这段代码需要放在 `src/bootstrap/worker.js`：

```javascript
//HTTP 服务启动前执行
think.beforeStartServer(async () => {
	let postTitle = {};
	(await think.model('post').field(['slug', 'title']).select()).forEach(item => {
		postTitle[item.slug] = item.title;
	});
	think.config('postTitle', postTitle);
});
```

3）Middleware 中获取到的 `ctx.body` 不是完整页面。

本方案只有最后一个分块内容才会赋值给 `ctx.body`，前面分块的输出则完全绕过了 Middleware，出现这种情况是正常的。如果你不能接受，还是老老实实用 Readable Stream 吧。

最后，看完本文，相信你对如何优化 ThinkJS 3 网站的 TTFB 时间有了足够了解，赶紧动手试试吧。遇到任何问题，欢迎留言讨论。

原文链接：[https://imququ.com/post/reduce-ttfb-on-thinkjs3-website.html](https://imququ.com/post/reduce-ttfb-on-thinkjs3-website.html)，[前往原文评论 »](https://imququ.com/post/reduce-ttfb-on-thinkjs3-website.html#comments)
