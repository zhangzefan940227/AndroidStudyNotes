# I/O 系统

# 1  输入输出
## 1.1  输入（Input）
<font style="color:rgb(0, 0, 0);">　　输入是指可以让程序从外部系统获得数据（核心含义是“</font>**<font style="color:rgb(255, 0, 0);">读</font>**<font style="color:rgb(0, 0, 0);">”，读取外部数据）。</font>

<font style="color:rgb(0, 0, 0);">　　例如：</font>

+ <font style="color:rgb(0, 0, 0);">读取硬盘上的文件内容到程序。播放器打开一个视频文件。</font>
+ <font style="color:rgb(0, 0, 0);">读物网络上某个位置的内容到程序。浏览器输入网之后，打开该网址对应的网页内容。</font>
+ <font style="color:rgb(0, 0, 0);">读取数据库系统的数据到程序。</font>

## 1.2  输入（Output）
<font style="color:rgb(0, 0, 0);">　　输出是指程序输出数据给外部系统从而可以操作外部系统（核心含义是“</font>**<font style="color:rgb(255, 0, 0);">写</font>**<font style="color:rgb(0, 0, 0);">”，将数据写出到外部系统）。</font>

<font style="color:rgb(0, 0, 0);">　　例如：</font>

+ <font style="color:rgb(0, 0, 0);">将数据写到硬盘中。编辑一个word文档后，将内容写到硬盘上进行保存。</font>
+ <font style="color:rgb(0, 0, 0);">将数据写到数据库中。注册一个网站会员，实际就是后台程序向数据库写入一条记录。</font>

## 1.3  数据源
<font style="color:rgb(0, 0, 0);">　　数据源分为：源设备、目标设备。</font>

+ <font style="color:rgb(0, 0, 0);">源设备：为程序提供数据，一般对应输入流。</font>
+ <font style="color:rgb(0, 0, 0);">目标设备：程序数据的目的地，一半对应输出流。</font>

![1619428208563-a5b4e7de-869f-4467-b0a0-511618265767.png](4%20Java/img/aqBXy9ySSNoEO2A5/1619428208563-a5b4e7de-869f-4467-b0a0-511618265767-038081.png)

# 2  I/O流
## 2.1  按流的方向分类
+ <font style="color:rgb(0, 0, 0);">输入流：数据源到程序。</font>
+ <font style="color:rgb(0, 0, 0);">输出流：程序到目的地。</font>

## 2.2  按处理的数据单元分类
+ <font style="color:rgb(0, 0, 0);">字节流：以字节为单位获取数据。命名上以 Stream 结尾的流一般是字节流。</font>
+ <font style="color:rgb(0, 0, 0);">字符流：以字符为单位以字符为单位获取数据。命名上以 Reader/Writer 结尾的流一般是字符流。</font>

## 2.3  按处理对象不同分类
+ <font style="color:rgb(0, 0, 0);">节点流：可以直接从数据源或目的地读写数据。</font>
+ <font style="color:rgb(0, 0, 0);">处理流：不直接连接到数据源或目的地。通过对其他流的处理提高程序性能，也叫包装流。</font>
    - <font style="color:rgb(0, 0, 0);">处理流又分为：缓冲流、转换流、数据流。</font>

<font style="color:rgb(0, 0, 0);">　　（个人理解：实际上就是对已形成的流进行了包装设计。）</font>

# 3  常用类
![1619428222615-67ff5abf-b2c2-498e-8b53-3c5016a8105b.png](4%20Java/img/aqBXy9ySSNoEO2A5/1619428222615-67ff5abf-b2c2-498e-8b53-3c5016a8105b-294631.png)

**<font style="color:rgb(0, 0, 0);">1. InputStream / OutputStream：</font>**

<font style="color:rgb(0, 0, 0);">　　字节流的抽象类。</font>

**<font style="color:rgb(0, 0, 0);">2. Reader / Writer：</font>**

<font style="color:rgb(0, 0, 0);">　　字符流的抽象类。</font>

**<font style="color:rgb(0, 0, 0);">3. FileInputStream / FileOutputStream：</font>**

