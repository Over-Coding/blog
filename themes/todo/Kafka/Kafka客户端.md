---
title: Kafka客户端
date: 2020-01-29 15:12:13
categories:
	- Kafka
tags: 
	- Kafka
	- 客户端
toc: true
typora-root-url: Kafka客户端
---



### 1、基础概念

本篇文章将讲解Kafka客户端的使用，在开始之前，我们需要先了解一下相关的基础概念。

* Broker：表示一个Kafka服务，在上一篇
* Topic：
* Partition：
* Producer：
* Consumer：
* Offset：



### 2、概述

Kafka 是一个消息队列，我们需要一个生产者将消息存储到消息队列中，需要一个消费者将消息从消息队列中取出，所以 Kafka 就给我们提供了客户端，当然我们这里只说 Java 语言的客户端，其他语言的请自行研究。这里介绍两种客户端，一种是Kafka 自带的脚本工具客户端，一种的是 Java 客户端。



### 3、脚本客户端

在 Kafka 安装的根目录下有一个 /bin 文件夹，其中有很多脚本工具，其中就有一个 kafka-console-producer.sh 脚本，这个就是生产者的脚本客户端。还有一个 kafka-console-consumer.sh 脚本，这个就是消费者的脚本客户端。

* 生产者脚本客户端

  使用脚本客户端连接 Kafka 集群，并指定向哪个主题中写消息，连接成功后，发送 Hello kafka。

  ```shell
  bin/kafka-console-producer.sh --broker-list 192.168.150.101:9092 --topic first
  >Hello kafka
  >
  ```

  * **--broker-list：**指定连接的 Kafka 集群地址。
  * **--topic：**指定向哪个主题中写消息。



* 消费者脚本客户端

  使用脚本客户端连接 Kafka 集群，并指定从哪个主题中取消息。

  ```shell
  bin/kafka-console-consumer.sh --bootstrap-server 192.168.150.101:9092 --topic first --from-beginning
  Hello kafka
  ```

  * **--bootstrap-server：**指定连接的 Kafka 集群地址。
  * **--topic：**指定从哪个主题中取消息。
  * **--from-beginning：**指定从开始将所有的消息都取出来。也可以不指定，即从当前时间获取新的消息。



### 4、Java 客户端

#### 3.1、生产者客户端

##### 3.1.1、示例代码

**step1：**添加 maven 引用

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>1.1.1</version>
</dependency>
```

注意：我这里用的版本是和Kafka 集群的版本是一致的。如果版本不同，最好修改成自己的版本。

**step2：**编写代码

```java
package com.ld.producer;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;

/**
 * @author ld
 * 生产者客户端示例
 */
public class KafkaProducerDemo {

    //kafka服务器地址
    private static String servers = "192.168.150.101:9092,192.168.150.102:9092,192.168.150.103:9092";
    //客户端id
    private static String clientId = "product.client.demo";
    //主题名称
    private static String topic = "first";

