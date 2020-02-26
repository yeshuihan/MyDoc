Client.java

```java
import java.net.*;
import java.io.*;
import java.util.Scanner;
 
public class Client {
 
    Thread read;
    Thread write;
    Socket s;
    public static void main(String[] args) {
        Client client=new Client();
        client.startClient();
    }
 
    public void startClient() {
        try {
            s=new Socket("127.0.0.1",65534);
            sendMessage("d");
            read=new Thread(new Runnable() {
                public void run() {
                    try {
                        while(true) {
                            InputStream is= s.getInputStream();
                            InputStreamReader isr =new InputStreamReader(is);
                            BufferedReader br =new BufferedReader(isr);
                            String info =null;
                            info=br.readLine();
 
                            if(info.indexOf("#")!=-1) {
                                String name=info.substring(0,info.indexOf("#"));
                                System.out.println(name+":"+info.substring(info.indexOf("#")+1));
                            }else{
                                System.out.println("Service:"+info);
                            }
                             
                        }
                    } catch(Exception e) {
                        e.printStackTrace();
                    }
 
                }
            });
            write=new Thread(new Runnable() {
                public void run() {
 
                    while(true) {
                        try {
                            BufferedReader buff=new BufferedReader(new InputStreamReader(System.in));
                            String str=buff.readLine().toString();
                            sendMessage(str);
                        } catch(Exception e) {
                            e.printStackTrace();
                        }
                    }
 
 
                }
            });
 
            write.start();
            read.start();
 
        } catch(Exception e) {
            e.printStackTrace();
        }
    }
 
    public void sendMessage(String str) {
        try {
            OutputStream out=s.getOutputStream();
            BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(s.getOutputStream()));
            bw.write(str);
            bw.newLine();
            bw.flush();
        } catch(Exception e) {
            e.printStackTrace();
        }
 
    }
 
}
```

Service.java

```java
import java.net.*;
import java.io.*;
import java.util.*;
 
public class Service {
 
    public static int socketNum=0;
     
    public Map<String,Socket> allSocket;
 
 
    public static void main(String[] args) {
 
        Service service=new Service();
        service.startService();
    }
    /**/
    public void startService() {
        ServerSocket serviceListener=null;
        allSocket=new HashMap<String,Socket>();
        try {
            serviceListener=new ServerSocket(65534);
 
            new Thread(new Runnable() {
                public void run() {
                    while(true) {
                        System.out.println("当前在线人数："+socketNum);
                        try {
                            Thread.sleep(2000);
                        } catch(Exception e) {
                            e.printStackTrace();
                        }
                    }
 
 
                }
            }).start();
 
            while(true) {
                Socket ls=serviceListener.accept();
             
                 
                Thread thread=new Thread(new Handle(ls));
                thread.start();
            }
 
        } catch(Exception e) {
            e.printStackTrace();
            try {
                if(serviceListener!=null) {
                    serviceListener.close();
                    System.out.println("Service over");
                }
            } catch(Exception e1) {
                e1.printStackTrace();
            }
        }
    }
 
 
 
 
 
 
 
 
    class Handle implements Runnable {
        Socket socket;
        String mName="";
        final int id=socketNum++;
        public Handle(Socket soc) {
            socket=soc;
        }
        public void run() {
            mName=getMessage();
            allSocket.put(mName,socket);
            System.out.println("mName:"+mName+" port:"+socket.getPort());
            try {
                sendMessage(socket,"你好！"+mName);
                while(true) {
                    String info=getMessage();
                    System.out.println(mName+"说："+info);
                    if(info.indexOf("#")!=-1) {
                        String name=info.substring(0,info.indexOf("#"));
                        if(!allSocket.containsKey(name)) {
             
                            sendMessage(socket,name+"不在线");
                        } else {
                             
                            sendMessage(allSocket.get(name),mName+info.substring(info.indexOf("#")));
                        }
                    }
                }
            } catch(Exception e) {
                e.printStackTrace();
                try {
                    if(socket!=null) {
                        socket.close();
                        System.out.println("客户端"+socket.getPort()+" over");
                        allSocket.remove(mName);
                        socketNum--;
                    }
                } catch(Exception e1) {
                    e1.printStackTrace();
                }
            }
        }
 
        public void sendMessage(Socket s,String str) {
            try {
 
                System.out.println("send客户端"+s.getPort());
                OutputStream out=s.getOutputStream();
                BufferedWriter bw=new BufferedWriter(new OutputStreamWriter(out));
                bw.write(str);
                bw.newLine();
                bw.flush();
            } catch(Exception e) {
                e.printStackTrace();
            }
 
        }
 
 
        public String getMessage() {
            try{
                InputStream is= socket.getInputStream();
                InputStreamReader isr =new InputStreamReader(is);
                BufferedReader br =new BufferedReader(isr);
                String info =null;
                info=br.readLine();
 
                return info;
            } catch(Exception e) {
                e.printStackTrace();
                 
            }
             
            return null;
        }
    }
 
 
 
}
```

