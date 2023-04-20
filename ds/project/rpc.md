# Socket

## TCP

基于TCP的Socket通信，实现前后端通信

### 服务器端

- 服务器端需要先运行，指定监听端口，等待客户端接入

```java
package TCP;

import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;


public class TCPServer {
    public static void main(String[] args) {

        try  {
            // create server socket, listening port
            ServerSocket serverSocket = new ServerSocket(8888);
            // call accept() function to get message
            System.out.println("---- TCP server is running, waiting for connecting");
            Socket socket = serverSocket.accept();
            // get input stream
            InputStream in = socket.getInputStream();
            // convert to string input stream
            InputStreamReader inreader = new InputStreamReader(in);
            // add buffer for input stream
            BufferedReader br = new BufferedReader(inreader);
            String info = null;
            while((info = br.readLine())!=null){
                System.out.println("This is TCP server, client says: "+info);

            }
            socket.shutdownInput();

            // get output stream, send to client
            OutputStream outputStream = socket.getOutputStream();
            PrintWriter printWriter = new PrintWriter(outputStream);
            printWriter.write("message from TCP server, welcome! ");
            printWriter.flush();
            socket.shutdownOutput();

            //close resource
            printWriter.close();
            outputStream.close();

            br.close();
            inreader.close();
            in.close();
            socket.close();
            serverSocket.close();

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

### 客户端

- 客户端要指定IP地址和接口，并使用序列化完成发送消息和接收消息。

```java
package TCP;

import java.io.*;
import java.net.Socket;

