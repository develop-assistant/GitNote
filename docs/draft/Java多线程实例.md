# Java多线程实例

## 1. 多线程下载

单文件多线程下载: 分段下载

```
import java.io.*;
import java.net.HttpURLConnection;
import java.net.URL;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;
import java.util.ArrayList;

public class DownLoader {
    String url;
    int num;

    public  void setUrl(String url) {
        this.url = url;
    }

    public void setNum(int num) {
        this.num = num;
    }
    
    public ArrayList getDataLeng(String url) throws Exception {
        ArrayList<Object> arr=new ArrayList<Object>();
        String []names=url.split("/");
        String name=names[names.length-1];
        URL urls= new URL(url);
        HttpURLConnection conn=(HttpURLConnection) urls.openConnection();
        int length=conn.getContentLength();
        arr.add(name);
        arr.add(length-1);
        return arr;
    }
    public int[] getindex(int length,int mun){
        int index=length/mun;
        int []a=new int[mun+1];
        for (int i = 0; i <=mun; i++) {
            if(i==(mun)){
                a[i]=length;
            }else {
                a[i]=index*i;
            }
        }
        return a;
    }
    public void mut_thread_donwloader(int mun,String url) throws Exception {
        setNum(mun);
        setUrl(url);
        ArrayList arr=getDataLeng(this.url);
        String name=(String)arr.get(0);
        int length=(int)arr.get(1);
        int []index=getindex(length,this.num);
        System.out.println("开始下载"+name);
        RandomAccessFile raf=new RandomAccessFile(name,"rw");
        raf.setLength(length);
        raf.close();
        for (int i = 0; i < this.num; i++) {
            new Thread(new thread(index[i]+1,index[i+1],this.url,name)).start();
        }
    }
    public static void main(String[] args) throws Exception {
           String url="http://c.hiphotos.baidu.com/image/pic/item/060828381f30e924aa20564a46086e061c95f7c1.jpg";
           DownLoader m= new DownLoader();
           m.mut_thread_donwloader(5,url); #这里我设置为5线程下载图片 基本秒下
    }
}

class thread implements Runnable{
    private  RandomAccessFile raf;
    private  int start,end;
    private  String url;
    private URL urls;
    private String path;
    int buf=1;
    int length;

    public thread(int start,int end,String url,String path) {
        this.path=path;
        this.start=start;
        this.end=end;
        this.url=url;
    }

    @Override
    public  void run() {
        if(start==1){
            start=0;
        }
        System.out.println(start+"...."+end);
        try {
            raf=new RandomAccessFile(path,"rwd");
            urls = new URL(url);
            HttpURLConnection conn=(HttpURLConnection) urls.openConnection();
            conn.setRequestProperty("Range","bytes="+ start + "-" + end);
            if(conn.getResponseCode()==206){
                InputStream is=conn.getInputStream();
                byte[]b=new byte[buf];
                FileChannel fc=new RandomAccessFile(path,"rw").getChannel();
                fc.position(start);
                ByteBuffer bb=ByteBuffer.allocateDirect(buf*1024);
                length=end-start;
                fc.lock(start,length,false);
                while (length-(b.length*1024)>0&&is.read(b)!=-1) {
                    length=length-b.length*1024;
                    bb.flip();
                    fc.write(bb.wrap(b));
                    bb.clear();
                }
                ByteBuffer bb2 = ByteBuffer.allocateDirect(length);
                while (is.read(b)!=-1) {
                    bb2.flip();
                    fc.write(bb2.wrap(b));
                    bb2.clear();
                }
                System.out.println(Thread.currentThread()+"下载结束");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```



## 2. 多线程socket

```
//因为是多线程，所以不在服务端直接做业务处理，而是在线程类里处理
public class NetServer {

    public static void go(){
        int PORT=7775;
        try {
            //指定端口专门处理这件事
            ServerSocket ss = new ServerSocket(PORT);
            System.out.println("服务器已启动");
            //死循环，目的是一直保持监听状态
            while (true) {
                //开启监听
                Socket s = ss.accept();
                //将连接的客户端交给一个线程去处理
                Thread t = new Thread(new ClentThread(s));
                //开启线程
                t.start();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    public static void main(String[] args){
        go();
    }
}
```

```
public class ClentThread implements Runnable {
    private Socket socket = null;
    public ClentThread(Socket s) {
        this.socket = s;
    }

    @Override
    public void run() {
        try {
            //接收客户端消息
            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            //按行读取客户端的消息内容，并且拼接在一起
            StringBuffer sb=new StringBuffer();
            String tmp="";
            while((tmp=in.readLine())!=null){
                sb.append(tmp);
            }
            System.out.println("客户端发送的是："+sb.toString());
            //转码
            String rc=new String(sb.toString().getBytes(),"UTF-8");
            
            //将消息返回给客户端
            PrintWriter out = new PrintWriter(new BufferedWriter(new OutputStreamWriter(socket.getOutputStream())), true);
            out.print(rc);
            //该关的都关掉
            out.close();
            in.close();
        } catch (Exception e) {
            System.out.println(e.toString());
        } finally {
            try {
                socket.close();
            } catch (Exception e) {
                System.out.println(e.toString());
            }
        }
    }
}
```



## 3. 多线程购票



## 4. 多线程消费

