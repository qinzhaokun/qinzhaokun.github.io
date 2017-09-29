---
title: Java IO
date: 2017-09-29 18:54:25
tags: Java
categories: Java
---

IO流技术

## 简介

I/O类库中使用“流”这个抽象概念。Java对设备中数据的操作是通过流的方式。表示任何有能力产出数据的数据源对象，或者是有能力接受数据的接收端对象。“流”屏蔽了实际的I/O设备中处理数据的细节。IO流用来处理设备之间的数据传输。设备是指硬盘、内存、键盘录入、网络等。

## 分类

流按操作数据类型的不同分为两种：**字节流**与**字符流**。
流按流向分为：输入流，输出流

### 字节流

#### InputStream

InputStream：字节输入流基类，抽象类是表示字节输入流的所有类的超类。

```
常用方法：  
    // 从输入流中读取数据的下一个字节  
    abstract int read()  
    // 从输入流中读取一定数量的字节，并将其存储在缓冲区数组 b中  
    int read(byte[] b)  
    // 将输入流中最多 len 个数据字节读入 byte 数组  
    int read(byte[] b, int off, int len)  
    // 跳过和丢弃此输入流中数据的 n个字节  
    long skip(long n)  
    // 关闭此输入流并释放与该流关联的所有系统资源  
    void close() 
```

#### OutputStream

字节输出流基类，抽象类是表示输出字节流的所有类的超类。

```
常用方法：  
    // 将 b.length 个字节从指定的 byte 数组写入此输出流  
    void write(byte[] b)  
    // 将指定 byte 数组中从偏移量 off 开始的 len 个字节写入此输出流  
    void write(byte[] b, int off, int len)  
    // 将指定的字节写入此输出流  
    abstract void write(int b)  
    // 关闭此输出流并释放与此流有关的所有系统资源  
    void close()  
    // 刷新此输出流并强制写出所有缓冲的输出字节  
    void flush() 
```
#### FileInputStream

字节文件输入流，从文件系统中的某个文件中获得输入字节，用于读取诸如图像数据之类的原始字节流。

```
构造方法：  
    // 通过打开一个到实际文件的连接来创建一个FileInputStream，该文件通过文件系统中的File对象file指定  
    FileInputStream(File file)  
    // 通过打开一个到实际文件的连接来创建一个FileInputStream，该文件通过文件系统中的路径name指定  
    FileInputStream(String name)  
 常用方法：覆盖和重写了父类的的常用方法。  
```

示例：
```
// 读取f盘下该文件f://hell/test.txt  
//构造方法1  
InputStream inputStream = new FileInputStream(new File("f://hello//test.txt"));  
int i = 0;  
//一次读取一个字节  
while ((i = inputStream.read()) != -1) {  
    // System.out.print(i + " ");// 65 66 67 68  
    //为什么会输出65 66 67 68？因为字符在底层存储的时候就是存储的数值。即字符对应的ASCII码。  
    System.out.print((char) i + " ");// A B C D  
}  
//关闭IO流  
inputStream.close();  
```

#### FileOutputStream

字节文件输出流是用于将数据写入到File，从程序中写入到其他位置。
```
构造方法：  
   // 创建一个向指定File对象表示的文件中写入数据的文件输出流  
   FileOutputStream(File file)  
   // 创建一个向指定File对象表示的文件中写入数据的文件输出流  
   FileOutputStream(File file, boolean append)  
   // 创建一个向具有指定名称的文件中写入数据的输出文件流  
   FileOutputStream(String name)  
   // 创建一个向具有指定name的文件中写入数据的输出文件流  
   FileOutputStream(String name, boolean append)  
常用方法：覆盖和重写了父类的的常用方法。 
```

示例：
```
OutputStream outputStream = new FileOutputStream(new File("test.txt"));  
        // 写出数据  
        outputStream.write("ABCD".getBytes());  
        // 关闭IO流  
        outputStream.close();  
        // 内容追加写入  
        OutputStream outputStream2 = new FileOutputStream("test.txt", true);  
        // 输出换行符  
        outputStream2.write("\r\n".getBytes());  
        // 输出追加内容  
        outputStream2.write("hello".getBytes());  
        // 关闭IO流  
        outputStream2.close();
```

