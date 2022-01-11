# 通信协议

代码详见：[代码](https://github.com/jkstack/natpass/tree/master/code/network)

1. 数据传输过程中的数据包格式如下

        +++++++++++++++++++++++++++++++++++++++++++++++++
        + size(16bit) | crc32(32bit) | protobuf payload +
        +++++++++++++++++++++++++++++++++++++++++++++++++
2. msg结构为数据传输的基本结构，[数据结构](https://github.com/jkstack/natpass/blob/master/code/network/msg.proto)如下

        message msg {
            enum type { // 消息类型定义
                ...
            }
            type      _type = 1; // 消息类型，详见type结构
            string     from = 2; // 数据来源client id
            uint32 from_idx = 3; // 来源连接id
            string       to = 4; // 数据目标client id
            uint32   to_idx = 5; // 目标连接id
            string  link_id = 6; // 虚拟链接id
            oneof payload { // 消息内容定义
                ...
            }
        }

## 握手

在创建tcp/tls连接后client会发送一个握手报文来表明身份，其中包含一个enc字段该字段的值为配置文件中的secret的md5码

TODO补图

## 虚拟链接