    /**
     * 生产者客户端配置
     * 这里只配置了几个必要的参数，其他参数后面会一一介绍。
     * @return
     */
    private static Properties config(){
        Properties properties = new Properties();

        /*************************优化前*****************************/
//        //kafka服务器地址
//        properties.put("bootstrap.servers",servers);
//        //key的序列化器类路径
//        properties.put("key.serializer","org.apache.kafka.common.serialization.StringSerializer");
//        //value的序列化器类路径
//        properties.put("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
//        //客户端id
//        properties.put("client.id",clientId);

        /*************************优化后*****************************/
        /**
         * 为了防止开发人员将参数名写错，所以客户端提供了一个 ProducerConfig 类，这个类中定义了所有要配置的参数名。
         * 所以我们可以使用这个类中的定义参数名。
         */
        //kafka服务器地址
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,servers);
        //key的序列化器类路径,序列化器可以直接获取类路径
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        //value的序列化器类路径
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,StringSerializer.class.getName());
        //客户端id
        properties.put(ProducerConfig.CLIENT_ID_CONFIG,clientId);

        //返回配置对象
        return properties;
    }


    public static void main(String[] args) {
        //获取客户端配置
        Properties properties = config();
        //创建KafkaProducer对象,传入客户端配置信息
        KafkaProducer<String, String> producer = new KafkaProducer<>(properties);
        //创建ProducerRecord对象,传入主题名称，消息
        ProducerRecord<String, String> record = new ProducerRecord<>(topic,"Hello World");

        //发送ProducerRecord对象
        try{
            //普通发送
            producer.send(record);

            //同步发送
//            RecordMetadata recordMetadata = producer.send(record).get();
//            //打印发送结果
//            System.out.println("topic:" + recordMetadata.topic() + ",partition:" + recordMetadata.partition() + ", offset" + recordMetadata.offset());

            //异步发送
//            producer.send(record, new Callback() {
//                @Override
//                public void onCompletion(RecordMetadata recordMetadata, Exception e) {
//                    //判断如果recordMetadata 不为空，说明发送成功，否则发送失败，打印异常。
//                    if(recordMetadata != null){
//                        System.out.println("topic:" + recordMetadata.topic() + ",partition:" + recordMetadata.partition() + ", offset" + recordMetadata.offset());
//                    }else{
//                        e.printStackTrace();
//                    }
//                }
//            });
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            //关闭KafkaProducer对象
            producer.close();
        }
    }
}
```

上述就是 Java 版的生产者客户端代码。下面简述一下其中几个重要的API。

* **创建生产者对象**

  **API列表：**

  * **KafkaProducer**(Map<String, Object> configs)
  * **KafkaProducer**(Map<String, Object> configs, Serializer<K> keySerializer, Serializer<V> valueSerializer)
  * **KafkaProducer**(Properties properties)
  * **KafkaProducer**(Properties properties, Serializer<K> keySerializer, Serializer<V> valueSerializer)

  

  **参数介绍：**

  | 参数名          | 说明                      |
  | --------------- | ------------------------- |
  | configs         | 使用map存储参数配置信息。 |
  | keySerializer   | 指定 key 序列化器对象。   |
  | valueSerializer | 指定 value 序列化器对象。 |
  | properties      | 指定参数配置信息。        |

  

* **创建消息对象**

  **API列表：**

  * **ProducerRecord**(String topic, Integer partition, Long timestamp, K key, V value, Iterable<Header> headers)
  * **ProducerRecord**(String topic, Integer partition, Long timestamp, K key, V value)
  * **ProducerRecord**(String topic, Integer partition, K key, V value, Iterable<Header> headers)
  * **ProducerRecord**(String topic, Integer partition, K key, V value)
  * **ProducerRecord**(String topic, K key, V value)
  * **ProducerRecord**(String topic, V value)

  

  **参数介绍：**

  | 参数名    | 说明                                                         |
  | --------- | ------------------------------------------------------------ |
  | topic     | 指定主题名称。                                               |
  | partition | 指定分区号。                                                 |
  | timestamp | 指定时间。                                                   |
  | key       | 指定 key 值，如果 key 不为 null，那么默认的分区器会对 key 进行哈希，根据得出的哈希值来计算分区号。 |
  | value     | 指定消息。                                                   |
  | headers   | 指定发送的头部信息。                                         |

  

* **发送消息**

  **API列表：**

  * **Future**<RecordMetadata> send(ProducerRecord<K, V> record)
  * **Future**<RecordMetadata> send(ProducerRecord<K, V> record, Callback callback)

  

  **参数介绍：**

  | 参数名   | 说明                          |
  | -------- | ----------------------------- |
  | record   | 要发送的 ProducerRecord对象。 |
  | callback | 回调函数。可以异步发送消息。  |

  

  **发送方式：**

  * 普通发送：即发送出去消息就没有然后了，也不知道成功还是失败。这种方式性能高，可靠性差。
  * 同步发送：即发送一条消息，然后就会阻塞线程，等待发送结果。从上面API的返回值可知，我们可以使用 Future 的 get() 方法阻塞当前线程，等待发送结果。这种方式性能差，可靠性高。
  * 异步发送：即发送一条消息，不会阻塞线程，而是通过回调函数来获取发送结果。这种方式性能高，可靠性高。这里有一点，因为消息是分区有序的，所有回调函数的调用也是分区有序的。



##### 3.1.2、拦截器

Kafka 一共有两种拦截器，分别是生产者拦截器和消费者拦截器。这里先讲一下生产者拦截器。生产者拦截器作用在发送消息前后，即在发送消息之前做一些准备或者附加工作，在发送消息之后做一些处理工作。Kafka 客户端中提供了一个 org.apache.kafka.clients.producer.ProducerInterceptor 接口，当想自定义生产者拦截器时，只需要实现 ProducerInterceptor 接口，并将实现类的类路径配置到创建 KafkaProducer 的配置参数中即可生效。下面请看示例代码。



**step1：**自定义一个 MyProducerInterceptor 类实现 ProducerInterceptor 接口。

```java
package com.ld.producer.interceptor;

import org.apache.kafka.clients.producer.ProducerInterceptor;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;

import java.util.Map;

/**
 * @author ld
 *
 * 自定义一个生产者拦截器
 */
public class MyProducerInterceptor implements ProducerInterceptor<String, String> {

    /**
     * 此方法用于发送消息之前调用
     * @param producerRecord 消息对象
     * @return
     */
    @Override
    public ProducerRecord<String, String> onSend(ProducerRecord<String, String> producerRecord) {
        System.out.println("准备发送消息......");
        /**
         * 这里可以修改要发送的消息，例如为消息添加前缀 test-
         */
        //获取消息
        String value = producerRecord.value();
        //添加前缀
        value = "test-" + value;
        return new ProducerRecord<String,String>(producerRecord.topic(),value);
    }

    /**
     * 此方法用于发送消息之后,发送结果被应答之前调用
     * @param recordMetadata 发送消息成功，返回的发送结果
     * @param e 发送失败，返回的异常信息
     */
    @Override
    public void onAcknowledgement(RecordMetadata recordMetadata, Exception e) {
        /**
         * 可以判断发送成功还是发送失败
         */
        if(recordMetadata != null){
            System.out.println("发送消息成功... topic: " + recordMetadata.topic() + " , partition: " + recordMetadata.partition());
        }else{
            System.out.println("发送消息失败... 失败原因：" + e.getMessage());
        }
    }

    /**
     * 此方法用于在关闭拦截器时调用
     */
    @Override
    public void close() {
        System.out.println("关闭拦截器......");
    }

