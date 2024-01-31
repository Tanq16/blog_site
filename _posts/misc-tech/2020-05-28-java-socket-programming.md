---
title: Basics of Socket Programming in Java
date: 2020-05-28 12:00:00 -0000
categories: [Computers and Fun]
tags: [java,programming,network,sockets]
---

## Simple Client Program

```java
import java.util.*;
import java.net.*;
import java.io.*;

public class client
{
    public static void main(String[] args) throws Exception
    {
        Socket s = new Socket("localhost", 12345);
        DataInputStream input = new DataInputStream(s.getInputStream());
        DataOutputStream output = new DataOutputStream(s.getOutputStream());
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));

        boolean x = true;
        while(x)
        {
            System.out.println("enter to send ");
            String message = reader.readLine();
            if(message.equals("stop"))
                    x = false;
            output.writeUTF(message);
            String response = input.readUTF();
            System.out.println("server says --- " + response);
        }
        s.close();
    }
}
```

## Simple Server Program

```java
import java.net.*;
import java.util.*;
import java.io.*;

public class server
{
    public static void main(String[] args) throws Exception
    {
        ServerSocket ss = new ServerSocket(12345);
        Socket s = ss.accept();
        DataInputStream input = new DataInputStream(s.getInputStream());
        DataOutputStream output = new DataOutputStream(s.getOutputStream());
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));

        boolean x = true;
        while(x)
        {
            String message = input.readUTF();
            System.out.println("client says --- " + message);
            if(message.equals("stop"))
                    x = false;
            if(x)
                    System.out.println("enter to send ");
            String respond = x?reader.readLine():"END_SESSION_!";
            output.writeUTF(respond);
        }
        s.close();
        ss.close();
    }
}
```

## Multi-threaded Server

```java
import java.util.*;
import java.net.*;
import java.io.*;

public class multithreadserver implements Runnable
{
    Socket client;
    multithreadserver(Socket client)
    {
        this.client = client;
    }

    public static void main(String[] args) throws Exception
    {
        ServerSocket ss = new ServerSocket(12345);
        while(true)
        {
            Socket sock = ss.accept();
            new Thread(new multithreadserver(sock)).start();
        }
    }

    public void run()
    {
        try
        {
            DataInputStream input = new DataInputStream(client.getInputStream());
            DataOutputStream output = new DataOutputStream(client.getOutputStream());
            BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
            InetAddress addr = client.getInetAddress();
            boolean x = true;
            while(x)
            {
                String message = input.readUTF();
                System.out.println(addr.getHostAddress() + "says --- " + message);
                if(message.equals("stop"))
                        x = false;
                if(x)
                        System.out.println("enter to send to " + addr.getHostAddress());
                String respond = x?reader.readLine():"END_SESSION_!";
                output.writeUTF(respond);
            }
            client.close();
        }
        catch(IOException e)
        {
            e.printStackTrace();
        }
    }
}
```

## Banner Grabbing - Get Request

```java
import java.util.*;
import java.net.*;
import java.io.*;

public class get_request
{
    public static void main(String[] args) throws Exception
    {
        InetAddress addr = InetAddress.getByName("www.avajava.com");
        Socket client = new Socket(addr, 80);
        PrintWriter output = new PrintWriter(client.getOutputStream());
        BufferedReader input = new BufferedReader(new InputStreamReader(client.getInputStream()));
        output.println("GET / HTTP/1.1");
        output.println("Host: www.avajava.com");
        output.println();
        output.flush();
        String line;
        while((line = input.readLine())!=null)
        {
            System.out.println(line);
        }
    }
}
```

## Download Image using Get Request

```java
import java.util.*;
import java.net.*;
import java.io.*;

public class download_image
{
    public static void main(String[] args) throws Exception
    {
        InetAddress addr = InetAddress.getByName("www.bits-pilani.ac.in");
        Socket client = new Socket(addr, 80);
        PrintWriter output = new PrintWriter(client.getOutputStream());
        DataInputStream input = new DataInputStream(client.getInputStream());

        output.println("GET /Uploads/Campus/BITS_Dubai_campus_logo.gif HTTP/1.1");
        output.println("Host: www.bits-pilani.ac.in");
        output.println();
        output.flush();

        FileOutputStream img = new FileOutputStream(new File("img.gif"));
        byte[] rec = new byte[2048];
        boolean eohfound = false;
        int count;

        while((count=input.read(rec))!=-1)
        {
            int start = 0;
            if(!eohfound)
            {
                String yu = new String(rec, 0, count);
                int index = yu.indexOf("\\r\\n\\r\\n");
                if(index!=-1)
                {
                    start = index + 4;
                    count = count - index - 4;
                    eohfound = true;
                }
                else
                    count = 0;
            }
            img.write(rec, start, count);
        }
        img.close();
        input.close();
        output.close();
        client.close();
    }
}
```

## Banner Grabbing - HttpURLConnection

```java
import java.util.*;
import java.net.*;
import java.io.*;

public class httprequest
{
    public static void main(String[] args) throws Exception
    {
        String url = "<http://www.java2s.com>";
        URL object = new URL(url);
        HttpURLConnection connection = (HttpURLConnection)object.openConnection();
        connection.setRequestMethod("GET");
        BufferedReader in = new BufferedReader(new InputStreamReader(connection.getInputStream()));
        String line;
        while((line=in.readLine())!=null)
        {
            System.out.println(line);
            System.out.flush();
        }
    }
}
```

## Download Image using HttpURLConnection

```java
import java.util.*;
import java.net.*;
import java.io.*;

public class httprequest_download
{
    public static void main(String[] args) throws Exception
    {
        URL object = new URL("<http://bits-pilani.ac.in/Uploads/Campus/BITS_Dubai_campus_logo.gif>");
        HttpURLConnection connection = (HttpURLConnection)object.openConnection();
        connection.setRequestMethod("GET");
        FileOutputStream webpage = new FileOutputStream(new File("img.gif"));
        DataInputStream in = new DataInputStream(connection.getInputStream());
        int count;
        byte[] buffer = new byte[2048];
        while((count = in.read(buffer))!=-1)
        {
            webpage.write(buffer, 0, count);
            webpage.flush();
        }
        in.close();
        webpage.close();
    }
}
```
