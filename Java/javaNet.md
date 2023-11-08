## 基本通信架构
通信架构主要可以分为Client-Server(CS)和Browser-Server(BS)架构，其中CS中的客户端会负责一些功能，但BS架构会将所有功能放到Server中。

## IP地址
JAVA有InetAddress类来代表IP地址

````
public static void main(String[] args) throws Exception {
    // 1. 获取自己IP地址对象
    InetAddress localIP = InetAddress.getLocalHost();
    System.out.println(localIP.getHostName());
    System.out.println(localIP.getHostAddress());

    // 2. 通过域名获取IP
    InetAddress ip = InetAddress.getByName("www.google.com");
    System.out.println(ip.getHostName());
    System.out.println(ip.getHostAddress());

    // 3. 查看IP地址是否可达
    System.out.println(ip.isReachable(5000));
}
````

## 端口号
周知端口号：0-1023<br>
注册端口号：1024-49151<br>
动态端口号：49152-65535

## UDP
UPD是无连接，不可靠的</br>
java中提供了java.net.DatagramSocket来实现通信

client:
````
public static void main(String[] args) throws Exception {
    // 1. 创建socket
    // 可指定端口号，没有指定则自动分配
    DatagramSocket sock = new DatagramSocket();

    // 2. 创建数据类
    // 在数据类中指定接收端的ip地址和端口号
    byte[] data = "hello world".getBytes();
    DatagramPacket packet = new DatagramPacket(data, data.length, InetAddress.getLocalHost(), 6666);

    // 3. 发送数据
    sock.send(packet);

    // 4. 关闭端口
    sock.close();
}
````

server:
````
public static void main(String[] args) throws Exception {
    // 绑定接听端口
    DatagramSocket sock = new DatagramSocket(6666);

    // 创建DatagramPacket接受数据
    byte data[] = new byte[1024];
    DatagramPacket packet = new DatagramPacket(data, data.length);

    // 接收数据
    sock.receive(packet);

    // 查看数据
    String msg = new String(data,0,packet.getLength());
    System.out.println(msg);

    // 查看发送者信息
    System.out.println(packet.getAddress());
    System.out.println(packet.getPort());
}
````

## TCP
tcp可靠，面向连接。</br>
java提供了java.net.Socket类和java.net.ServerSocket来实现TCP.

client:
````
public static void main(String[] args) throws Exception {
    // 1. 创建socket并指定服务端IP和port
    Socket socket = new Socket(InetAddress.getLocalHost(), 6666);

    // 2. 获取输出流
    OutputStream os = socket.getOutputStream();
    //将输出流封装为Dataoutputsetream, 这步可有可无
    DataOutputStream dos = new DataOutputStream(os);   
    
    // 3. 获取用户输入并发送
    Scanner sc = new Scanner(System.in);
    while (true) {
        // 获取用户输入
        System.out.println("please input (exit to quit): ");
        String msg = sc.nextLine();
        if(msg.equals("exit")){
            break;
        }

        // 发送信息
        dos.writeUTF(msg);
    }
    System.out.println("client end.");

    // 释放资源
    dos.close();
    socket.close();
}
````

server:
````
public static void main(String[] args) throws Exception {
    // 1. 创建监听socket,并绑定端口号
    ServerSocket listenSocket = new ServerSocket(6666);

    // 2. 等待连接，如有连接创建socket与其交流
    // 对于创建的socket，创建线程处理任务来保证listenSocket可以一直监听
    while (true) {
        Socket socket = listenSocket.accept();
        ServerSocketThread thread = new ServerSocketThread(socket);
        thread.start();
    }
}
````

server thread:
````
public class ServerSocketThread extends Thread {
    private Socket socket;

    public ServerSocketThread(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        try {
            // 接收消息
            // 拿到输入流并封装
            InputStream is = socket.getInputStream();
            DataInputStream dis = new DataInputStream(is);
            // 输出消息
            while (true) {
                // 如果客户端关闭，dis.readUTF会出现异常
                try {
                    System.out.println(dis.readUTF());
                } catch (Exception e) {
                    System.out.println(socket.getRemoteSocketAddress() + " closed");
                    break;
                }
            }
            // 释放资源
            dis.close();
            socket.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
````