    @Override
    public void configure(Map<String, ?> map) {

    }
}
```

上述代码中实现了 ProducerInterceptor 接口的三个方法。

* onSend：发送消息之前被调用。
* onAcknowledgement：发送消息之后，发送结果被应答之前被调用。
* close：用于关闭拦截器时调用。

发现还有一个 configure() 方法，此方法不是 ProducerInterceptor 接口中的方法，而是其父接口 Configurable 中的方法。Configurable 接口中的 configure() 主要用来获取配置信息及初始化数据。



**step2：**编写生产者客户端，配置拦截器参数。

```java
package com.ld.producer.interceptor;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;

/**
 * @author ld
 *
 * 生产者拦截器的生产者客户端
 */
public class ProducerInterceptorKafkaProducer {

    //kafka服务器地址
    private static String servers = "192.168.150.101:9092,192.168.150.102:9092,192.168.150.103:9092";
    //客户端id
    private static String clientId = "product.client.producer.interceptor";
    //主题名称
    private static String topic = "first";


    /**
     * 生产者客户端配置
     * 这里只配置了几个必要的参数，其他参数后面会一一介绍。
     * @return
     */
    private static Properties config(){
        Properties properties = new Properties();
        //kafka服务器地址
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,servers);
        //key的序列化器类路径,序列化器可以直接获取类路径
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        //value的序列化器类路径
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,StringSerializer.class.getName());
        //客户端id
        properties.put(ProducerConfig.CLIENT_ID_CONFIG,clientId);

        //配置生产者拦截器,还可以配置多个拦截器（以逗号分隔）,形成拦截器链,拦截器链中拦截器的执行顺序是按照配置的顺序执行。
        properties.put(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG,MyProducerInterceptor.class.getName());

        //返回配置对象
        return properties;
    }

    public static void main(String[] args) {
        //获取客户端配置
        Properties properties = config();
        //创建KafkaProducer对象,传入客户端配置信息
        KafkaProducer<String, String> producer = new KafkaProducer<>(properties);
        for(int i = 0;i < 10;i++){
            //创建ProducerRecord对象,传入主题名称，消息
            ProducerRecord<String, String> record = new ProducerRecord<>(topic,"消息" + i);

            //发送ProducerRecord对象
            try{
                //普通发送
                producer.send(record);
            }catch (Exception e){
                e.printStackTrace();
            }
        }
        //关闭KafkaProducer对象
        producer.close();
    }

}
```

上述代码中添加了一个新的配置即：properties.put(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG,MyProducerInterceptor.class.getName());

* key：interceptor.classes。
* value：配置拦截器的类路径。并且支持配置多个拦截器（以逗号分隔），形成拦截器链，拦截器链中拦截器的执行顺序是按照配置的顺序执行。如果前后多个拦截器有依赖关系，那么前一个拦截器执行失败，会影响到下一个拦截器无法执行。



**step3：**查看消费消息结果。

![](生产者拦截器.PNG)



##### 3.1.3、序列化器

生产者需要用序列化器将对象转换成字节数组才能通过网络发送给 Kafka。消费者需要用反序列化器把从 Kafka 中收到的字节数组转换成相应的对象。所以序列化器和反序列化器是成对出现的，如果使用不匹配的序列化器和反序列化器时，就会出现问题。

Kafka 客户端中提供了一个 org.apache.kafka.common.serialization.Serializer 接口，如果想自定义序列化器时，只需实现这个接口即可。

Kafka 客户端中还默认提供了好多序列化器，如：StringSerializer、IntergerSerializer、DoubleSerializer、LongSerializer等等。

下面自定义一个序列化器，请看示例代码：

**step1：**创建一个 Person 实体类 。

```java
package com.ld.producer.serializer;

/**
 * @author ld
 * 创建一个 person 类
 */
public class Person {

    //名称
    private String name;

    //年龄
    private int age;

    public Person() {
    }

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```



**step2：**自定义一个 MySerializer 类，实现 Serializer 接口。

```java
package com.ld.producer.serializer;

import org.apache.kafka.common.serialization.Serializer;

import java.nio.ByteBuffer;
import java.util.Map;

/**
 * @author ld
 *
 * 自定义一个序列化器
 */
public class MySerializer implements Serializer<Person> {

    /**
     * 配置参数信息，在创建 KafkaProducer 实例时被调用
     * @param map
     * @param b
     */
    @Override
    public void configure(Map<String, ?> map, boolean b) {}

    /**
     * 用于实现序列化逻辑
     * @param topic 主题
     * @param person 数据
     * @return
     */
    @Override
    public byte[] serialize(String topic, Person person) {
        if(person == null){
            return null;
        }

        byte[] name, age;
        try{
            if(person.getName() != null){
                name = person.getName().getBytes("UTF-8");
            }else {
                name = new byte[0];
            }

            if(person.getAge() > 0){
                age = String.valueOf(person.getAge()).getBytes("UTF-8");
            }else {
                age = new byte[0];
            }

            ByteBuffer buffer = ByteBuffer.allocate(4 + 4 + name.length + age.length);
            buffer.putInt(name.length);
            buffer.put(name);
            buffer.putInt(age.length);
            buffer.put(age);
            return buffer.array();
        }catch (Exception e){
            e.printStackTrace();
        }
        return new byte[0];
    }

