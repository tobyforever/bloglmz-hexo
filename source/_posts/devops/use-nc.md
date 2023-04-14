---
title: 通过nc创建socket发数据给kafka
date: 2023-04-11 20:45:51
tags: linux
toc: true
published: true
---
# 使用nc实时输入数据

在大数据的实验中，需要启动一个socket，实时给kafka输入数据：

在 bigdata01 机器上，您可以使用 nc（Netcat）工具启动一个简单的 socket 服务器，监听 9999 端口。如果 nc 未安装在您的系统中，请首先安装它。在大多数 Linux 发行版中，您可以使用包管理器（如 apt、yum 或 pacman）来安装 nc。
在安装 nc 之后，您可以通过运行以下命令在 bigdata01 机器上启动 socket 服务器：
```
nc -lk 9999
```
此命令将启动一个 socket 服务器，监听 9999 端口。服务器将保持运行状态，直到您手动停止它。在此模式下，您可以通过控制台直接输入消息，然后按回车键发送。当有客户端连接到此服务器时，它们将接收到您通过控制台输入的消息。要停止服务器，只需按 Ctrl + C。
请注意，如果您希望在后台启动 socket 服务器并将其输出重定向到文件，可以使用以下命令：
```
nc -lk 9999 > output.log 2>&1 &
```
此命令将在后台启动 socket 服务器，并将所有输出（包括发送的消息）重定向到名为 output.log 的文件中。要停止在后台运行的服务器，您需要使用 kill 命令终止其进程。首先，使用 ps 和 grep 命令找到服务器进程的 ID：
```
ps aux | grep "nc -lk 9999"
```
然后使用 kill 命令终止该进程：
```
kill [process_id]
```
请将 [process_id] 替换为实际的进程 ID。

用法参考
https://github.com/tobyforever/geektime-homework-flink/tree/master/src/main/java/com/geektime/kafka

# flink怎么从socket直接读取数据
要从Socket直接读取数据，可以使用Flink提供的SocketTextStreamFunction函数。这个函数可以用来创建一个从Socket中读取数据的DataStream。以下是一个示例代码：

```java
import org.apache.flink.streaming.api.functions.source.SocketTextStreamFunction;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.datastream.DataStream;

public class SocketSourceExample {
    public static void main(String[] args) throws Exception {

        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        String hostname = "localhost";
        int port = 9000;

        // 从Socket中读取数据
        DataStream<String> text = env.addSource(new SocketTextStreamFunction(hostname, port, "\n", -1));

        // 打印DataStream中的数据
        text.print();

        env.execute("SocketSourceExample");
    }
}

```
在上面的代码中，我们创建了一个从localhost的9000端口读取数据的DataStream，并使用print()方法将其打印出来。注意，我们需要指定换行符（"\n"）和超时时间（-1）来告诉Flink如何从Socket中读取数据。

当我们运行这个程序时，它将等待来自Socket的输入数据。你可以使用telnet或nc命令（例如：nc -lk 9000）在另一个终端上向该端口发送数据，Flink将会实时处理输入的数据流。

# 如果hostname对应的机器和flink程序所在的机器不是同一个，能读的到数据吗？
上面的flink程序在本地localhost运行，运行nc socket的程序在bigdata02上，hostname改成bigdata02，是可以的。

如果hostname对应的机器和Flink程序所在的机器不是同一个，则需要确保两台机器之间能够建立网络连接，并且在网络中是可达的。否则，Flink程序将无法从远程机器的Socket读取数据。

例如，如果Flink程序运行在本地机器，而hostname对应的机器在另一台远程机器上，则需要确保两台机器之间能够建立网络连接，并且在本地机器上可以访问远程机器的hostname和端口。如果网络连接存在问题，例如防火墙阻止了网络连接，则Flink程序将无法读取来自远程机器的数据。

需要注意的是，如果数据的产生和Flink程序的运行环境不在同一台机器上，还需要考虑数据传输的带宽和延迟等问题。如果数据量很大或者网络延迟很高，则可能会影响程序的性能和实时性。