public class TCPClient {
    public static void main(String[] args) {
        try {
            // create socket to connect, assign host and port
            Socket socket = new Socket("127.0.0.1",8888);
            // create output stream, send message to server
            OutputStream outputStream = socket.getOutputStream();
            // convert to print stream
            PrintWriter pw = new PrintWriter(outputStream);
            // send message
            pw.write("message from TCP client");
            pw.flush();
            socket.shutdownOutput();

            // get server stream
            InputStream inputStream = socket.getInputStream();
            // add buffer for input stream
            BufferedReader br = new BufferedReader(new InputStreamReader(inputStream));
            String info = null;
            while((info = br.readLine())!=null){
                System.out.println("This is TCP client, server says: "+info);

            }
            socket.shutdownInput();

            // close resource
            br.close();
            inputStream.close();
            pw.close();
            outputStream.close();
            socket.close();

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

## UDP

- 与TCP类似，需要`DatagramSocket`和`DatagramPacket`库

### 服务器端

- 服务器端需要先运行，指定监听端口，等待客户端接入

```java
package UDP;

import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.net.SocketException;

public class UDPServer {
    public static void main(String[] args) {

        try  {
            // create DatagramSocket, port:8800
            DatagramSocket socket = new DatagramSocket(8800);
            // byte array to receive client message
            byte[] data = new byte[1024];
            DatagramPacket packet = new DatagramPacket(data,data.length);
            // receive data package
            socket.receive(packet);
            // read data
            String info = new String(data,0,packet.getLength());
            System.out.println("I am Server,Client say:"+info);


            // define port and ip
            InetAddress address = packet.getAddress();
            int port = packet.getPort();
            byte[] data2 = "This is UTP server, welcome ".getBytes();
            DatagramPacket packet1 = new DatagramPacket(data2,data2.length,address,port);
            socket.send(packet1);

            socket.close();

        } catch (SocketException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 客户端

- 需要指定IP地址和端口，使用`DatagramPacket`包装消息发送

```java
package UDP;

import java.io.IOException;
import java.net.*;


public class UDPClient {
    public static void main(String[] args) {

        try {

            // define IP and port
            InetAddress inetAddress = InetAddress.getByName("localhost");
            int port = 8800;
            byte[] data = "UDP send message, socket connect successful!".getBytes();
            // create DatagramSocket, port:8800
            DatagramPacket packet = new DatagramPacket(data,data.length,inetAddress,port);
            // create data socket
            DatagramSocket socket = new DatagramSocket();
            // send data pocket to server
            socket.send(packet);
            //socket.close();


            // receive server response
            byte[] dataR = new byte[1024];
            DatagramPacket packet1 = new DatagramPacket(dataR,dataR.length);
            socket.receive(packet1);
            // read message
            String info = new String(dataR,0,packet1.getLength());
            System.out.println("This is UDP client,sever says:"+info);

            socket.close();



        } catch (UnknownHostException e) {
            e.printStackTrace();
        } catch (SocketException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

# RPC

- RPC通信比较复杂，但主要思路还是很简单：**进程通过参数传递的方式调用另一进程（通常在远程机器）中的一个函数或方法，并得到返回的结果**
- 进程：客户端
- 另一进程：服务器
- 函数或方法：服务
- RPC在使用形式上像调用本地函数（或方法）一样去调用远程的函数（或方法）

## 服务器端

- 主要由四个java程序构成，分别为`main`（主入口），`RPCServer`（具体实现），`HelloService`（示例函数）

### RPCServer

- 对需要的函数（服务）分一个线程进行阻塞等待
- 主要接收客户端传来的函数名和参数，并使用`反射`进行调用，返回结果

```java
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;


public class RPCServer {

    private ExecutorService threadPool;
    private static final int DEFAULT_THREAD_NUM = 10;

    public RPCServer(){
        threadPool = Executors.newFixedThreadPool(DEFAULT_THREAD_NUM);
    }

    public void register(Object service, int port){
        try {
            System.out.println("server starts...");
            ServerSocket server = new ServerSocket(port);
            Socket socket = null;
            while((socket = server.accept()) != null){
                System.out.println("client connected...");
                // thread execute new service
                threadPool.execute(new Processor(socket, service));
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // implement Runnable interface(thread)
    class Processor implements Runnable{
        Socket socket;
        Object service;

        public Processor(Socket socket, Object service){
            this.socket = socket;
            this.service = service;
        }
        public void process(){

        }
        @Override
        public void run() {
            try {
                //
                ObjectInputStream in = new ObjectInputStream(socket.getInputStream());
                String methodName = in.readUTF();
                Class<?>[] parameterTypes = (Class<?>[]) in.readObject();
                Object[] parameters = (Object[]) in.readObject();
                Method method = service.getClass().getMethod(methodName, parameterTypes);
                try {
                    Object result = method.invoke(service, parameters);
                    ObjectOutputStream out = new ObjectOutputStream(socket.getOutputStream());
                    out.writeObject(result);
                } catch (IllegalAccessException e) {
                    e.printStackTrace();
                } catch (IllegalArgumentException e) {
                    e.printStackTrace();
                } catch (InvocationTargetException e) {
                    e.printStackTrace();
                }
            } catch (IOException e) {
                e.printStackTrace();
            } catch (NoSuchMethodException e) {
                e.printStackTrace();
            } catch (SecurityException e) {
                e.printStackTrace();
            } catch (ClassNotFoundException e1) {
                e1.printStackTrace();
            }
        }
    }
}
```

### HelloService

- 一个简单的函数实现，不过多介绍

```java

public interface HelloService{

    public String hello(String name);

}

public class HelloServiceImpl implements HelloService{

    @Override
    public String hello(String name) {
        return "Hello, " + name;
    }
}

```

### Main

- Server端的入口，首先需要新建一个`RPCServer`类，然后对需要的函数进行注册

```java
public class Main {

    public static void main(String args[]){
        HelloService helloService = new HelloServiceImpl();

        RPCServer server = new RPCServer();
        server.register(helloService,50001);
    }
}
```

## 客户端

- 主要由`RPCClient`（主体功能）和`HelloService`（需要服务的接口）组成

### RPCClient

- 使用`newProxyInstance`进行代理，当客户端访问该服务（函数）时，自动调用改写的`invoke`方法，代表客户端与服务器端进行通信。
- 注意，这里使用了匿名内部类和模板类实现。

```java
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.net.Socket;

public class RPCClient {

    public static void main(String args[]){
        // This helloService is a proxy object,it will call remote real object to finish method
        // need helloService interface
        HelloService helloService = getClient(HelloService.class, "127.0.0.1", 50001);
        System.out.println(helloService.hello("duyuntao"));
    }

    @SuppressWarnings("unchecked")
    public static <T> T getClient(Class<T> clazz, String ip, int port){
        return  (T) Proxy.newProxyInstance(RPCClient.class.getClassLoader(), new Class<?>[]{clazz}, new InvocationHandler() {
            // this is a anonymous inner class
                @Override
                public Object invoke(Object arg0, Method method, Object[] arg) throws Throwable {
                    Socket socket = new Socket(ip, port);
                    ObjectOutputStream out = new ObjectOutputStream(socket.getOutputStream());
                    out.writeUTF(method.getName());
                    out.writeObject(method.getParameterTypes());
                    out.writeObject(arg);
                    ObjectInputStream in = new ObjectInputStream(socket.getInputStream());
                    return in.readObject();
            }
        });
    }
}
```

### HelloService

- 只提供一个接口用于告诉java这个所需的类里面有什么服务
- 实际的函数调用在服务器端

```java
public interface HelloService{

    public String hello(String name);

}
```

# Reference

- [在Ubuntu下安装Java](https://blog.csdn.net/kongfu_cat/article/details/79510885)
- [Java Socket编程实例](https://blog.csdn.net/qq_32832023/article/details/78703810)
- [rpc原理 java实现](https://blog.csdn.net/u010900754/article/details/78081428)
- [Java的代理](https://www.cnblogs.com/xdp-gacl/p/3971367.html)
- [匿名内部类](https://www.cnblogs.com/nerxious/archive/2013/01/25/2876489.html)
- [接口和抽象类的区别](https://www.cnblogs.com/dolphin0520/p/3811437.html)
- [线程的启动](https://www.cnblogs.com/echo-cheng/p/6814909.html)