    /**
     * 关闭序列化器时使用
     */
    @Override
    public void close() {
        System.out.println("关闭序列化器......");
    }
}
```

上述代码中实现了 Serializer 接口的三个方法。

* configure：配置参数信息，在创建 KafkaProducer 实例时被调用。
* serialize：用于实现序列化逻辑。
* close：用于关闭序列化器时调用。



**step3：**编写生产者客户端，配置序列化器参数。

```java
package com.ld.producer.serializer;

import com.ld.producer.interceptor.MyProducerInterceptor;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;

/**
 * @author ld
 */
public class SerializerKafkaProducer {

    //kafka服务器地址
    private static String servers = "192.168.150.101:9092,192.168.150.102:9092,192.168.150.103:9092";
    //客户端id
    private static String clientId = "product.client.producer.interceptor";
    //主题名称
    private static String topic = "first";


    /**
     * 生产者客户端配置
     * 这里只配置了几个必要的参数，其他参数后面会一一介绍。
     * @return
     */
    private static Properties config(){
        Properties properties = new Properties();
        //kafka服务器地址
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,servers);
        //key的序列化器类路径,序列化器可以直接获取类路径
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        //value的序列化器类路径,配置自定义 person 的序列化器
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,MySerializer.class.getName());
        //客户端id
        properties.put(ProducerConfig.CLIENT_ID_CONFIG,clientId);
        //返回配置对象
        return properties;
    }

    public static void main(String[] args) {
        //获取客户端配置
        Properties properties = config();
        //创建KafkaProducer对象,传入客户端配置信息
        KafkaProducer<String, Person> producer = new KafkaProducer<>(properties);
        //创建ProducerRecord对象,传入主题名称，消息
        ProducerRecord<String, Person> record = new ProducerRecord<>(topic,new Person("ld",18));

        //发送ProducerRecord对象
        try{
            //普通发送
            producer.send(record);
        }catch (Exception e){
            e.printStackTrace();
        }
        //关闭KafkaProducer对象
        producer.close();
    }
}
```

上述代码中将value的序列化器配置修改为自定义的序列化器类路径即：properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,MySerializer.class.getName());

* key：value.serializer。
* value：配置自定义的序列化器类路径。



##### 3.1.4、分区器

当发送消息时需要确定消息发往哪一个分区。如果消息 ProducerRecord 中指定了 partition 字段，那么就不需要分区器来确定分区，因为 partition 代表的就是分区号；如果消息 ProducerRecord 中未指定 partition 字段，那么就需要依赖分区器，根据 key 值来计算 partition。分区器就是为消息分配分区。

Kafka 客户端中提供了一个org.apache.kafka.clients.producer.Partitioner 接口，如果想自定义分区器，只需实现这个接口即可。

客户端中默认提供了一个分区器是 org.apache.kafka.clients.producer.internals.DefaultPartitioner，其分区逻辑是：如果 key 为 null 时，那么消息将会以轮询的方式发往主题内的各个可用分区；如果 key 不为 null 时，那么默认的分区器会对 key 进行哈希，通过得到的哈希值来计算分区号，拥有相同 key 的消息会被写入同一个分区中。

当然也可以自定义分区器，下面请看示例代码：



**step1：**自定义一个 MyPartitioner 类，实现 Partitioner 接口。

```java
package com.ld.producer.partitioner;

import org.apache.kafka.clients.producer.Partitioner;
import org.apache.kafka.common.Cluster;
import org.apache.kafka.common.PartitionInfo;
import org.apache.kafka.common.utils.Utils;

import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentMap;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * @author ld
 * 自定义分区器
 */
public class MyPartitioner implements Partitioner {
    private final AtomicInteger counter = new AtomicInteger(0);

    /**
     * 自定义分区策略
     * @param topic 主题名
     * @param key key 值
     * @param keyBytes key字节数组
     * @param value value 值
     * @param valueBytes value 字节数组
     * @param cluster 集群对象
     * @return
     */
    @Override
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        //获取主题的分区数
        List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
        int numPartitions = partitions.size();
        if (keyBytes == null) {
            //如果key值为null时，轮询发送
            return counter.getAndIncrement() % numPartitions;
        } else {
            //如果key值不为 null时，计算 key 的哈希值，通过哈希值计算分区号。
            return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
        }
    }

    /**
     * 关闭分区器时调用
     */
    @Override
    public void close() {
        System.out.println("关闭分区器......");
    }

    @Override
    public void configure(Map<String, ?> map) {

    }
}
```

上述代码中实现了 Partitioner 接口的两个方法。

* partition：用于实现分区逻辑。
* close：用于关闭分区时调用。

发现还有一个 configure() 方法，此方法不是 Partitioner 接口中的方法，而是其父接口 Configurable 中的方法。Configurable 接口中的 configure() 主要用来获取配置信息及初始化数据。



**step2：**编写生产者客户端，配置分区器参数。

```java
package com.ld.producer.partitioner;

import com.ld.producer.interceptor.MyProducerInterceptor;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;

/**
 * @author ld
 */
public class PartitionerKafkaProducer {

    //kafka服务器地址
    private static String servers = "192.168.150.101:9092,192.168.150.102:9092,192.168.150.103:9092";
    //客户端id
    private static String clientId = "product.client.producer.interceptor";
    //主题名称
    private static String topic = "first";


