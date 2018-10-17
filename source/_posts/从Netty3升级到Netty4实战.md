---
title: 从Netty3升级到Netty4实战
date: 2018-10-12 18:11:32
tags:
    - netty
---
![homePage](/upload/homePage/20181017164401.jpg)
<!--more-->

## 情景
原来有一版程序，使用的Netty版本是3.7.0Final，最近正好要修改，就顺带把Netty版本升级一下，目标版本4.0.36.Final。

## 依赖

因为历史原因（作者从jboss跑路）、Channel状态模型变化等因素影响，Netty4并不能向下兼容Netty3，因此升级需要修改不兼容的代码。

首先变更的就是依赖关系，调查发现，Netty团队大概从3.3.0开始，将dependency groupId由org.jboss.netty改为了io.netty。

由于之前所使用版本是3.7.0Final，在3.3.0之后Maven这里不需要修改，但是代码内部引用的接口、类，在3.7.0Final版本仍为org.jboss.netty，也就说虽然依赖在3.3.0就已经从org.jboss.netty迁到了io.netty，但是内部代码的包结构到3.7.0Final还是原来的org.jboss.netty，如下图。

![Netty3升级Netty4_1.png](/upload/Netty3升级Netty4/Netty3升级Netty4_1.png)

因此升级4.0.36.Final，所有原来引用Netty的部分代码，引用包结构都需要修改为io.netty。

## Channel状态模型
这里借用网上的两张图，第一幅图是Netty3的Channel状态模型，第二幅图是Netty4优化之后的模型，可以看到channelOpen，channelBound和channelConnected都已经被channelActive替代。channelDisconnected，channelUnbound和channelClosed也被channelInactive替代。

Netty3 Channel状态模型：

![Netty3升级Netty4_2.png](/upload/Netty3升级Netty4/Netty3升级Netty4_2.png)

Netty4 Channel状态模型：

![Netty3升级Netty4_3.png](/upload/Netty3升级Netty4/Netty3升级Netty4_3.png)

因此原先Netty3代码使用的部分类，已经被移除，但是会有其他的替代方案。

## 连接建立
Netty3代码：
```
ServerBootstrap bootstrap = new ServerBootstrap(
        new NioServerSocketChannelFactory(Executors.newCachedThreadPool(),executorService));
  bootstrap.setOption("RCVBUF_ALLOCATOR",
          new AdaptiveRecvByteBufAllocator(64, 2000, 65536));
  bootstrap.setOption(ChannelOption.RCVBUF_ALLOCATOR,
          new AdaptiveRecvByteBufAllocator(64, 1024, 65536));
// 设置一个处理客户端消息和各种消息事件的类(Handler)
bootstrap.setPipelineFactory(new ChannelPipelineFactory() {
    @Override
    public ChannelPipeline getPipeline() throws Exception {
        return Channels.pipeline(new ConnectServerHandler(),
                new IdleStateHandler(timer,180,0,0),
                new HeartBeatServerHandler(deviceManager),
                new ReciveMessageChannelHandler(deviceManager, kafkaTemplate));
}});
bootstrap.bind(new InetSocketAddress(port));
```

Netty4代码：
```
EventLoopGroup parentGroup = new NioEventLoopGroup(parentThreadGroupSize);
EventLoopGroup childGroup = new NioEventLoopGroup(childThreadGroupSize);

try {
    ServerBootstrap b = new ServerBootstrap();
    b.group(parentGroup, childGroup)
            .channel(NioServerSocketChannel.class)
            .option(ChannelOption.RCVBUF_ALLOCATOR, new FixedRecvByteBufAllocator(2048))
            .childOption(ChannelOption.RCVBUF_ALLOCATOR,
                    new AdaptiveRecvByteBufAllocator(64, 2048, 65536))
            .childHandler(new ChannelInitializer<SocketChannel>() {
                @Override
                public void initChannel(SocketChannel ch) throws Exception {
                    ch.pipeline().addLast(new ConnectServerHandler(),
                            new HeartBeatServerHandler(180, 0, 0),
                            new LengthFieldBasedFrameDecoder(1024 * 1024 * 1024, 1, 2,0,0),
                            new ReceiveMessageChannelHandler(deviceManager, kafkaTemplate));
                }
            });

    // Wait until the server socket is closed.
    ChannelFuture f = b.bind(port).sync();
} catch (InterruptedException e) {
    logger.error("启动netty异常！", e);
} finally {
    // Shut down all event loops to terminate all threads.
    parentGroup.shutdownGracefully();
    childGroup.shutdownGracefully();
}
```

## 心跳
Netty3时，实现服务器端实现设备心跳使用的是IdleStateAwareChannelHandler，由于之前描述的模型变更影响，这个类现在已经被移除了，但是IdleStateHandler还在，在Netty4中使用IdleStateHandler替换IdleStateAwareChannelHandler实现心跳机制。

Netty3代码：
```
import org.jboss.netty.handler.timeout.IdleStateAwareChannelHandler;

public class HeartBeatServerHandler extends IdleStateAwareChannelHandler {

    @Override
    public void channelIdle(ChannelHandlerContext ctx, IdleStateEvent e)
            throws Exception {
        log.info("channelIdle {}", e.getChannel());
        super.channelIdle(ctx, e);
        Channel channel = e.getChannel();
        //...
    }
}
```

