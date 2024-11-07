# Checkpoint0-实验报告

## 1 环境配置
1. CS144实验需要GNU/Linux操作系统，因此配置虚拟机并安装Ubuntu

| 虚拟机   | VMware Workstation 16 Pro |
| -------- | ------------------------- |
| 操作系统 | Ubuntu 22.04.1 LTS        |

2. 同时为了进行实验，根据要求安装如下工具链： ![image-20241106221824357](C:\Users\Princess\AppData\Roaming\Typora\typora-user-images\image-20241106221824357.png)




## 2 手动联网
根据实验要求，在本部分需要完成手动抓取网页和发送邮件的任务。
### 2.1 获取网页
1. 首先访问指定网页 http://cs144.keithw.org/hello，可以看到网页显示的句子

![image-20241106223838060](C:\Users\Princess\AppData\Roaming\Typora\typora-user-images\image-20241106223838060.png)

2. 接下来在命令行终端输入命令，该命令可以使用http服务在两台主机间建立一个可靠的字节流。效果如下：

​	![image-20241106224331557](C:\Users\Princess\AppData\Roaming\Typora\typora-user-images\image-20241106224331557.png)

3. 接着在终端中输入以下 HTTP 请求：

   - 输入 `GET /hello HTTP/1.1` 并按下回车键。这条指令告诉服务器请求路径部分（即 URL 中第三个斜杠后的部分）。
   - 输入 `Host: cs144.keithw.org` 并按下回车键。这条指令告知服务器请求的主机名。
   - 输入 `Connection: close` 并按下回车键。这条指令告诉服务器在响应完成后关闭连接。
   - 再次按下回车键，发送一个空行，表示 HTTP 请求已完成

   **请求超时**：刚开始尝试时出现几次Timeout的提示，经查询 `408 Request Timeout` 错误通常是因为服务器等待请求的时间过长，导致请求超时。如下图所示：
   ![image-20241031192830610](C:\Users\Princess\AppData\Roaming\Typora\typora-user-images\image-20241031192830610.png)

   **请求成功**：经过调整后，在 `telnet` 会话中手动输入 HTTP 请求时，快速输入每一行并按下回车。
   ![image-20241031192751050](C:\Users\Princess\AppData\Roaming\Typora\typora-user-images\image-20241031192751050.png)
   
4. **assignment**：根据前面学习的手动抓取网页的方法，访问指定的 URL `http://cs144.keithw.org/lab0/sunetid`，并将 `sunetid` 替换为复旦大学学号。服务器返回一个响应，其中包含 `X-Your-Code-Is` 头部，也就是成功获取了包含密钥的响应头信息，验证了通过手动 HTTP 请求获取网页的技术。![image-20241106225755396](C:\Users\Princess\AppData\Roaming\Typora\typora-user-images\image-20241106225755396.png)



### 2.2 发邮件

该部分实验要求使用 SMTP 协议，通过可靠的字节流手动发送电子邮件。由于没有斯坦福的邮箱，无法完成与邮件服务器的交互，这里只展示运行telnet命令连接到 SMTP 服务器并加载出服务器欢迎信息的效果：![image-20241031201047540](C:\Users\Princess\AppData\Roaming\Typora\typora-user-images\image-20241031201047540.png)

### 2.3 服务器监听

该部分实验使用 `netcat` 创建一个简单的服务器，这个服务器将等待客户端的连接请求并响应。

1. **启动服务器**：在一个终端窗口中，运行命令`netcat -v -l -p 9090` 在端口 9090 上启动 `netcat` 服务器。成功启动后，终端显示信息如下：

​	![image-20241107133830787](C:\Users\Princess\AppData\Roaming\Typora\typora-user-images\image-20241107133830787.png)

2. **启动客户端**：保持 `netcat` 服务器运行状态，打开另一个终端窗口，运行命令 `telnet localhost 9090` 使用 `telnet` 连接到本地服务器：

​	![image-20241107134204772](C:\Users\Princess\AppData\Roaming\Typora\typora-user-images\image-20241107134204772.png)

3. **测试数据传输**：在任一终端窗口中输入文本，并按回车键。可以观察到输入的文本会在另一个窗口中显示，实现了客户端与服务器之间的通信。

	![image-20241031204927674](C:\Users\Princess\AppData\Roaming\Typora\typora-user-images\image-20241031204927674.png) 
	![image-20241031204942531](C:\Users\Princess\AppData\Roaming\Typora\typora-user-images\image-20241031204942531.png)



## 3. 编写Webget

1. **获取和编译初始代码**：

   - 在虚拟机中运行以下命令，克隆实验所需的代码库 Minnow：

     ```
     git clone https://github.com/cs144/minnow
     ```

   - 进入实验目录：

     ```
     cd minnow
     ```

   - 创建构建目录并编译代码：

     ```
     cmake -S . -B build
     cmake --build build
     ```

   - 遇到的问题：

     在执行编译步骤时，出现了如下图错误信息。此错误是由于函数 `boolStr` 被声明为 `constexpr`，但其返回类型为 `std::string`。在 C++ 中，`constexpr` 函数的返回值必须在编译时确定，而 `std::string` 不支持这种操作，因为它在编译时无法完全构造。这里简单处理删除`constexpr`，以解决不兼容问题。![image-20241031222102467](C:\Users\Princess\AppData\Roaming\Typora\typora-user-images\image-20241031222102467.png)

2. **实现 `webget` 程序**：

   - 打开 `apps/webget.cc` 文件，根据注释的提示编辑 `get_URL` 函数，实现一个简单的 Web 客户端来请求网页内容。

     ```python
     
     void get_URL( const string& host, const string& path )
     {
       TCPSocket s;
       s.connect(Address(host, "http"));
       s.write("GET " + path + " HTTP/1.1\r\n");
       s.write("Host: " + host + "\r\n");
       s.write("Connection: close\r\n");
       s.write("\r\n"); 
       s.shutdown(SHUT_WR);
       std::string buffer;
       while(!s.eof())
       {
         s.read(buffer);
         cout <<  buffer;      
       }
       s.close();
     }
     ```

3. **编译并测试程序**：

   - 编译程序：

     ![image-20241104162633921](C:\Users\Princess\AppData\Roaming\Typora\typora-user-images\image-20241104162633921.png)
   - 测试程序功能：
   
     ```
     ./apps/webget cs144.keithw.org /hello
     ```
   
     ![image-20241104162704241](C:\Users\Princess\AppData\Roaming\Typora\typora-user-images\image-20241104162704241.png)
   
4. **自动化测试**：

   - 通过上面的简单测试后，运行自动化测试，输出显示 “100% tests passed”。
   
     ```
     cmake --build build --target check_webget
     ```
   
     ![image-20241104170932873](C:\Users\Princess\AppData\Roaming\Typora\typora-user-images\image-20241104170932873.png)

5. 成功实现了通过流式套接字获取网页内容的功能，能够从指定的服务器地址和路径请求并显示网页内容，验证了操作系统流式套接字在网络通信中的应用。