<font style="color:rgb(0, 0, 0);">　　节点流：以字节为单位直接操作“文件”。</font>

**<font style="color:rgb(0, 0, 0);">【例3-1】</font>**<font style="color:rgb(0, 0, 0);">使用 FileInputStream 读取数据</font>

<font style="color:rgb(0, 0, 0);">　　首先在该工程的FileTest目录下创建一个文本文件“fan.txt”，在文件中输入内容 “Hello World”。</font>

```java
package com.ZZF.io;

import java.io.*;

public class FISDemo {
    public static void main(String[] args) {
        int temp;                  //定义一个int类型的变量temp，记住每次读取的一个字节
        FileInputStream fis = null;
        try {
            fis = new FileInputStream("FileTest/fan.txt");
            //创建一个文件字节输入流
            while (true) {
                temp = fis.read();     //变量temp记住读取的每一个字节
                if (temp == -1) {
                    break;          //如果读取的字节为-1，跳出while循环
                }
                System.out.print(temp + "\t");     //输出temp
            }
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try {
                if (fis != null) 
                    fis.close();    //关闭
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

<font style="color:rgb(0, 0, 0);">运行结果如下图所示：</font>

![1619428326020-c0e3fca5-1083-48bc-9573-982494edf1b2.png](4%20Java/img/aqBXy9ySSNoEO2A5/1619428326020-c0e3fca5-1083-48bc-9573-982494edf1b2-470514.png)

<font style="color:rgb(0, 0, 0);">　　这个例子中，创建的字节流 FileInputStream 通过 read() 方法将指定目录文件 “FileTest\fan.txt” 中的数据读取并打印。之所以输出数字是因为硬盘上的文件是以字节的形式存在的。需要注意的是，“Hello World” 中，空格也占一个字节，其对应的ASCII码值是32。一旦遇到 IO 异常，IO 流的 close() 方法将无法得到执行，流对象所占用的系统资源将得不到释放，因此，为了保证 IO 流的 close() 方法必须执行，通常将关闭流的操作写在 finally 代码块中。</font>



**<font style="color:rgb(0, 0, 0);">【例3-2】</font>**<font style="color:rgb(0, 0, 0);">使用FileOutputStream输出数据</font>

```java
package com.ZZF.io;

import java.io.*;

