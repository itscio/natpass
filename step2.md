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
3. 对于数据传输过程中的编码，目前已有统一的封装，详情见[代码](https://github.com/jkstack/natpass/blob/master/code/network/network.go)中的ReadMessage和WriteMessage接口

## 握手

在创建tcp/tls连接后client会发送一个握手报文来表明身份，其中包含一个enc字段该字段的值为配置文件中的secret的md5码

TODO补图

## 虚拟链接

在终端页面创建任何一个链接时，都会发起一个`connect_req`请求，该请求使用[connect_request](https://github.com/jkstack/natpass/blob/master/code/network/connect.proto)结构进行包装

        message connect_request {
            enum type {
                tcp   = 0; // tcp reverse proxy
                udp   = 1; // udp reverse proxy
                shell = 2; // shell
                vnc   = 3; // vnc
            }
            string name = 1; // rule name
            type  _type = 2; // rule type
            oneof payload {
                connect_addr   caddr = 10; // for reverse proxy
                connect_shell cshell = 11; // for shell
                connect_vnc     cvnc = 12; // for vnc
            }
        }

当受控端收到`connect_req`请求后会进行相应的处理，详见[代码](https://github.com/jkstack/natpass/blob/master/code/client/app/connect.go)

1. 收到shell请求后
   - 若给定exec参数，则拉起exec参数指定的程序，并接管pty设备
   - 否则根据当前操作系统拉起指定程序
     - windows: 若当前操作系统有powershell则拉起powershell程序，否则拉起cmd程序
     - 非windows: 若当前系统有bash则拉起bash程序，否则拉起sh程序
   - 返回`connect_response`消息，其中包含是否成功或错误信息
2. 收到vnc请求后
   - 从6155~6955之间顺序挑一个端口创建一个websocket服务，用于与后续创建的子进程通信
   - 创建子进程，`action`参数设置为vnc.worker，并传入规则名称和websocket端口号
   - 返回`connect_response`消息，其中包含是否成功或错误信息