    /**
     * 生产者客户端配置
     * 这里只配置了几个必要的参数，其他参数后面会一一介绍。
     * @return
     */
    private static Properties config(){
        Properties properties = new Properties();
        //kafka服务器地址
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,servers);
        //key的序列化器类路径,序列化器可以直接获取类路径
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        //value的序列化器类路径
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,StringSerializer.class.getName());
        //客户端id
        properties.put(ProducerConfig.CLIENT_ID_CONFIG,clientId);

        //配置分区器
        properties.put(ProducerConfig.PARTITIONER_CLASS_CONFIG,MyPartitioner.class.getName());

        //返回配置对象
        return properties;
    }

    public static void main(String[] args) {
        //获取客户端配置
        Properties properties = config();
        //创建KafkaProducer对象,传入客户端配置信息
        KafkaProducer<String, String> producer = new KafkaProducer<>(properties);
        //创建ProducerRecord对象,传入主题名称，消息
        ProducerRecord<String, String> record = new ProducerRecord<>(topic,"testPartitioner");

        //发送ProducerRecord对象
        try{
            //普通发送
            producer.send(record);
        }catch (Exception e){
            e.printStackTrace();
        }
        //关闭KafkaProducer对象
        producer.close();
    }
}
```

上述代码中添加了一个新的配置即：properties.put(ProducerConfig.PARTITIONER_CLASS_CONFIG,MyPartitioner.class.getName());

* key：partitioner.class。
* value：配置分区器的类路径。



##### 3.1.5、原理分析

上面介绍了生产者客户端的使用，现在研究一下生产者客户端的内部原理。



从上图中可以看出整个生产者客户端由三部分组成，主线程，消息累加器和Sender线程。下面分别详细介绍下每部分：

* **主线程：**在主线程中由 KafkaProducer 创建消息，然后通过可能的拦截器、序列化器和分区器的作用之后缓存到消息累加器中。
* **消息累加器（RecordAccumulator）：**RecordAccumulator主要用来缓存消息以便 Sender 线程可以批量发送，进而减少网络传输的资源消耗以提升性能。RecordAccumulator 缓存的大小可以通过生产者客户端参数**buffer.memory** 配置，默认值为 33554432B，即32MB。
  * **双端队列**：RecordAccumulator内部为主题的每个分区都维护了一个双端队列，队列中的内容是一个一个 ProducerBatch，即 Deque<ProducerBatch>；
  * **ProducerBatch：**即消息批次，ProducerBatch中包含了一个或多个ProducerRecord。被Sender线程从双端队列的头部读取一个一个的ProducerBatch，发送到Kafka 服务器端。
  * **ProducerRecord：**是从主线程中发送过来的消息，并被追加到某个双端队列的尾部。
  * **ByteBuffer：**实现消息内存的创建和释放。即当一条消息被缓存到RecordAccumulator时，会先寻找与消息分区所对应的双端队列（如果没有则新建），再从这个双端队列的尾部获取一个 ProducerBatch（如果没有则新建），查看 ProducerBatch 中是否还可以写入这个ProducerRecord，如果可以则写入，如果不可以则需要创建一个新的ProducerBatch。在新建ProducerBatch 时评估这条消息的大小是否超过 batch.size 参数的大小，如果不超过，那么就以 **batch.size（默认值为16384B，即16KB）** 参数大小来创建ProducerBatch，这样在使用完这对段内存区域之后，可以通过 BufferPool 来管理进行复用；如果超过，那么就以评估的大小来创建 ProducerBatch，这段内存区域不会被复用。
  * **BufferPool：**是用来管理可复用的ByteBuffer。BufferPool中只管理特定大小的ByteBuffer，其他大小的ByteBuffer不会缓存到BufferPool中。
* **Sender线程：**从 RecordAccumulator 中获取缓存的消息，然后将原本<分区，Deque<ProducerBatch>>的保存形式转变成<Node，List<ProducerBatch>>的形式，其中Node表示Kafka集群的broker节点。Sender还会进一步将<Node，List<ProducerBatch>> 封装成<Node,Request>的形式，将Request 请求发送给 kafka 服务器端。
  * **InFlightRequests：**请求在从Sender 线程发往 Kafka 之前还会保存到 InFlightRequests 中，InFlightRequests 保存对象的具体形式为 Map<NodeId, Deque<Request>>，它的主要作用是缓存了已经发出去但还没有收到响应的请求（NodeId 是一个 String 类型，表示节点的 id 编号）。
  * **leastLoadedNode：**InFlightRequests 还可以获得 leastLoadedNode，即所有 Node 中负载最小的那一个。这里的负载最小是通过每个 Node 在 InFlightRequests 中还未确认的请求决定的，未确认的请求越多则认为负载越大。



##### 3.1.6、配置参数介绍

上面的示例代码中只配置了几个必要参数，但是当需要对客户端进行优化时，就需要配置更多参数，下面进行一一介绍：

| 参数名                                 | 默认值                                                       | 说明                                                         |
| -------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| bootstrap.servers                      | ""                                                           | 指定连接kafka集群所需的broker地址清单。                      |
| key.serializer                         | ""                                                           | 指定消息中 key 对应的序列化类路径。                          |
| value.serializer                       | “”                                                           | 指定消息中 value 对应的序列化类路径。                        |
| buffer.memory                          | 33554432B（32MB）                                            | 指定生产者客户端中用于缓存消息的缓冲区大小。                 |
| batch.size                             | 16384B（16KB）                                               | 指定 ProducerBatch 可以复用内存区域的大小。                  |
| client.id                              | ""                                                           | 指定生产者客户端的id。                                       |
| max.block.ms                           | 60000                                                        | 用来控制 KafkaProducer 中 send() 方法和 partitionsFor() 方法的阻塞时间。当生产者的发送缓冲区已满，或者没有可用的元数据时，这些方法就会被阻塞。 |
| partitioner.class                      | org.apache.kafka.clients. producer.internals.DefaultPartitioner | 指定分区器类路径。                                           |
| interceptor.classes                    | ""                                                           | 指定生产者拦截器类路径。                                     |
| enable.idempotence                     | false                                                        | 是否开启幂等性功能。                                         |
| max.in.flight.requests.per .connection | 5                                                            | 限制每个连接（也就是客户端与 Node 之间的连接）最多缓存的请求数。 |
| metadata.max.age.ms                    | 300000（5分钟）                                              | 如果在这个时间内元数据没有更新的话会被强制更新。             |
| acks                                   | 1                                                            | 指定分区中必须要有多少个副本收到这条消息，之后生产者才会认为这条消息是成功写入的。参数值如下：acks=1，生产者发送消息之后，只要分区的 leader 副本成功写入消息，那么它就会收到来自服务端的成功响应；acks=0，生产者发送消息之后不需要等待任何服务端的响应；acks=-1或acks=all，生产者在消息发送之后，需要等待 ISR 中的所有副本都成功写入消息之后才能够收到来自服务端的成功响应。 |
| max.request.size                       | 1048576B（1MB）                                              | 用来限制生产者客户端能发送的消息的最大值。                   |
| retries                                | 0                                                            | 用来配置生产者重试的次数，即在发送异常的时候进行重新发送。   |
| retry.backoff.ms                       | 100                                                          | 用来设定两次重试之间的时间间隔。                             |
| compression.type                       | none                                                         | 用来指定消息的压缩方式，默认请款下消息不会被压缩。该参数还可以配置为 “gzip”，“snappy” 和 “lz4”。 |
| connections.max.idle.ms                | 540000ms（9分钟）                                            | 用来指定在多久之后关闭空闲的连接。                           |
| linger.ms                              | 0                                                            | 用来指定生产者发送 ProducerBatch 之前等待更多消息加入ProducerBatch 的时间。生产者客户端会在 ProducerBatch 被填满或等待时间超过 linger.ms 值时发送出去。增大这个参数的值会增加消息的延迟，但是同时能提升一定的吞吐量。 |
| receive.buffer.bytes                   | 32768B（32KB）                                               | 用来设置 socket 接收消息缓冲区的大小。                       |
| send.buffer.bytes                      | 131072B（128KB）                                             | 用来设置socket 发送消息缓冲区的大小。                        |
| request.timeout.ms                     | 30000ms                                                      | 用来设置 Producer 等待请求响应的最长时间。                   |



#### 3.2、消费者客户端

##### 3.2.1、示例代码

**step1：**添加 maven 引用

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>1.1.1</version>
</dependency>
```