出的目的地文件不存在，则会自动创建，不指定盘符的话，默认创建在项目目录下;输出换行符时一定要写\r\n不能只写\n,因为不同文本编辑器对换行符的识别存在差异性

#### BufferedInputStream

字节缓冲流，字节缓冲输入流，提高了读取效率
```
构造方法：  
    // 创建一个 BufferedInputStream并保存其参数，即输入流in，以便将来使用。  
    BufferedInputStream(InputStream in)  
    // 创建具有指定缓冲区大小的 BufferedInputStream并保存其参数，即输入流in以便将来使用  
    BufferedInputStream(InputStream in, int size)  
```

```
InputStream in = new FileInputStream("test.txt");  
       // 字节缓存流  
       BufferedInputStream bis = new BufferedInputStream(in);  
       byte[] bs = new byte[20];  
       int len = 0;  
       while ((len = bis.read(bs)) != -1) {  
           System.out.print(new String(bs, 0, len));  
           // ABCD  
           // hello  
       }  
       // 关闭流  
       bis.close();  
```

**注意，构造方法中传入的是一个InputStream对象，这里是使用了装饰器模式**

#### BufferedOutputStream

字节缓冲输出流，提高了写出效率。同理，使用了**装饰器模式**

```
构造方法：  
    // 创建一个新的缓冲输出流，以将数据写入指定的底层输出流  
    BufferedOutputStream(OutputStream out)  
    // 创建一个新的缓冲输出流，以将具有指定缓冲区大小的数据写入指定的底层输出流  
    BufferedOutputStream(OutputStream out, int size)  
    常用方法：  
    // 将指定 byte 数组中从偏移量 off 开始的 len 个字节写入此缓冲的输出流  
    void write(byte[] b, int off, int len)  
    // 将指定的字节写入此缓冲的输出流  
    void write(int b)  
    // 刷新此缓冲的输出流  
    void flush()  
```
```
BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("test.txt", true));  
       // 输出换行符  
       bos.write("\r\n".getBytes());  
       // 输出内容  
       bos.write("Hello Android".getBytes());  
       // 刷新此缓冲的输出流  
       bos.flush();  
       // 关闭流  
       bos.close();  
```

### 字符流

#### Reader

读取字符流的抽象类.
```
常用方法：  
   // 读取单个字符  
   int read()  
   // 将字符读入数组  
   int read(char[] cbuf)  
   // 将字符读入数组的某一部分  
   abstract int read(char[] cbuf, int off, int len)  
   // 跳过字符  
   long skip(long n)  
   // 关闭该流并释放与之关联的所有资源  
   abstract void close()  
```

#### Writer
写入字符流的抽象类.
```
常用方法：  
    // 写入字符数组  
     void write(char[] cbuf)  
    // 写入字符数组的某一部分  
    abstract void write(char[] cbuf, int off, int len)  
    // 写入单个字符  
    void write(int c)  
    // 写入字符串  
    void write(String str)  
    // 写入字符串的某一部分  
    void write(String str, int off, int len)  
    // 将指定字符添加到此 writer  
    Writer append(char c)  
    // 将指定字符序列添加到此 writer  
    Writer append(CharSequence csq)  
    // 将指定字符序列的子序列添加到此 writer.Appendable  
    Writer append(CharSequence csq, int start, int end)  
    // 关闭此流，但要先刷新它  
    abstract void close()  
    // 刷新该流的缓冲  
    abstract void flush()  
```

### InputStreamReader

字节流转字符流，它使用的字符集可以由名称指定或显式给定，否则将接受平台默认的字符集。它依赖于InputStream，**装饰器模式**
```
构造方法：  
   // 创建一个使用默认字符集的 InputStreamReader  
   InputStreamReader(InputStream in)  
   // 创建使用给定字符集的 InputStreamReader  
   InputStreamReader(InputStream in, Charset cs)  
   // 创建使用给定字符集解码器的 InputStreamReader  
   InputStreamReader(InputStream in, CharsetDecoder dec)  
   // 创建使用指定字符集的 InputStreamReader  
   InputStreamReader(InputStream in, String charsetName)  
特有方法：  
   //返回此流使用的字符编码的名称   
   String getEncoding()   
```
```
//使用默认编码          
 InputStreamReader reader = new InputStreamReader(new FileInputStream("test.txt"));  
 int len;  
 while ((len = reader.read()) != -1) {  
     System.out.print((char) len);
 }  
 reader.close();  
  //指定编码   
 InputStreamReader reader = new InputStreamReader(new FileInputStream("test.txt"),"utf-8");  
 int len;  
 while ((len = reader.read()) != -1) {  
     System.out.print((char) len);
 }  
 reader.close();  
```

