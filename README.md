
> RXTX 项目提供了 Windows、Linux、Mac os X、Solaris 操作系统下的兼容 javax.comm 串口通讯包 API 的实现，为其他研发人员在此类系统下研发串口应用提供了相当的方便。本项目基于 RXTX 封装重构了部分代码，使开发人员只需简单几步即可实现 java 串口通信，需要注意的是，这不是一个可执行的程序，如果你要使用它，请将它作为你项目的依赖库或者编译成 jar 包引入到项目

## 准备工作
点击[这里](https://pan.baidu.com/s/1i4X3Fxj)下载准备资源，密码：1234
1. 下载 `rxtx-2.2pre2-bins.zip`，根据不同的操作系统将 RXTX 开源包配置好，这里提供 windows 平台配置方法，其他平台请自行阅读官方文档
 - 将 `rxtxSerial.dll` 放入 `<系统盘>\Windows\System32`
 - 将 `rxtxSerial.dll` 放入 `<JAVA_HOME>\jre\bin`
 - 将 `RXTXcomm.jar` 放入 `<JAVA_HOME>\jre\lib\ext`
2. 将 `RXTXcomm.jar` 和 log4j 依赖的 jar 包加入到你的项目中
3. 将此模块编译成的 `simple_serial_port.jar` 加入到你的项目中，或者直接将该模块作为依赖库引入项目中
## 正式使用
1. 查找可用串口

```java
ArrayList<String> ports = SerialTool.findPort();
```
2. 打开上述找到的串口
```java
// 第一个参数用于给 serialPortVo 指定一个名字，便于识别不同的 serialPortVo ，可通过 serialPortVo.getName() 获取
// 这里只是打开了上一步骤找到了串口集合中的第一个串口，可根据需求更改
SerialPortVo serialPortVo = SerialTool.openPort("我是dtu的串口", ports.get(0), baudRate);
```
3. 新建监听器继承 `PortListener` 父类并重写 `onReceive()` 和 `onReadException()` 方法，当串口接收到数据自动调用 `onReceive()` 方法，当接收数据时发生异常将调用 `onReadException()` 方法
```java
public class MyListener extends PortListener {
    @Override
    public void onReceive(byte[] data) {
        String dataStr = new String(data).trim();
        System.out.println("串口接收到数据: " + dataStr);
    }
    
    @Override
    public void onReadException(Exception e) {
        System.out.println("发生异常: " + e.getMessage());
    }
}
```
4. 利用打开串口时返回的 `SerialPortVo` 对象绑定监听器
```java
// 第三个参数 500 为串口收到数据后等待 500 ms后再去读取串口数据，
// 防止读串口时数据没有完全到达串口，可根据数据大小适当调整等待时间
serialPortVo.bindListener(new MyListener(), 500);
```
5. 给串口发送数据
- 方式一：在监听器外，使用打开串口时返回的 `SerialPortVo` 对象发送字节数组类型的数据
```java
// 使用打开串口返回的 SerialPortVo 对象发送方法
String reply = "我是在监听器类外面给你发送的消息";
serialPortVo.sendData(reply.getBytes());
```
- 方式二：在监听器内，直接使用监听器内部 `SerialPortVo` 对象调用发送字节数组类型的数据
```java
public class MyListener extends PortListener {
    @Override
    public void onReceive(byte[] data) {
        String dataStr = new String(data).trim();
        // 直接使用内置 SerialPortVo 对象调用发送方法
        String reply = "我收到你发送的数据啦";
        serialPortVo.sendData(reply.getBytes());
    }
    
    @Override
    public void onReadException(Exception e) {
        //...
    }
}
```