注意：我这里用的版本是和Kafka 集群的版本是一致的。如果版本不同，最好修改成自己的版本。

**step2：**编写代码

```java
package com.ld.consumer;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;

import java.util.Arrays;
import java.util.Properties;

/**
 * @author ld
 * kafka消费者示例代码
 */
public class KafkaConsumerDemo {

    //kafka服务器地址
     private static String servers = "192.168.150.101:9092,192.168.150.102:9092,192.168.150.103:9092";
     //消费者组id
     private static String groupId = "consumer-demo";
     //客户端id
     private static String clientId = "consumer.client.demo";
     //主题名称
     private static String topic = "first";

     /**
     * 消费者客户端配置信息
     * @return
     */
    private static Properties config(){
        Properties properties = new Properties();

        /*************************优化前*****************************/
//        //kafka服务器地址
//        properties.put("bootstrap.servers",servers);
//        //key 反序列化器类路径
//        properties.put("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
//        //value 反序列化器类路径
//        properties.put("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
//        //消费者组id
//        properties.put("group.id",groupId);
//        //客户端id
//        properties.put("client.id",clientId);

        /*************************优化后*****************************/
        /**
         * 为了防止开发人员将参数名写错，所以客户端提供了一个 ConsumerConfig 类，这个类中定义了所有要配置的参数名。
         * 所以我们可以使用这个类中的定义参数名。
         */
        //kafka服务器地址
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,servers);
        //key 反序列化器类路径
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        //value 反序列化器类路径
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,StringDeserializer.class.getName());
        //消费者组id
        properties.put(ConsumerConfig.GROUP_ID_CONFIG,groupId);
        //客户端id
        properties.put(ConsumerConfig.CLIENT_ID_CONFIG,clientId);

        return properties;
    }

    public static void main(String[] args) {
        //获取消费者客户端配置
        Properties properties = config();
        //创建 KafkaConsumer 对象,传入配置信息
        KafkaConsumer<String,String> kafkaConsumer = new KafkaConsumer<>(properties);
        //1.订阅主题
        kafkaConsumer.subscribe(Arrays.asList(topic));
        //2.通过正则表达式的方式订阅以 test- 开头的主题
//        kafkaConsumer.subscribe(Pattern.compile("test-*"));
        //3.指定主题，分区消费
//        kafkaConsumer.assign(Arrays.asList(new TopicPartition(topic,0)));
        //如果不知道分区数的话，可以先通过topic获取其分区数
//        List<PartitionInfo> partitionInfos = kafkaConsumer.partitionsFor(topic);

        try{
            //轮询消费主题
            while (true){
                //拉取消息
                ConsumerRecords<String, String> records = kafkaConsumer.poll(1000);
                //遍历输出消息
                for (ConsumerRecord<String, String> record : records) {
                        System.out.println("topic：" + record.topic() + ", partition：" + record.partition()
                                + ", offset：" + record.offset() + ", key：" + record.key() + ", value：" + record.value());
                }
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            //关闭消费者客户端
            kafkaConsumer.close();
        }
    }
}
```