#### OutputStreamWriter

字节流转字符流。**装饰器模式**
```
构造方法：  
   // 创建使用默认字符编码的 OutputStreamWriter  
   OutputStreamWriter(OutputStream out)  
   // 创建使用给定字符集的 OutputStreamWriter  
   OutputStreamWriter(OutputStream out, Charset cs)  
   // 创建使用给定字符集编码器的 OutputStreamWriter  
   OutputStreamWriter(OutputStream out, CharsetEncoder enc)  
   // 创建使用指定字符集的 OutputStreamWriter  
   OutputStreamWriter(OutputStream out, String charsetName)  
特有方法：  
   //返回此流使用的字符编码的名称   
   String getEncoding()   
```

#### BufferedReader

字符缓冲流，从字符输入流中读取文本，缓冲各个字符，从而实现字符、数组和行的高效读取
```
构造方法：  
   // 创建一个使用默认大小输入缓冲区的缓冲字符输入流  
   BufferedReader(Reader in)  
   // 创建一个使用指定大小输入缓冲区的缓冲字符输入流  
   BufferedReader(Reader in, int sz)  
特有方法：  
   // 读取一个文本行  
   String readLine()  
```
```
//生成字符缓冲流对象  
  BufferedReader reader = new BufferedReader(new InputStreamReader(new FileInputStream("test.txt")));  
  String str;  
  //一次性读取一行  
  while ((str = reader.readLine()) != null) {  
      System.out.println(str);// 爱生活，爱Android  
  }  
  //关闭流  
  reader.close();  
```
`BufferedReader reader = new BufferedReader(new InputStreamReader(new FileInputStream("test.txt"))); `可以看出用了**三层装饰器**

#### BufferedWriter

同理，字符缓冲流，将文本写入字符输出流，缓冲各个字符，从而提供单个字符、数组和字符串的高效写入。
```
构造方法：  
   // 创建一个使用默认大小输出缓冲区的缓冲字符输出流  
   BufferedWriter(Writer out)  
   // 创建一个使用给定大小输出缓冲区的新缓冲字符输出流  
   BufferedWriter(Writer out, int sz)  
特有方法：  
   // 写入一个行分隔符  
   void newLine()   
```

#### FileReader、FileWriter

FileReader：InputStreamReader类的直接子类，用来读取字符文件的便捷类，使用默认字符编码。  
FileWriter：OutputStreamWriter类的直接子类，用来写入字符文件的便捷类，使用默认字符编码。

他们封装过了，无需显示使用装饰器模式构造
```
FileReader fileReader = new FileReader(fileName);
FileReader fileReader = new FileReader(new File(fileName));
```
可以让BufferedReader装饰它。
```
BufferedReader br = new BufferedReader(new FileReader(fileName)); 
```

**这是读取文件的常用方式**

## 总结

I/O分为操作字节和字符的，基类分别是InputStream/OutStream, Reader/Writer, 抽象类，基本没有实现不能自己用

字节流：

1. `FileInputStream/FileOutStream` 可以单独读写文件的byte

2. `BufferedInputStream/BufferedOutputStream`使用了缓冲区，并且使用了**装饰器模式**，装饰了一个`InputStream/OutputStream`

字符流：
1. 不能直接使用，一定要装饰字节流`InputStream/OutStream`

2. `InputStreamReader/OutputStreamReader`要装饰一个`InputStream/OutputStream`，这和`BufferedInputStream/BufferedOutputStream`一样。

3. `BufferedReader/BufferedWriter`增加了缓冲区，提高效率，但同样使用了**装饰器模式**，但是它装饰的是`Reader/Writer`.因此，使用它时，看起来被装饰了三层。`BufferedReader reader = new BufferedReader(new InputStreamReader(new FileInputStream("test.txt"))); `

4. `FileReader/FileWriter`继承自`InputStreamReader/OutputStreamReader`所以他们是平级的。
