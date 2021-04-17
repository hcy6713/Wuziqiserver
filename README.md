# WuziqiServer
课程作业——基于C#和WPF的五子棋对战服务器设计


## 一、五子棋服务器运行环境

IDE:Visual Studio 2017(C# + WPF)

框架：.NET Framework 4.6.1

## 二、连接服务器

ipAddress为五子棋服务器程序运行计算机ipv4地址，可以通过WIN + R打开运行框，输入cmd进入命令行终端，输入ipconfig命令查看

port为五子棋服务器监听的端口，通过port框进行设置

## 二、游戏开始

当连接服务器的客户端数量为2时自动开始游戏，并通过随机数的方式决定哪方先走，且会给先落子一方发送开始游戏指令

## 三、游戏规则

- 当客户端接收到开始游戏指令时，需要发送位置信息告诉服务器落子位置
  - 由于棋盘大小为15 * 15，所以 x, y ∈ [0, 14]
- 落子后服务器会检查落子一方是否胜利，若胜利，则给胜利一方发送胜利信息，失败一方发送失败信息，并结束游戏
- 若未胜利，服务器会将落子位置信息转发给对手方，对手方需要发送位置信息告诉服务器落子位置

## 四、通信协议

数据包(只含命令) = 帧头(1字节) + nameID(1字节) + commandID(1字节)  + 校验码(4字节) + 帧尾(1字节)

数据包(含命令与信息) = 帧头(1字节) + nameID(1字节) + commandID(1字节) + 信息体长度(4字节) + 信息体 + 校验码(4字节) + 帧尾(1字节)

帧头：0xA5

帧尾：0x5A

信息体：单个信息长度(4字节) + 单个信息（通知客户端执棋颜色和通知落子位置时需要带信息体）

例：在(x, y)处落子的信息： x的长度(4字节) + x + y的长度 + y

校验码：nameID * commandID * 信息体长度（不含信息时不计算信息体长度）

| nameID |    信息来源    |
| :----: | :------------: |
|  0xAA  | 信息来自服务器 |
|  0xBB  | 信息来自黑色方 |
|  0xCC  | 信息来自白色方 |

| commandID |                          含义                           |
| :-------: | :-----------------------------------------------------: |
|   0x11    |           通知客户端服务器已有两个客服端连接            |
|   0x22    |                   通知客户端执棋颜色                    |
|   0x33    |         通知先落子一方客户端游戏开始，落第一子          |
|   0x44    |               通知客户端或服务器落子位置                |
|   0x55    |               通知客户端落子位置已有棋子                |
|   0x66    |                 通知客户端不是你的回合                  |
|   0x77    | 通知客户端数据包内容(nameID、commandID等)有误，不能识别 |
|   0x88    |                     通知客户端胜利                      |
|   0x99    |                     通知客户端失败                      |
|   0xAA    |                     通知服务器认输                      |
|   0xBB    |    数据包格式有误（校验码不符合和数据包格式不匹配）     |