上述就是 Java 版的消费者客户端代码。下面简述一下其中几个重要的API。

* **创建消费者对象**

  **API列表：**

  * **KafkaConsumer**(Map<String, Object> configs)
  * **KafkaConsumer**(Map<String, Object> configs, Deserializer<K> keyDeserializer, Deserializer<V> valueDeserializer)
  * **KafkaConsumer**(Properties properties)
  * **KafkaConsumer**(Properties properties, Deserializer<K> keyDeserializer, Deserializer<V> valueDeserializer)

  

  **参数介绍：**

  | 参数名            | 说明                        |
  | ----------------- | --------------------------- |
  | configs           | 使用map存储参数配置信息。   |
  | keyDeserializer   | 指定 key 反序列化器对象。   |
  | valueDeserializer | 指定 value 反序列化器对象。 |
  | properties        | 指定参数配置信息。          |



* **订阅主题与分区**

  **API列表：**

  * subscribe(Collection<String> topics)
  * subscribe(Collection<String> topics, ConsumerRebalanceListener listener)
  * subscribe(Pattern pattern)
  * subscribe(Pattern pattern, ConsumerRebalanceListener listener)
  * assign(Collection<TopicPartition> partitions)

  

  **参数介绍**

  | 参数名     | 说明                             |
  | ---------- | -------------------------------- |
  | topics     | 指定要订阅的主题集合。           |
  | pattern    | 采用正则表达式的方式订阅主题。   |
  | listener   | 用来设置相应的再均衡监听器。     |
  | partitions | 用来指定订阅某些主题的特定分区。 |

  

##### 3.2.2、消费者组

在 3.2.1 的消费者配置参数中有一个消费者组id 的配置，下面详细讲解一下消费者和消费者组之间的关系。

在kafka中的每一个消费者都有一个对应的消费者组。当消息发布到主题后，只会被投递给订阅它的每个消费者组中的一个消费者。



##### 3.2.3、订阅和取消订阅主题

* 订阅集合主题
* 支持正则订阅
* 支持指定分区的订阅



##### 3.2.4、消费消息

* 普通消费

```java
try{
    //轮询消费主题
    while (true){
        //拉取消息,即消费消息，timeout：表示在没有消费到消息时的阻塞时间
        ConsumerRecords<String, String> records = kafkaConsumer.poll(1000);

        //普通遍历输出消息
        for (ConsumerRecord<String, String> record : records) {
            System.out.println("topic：" + record.topic() + ", partition：" + 		                     record.partition() + ", offset：" + record.offset() + ", key：" +                         record.key() + ", value：" + record.value());
        }
    }
}catch (Exception e){
    e.printStackTrace();
}finally {
    //关闭消费者客户端
    kafkaConsumer.close();
}
```



* 按主题维度消费

```java
try{
    //轮询消费主题
    while (true){
        //拉取消息,即消费消息，timeout：表示在没有消费到消息时的阻塞时间
        ConsumerRecords<String, String> records = kafkaConsumer.poll(1000);

        //按照主题维度来进行消费
        //如果订阅了多个主题时，通过主题信息获取当前主题的的所有消息，并进行遍历
        for (ConsumerRecord<String, String> record : records.records(topic)) {
        	System.out.println("topic：" + record.topic() + ", partition：" +                             record.partition() + ", offset：" + record.offset() + ", key：" +                         record.key() + ", value：" + record.value());
        }
    }
}catch (Exception e){
    e.printStackTrace();
}finally {
    //关闭消费者客户端
    kafkaConsumer.close();
}
```



* 按分区维度消费

```java
try{
    //轮询消费主题
    while (true){
        //拉取消息,即消费消息，timeout：表示在没有消费到消息时的阻塞时间
        ConsumerRecords<String, String> records = kafkaConsumer.poll(1000);

        //按照分区维度来进行消费
        //获取消息中所有的分区信息，并进行遍历
        for (TopicPartition partition : records.partitions()) {
        //通过分区信息获取当前分区的所有消息，并进行遍历
        	for (ConsumerRecord<String, String> record : records.records(partition)) {
        		System.out.println("topic：" + record.topic() + ", partition：" +                             record.partition() + ", offset：" + record.offset() + ", key：" +                         record.key() + ", value：" + record.value());
            }
        }
    }
}catch (Exception e){
    e.printStackTrace();
}finally {
    //关闭消费者客户端
    kafkaConsumer.close();
}
```



* 指定位移消费



##### 3.2.5、提交位移

