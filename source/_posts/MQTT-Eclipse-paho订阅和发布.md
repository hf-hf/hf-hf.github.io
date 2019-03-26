---
title: MQTT Eclipse paho订阅和发布
date: 2019-03-26 09:14:46
tags:
    - mqtt
    - eclipse paho
categories:
    - mqtt
---
![homePage](/upload/homePage/20190326091707.jpg)
<!--more-->

## 简介
MQTT(Message Queuing Telemetry Transport，是一种基于发布/订阅（publish/subscribe）模式的"轻量级"通讯协议，该协议构建于TCP/IP协议上，由IBM在1999年发布。MQTT最大优点在于，可以以极少的代码和有限的带宽，为连接远程设备提供实时可靠的消息服务。作为一种低开销、低带宽占用的即时通讯协议，使其在物联网、小型设备、移动应用等方面有较广泛的应用。

## Broker配置
MQTT工作在TCP/IP协议族上，是为硬件性能低下的远程设备以及网络状况糟糕的情况下而设计的发布/订阅型消息协议，为此，它需要一个消息中间件。本文使用的消息中间件为Apache Apollo，详细安装教程见官网[apollo getting-started](http://activemq.apache.org/apollo/documentation/getting-started.html)。

![Apache Apollo Console](/upload/mqttBroker/mqtt_broker_1.png)

## Maven依赖
```
<dependency>
    <groupId>org.eclipse.paho</groupId>
    <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
    <version>1.2.0</version>
</dependency>
```

Eclipse paho选用版本1.2.0，最后更新时间为2017年8月，截止本文撰写日，Eclipse paho已更新至1.2.1，相关更新详情见[Github paho.mqtt.java](https://github.com/eclipse/paho.mqtt.java)。

## MQTT通配符说明
/用于分割主题层级，称为主题层级分隔符;#为多层通配符，可以匹配于多层主题；+为单层通配符只能匹配一层主题。

例如：订阅topic为`RFID/SS/#`，设备123456初始化指令发布topic为`RFID/SS/123456/INIT`。

PS: 单层通配符和多层通配符只能用于订阅消息而不能用于发布消息，层级分隔符两种情况下均可使用。

## 订阅发布代码
MQTT工具类：

```
/**
 * mqtt client
 * @author hefan
 * @date 2019/3/8 11:15
 */
@Slf4j
@Component
public class MqttUtils {

    private static boolean startReconnect = false;

    private static MqttClient client;

    private static MqttConfig config;

    private static ScheduledExecutorService scheduler;

    @Autowired
    public void setConfig(MqttConfig config) {
        MqttUtils.config = config;
    }

    private static Sender sender;

    @Autowired
    public void setSender(Sender sender) {
        MqttUtils.sender = sender;
    }

    /**
     * 生成配置对象，用户名，密码等
     * @return
     */
    public static MqttConnectOptions getOptions() {
        MqttConnectOptions options = new MqttConnectOptions();
        options.setCleanSession(config.getCleanSession());
        options.setUserName(config.getUsername());
        options.setPassword(config.getPassword().toCharArray());
        //options.setConnectionTimeout(config.getConnectionTimeout());
        options.setKeepAliveInterval(config.getKeepAliveInterval());
        return options;
    }

    /**
     * 连接
     * @throws MqttException
     */
    public static void connect() throws MqttException {
        // 防止重复创建MQTTClient实例
        if (null == client) {
            client = new MqttClient(config.getHost(), config.getId(),
                    new MemoryPersistence());
            client.setCallback(new ReceiveCallback(config.getId(), sender));
        }
        MqttConnectOptions options = getOptions();
        // 遗嘱消息设置
        //options = setWill(options);
        if (!client.isConnected()) {
            client.connect(options);
            log.debug("first connect success!");
            //client.subscribe(config.getTopic(), config.getQos());
        } else {
            // 这里的逻辑是如果连接成功就重新连接
            client.reconnect();
            log.debug("reconnect success!");
        }
    }

    public static MqttConnectOptions setWill(MqttConnectOptions options){
        MqttTopic topic = client.getTopic(config.getWillTopic());
        options.setWill(topic, (config.getId()).getBytes(), 0, false);
        return options;
    }

    /**
     * 开启重新连接
     */
    public static void startReconnect() {
        if(startReconnect){
            log.error("reconnect scheduler is running!");
            return;
        }
        scheduler = new ScheduledThreadPoolExecutor(1);
        scheduler.scheduleAtFixedRate(() -> {
            if (!client.isConnected()) {
                try {
                    connect();
                } catch (MqttException e) {
                    log.error("reconnect error！", e);
                }
            }
        }, 0, 5, TimeUnit.SECONDS);
        startReconnect = true;
    }

    /**
     * 关闭重新连接
     */
    public static void closeReconnect() {
        if(!startReconnect){
            log.debug("reconnect scheduler not started!");
            return;
        }
        scheduler.shutdown();
        startReconnect = false;
    }

    /**
     * 发布消息
     * @param topic
     * @param message
     * @throws MqttException
     */
    public static void publish(String topic, MqttMessage message) throws MqttException {
        if(null == client || !client.isConnected()){
            log.error("mqtt client disconnect!");
            return;
        }
        MqttTopic mqttTopic = client.getTopic(topic);
        MqttDeliveryToken token = mqttTopic.publish(message);
        token.waitForCompletion();
        log.debug("publish result:" + token.isComplete());
    }

    /**
     * 发布消息
     * @param topic
     * @param payload
     * @throws MqttException
     */
    public static void publish(String topic, byte[] payload) throws MqttException {
        MqttMessage message = new MqttMessage();
        //在Qos2情况下，Broker肯定会收到消息，且只收到一次
        message.setQos(config.getQos());
        message.setRetained(config.getRetained());
        message.setPayload(payload);
        publish(topic, message);
    }

}
```

开启连接并启动断线重连。
```
/**
 * mqtt client引导
 * @author hefan
 * @date 2019/3/8 14:03
 */
@Slf4j
@Component
public class MqttBootStrap implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {
        MqttUtils.connect();
        MqttUtils.startReconnect();
    }
}
```

发布消息。
```
MqttUtils.publish("topic", StrUtil.hexStringToBytes("cmd"));
```

## 注意事项
- 发布消息时，如无特殊需要，请不要将消息格式设置为保留，否则当连接断开重新连接时，会收到的重复的消息。保留消息主要应用：发布方定时发布retain消息向订阅方通知自身状态，订阅方获取该消息推测发布方状态，当订阅方断线重连后仍能接收到最新的发布方状态。
- 若要配置遗嘱消息，需放开setWill(options)行的注释，并设置单独的topic接收client离线消息。
- 客户端重新连接时，请设置cleanSession为false，否则会导致离线期间发送的消息，重连后无法接收。
- 当程序订阅一次后，不清除Session无需再次订阅，若断线重连后再次订阅，会接收不到部分消息。MqttUtils中client.subscribe(config.getTopic(), config.getQos())仅第一次上线时需放开。