Netty4代码：
```
import io.netty.handler.timeout.IdleStateHandler;

public class HeartBeatServerHandler extends IdleStateHandler {

  public class HeartBeatServerHandler extends IdleStateHandler {
    public HeartBeatServerHandler(int readerIdleTimeSeconds, 
                                  int writerIdleTimeSeconds, int allIdleTimeSeconds) {
      super(readerIdleTimeSeconds, writerIdleTimeSeconds, allIdleTimeSeconds);
    }
  }
}
```

在心跳之后的Handler中捕获心跳事件：
```
@Override
public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
    if (evt instanceof IdleStateEvent) {
        IdleStateEvent event = (IdleStateEvent) evt;
        if (event.state() == IdleState.READER_IDLE) {
            //读超时
            logger.info("READER_IDLE 读超时");
            Channel channel = ctx.channel();
            logger.info("channelIdle {}", ctx.channel());
            if (channel.remoteAddress() != null) {
                String address = channel.remoteAddress().toString();
                Device device = deviceManager.getDeviceByAddress(address);
                if (null != device) {
                    logger.info("Idle {}-{}", device, channel);
                } else {
                    channel.close();
                }
            } else {
                logger.info("Idle udp null-{}", channel);
            }
            ctx.disconnect();
        } else if (event.state() == IdleState.WRITER_IDLE) {
            //写超时
            logger.info("WRITER_IDLE 写超时");
        } else if (event.state() == IdleState.ALL_IDLE) {
            //总超时
            logger.info("ALL_IDLE 总超时");
        }
    }
    ctx.fireUserEventTriggered(evt);
}
```

这里Netty4与Netty3获取channel以及remoteAddress的方式也有了变化，在Netty4中这两者都可以从ChannelHandlerContext上下文直接获取了，不再需要Netty3的MessageEvent。

## 消息处理
对于消息处理这方面，原来Netty3中我使用的是SimpleChannelUpstreamHandler，Netty4中使用ChannelDuplexHandler替换。

Netty3代码：
```
public class ReciveMessageChannelHandler extends SimpleChannelUpstreamHandler {

    @Override
    protected void finalize() throws Throwable {
        super.finalize();
    }
    
    @Override
    public void messageReceived(ChannelHandlerContext ctx, MessageEvent e) throws Exception {
        if (e.getMessage() instanceof ChannelBuffer) {
            ChannelBuffer cb = ((ChannelBuffer) e.getMessage());
            //消息处理...
        }
        super.messageReceived(ctx, e);
     }

    @Override
    public void channelDisconnected(ChannelHandlerContext ctx, ChannelStateEvent e)
              throws Exception {
         super.channelDisconnected(ctx, e);
          logger.info("**************************^_^^_^^_^**************************");
    }
    
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, ExceptionEvent e) {
      e.getChannel().close();
    }
}
```

Netty4代码：
```
public class ReceiveMessageChannelHandler extends ChannelDuplexHandler {

    private static Logger logger = LoggerFactory.getLogger(ReceiveMessageChannelHandler.class);

    /**
     * 用来生成key
     */
    private static final String KAFKA_KEY = "test";

    private static final String IF_PARTITION = "0";

    private static final Integer PARTITION_NUM = 1;

    /**
     * 默认topic名称
     */
    private static final String DEFAULT_TOPIC_NAME = "nettytest";

    private KafkaProducerServer kafkaProducer;

    private DeviceManager deviceManager;

    public DeviceManager getDM() {
        return deviceManager;
    }

    public ReceiveMessageChannelHandler(DeviceManager deviceManager,
                                        KafkaTemplate<String, String> kafkaTemplate) {
        this.deviceManager = deviceManager;
        this.kafkaProducer = new KafkaProducerServer(kafkaTemplate);
    }

    public ReceiveMessageChannelHandler(DeviceManager deviceManager) {
        this.deviceManager = deviceManager;
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf buffer = ((ByteBuf) msg);
        //消息处理...
        super.channelRead(ctx, msg);
    }

    @Override
    public void disconnect(ChannelHandlerContext ctx, ChannelPromise future) throws Exception {
        super.disconnect(ctx, future);
        logger.info("*****************disconnect*********^_^^_^^_^**************************");
    }

    @Override
    public void close(ChannelHandlerContext ctx, ChannelPromise future) throws Exception {
        super.close(ctx, future);
        logger.info("*****************close*********^_^^_^^_^**************************");
    }

}
```

## 消息发送
消息读取方面，原来Netty3的ChannelBuffer已经由ByteBuf，并且Netty4还专门将其拆分出一个单独的包，其意义在于即使你不用Netty也可以方便的使用ByteBuf处理字节数据。

消息发送方面，Netty3中通过ctx.getChannel().write(ChannelBuffers.wrappedBuffer(replyMsg), e.getRemoteAddress())进行消息发送，需要指定remoteAddress，在Netty4中改变为了ctx.channel().writeAndFlush(Unpooled.buffer().writeBytes(replyMsg))，将数据写入缓冲区并刷新缓冲区，而且也不需要指定需要指定remoteAddress。