* 自动提交
* 手动同步提交
* 手动异步提交



##### 3.2.6、拦截器



##### 3.2.7 、反序列化器

在 3.1.3 节中自定义一个生产者的序列化器，这里就对应的要有一个消费者的反序列化器。

Kafka 客户端中提供了一个 org.apache.kafka.common.serialization.Deserializer 接口，如果想自定义反序列化器时，只需实现这个接口即可。

Kafka 客户端中还默认提供了好多反序列化器，如：StringDeserializer、IntergerDeserializer、DoubleDeserializer、LongDeserializer等等。

下面自定义一个反序列化器，请看示例代码：

**step1：**创建一个 Person 实体类 。

```java
package com.ld.producer.serializer;

/**
 * @author ld
 * 创建一个 person 类
 */
public class Person {

    //名称
    private String name;

    //年龄
    private int age;

    public Person() {
    }

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```



**step2：**自定义一个 MyDeserializer 类，实现 Deserializer 接口。

```java
package com.ld.consumer.deserializer;

import com.ld.producer.serializer.Person;
import org.apache.kafka.common.serialization.Deserializer;

import java.io.UnsupportedEncodingException;
import java.nio.ByteBuffer;
import java.util.Map;

/**
 * @author ld
 * 自定义一个反序列化器
 */
public class MyDeserializer implements Deserializer<Person> {

    /**
     * 用于配置当前类，在创建 kafkaConsumer 实例时被调用
     * @param map
     * @param b
     */
    @Override
    public void configure(Map<String, ?> map, boolean b) {
        System.out.println("测试反序列化器......");
    }

    /**
     * 用来执行反序列化逻辑
     * @param topic 主题
     * @param data 数据
     * @return
     */
    @Override
    public Person deserialize(String topic, byte[] data) {
        if(data == null){
            return null;
        }

        //读取数据，注意：这里要根据序列化的顺序读取
        ByteBuffer buffer = ByteBuffer.wrap(data);
        //获取name的长度
        int nameLen = buffer.getInt();
        //获取name
        byte[] nameBytes = new byte[nameLen];
        buffer.get(nameBytes);
        //获取age的长度
        int ageLen = buffer.getInt();
        //获取age
        byte[] ageBytes = new byte[ageLen];
        buffer.get(ageBytes);

        //转化byte[]数组
        try{
            String name = new String(nameBytes, "UTF-8");
            int age = Integer.valueOf(new String(ageBytes, "UTF-8"));
            return new Person(name,age);
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 用来关闭当前的反序列器
     */
    @Override
    public void close() {
        System.out.println("关闭反序列化器......");
    }
}
```

上述代码中实现了 Deserializer 接口的三个方法。

* configure：配置参数信息，在创建 KafkaConsumer实例时被调用。
* deserialize：用于实现反序列化逻辑。
* close：用于关闭反序列化器时调用。



**step3：**编写消费者客户端，配置反序列化器参数。

```java
package com.ld.consumer.deserializer;

import com.ld.producer.serializer.Person;
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;

import java.util.Arrays;
import java.util.Properties;

/**
 * @author ld
 *
 * 自定义反序列化器的主测试类
 */
public class DeserializerKafkaConsumer {

    //kafka服务器地址
    private static String servers = "192.168.150.101:9092,192.168.150.102:9092,192.168.150.103:9092";
    //消费者组id
    private static String groupId = "consumer-deserializer-group";
    //客户端id
    private static String clientId = "consumer.client.producer.interceptor";
    //主题名称
    private static String topic = "first";

    /**
     * 消费者客户端配置信息
     * @return
     */
    private static Properties config(){
        Properties properties = new Properties();
        //kafka服务器地址
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,servers);
        //key 反序列化器类路径
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        //value 反序列化器类路径
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, MyDeserializer.class.getName());
        //消费者组id
        properties.put(ConsumerConfig.GROUP_ID_CONFIG,groupId);
        //客户端id
        properties.put(ConsumerConfig.CLIENT_ID_CONFIG,clientId);
        return properties;
    }

    public static void main(String[] args) {
        //获取消费者客户端配置
        Properties properties = config();
        //创建 KafkaConsumer 对象,传入配置信息
        KafkaConsumer<String, Person> kafkaConsumer = new KafkaConsumer<>(properties);
        //订阅主题
        kafkaConsumer.subscribe(Arrays.asList(topic));

        try{
            //轮询消费主题
            while (true){
                //拉取消息,即消费消息，timeout：表示在没有消费到消息时的阻塞时间
                ConsumerRecords<String, Person> records = kafkaConsumer.poll(1000);

                /**
                 * 消费消息
                 */
                //普通遍历输出消息
                for (ConsumerRecord<String, Person> record : records) {
                    System.out.println("topic：" + record.topic() + ", partition：" + record.partition()
                            + ", offset：" + record.offset() + ", key：" + record.key() + ", value：" + record.value());
                }
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            //关闭消费者客户端
            kafkaConsumer.close();
        }
    }
}
```

上述代码中将value的反序列化器配置修改为自定义的反序列化器类路径即：properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, MyDeserializer.class.getName());

* key：value.deserializer。
* value：配置自定义的反序列化器类路径。



##### 3.2.8、分区分配策略



##### 3.2.9、再均衡监听器



##### 3.2.10、配置参数介绍























