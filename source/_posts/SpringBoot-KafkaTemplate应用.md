---
title: SpringBoot KafkaTemplate应用
date: 2019-01-04 15:32:57
tags:
    - Spring Boot
    - kafka
---
![homePage](/upload/homePage/20190104095507.jpg)
<!--more-->

## 简介
Apache Kafka是一种高吞吐量的分布式发布订阅消息系统，有如下特性：
- 通过O(1)的磁盘数据结构提供消息的持久化，这种结构对于即使数以TB的消息存储也能够保持长时间的稳定性能。
- 高吞吐量：即使是非常普通的硬件Kafka也可以支持每秒数百万的消息。
- 支持通过Kafka服务器和消费机集群来分区消息。
- 支持Hadoop并行数据加载。

PS：因Kafka安装部署项较多，且网上教程比较全面，本文着重从代码以及配置项方面说明，相关安装说明请参考[Kafka官网](http://kafka.apache.org/)。

## Maven依赖
```
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>2.1.3.RELEASE</version>
</dependency>
```

## Kafka消费端配置
```
/**
 * kafka消费配置
 * @author hefan
 * @date 2019/3/8 15:11
 */
@Configuration
public class KafkaConsumerConfig {

    @Value("${kafka.consumer.servers}")
    private String servers;

    @Value("${kafka.consumer.enable.auto.commit}")
    private boolean enableAutoCommit;

    @Value("${kafka.consumer.session.timeout}")
    private String sessionTimeout;

    @Value("${kafka.consumer.auto.commit.interval}")
    private String autoCommitInterval;

    @Value("${kafka.consumer.group.id}")
    private String groupId;

    @Value("${kafka.consumer.auto.offset.reset}")
    private String autoOffsetReset;

    @Value("${kafka.consumer.concurrency}")
    private int concurrency;

    @Value("${kafka.consumer.maxPollRecordsConfig}")
    private int maxPollRecordsConfig;

    @Value("${kafka.consumer.client.id}")
    private String clientId;

    @Bean
    public KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<String, String>> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        factory.setConcurrency(concurrency);
        factory.getContainerProperties().setPollTimeout(1500);
        //@KafkaListener 批量消费  每个批次数量在Kafka配置参数中设置ConsumerConfig.MAX_POLL_RECORDS_CONFIG
        factory.setBatchListener(false);
        //设置提交偏移量的方式
        //factory.getContainerProperties().setAckMode(AbstractMessageListenerContainer.AckMode.MANUAL_IMMEDIATE);
        return factory;
    }

    public ConsumerFactory<String, String> consumerFactory() {
        return new DefaultKafkaConsumerFactory<>(consumerConfigs());
    }

    public Map<String, Object> consumerConfigs() {
        Map<String, Object> propsMap = new HashMap<>(8);
        propsMap.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, servers);
        propsMap.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, enableAutoCommit);
        propsMap.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, autoCommitInterval);
        propsMap.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, sessionTimeout);
        propsMap.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        propsMap.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        propsMap.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);
        propsMap.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, autoOffsetReset);
        propsMap.put(ConsumerConfig.CLIENT_ID_CONFIG, clientId);
        //每个批次获取数
        //propsMap.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, maxPollRecordsConfig);
        //设置限制byte 623 * 3000 = 1869000
        //propsMap.put(ConsumerConfig.MAX_PARTITION_FETCH_BYTES_CONFIG, 1_869_000);
        return propsMap;
    }
}
```

这里注意可以配置批量消费，设置BatchListener为true，即可开启批量消费，每个批次数量为参数中设置的ConsumerConfig.MAX_POLL_RECORDS_CONFIG。

## Kafka生产端配置
```
/**
 * Kafka生产配置
 * @author hefan
 * @date 2019/3/8 11:54
 */
@Configuration
public class KafkaProducerConfig {

    @Value("${kafka.producer.servers}")
    private String servers;

    @Value("${kafka.producer.retries}")
    private int retries;

    @Value("${kafka.producer.batch.size}")
    private int batchSize;

    @Value("${kafka.producer.linger}")
    private int linger;

    @Value("${kafka.producer.buffer.memory}")
    private int bufferMemory;

    @Value("${kafka.producer.client.id}")
    private String clientId;

    @Value("${kafka.producer.compression.type}")
    private String compressionType;

    @Bean(name = "kafkaProducerTemplate")
    public KafkaTemplate<String, String> kafkaTemplate(@Qualifier("kafkaProducerListener") KafkaProducerListener listener) {
        KafkaTemplate kafkaTemplate = new KafkaTemplate(producerFactory());
        kafkaTemplate.setProducerListener(listener);
        return kafkaTemplate;
    }

    @Bean
    public KafkaProducerListener kafkaProducerListener(){
        return new KafkaProducerListener();
    }

    public ProducerFactory<String, String> producerFactory() {
        return new DefaultKafkaProducerFactory<>(producerConfigs());
    }

    public Map<String, Object> producerConfigs() {
        Map<String, Object> props = new HashMap<>();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, servers);
        props.put(ProducerConfig.RETRIES_CONFIG, retries);
        props.put(ProducerConfig.BATCH_SIZE_CONFIG, batchSize);
        props.put(ProducerConfig.LINGER_MS_CONFIG, linger);
        props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, bufferMemory);
        props.put(ProducerConfig.CLIENT_ID_CONFIG, clientId);
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        //设置压缩方式
        props.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, compressionType);
        return props;
    }
}
```

此处配置了生产端消息压缩方式ProducerConfig.COMPRESSION_TYPE_CONFIG，Kafka支持三种压缩算法lz4、snappy、gzip，这三者压缩比、生成性能和消费性能等，可以参考该篇文章[Kafka压缩](https://www.jianshu.com/p/d69e27749b00)。
另外还配置了生成端的监听器KafkaProducerListener，在此可以输出一些生产消息的日志。
更多Kafka配置详见[Kafka Documentation](https://kafka.apache.org/documentation/)。

## 生产端监听器(KafkaProducerListener)
```
/**
 * kafkaProducer监听器，在producer配置文件中开启
 * @author hefan
 * @date 2018/11/23 18:37
 */
@SuppressWarnings("rawtypes")
@Slf4j
public class KafkaProducerListener implements ProducerListener {
	/**
	 * 发送消息成功后调用
	 */
	@Override
	public void onSuccess(String topic, Integer partition, Object key, Object value, RecordMetadata recordMetadata) {
		log.info("==========kafka发送数据成功（日志开始）==========");
		log.info("----------topic:"+topic);
		log.info("----------partition:"+partition);
		log.info("----------key:"+key);
		log.info("----------value:"+value);
		log.info("----------RecordMetadata:"+recordMetadata);
		log.info("~~~~~~~~~~kafka发送数据成功（日志结束）~~~~~~~~~~");
	}

	/**
	 * 发送消息错误后调用
	 */
	@Override
	public void onError(String topic, Integer partition, Object key, Object value, Exception exception) {
		log.info("==========kafka发送数据错误（日志开始）==========");
		log.info("----------topic:"+topic);
		log.info("----------partition:"+partition);
		log.info("----------key:"+key);
		log.info("----------value:"+value);
		log.info("----------Exception:"+exception);
		log.info("~~~~~~~~~~kafka发送数据错误（日志结束）~~~~~~~~~~");
		//exception.printStackTrace();
        log.error("发送消息错误",exception);
	}

	/**
	 * 方法返回值代表是否启动kafkaProducer监听器
	 */
	@Override
	public boolean isInterestedInSuccess() {
		log.info("///kafkaProducer监听器启动///");
		return true;
	}

}
```

## Kafka消费端
```
/**
 * 消费端
 * @author hefan
 * @date 2018/11/23 18:37
 */
@Component
@Slf4j
public class Receiver {

    @Value("${consumer.thread.num}")
    private Integer consumerThreadNum;

    private static ExecutorService service;

    @PostConstruct
    public void init(){
        service = new ThreadPoolExecutor(consumerThreadNum, consumerThreadNum,
                0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<>());
    }

    @KafkaListener(id = "${consumer.id}", topics = {"${consumer.topicName}"},
            containerFactory = "kafkaListenerContainerFactory")
    public void listen(ConsumerRecord<?, String> record) {
        try {
            //转换数据
            MqttDownLinkCmd cmd = JSON.parseObject(record.value(), MqttDownLinkCmd.class);

            CompletableFuture.supplyAsync(() -> {
                try {
                    MqttUtils.publish(cmd.getTopic(),
                            StrUtil.hexStringToBytes(cmd.getCmd()));
                    log.info("publish topic:{},cmd:{}", cmd.getTopic(), cmd.getCmd());
                } catch (MqttException e) {

                }
                return 0;
            }, service);
        } catch (Exception e) {
            log.error("kafka listener error", e);
        }
    }

}
```

## Kafa生产端
```
/**
 * 生产端
 * @author hefan
 * @date 2019/3/8 15:10
 */
@Component
@Slf4j
public class Sender {

    @Value("${producer.topicName}")
    private String topicName;

    @Autowired
    @Qualifier("kafkaProducerTemplate")
    private KafkaTemplate<String, String> kafkaTemplate;

    public void send(String data) {
        kafkaTemplate.send(topicName, data);
    }
}
```

## 注释事项
- 若设置KafaTemplate的autoFlush为true，则发送部分会变为阻塞。
- Kafka可设置提交方式，配置enable.auto.commit，可以切换自动/手动两种模式，手动提交即需要在消费时手动commit，自动提交则是消费端会在后台周期性的去commit。