public class FOSDemo {
    public static void main(String[] args) {
        FileOutputStream fos = null;
        String str = "张小凡的博客";
        byte[] b = str.getBytes();
        try {
            fos = new FileOutputStream("FileTest/fan1.txt");
            for (int i = 0; i < b.length; i++) {
                fos.write(b[i]);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (fos != null)
                    fos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

<font style="color:rgb(0, 0, 0);">程序运行后，会在指定目录 “FileTest” 下生成一个新的文本文件 “fan1.txt”，打开文件会看到如下图所示的内容：</font>

![1619428347039-ad4c4155-109d-4546-baa0-f8ab3f498c80.png](4%20Java/img/aqBXy9ySSNoEO2A5/1619428347039-ad4c4155-109d-4546-baa0-f8ab3f498c80-590885.png)

<font style="color:rgb(0, 0, 0);">　　需要注意的是，如果通过 FileOutputStream 向一个已经存在的文件中写入数据，那么该文件中的数据首先会被清空，再写入新数据。若希望在已存在的文件内容之后追加新内容，则可使用 FileOutputStream 构造函数 FileOutputStream(String fileName, boolean append) 来创建文件输出流对象，并把 append 参数值设为true。对例3-2 中关键代码进行如下修改：</font>

```java
fos = new FileOutputStream("FileTest/fan1.txt", true);
```

**<font style="color:rgb(0, 0, 0);">4. ByteArrayInputStream / ByteArrayOutputStream：</font>**

<font style="color:rgb(0, 0, 0);">　　节点流：以字节为单位直接操作“字节数组对象”。</font>

**<font style="color:rgb(0, 0, 0);">【例3-3】</font>**<font style="color:rgb(0, 0, 0);">简单测试 ByteArrayInputStream 的使用</font>

```java
package com.ZZF.io;

import java.io.*;

public class TestByteArray {
    public static void main(String[] args) {
        byte[] b = "Hello World".getBytes();    //将字符串转换成字节数组
        test(b);
    }

    public static void test(byte[] bytes) {
        ByteArrayInputStream bais = null;
        StringBuilder sb = new StringBuilder();
        int temp = 0;
        int num = 0;
        //该构造方法的参数是一个字节数组，这个字节数组就是数据源
        try {
            bais = new ByteArrayInputStream(bytes);
            while ((temp = bais.read()) != -1) {
                sb.append((char) temp);
                num++;
            }
            System.out.println(sb);
            System.out.println("读取的字节数：" + num);
        } finally {
            try {
                if (bais != null) {
                    bais.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }

        }
    }
}
```

<font style="color:rgb(0, 0, 0);">输入结果如下图所示：</font>

![1619428421996-b810bbb7-584a-492e-94ed-21b513fc103a.png](4%20Java/img/aqBXy9ySSNoEO2A5/1619428421996-b810bbb7-584a-492e-94ed-21b513fc103a-813645.png)

<font style="color:rgb(0, 0, 0);">　　ByteArrayInputStream 和 ByteArrayOutputStream 经常用在需要流和数组之间转化的情况。</font>

<font style="color:rgb(0, 0, 0);">　　其实，FileInputStream 是把文件当做数据源，ByteArrayInputStream 则是把内存中的 “某个字节数组对象” 当做数据源。</font>

**<font style="color:rgb(0, 0, 0);">5. ObjectInputStream / ObjectOutputStream：</font>**

<font style="color:rgb(0, 0, 0);">　　处理流：以字节为单位直接操作“对象”。</font>

**<font style="color:rgb(0, 0, 0);">6. DataInputStream / DataOutputStream：</font>**

<font style="color:rgb(0, 0, 0);">　　处理流：以字节为单位直接操作“基本数据类型与字符串类型”。</font>

**<font style="color:rgb(0, 0, 0);">【例3-4】</font>**<font style="color:rgb(0, 0, 0);">DataInputStream 和 DataOutputStream 的使用</font>

```java
package com.ZZF.io;

import java.io.*;

public class TestDataStream {
    public static void main(String[] args) {
        DataOutputStream dos = null;
        DataInputStream dis = null;
        FileOutputStream fos = null;
        FileInputStream fis = null;

        try {
            fis = new FileInputStream("FileTest/fan1.txt");
            fos = new FileOutputStream("FileTest/fan1.txt");
            //使用数据流对缓冲流进行包装，新增缓冲功能
            dos = new DataOutputStream(new BufferedOutputStream(fos));
            dis = new DataInputStream(new BufferedInputStream(fis));
            //将如下数据写入到文件中
            dos.writeChar('a');
            dos.writeInt(1);
            dos.writeUTF("张小凡的博客");
            //手动刷新缓冲区，将流中的数据写入到文件中
            dos.flush();
            //直接读取数据：读取的顺序要与写入的顺序一致，否则不能正确读取数据。
            System.out.println("char: " + dis.readChar());
            System.out.println("int: " + dis.readInt());
            System.out.println("String: " + dis.readUTF());
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if(dos!=null){
                    dos.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if(dis!=null){
                    dis.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if(fos!=null){
                    fos.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if(fis!=null){
                    fis.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

<font style="color:rgb(0, 0, 0);">运行结果如下图所示：</font>

![1619428454254-8ab54ebe-3d4e-43a1-a6b8-f61c53601ffa.png](4%20Java/img/aqBXy9ySSNoEO2A5/1619428454254-8ab54ebe-3d4e-43a1-a6b8-f61c53601ffa-187347.png)

<font style="color:rgb(0, 0, 0);">　　存在的问题是，打开所操作的文件时会发现，文本中出现乱码或者不显示 int 类型的数字1。网上进行查阅之后，说是writeInt底层使用的是位操作，实际操作的是字符。可以正确读取就可以了。</font>

**<font style="color:rgb(0, 0, 0);">7. FileReader / FileWriter</font>**

<font style="color:rgb(0, 0, 0);">　　节点流：以字符为单位直接操作“文本文件”(注意：只能读写文本文件)。</font>

**<font style="color:rgb(0, 0, 0);">【例3-5】</font>**<font style="color:rgb(0, 0, 0);">使用FileReader读取文件</font>

```java
package com.ZZF.io;

import java.io.*;

public class FRDemo {
    public static void main(String[] args) {
        FileReader fr = null;       //创建一个FileReader对象来读取文件中的字符
        int temp;                   //定义一个变量用于记录
        try {
            fr = new FileReader("FileTest/fan.txt");
            while ((temp = fr.read()) != -1) {
                System.out.print((char)temp);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try{
                if (fr != null) {
                    fr.close();     //关闭文件读取流，释放资源         
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

<font style="color:rgb(0, 0, 0);">运行结果如下图所示：</font>

![1619428477307-9f2762be-b7ab-4687-8e84-58f55592d8b8.png](4%20Java/img/aqBXy9ySSNoEO2A5/1619428477307-9f2762be-b7ab-4687-8e84-58f55592d8b8-014811.png)

<font style="color:rgb(0, 0, 0);">　　字符输入流的 read() 方法返回的是int类型的值，要想获得字符就需要进行强制类型转换。</font>

**<font style="color:rgb(0, 0, 0);">【例3-6】</font>**<font style="color:rgb(0, 0, 0);">使用FileWriter将字符写入文件</font>

<font style="color:rgb(0, 0, 0);">　　在已有的 “fan.txt” 文件中换行并写入 “你好，世界！”。</font>

```java
package com.ZZF.io;

import java.io.*;

public class FWDemo {
    public static void main(String[] args) {
        FileWriter fw = null;       //创建一个FileWriter对象用于向文件中写入数据
        String str = "你好，世界！";

        try {
            fw = new FileWriter("FileTest/fan.txt",true);
            fw.write("\r\n");
            fw.write(str);

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try{
                if (fw != null) {
                    fw.close();     //关闭写入流，释放资源
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

<font style="color:rgb(0, 0, 0);">打开 “fan.txt” 文件会看到如下图所示内容：</font>

![1619428497139-9acb2ac9-2ed8-4477-a0f2-306a91e5c59b.png](4%20Java/img/aqBXy9ySSNoEO2A5/1619428497139-9acb2ac9-2ed8-4477-a0f2-306a91e5c59b-747342.png)

**<font style="color:rgb(0, 0, 0);">8. BufferedReader / BufferedWriter</font>**

<font style="color:rgb(0, 0, 0);">        处理流：将Reader/Writer对象进行包装，增加缓存功能，提高读写效率。</font>

**<font style="color:rgb(0, 0, 0);">【例3-7】</font>**<font style="color:rgb(0, 0, 0);">使用 BufferedReader/BufferedWriter对文件内容进行复制</font>

```java
package com.ZZF.io;

import java.io.*;

public class BRWDemo {
    public static void main(String[] args) {
        // 注：处理文本文件时，实际开发中可以用如下写法，简单高效！！
        FileReader fr = null;
        FileWriter fw = null;
        BufferedReader br = null;
        BufferedWriter bw = null;
        String temp;
        try {
            fr = new FileReader("FileTest/src.txt");
            fw = new FileWriter("FileTest/des.txt");
            //使用缓冲字符流进行包装
            br = new BufferedReader(fr);
            bw = new BufferedWriter(fw);
            //BufferedReader提供了更方便的readLine()方法，直接按行读取文本
            //br.readLine()方法的返回值是一个字符串对象，即文本中的一行内容
            while ((temp = br.readLine()) != null) {
                //将读取的一行字符串写入文件中
                bw.write(temp);
                //下次写入之前先换行，否则会在上一行后边继续追加，而不是另起一行
                bw.newLine();
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (bw != null) {
                    bw.close();
                }
            } catch (IOException e1) {
                e1.printStackTrace();
            }
            try {
                if (br != null) {
                    br.close();
                }
            } catch (IOException e1) {
                e1.printStackTrace();
            }
            try {
                if (fw != null) {
                    fw.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if (fr != null) {
                    fr.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

**<font style="color:rgb(0, 0, 0);">注意</font>**

+ <font style="color:rgb(0, 0, 0);">readLine()方法是BufferedReader特有的方法，可以对文本文件进行更加方便的读取操作。</font>
+ <font style="color:rgb(0, 0, 0);">写入一行后要记得使用newLine()方法换行。</font>

**<font style="color:rgb(0, 0, 0);">9. BufferedInputStream / BufferedOutputStream</font>**

<font style="color:rgb(0, 0, 0);">        处理流：将InputStream/OutputStream对象进行包装，增加缓存功能，提高读写效率。</font>

**<font style="color:rgb(0, 0, 0);">【例3-8】</font>**<font style="color:rgb(0, 0, 0);">使用 BufferedInputStream/BufferedOutputStream 实现文件的复制</font>

```java
package com.ZZF.io;

import java.io.*;

public class BIOSDemo {
    public static void main(String[] args) {
        FileInputStream fis = null;
        BufferedInputStream bis = null;
        FileOutputStream fos = null;
        BufferedOutputStream bos = null;
        int temp;　　　　　　
        try {
            fis = new FileInputStream("FileTest/src.txt");
            bis = new BufferedInputStream(fis);
            fos = new FileOutputStream("FileTest/des.txt");
            bos = new BufferedOutputStream(fos);
            while ((temp = bis.read()) != -1) {
                bos.write(temp);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //注意：增加处理流后，注意流的关闭顺序“后开的先关闭”
            try {
                if (bos != null) { bos.close(); }
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if (bis != null) { bis.close(); }
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if (fos != null) { fos.close(); }
            } catch (IOException e) {
                e.printStackTrace();
            }            try {
                if (fis != null) { fis.close(); }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

**<font style="color:rgb(0, 0, 0);">10. InputStreamReader / OutputStreamWriter</font>**

<font style="color:rgb(0, 0, 0);">        处理流：将字节流对象转化成字符流对象。</font>

<font style="color:rgb(0, 0, 0);">　　System.in是字节流对象，代表键盘的输入，如果我们想按行接收用户的输入时，就必须用到缓冲字符流BufferedReader特有的方法readLine()，但是经过观察会发现在创建BufferedReader的构造方法的参数必须是一个Reader对象，这时候我们的转换流InputStreamReader就派上用场了。</font>

<font style="color:rgb(0, 0, 0);">       而System.out也是字节流对象，代表输出到显示器，按行读取用户的输入后，并且要将读取的一行字符串直接显示到控制台，就需要用到字符流的write(String str)方法，所以我们要使用OutputStreamWriter将字节流转化为字符流。</font>

**<font style="color:rgb(0, 0, 0);">11. PrintStream</font>**

<font style="color:rgb(0, 0, 0);">        处理流：将OutputStream进行包装，可以方便地输出字符，更加灵活。</font>

### **<font style="color:rgb(0, 0, 0);">装饰器模式</font>**
<font style="color:rgb(0, 0, 0);">　　装饰器模式可以动态的把新的职责添加到对象上，在扩展性方面比通过继承实现扩展更富有弹性。这里关键点是“动态”，也就是运行时；而继承在编译的时候已经确定了。</font>

<font style="color:rgb(0, 0, 0);">　　这种设计模式创建了一个装饰类，用来包装原有的类，并在保持类方法签名完整性的前提下，提供了额外的功能。</font>

![1619428529339-731da654-34ef-4aa3-8178-d8c99fbbc692.png](4%20Java/img/aqBXy9ySSNoEO2A5/1619428529339-731da654-34ef-4aa3-8178-d8c99fbbc692-435968.png)















> 更新: 2023-11-18 15:03:51  
> 原文: <https://www.yuque.com/zhangxiaofani4cu/xih3ez/hvc8sg>