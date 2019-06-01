---
layout: post
notes: true
subtitle: "Netty进阶之路：跟着案例学Netty"
comments: false
author: "李林锋"
date: 2019-04-20 00:00:00

---

![](/img/notes/network/nettyAdvanced/netty_advanced.jpg)

*   目录
{:toc }

# 第1章 Netty服务端意外退出案例

在使用Netty进行服务端程序开发时，主要涉及端口监听、EventLoop线程池创建、NioServerSocketChannel和ChannelPipeline初始化等。初学者如果对服务端相关类库的工作原理和用法不熟悉，则会导致程序启动或者退出发生问题。

## 1.1 Netty服务端意外退出问题

#### 案例1 通过阻塞方式绑定监听端口，启动服务端之后，没发生任何异常，程序退出

代码如下：

	try {
		ServerBootStrap b = new ServerBootStrap();
		b.group(bossGroup, workerGroup)
			.channel(NioServerSocketChannel.class)
			.option(ChannelOption.SO_BACKLOG, 100)
			.handler(new LoggingHandler(LogLevel.INFO))
			.childHandler(new ChannelInitializer<SocketChannel>() {
				@Override
				public void initChannel(SocketChannel ch) throws Exception {
					ChannelPipeline p = ch.pipeline();
					p.addLast(new LoggingHandler(LogLevel.INFO));
				}
			});
		b.bind(18080).sync(); // 用同步方式绑定服务端监听端口
	} finally {
		bossGroup.shutdownGracefully();
		workerGroup.shutdownGracefully();
	}
	
#### 案例2 对案例1进行排查时，发现没有监听Close Future，于是对代码进行修改，还是会发生服务器套接字直接关闭、进程退出的问题

代码如下：

	try {
		ServerBootStrap b = new ServerBootStrap();
		b.group(bossGroup, workerGroup)
			.channel(NioServerSocketChannel.class)
			.option(ChannelOption.SO_BACKLOG, 100)
			.handler(new LoggingHandler(LogLevel.INFO))
			.childHandler(new ChannelInitializer<SocketChannel>() {
				@Override
				public void initChannel(SocketChannel ch) throws Exception {
					ChannelPipeline p = ch.pipeline();
					p.addLast(new LoggingHandler(LogLevel.INFO));
				}
			});
		ChannelFuture f = b.bind(18080).sync(); // 用同步方式绑定服务端监听端口
		f.channel().closeFuture().addListener(new ChannelFutureListener() {
			@Override
			public void operationComplete(ChannelFuture future) throws Exception {
				// 业务逻辑处理代码，此处省略
				logger.info(future.channel().toString() + " 链路关闭");
			}
		});
	
	} finally {
		bossGroup.shutdownGracefully();
		workerGroup.shutdownGracefully();
	}

	
