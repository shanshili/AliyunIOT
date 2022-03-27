
# 步骤：

## 硬件：

5V供电，Rx接Tx，Tx接Rx

Baud: 115200

send: AT+RST

#### TCP Client端配置：

AT+CWMODE=1 //配置为station模式

   OK
   
AT+CWJAP_CUR="$WIFI名","$WIFI密码"  //连接到AP

   WIFI CONNECTED
   
   WIFI GOT IP
   
   OK
   
AT+CIFSR  //查询本机IP

   +CIFSR:STAIP,"  "
   
   +CIFSR:STAMAC,"  "
   
   OK  
   
AT+CIPMUX=0 //开启单连接

   OK

#### 透传：

AT+CIPSTART="TCP","Confidentiality confidentiality",1883

  CONNECT
    
  OK
    
  CLOSED

## 云端：

1、云平台创建产品、设备、物模型

2、mqtt调试json

3、网络助手调试报文

#### 三元组

```json
{
  "ProductKey": "   ",
  "DeviceName": "   ",
  "DeviceSecret": "    "
}
```

#### 接入域名 

${a1UUBhveKMv}.iot-as-mqtt.${cn-shanghai}.aliyuncs.com:1883



**client :**

|MQTT项目 | 内容|
| ---------------- | ---------------------------------------- |
| **mqttclientId**   |    |
| **mqttUsername**  |         |
| **mqttPassword** |  |
| **mqttPassword HMAC-SHA1** |   |
| **KeepAlive时长（s）** |  |

#### topic：



# MQTT报文



### Ⅰ连接报文

#### 一、报头

示例：10 <LENGTH> 00 04 4D 51 54 54 04 C2 00 64

##### 1、固定报头

**固定位** 10

**剩余长度字段 <LENGTH>**：固定报头之后的所有字段长度，等于可变报头的长度加上有效载荷的长度。

##### 2、可变报头

**协议名** 00 04 4D 51 54 54

**协议版本** 04

**连接标志** C2

**KeepAlive时长** 00 64, 表示系统在未收到设备消息的多长时间后自动断开连接，这里设置为100秒

#### 二、载荷

<LENGTH> <LENGTH> ${mqttclientId} <LENGTH> <LENGTH> ${mqttUsername} <LENGTH> <LENGTH> ${mqttPassword(hmacsha1)}

载荷中的参数已在第三节给出，最后的mqttPassword是加密后的字符串，每项参数前面的两个字节表示该参数的长度。

**成功** 20 02 00 00



### Ⅱ订阅SUBSCRIBE

正如前面提到的，TOPIC首先需要订阅才能使用。

#### 一、报头

示例：82 <LENGTH> 00 0A

##### 1、固定报头

**固定位** 82

**剩余长度字段 <LENGTH>**： 固定报头之后的所有字段长度

##### 2、可变报头

00 0A

#### 二、载荷

<LENGTH> <LENGTH> ${TOPIC} <QoS>

其中QoS为服务质量等级， 00为无需认证，01为认证一次，02为至少认证一次，这里选择00无需认证即可。

##### 响应

**成功** 90 <LENGTH> <ID> 00



### Ⅲ发布PUBLISH （QoS0）

通过PUBLISH，向相应的TOPIC发送信息。

#### 一、报头

示例：30 <LENGTH> <LENGTH> <LENGTH> ${TOPIC}

##### 1、固定报头

**固定位** 30

**剩余长度字段 <LENGTH>**： 固定报头之后的所有字段长度

##### 2、可变报头

 <LENGTH> <LENGTH> ${TOPIC}

#### 二、载荷

示例：

```json
{
    "id":"1562283421",
    "version":"1.0.0",
    "params":{"temp":12},
    "method":"thing.event.property.set"
}
```

其中“method”同Topic的后半部分；“id”为每条消息的自定义id，其实可以重复，但不建议重复，可以随机；“params”即需要设置的设备属性，把设备看作一个对象，对象里的值为设备的属性；“version”为固定部分。

QOS=0时TOPIC后没有报文标识符

*（上云工具在TOPIC后多了00 32，因此总长度计算也不对）*
 
**正确的：** 需在 30 92 后加入 01 ，不知道为什么(不会是因为超出了127？)



### ⅣPING

**报文** 

> C0 00

**响应** D0 00



### Ⅴ断开 DISCONNECT

**报文** 

> E0 00

以上是对于MQTT协议简单的分析，掌握了MQTT的套路后可以根据官方协议手册自行取用。






