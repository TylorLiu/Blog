---
title: Spring定时任务BLOCK问题排查
date: 2018/3/16 16:33
tags: Spring
category: troubleshooting
---


#### 问题现象

设备状态不正确，查日志发现间定时轮询设备状态的任务 已经快一天没打日志了。
另外：
1. 日志没有相关错误信息
2. 其它定时任务正常执行

### 解决过程

在StackOVerFlow上搜到一个类似的[问题](http://stackoverflow.com/questions/17909404/spring-scheduler-stops-unexpectedly)，是由定时任务中的HttpClient阻塞引起的。
相似度很高，都是在Scheduled Task中发起远程调用。
参考回答，先用jstack -F pid 拿到线程快照，查了一下，发现该任务的堆栈Block在ArrayBlockingQueue.take()方法上，代码如下：
````java
public class HkpClientHandler extends SimpleChannelInboundHandler<MessagePackage> {
	private BlockingQueue<Object> response = new ArrayBlockingQueue<Object>(1);

	@Override
	protected void channelRead0(ChannelHandlerContext ctx, MessagePackage msg) throws Exception {
		response.add(msg.getXmlMessage());
		ctx.close();
	}
	public String getResponse() throws InterruptedException {
  		Object result = response.take();
  		if (result instanceof Throwable) {
  			Throwable cause = (Throwable) result;
  			throw new ServerException(ResponseCode.FAIL, cause);
  		}
  		return (String) result;
  	}
  	//省略
}
````
此处在netty的ClientHandler中用BlockingQueue存放Decoded Response，这样调用HkpClientInitializer.getResponse()就可以阻塞直到服务端返回。
````java
public class HkpClientInitializer extends ChannelInitializer<SocketChannel> {
	private HkpClientHandler clientHandler = new HkpClientHandler();

	@Override
	protected void initChannel(SocketChannel ch) throws Exception {
		ChannelPipeline pipeline = ch.pipeline();

		pipeline.addLast(new LoggingHandler());
		pipeline.addLast("MessageDecoder", new MessagePackageDecoder(1024 * 1024, 4, 4, -8, 0));
		pipeline.addLast("MessageEncoder", new MessagePackageEncoder());
		pipeline.addLast("ReadTimeout", new ReadTimeoutHandler(5));
		// 客户端的逻辑
		pipeline.addLast("handler", clientHandler);
	}
	public String getResponse() throws InterruptedException {
  		return clientHandler.getResponse();
  	}
}
````
但是万万没想到，都已经设置了ReadTimeout，还是被阻塞了，对照了服务端的日志，发现服务端返回毫无问题。由于时间紧急，考虑到问题出现概率较小、单次状态查询失败几无影响，阻塞的地方改成快速失败足以解决问题：
````java
Object result = response.poll(8, TimeUnit.SECONDS);
if (result == null){
    throw new ServerException("Timeout recieving response from remote!");
}
````
### 根因分析
虽然暂时解决了问题，但是这个Netty客户端无疑还有瑕疵，偶尔收不到服务端的返回对于某些业务是无法接受的。
因此，还需要找到问题的根本原因。
从channelRead0入手，往上追溯，看看信息在哪里被遗漏掉了。