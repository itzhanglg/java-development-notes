### 一.File类
java.io.File类：**文件和文件目录**路径的抽象表示形式，与平台无关
1. File 能**新建、删除、重命名**文件和目录，但 File 不能访问文件内容本身。如果需要访问文件内容本身，则需要使用输入/输出流。
2. 想要在Java程序中表示一个真实存在的文件或目录，那么必须有一个File对象，但是Java程序中的一个File对象，可能没有一个真实存在的文件或目录。
3. File对象可以作为**参数传递给流的构造器**

#### 1.常用构造器
1.`public File(String pathname)`
以pathname为路径创建File对象，可以是 绝对路径或者相对路径，如果pathname是相对路径，则默认的当前路径在系统属性user.dir中存储。
- 绝对路径：是一个固定的路径,从盘符开始
- 相对路径：是相对于某个位置开始

2.`public File(String parent,String child)`
	以parent为父路径，child为子路径创建File对象。

3.`public File(File parent,String child)`
	根据一个父File对象和子文件路径创建File对象
#### 2.常用方法

##### 1.获取功能
- public String getPath() ：获取路径
- public String getName() ：获取名称
- public String getParent()：获取上层文件目录路径。若无，返回null
- public long length() ：获取文件长度（即：字节数）。不能获取目录的长度。
- public long lastModified() ：获取最后一次的修改时间，毫秒值
- 适用于文件目录:
- public String[] list() ：获取指定目录下的所有文件或者文件目录的名称数组
- public File[] listFiles() ：获取指定目录下的所有文件或者文件目录的File数组


##### 2.重命名功能
- public boolean renameTo(File dest):把文件重命名为指定的文件路径(移动/重命名)


##### 3.判断功能
- public boolean isDirectory()：判断是否是文件目录
- public boolean isFile() ：判断是否是文件
- public boolean exists() ：判断是否存在
- public boolean canRead() ：判断是否可读
- public boolean canWrite() ：判断是否可写
- public boolean isHidden() ：判断是否隐藏


##### 4.创建功能
- public boolean createNewFile() ：创建文件。若文件存在，则不创建，返回false
- public boolean mkdir() ：创建文件目录。如果此文件目录存在，就不创建了。如果此文件目录的上层目录不存在，也不创建。
- public boolean mkdirs() ：创建文件目录。如果上层文件目录不存在，一并创建
  注意事项：如果你创建文件或者文件目录没有写盘符路径，那么， 默认在项目路径下


##### 5.删除功能
- public boolean delete()：删除文件或者文件夹
    删除注意事项：
        Java中的删除不走回收站。
        要删除一个文件目录，请注意该文件目录内不能包含文件或者文件目录


#### 3.File类使用示例

```java
package com.cykj.file;

import org.junit.Test;

import java.io.File;
import java.io.IOException;
import java.util.Date;

/**
 * 1.File类的一个对象，代表一个文件或一个文件目录
 * 2.File类声明在java.io包下
 *
 * @author zlg
 * @create 2019-10-06 15:19
 */
public class FileTest {

    /**
     * 常用构造器：
     * 1.如何创建File类的实例
     *      File(String filepath)
     *      File(String parent,String child)
     *      File(File parent,String child)
     *
     * 2.相对路径：相较于某个路径下，指明的路径
     *   绝对路径：包含盘符在内的文件或文件目录的路径
     * IDEA中：
        如果大家开发使用JUnit中的单元测试方法测试，相对路径即为当前Module下。
        如果大家使用main()测试，相对路径即为当前的Project下。
       Eclipse中：
        不管使用单元测试方法还是使用main()测试，相对路径都是当前的Project下。
     *
     * 3.路径分隔符
     *      windows：\\
     *      unix：/
     *      通用：File.separator
     */
    @Test
    public void test1(){

        //构造器1
        //相对于当前module
        File file1 = new File("hello.txt");
        //绝对路径
        File file2 = new File("E:\\ideaworkspace\\java-high\\io\\he.txt");

        System.out.println(file1);  // hello.txt
        System.out.println(file2);  // E:\ideaworkspace\java-high\io\he.txt

        //构造器2
        File file3 = new File("E:\\ideaworkspace", "java-high");
        System.out.println(file3);  // E:\ideaworkspace\java-high

        //构造器3
        File file4 = new File(file3, "hi.txt");
        System.out.println(file4);  // E:\ideaworkspace\java-high\hi.txt

    }


    /**
     * 常用方法：
     * 1.获取功能
     public String getAbsolutePath()：获取绝对路径
     public String getPath() ：获取路径
     public String getName() ：获取名称
     public String getParent()：获取上层文件目录路径。若无，返回null
     public long length() ：获取文件长度（即：字节数）。不能获取目录的长度。
     public long lastModified() ：获取最后一次的修改时间，毫秒值
     适用于文件目录:
     public String[] list() ：获取指定目录下的所有文件或者文件目录的名称数组
     public File[] listFiles() ：获取指定目录下的所有文件或者文件目录的File数组
     */
    @Test
    public void test2(){
        File file1 = new File("hello.txt");
        System.out.println(file1.getAbsolutePath());
        System.out.println(file1.getPath());
        System.out.println(file1.getName());    //文件名称
        System.out.println(file1.getParent());  //形参为相对路径，找不到上一级

        System.out.println(file1.length());
        System.out.println(new Date(file1.lastModified()));

        File file2 = new File("E:\\ideaworkspace", "java-high");
        for(String s:file2.list()){     //名称数组
            System.out.println(s);
        }
        for(File f:file2.listFiles()){  //File数组
            System.out.println(f);
        }

    }


    /**
     * 2.重命名功能
     public boolean renameTo(File dest):把文件重命名为指定的文件路径(移动/重命名)
     */
    @Test
    public void test3(){

        File file1 = new File("hello.txt"); //目标文件不存在
        File file2 = new File("hi.txt");    //原文件存在
        boolean bool = file2.renameTo(file1);
        System.out.println(bool);

    }


    /**
     * 3.判断功能
     public boolean isDirectory()：判断是否是文件目录
     public boolean isFile() ：判断是否是文件
     public boolean exists() ：判断是否存在
     public boolean canRead() ：判断是否可读
     public boolean canWrite() ：判断是否可写
     public boolean isHidden() ：判断是否隐藏
     */
    @Test
    public void test4(){

        File file = new File("hello.txt");
        System.out.println(file.isDirectory());     //是否是文件夹
        System.out.println(file.isFile());          //是否是文件
        System.out.println(file.exists());          //是否存在
        System.out.println(file.canRead());
        System.out.println(file.canWrite());
        System.out.println(file.isHidden());

    }


    /**
     * 4.创建功能
     public boolean createNewFile() ：创建文件。若文件存在，则不创建，返回false
     public boolean mkdir() ：创建文件目录。如果此文件目录存在，就不创建了。如果此文件目录的上层目录不存在，也不创建。
     public boolean mkdirs() ：创建文件目录。如果上层文件目录不存在，一并创建
     注意事项：如果你创建文件或者文件目录没有写盘符路径，那么， 默认在项目路径下
     */
    @Test
    public void test5() throws IOException {
        //文件创建
        File file1 = new File("world.txt");
        if(!file1.exists()){
            file1.createNewFile();
            System.out.println("创建成功！");
        }else{
            file1.delete();
            System.out.println("删除成功！");
        }

    }


    /**
     * 5.删除功能
     public boolean delete()：删除文件或者文件夹
     删除注意事项：
        Java中的删除不走回收站。
        要删除一个文件目录，请注意该文件目录内不能包含文件或者文件目录
     */
    public void test6(){

        File file = new File("hello.txt");
        //删除文件或文件夹
        deleteDirectory(file);

    }


    //删除文件或文件夹
    public void deleteDirectory(File file) {
        // 如果file是文件，直接delete
        // 如果file是目录，先把它的下一级干掉，然后删除自己
        if (file.isDirectory()) {
            File[] all = file.listFiles();
            // 循环删除的是file的下一级
            for (File f : all) {// f代表file的每一个下级
                deleteDirectory(f);
            }
        }
        // 删除自己
        file.delete();
    }


}

```

#### 4.应用

##### 1.查找指定文件

```java
/**
 * 应用：判断指定目录下是否有后缀名为.jpg的文件，如果有，就输出该文件名称
 *
 */
public class FindJPGFileTest {

	@Test
	public void test1(){
		File srcFile = new File("d:\\code");
		
		String[] fileNames = srcFile.list();
		for(String fileName : fileNames){
			if(fileName.endsWith(".jpg")){
				System.out.println(fileName);
			}
		}
	}
	@Test
	public void test2(){
		File srcFile = new File("d:\\code");
		
		File[] listFiles = srcFile.listFiles();
		for(File file : listFiles){
			if(file.getName().endsWith(".jpg")){
				System.out.println(file.getAbsolutePath());
			}
		}
	}
	/*
	 * File类提供了两个文件过滤器方法
	 * public String[] list(FilenameFilter filter)
	 * public File[] listFiles(FileFilter filter)

	 */
	@Test
	public void test3(){
		File srcFile = new File("d:\\code");
		
		File[] subFiles = srcFile.listFiles(new FilenameFilter() {
			
			@Override
			public boolean accept(File dir, String name) {
				return name.endsWith(".jpg");
			}
		});
		
		for(File file : subFiles){
			System.out.println(file.getAbsolutePath());
		}
	}
	
}
```

##### 2.遍历指定目录

```java
/**
 * 应用：遍历指定目录所有文件名称，包括子文件目录中的文件。
	拓展1：并计算指定目录占用空间的大小
	拓展2：删除指定文件目录及其下的所有文件
 *
 */
public class ListFilesTest {

	public static void main(String[] args) {
		// 递归:文件目录
		/** 打印出指定目录所有文件名称，包括子文件目录中的文件 */

		// 1.创建目录对象
		File dir = new File("E:\\teach\\01_javaSE\\_尚硅谷Java编程语言\\3_软件");

		// 2.打印目录的子文件
		printSubFile(dir);
	}

	public static void printSubFile(File dir) {
		// 打印目录的子文件
		File[] subfiles = dir.listFiles();

		for (File f : subfiles) {
			if (f.isDirectory()) {// 文件目录
				printSubFile(f);
			} else {// 文件
				System.out.println(f.getAbsolutePath());
			}

		}
	}

	// 方式二：循环实现
	// 列出file目录的下级内容，仅列出一级的话
	// 使用File类的String[] list()比较简单
	public void listSubFiles(File file) {
		if (file.isDirectory()) {
			String[] all = file.list();
			for (String s : all) {
				System.out.println(s);
			}
		} else {
			System.out.println(file + "是文件！");
		}
	}

	// 列出file目录的下级，如果它的下级还是目录，接着列出下级的下级，依次类推
	// 建议使用File类的File[] listFiles()
	public void listAllSubFiles(File file) {
		if (file.isFile()) {
			System.out.println(file);
		} else {
			File[] all = file.listFiles();
			// 如果all[i]是文件，直接打印
			// 如果all[i]是目录，接着再获取它的下一级
			for (File f : all) {
				listAllSubFiles(f);// 递归调用：自己调用自己就叫递归
			}
		}
	}

	// 拓展1：求指定目录所在空间的大小
	// 求任意一个目录的总大小
	public long getDirectorySize(File file) {
		// file是文件，那么直接返回file.length()
		// file是目录，把它的下一级的所有大小加起来就是它的总大小
		long size = 0;
		if (file.isFile()) {
			size += file.length();
		} else {
			File[] all = file.listFiles();// 获取file的下一级
			// 累加all[i]的大小
			for (File f : all) {
				size += getDirectorySize(f);// f的大小;
			}
		}
		return size;
	}

	// 拓展2：删除指定的目录
	public void deleteDirectory(File file) {
		// 如果file是文件，直接delete
		// 如果file是目录，先把它的下一级干掉，然后删除自己
		if (file.isDirectory()) {
			File[] all = file.listFiles();
			// 循环删除的是file的下一级
			for (File f : all) {// f代表file的每一个下级
				deleteDirectory(f);
			}
		}
		// 删除自己
		file.delete();
	}

}
```

### 二.文件流(节点流)

#### 1.FileReader/FileWriter应用

##### 1.将hello.txt文件内容读入程序中，并输出到控制台

说明点：

1. `read()`的理解：返回读入的一个字符。如果达到文件末尾，返回**-1**
2. 异常的处理：为了保证流资源一定可以执行关闭操作。需要使用`try-catch-finally`处理
3. 读入的文件一定要存在，否则就会报`FileNotFoundException`。

```java
	@Test
    public void fileReaderTest1()  {
        FileReader fileReader = null;
        try {
            //1.实例化File对象
            File file = new File("hello.txt");
            //2.提供具体流
            fileReader = new FileReader(file);
            //3.数据读取
            //read():返回读入的一个字符。如果达到文件末尾，返回-1
            int data = -1;
            while ( (data = fileReader.read()) != -1){
                System.out.print((char) data);  //将int类强制转化为char输出
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //4.关闭流
            if(fileReader != null){ //先判断是否为null，避免NullPointerException
                try {
                    fileReader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

##### 2.对read()操作升级

使用read的重载方法 (read(char[] cbuf))

```java
	@Test
    public void fileReaderTest2()  {
        FileReader fileReader = null;
        try {
            //1.File类的实例化
            File file = new File("hello.txt");
            //2.具体流的提供
            fileReader = new FileReader(file);
            //3.数据读取
            //read(char[] cbuf):返回每次读入cbuf数组中的字符的个数。如果达到文件末尾，返回-1
            char[] cbuf = new char[5];
            int len = -1;
            while ((len = fileReader.read(cbuf)) != -1){
                //第一种写法：
//                for(int i =0;i<len;i++){
//                    System.out.print(cbuf[i]);
//                }
                //错误写法
//                for(int i=0;i<cbuf.length;i++){
//                    System.out.print(cbuf[i]);
//                }
                //第二种写法：
                String data = new String(cbuf,0,len);
                System.out.print(data);
                //错误写法
//                String data = new String(cbuf);
//                System.out.print(data);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(fileReader!=null){
                //4.流的关闭
                try {
                    fileReader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

##### 3.从内存中写出数据到硬盘的文件里

说明：

1. 输出操作，对应的File可以不存在的。并不会报异常
2. File对应的硬盘中的文件
  如果不存在，在输出的过程中，会自动创建此文件。
  如果存在：
  如果流使用的构造器是：`FileWriter(file,false) / FileWriter(file)`:对原有文件的覆盖
  如果流使用的构造器是：`FileWriter(file,true)`:不会对原有文件覆盖，而是在原有文件基础上追加内容

```java
	@Test
    public void fileWriterTest() {
        FileWriter fileWriter = null;
        try {
            //1.File类实例化
            File file = new File("hello1.txt");
            //2.具体流实例化
            fileWriter = new FileWriter(file,false);
            //3.数据写入
            fileWriter.write("I have a dream!\n");
            fileWriter.write("You need to have a dream!");
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(fileWriter!=null){
                //4.关闭流
                try {
                    fileWriter.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

##### 4.实现文本文件的复制

1. 源文件必须存在
2. 目标文件可以不存在
3. 读取时常用：`read(char[] cbuf)`
   写入时常用：`write(char[] cbuf,int off,int len)`
4. 不能使用字符流来处理图片等字节数据

```java
	@Test
    public void copyFileTest()  {
        FileReader fr = null;
        FileWriter fw = null;
        try {
            //File实例化
            File srcFile = new File("hello.txt");
            File destFile = new File("hello2.txt");
            //具体流实例化
            fr = new FileReader(srcFile);
            fw = new FileWriter(destFile);
            //数据读取与写入
            char[] cbuf = new char[5];  //每次读取5个字符并存放在数组中
            int len;    //记录每次读入到cbuf数组中的字符的个数
            while((len = fr.read(cbuf)) != -1){
                //每次写出len个字符
                fw.write(cbuf,0,len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //关闭流
            try {
                if(fr!=null)
                fr.close();
            } catch (IOException e) {
                e.printStackTrace();
            }   //若第一个try-catch发送异常，会进行处理，后面语句会执行
            try {
                if(fw!=null)
                fw.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```

#### 2.FileInputStream/FileOutputStream应用

##### 1.处理文本文件，可能出现乱码

* 1.若文本没有中文，是没有乱码（英文对应的ASCII码，一个字节）
* 2.若有中文时，可能会主线乱码（一个中文占三个字节）

```java
	@Test
    public void copyDocFileTest() {
        FileInputStream fis = null;
        try {
            //实例化File
            File file = new File("hello.txt");
            //实例化Stream
            fis = new FileInputStream(file);
            //数据操作
            byte[] buffer = new byte[5];
            int len;
            while( (len = fis.read(buffer)) != -1){
                String str = new String(buffer,0,len);
                System.out.println(str);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if( fis != null){
                //关闭Stream
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

##### 2.实现对图片的复制操作

```java
	@Test
    public void copyImgFileTest() {
        FileInputStream fis = null;
        FileOutputStream fos = null;
        try {
            //实例化File
            File srcFile = new File("爱情与友情.jpg");
            File destFile = new File("爱情与友情dest.jpg");
            //实例化Stream
            fis = new FileInputStream(srcFile);
            fos = new FileOutputStream(destFile);
            //数据操作
            byte[] buffer =new byte[5];
            int len;
            while( (len = fis.read(buffer)) != -1){
                fos.write(buffer,0,len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //关闭Stream
            if(fis != null){
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(fos != null){
                try {
                    fos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

##### 3.封装复制操作方法，对指定路径下文件复制

```java
	@Test
    public void copyFileTest(){
        //对非文本文件复制操作
//        String srcPath = "D:\\BaiduNetdiskDownload\\videotest.avi";
//        String destPath = "D:\\BaiduNetdiskDownload\\videotest_dest.avi";

        //对文本文件复制操作
        String srcPath = "hello.txt";
        String destPath = "hello_dest.txt";

        long start = System.currentTimeMillis();
        copyFile(srcPath,destPath);
        long end = System.currentTimeMillis();
        System.out.println("复制完成！耗时："+(end-start)+"ms！");

    }

	//非文本或文本文件都可以使用字节流进行复制操作
    public void copyFile(String srcPath, String destPath){
        FileInputStream fis = null;
        FileOutputStream fos = null;
        try {
            //实例化File
            File srcFile = new File(srcPath);
            File destFile = new File(destPath);
            //实例化Stream
            fis = new FileInputStream(srcFile);
            fos = new FileOutputStream(destFile);
            //数据操作
            byte[] buffer =new byte[1024];
            int len;
            while( (len = fis.read(buffer)) != -1){
                fos.write(buffer,0,len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //关闭Stream
            if(fis != null){
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(fos != null){
                try {
                    fos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

#### 3.结论

1. 对于**文本**文件(.txt,.java,.c,.cpp)，使用**字符流**处理
2. 对于**非文本**文件(.jpg,.mp3,.mp4,.avi,.doc,.ppt,.xsl,...)，使用**字节流**处理
3. 对于**复制**操作，文本文件也可以使用**字节流**处理，相当于进行底层搬运；
    反之，非文本文件，是不可以用字符流来处理的


### 三.缓存流(处理流)

#### 1.概念

1.  为了提高数据读写的速度，Java API提供了带缓冲功能的流类，在使用这些流类时，会创建一个内部缓冲区数组，缺省使用8192个字节(8Kb)的缓冲区。
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124162902571.png)

2.  缓冲流要“套接”在相应的节点流之上，根据数据操作单位可以把缓冲流分为：

    `BufferedInputStream` 和 和 `BufferedOutputStream`
    `BufferedReader` 和 和 `BufferedWriter`

#### 2.原理

1.  当读取数据时，数据按块读入缓冲区，其后的读操作则直接访问缓冲区
2.  当使用`BufferedInputStream`读取字节文件时，`BufferedInputStream`会一次性从文件中读取8192个(8Kb)，存在缓冲区中，直到缓冲区装满了，才重新从文件读取下一个8192个字节数组。
3.  向流中写入字节时，不会直接写到文件，先写到缓冲区中直到缓冲区写满，`BufferedOutputStream`才会把缓冲区中的数据一次性写到文件里。使用方法`flush()`可以强制将缓冲区的内容全部写入输出流
4.  关闭流的顺序和打开流的顺序相反。只要关闭最外层流即可，关闭最外层流也会相应关闭内层节点流
5.  `flush()`方法的使用：手动将buffer中内容写入文件
6.  如果是带缓冲区的流对象的`close()`方法，不但会关闭流，还会在关闭流之前刷新缓冲区，关闭后不能再写出
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124162918455.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

#### 3.示例

```java
/**
 * 缓冲流的使用(处理流的一种)
 * 1.缓冲流
 *  BufferedInputStream
 *  BufferedOutputStream
 *  BufferedReader
 *  BufferedWriter
 *
 * 2.作用：提供流的读取、写入的速度
 *   提高读写速度的原因：内部提供了一个缓冲区
 *
 * 3.处理流，就是“套接”在已有的流的基础上。
 *
 * @author zlg
 * @create 2019-10-08 23:58
 */
public class BufferedStreamTest {

    /**
     * 实现非文本文件的复制
     */
    @Test
    public void copyFileTest(){

        String srcPath = "D:\\BaiduNetdiskDownload\\videotest.avi";
        String destPath = "D:\\BaiduNetdiskDownload\\videotest_dest2.avi";

        long start = System.currentTimeMillis();
        copyFile(srcPath,destPath);
        long end = System.currentTimeMillis();

        System.out.println("复制完成！耗时："+(end-start)+"ms！");   // 792ms  1850ms

    }

    //封装文件复制的方法
    public void copyFile(String srcPath,String destPath){
        BufferedInputStream bis = null;
        BufferedOutputStream bos = null;
        try {
            //1.file实例化
            File srcFile = new File(srcPath);
            File destFile = new File(destPath);
            //2.stream实例化
            //2.1filestream实例化
            FileInputStream fis = new FileInputStream(srcFile);
            FileOutputStream fos = new FileOutputStream(destFile);
            //2.2bufferedstream实例化
            bis = new BufferedInputStream(fis);
            bos = new BufferedOutputStream(fos);
            //3.data处理
            byte[] buffer = new byte[1024];
            int len;
            while ( (len = bis.read(buffer)) != -1){
                bos.write(buffer,0,len);
//              bos.flush();//刷新缓冲区
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //4.stream关闭
            //要求：先关闭外层的流，再关闭内层的流
            if (bis != null){
                try {
                    bis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (bos != null){
                try {
                    bos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            //说明：关闭外层流的同时，内层流也会自动的进行关闭。关于内层流的关闭，我们可以省略.
//          fos.close();
//          fis.close();
        }
    }


    /**
     * 使用BufferedReader和BufferedWriter实现文本文件的复制
     */
    @Test
    public void bufferedCharStreamTest(){
        BufferedReader br = null;
        BufferedWriter bw = null;
        try {
            //stream实例化
            br = new BufferedReader(new FileReader("dbcp.txt"));
            bw = new BufferedWriter(new FileWriter("dbcp_dest.txt"));
            //data处理
            //方式一：
//            char[] cbuf = new char[1024];
//            int len;
//            while ( (len = br.read(cbuf)) != -1){
//                bw.write(cbuf,0,len);
//            }
            //方式二：
            String data;
            while ( (data = br.readLine()) != null) {
                bw.write(data);     //data中不包含换行符
                bw.newLine();   //提供换行的操作
//                bw.write(data + "\n");//data中不包含换行符
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //stream关闭
            if(br != null){
                try {
                    br.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(bw != null){
                try {
                    bw.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

#### 4.应用

##### 1.图片的加密与解密

```java
public class PicTest {

    //图片的加密
    @Test
    public void test1() {

        FileInputStream fis = null;
        FileOutputStream fos = null;
        try {
            fis = new FileInputStream("爱情与友情.jpg");
            fos = new FileOutputStream("爱情与友情secret.jpg");

            byte[] buffer = new byte[20];
            int len;
            while ((len = fis.read(buffer)) != -1) {
                //字节数组进行修改
                //错误的
                //            for(byte b : buffer){
                //                b = (byte) (b ^ 5);
                //            }
                //正确的
                for (int i = 0; i < len; i++) {
                    buffer[i] = (byte) (buffer[i] ^ 5);
                }


                fos.write(buffer, 0, len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fos != null) {
                try {
                    fos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
            if (fis != null) {
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
        }
    }


    //图片的解密
    @Test
    public void test2() {

        FileInputStream fis = null;
        FileOutputStream fos = null;
        try {
            fis = new FileInputStream("爱情与友情secret.jpg");
            fos = new FileOutputStream("爱情与友情4.jpg");

            byte[] buffer = new byte[20];
            int len;
            while ((len = fis.read(buffer)) != -1) {
                //字节数组进行修改
                //错误的
                //            for(byte b : buffer){
                //                b = (byte) (b ^ 5);
                //            }
                //正确的
                for (int i = 0; i < len; i++) {
                    buffer[i] = (byte) (buffer[i] ^ 5);
                }

                fos.write(buffer, 0, len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fos != null) {
                try {
                    fos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
            if (fis != null) {
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
        }
    }
}
```

##### 2.获取文本上字符出现的次数,把数据写入文件

```java
/**
 * 获取文本上字符出现的次数,把数据写入文件
 *
 * 思路：
 * 1.遍历文本每一个字符
 * 2.字符出现的次数存在Map中
 *
 * Map<Character,Integer> map = new HashMap<Character,Integer>();
 * map.put('a',18);
 * map.put('你',2);
 *
 * 3.把map中的数据写入文件
 */
public class WordCount {
    /*
    说明：如果使用单元测试，文件相对路径为当前module
          如果使用main()测试，文件相对路径为当前工程
     */
    @Test
    public void testWordCount() {
        FileReader fr = null;
        BufferedWriter bw = null;
        try {
            //1.创建Map集合
            Map<Character, Integer> map = new HashMap<Character, Integer>();

            //2.遍历每一个字符,每一个字符出现的次数放到map中
            fr = new FileReader("dbcp.txt");
            int c = 0;
            while ((c = fr.read()) != -1) {
                //int 还原 char
                char ch = (char) c;
                // 判断char是否在map中第一次出现
                if (map.get(ch) == null) {
                    map.put(ch, 1);
                } else {
                    map.put(ch, map.get(ch) + 1);
                }
            }

            //3.把map中数据存在文件count.txt
            //3.1 创建Writer
            bw = new BufferedWriter(new FileWriter("wordcount.txt"));

            //3.2 遍历map,再写入数据
            Set<Map.Entry<Character, Integer>> entrySet = map.entrySet();
            for (Map.Entry<Character, Integer> entry : entrySet) {
                switch (entry.getKey()) {
                    case ' ':
                        bw.write("空格=" + entry.getValue());
                        break;
                    case '\t'://\t表示tab 键字符
                        bw.write("tab键=" + entry.getValue());
                        break;
                    case '\r'://
                        bw.write("回车=" + entry.getValue());
                        break;
                    case '\n'://
                        bw.write("换行=" + entry.getValue());
                        break;
                    default:
                        bw.write(entry.getKey() + "=" + entry.getValue());
                        break;
                }
                bw.newLine();
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //4.关流
            if (fr != null) {
                try {
                    fr.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (bw != null) {
                try {
                    bw.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }
}
```

### 四.转换流(处理流)

#### 1.概念

1.  转换流提供了在**字节流和字符流**之间的转换

2.  javaAPI提供了两个转换流：

    **`InputStreamReader`**：将InputStream转换为Reader

    **`OutPutStreamWriter`**：将Writer转换为OutputStream

3.  字节流中的数据都是字符时，转成字符流操作更高效

4.  可以使用转换流来处理**文件乱码**问题。实现编码和解码功能

#### 2.API介绍

1.  `InputStreamReader`

    实现将**字节**的输入流按指定**字符集**转换为**字符**的输入流

    需要和 **InputStream** “套接”

    构造器：

    ​	`public InputStreamReader(InputStream in)`

    ​	`public InputStreamReader(InputStream in, String charsetName)`

    如：`Reader isr = new InputStreamReader(System.in, "gbk");`

2.  `OutputStreamWriter`

    实现将**字符**的输出流按指定**字符集**转换为**字节**的输出流

    需要和 **OutputStream** “套接”

    构造器：

    ​	`public OutputStreamWriter(OutputStream out)`

    ​	`public OutputStreamWriter(OutputStream out, String charsetName)`

#### 3.图例
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124163036754.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

#### 4.示例

```java
/**
 * 处理流之二：转换流的使用
 * 1.转换流：属于字符流
 *   InputStreamReader：将一个字节的输入流转换为字符的输入流
 *   OutputStreamWriter：将一个字符的输出流转换为字节的输出流
 *
 * 2.作用：提供字节流与字符流之间的转换
 *
 * 3. 解码：字节、字节数组  --->字符数组、字符串      InputStreamReader(读取)
 *    编码：字符数组、字符串 ---> 字节、字节数组      OutputStreamWriter(写入)
 *
 * @author zlg
 * @create 2019-10-10 23:33
 */
public class TransformStream {

    /**
     * 将utf-8编码的dbcp.txt文件复制转为gbk编码的dbcp_gbk.txt文件
     * 系统(idea)默认字符集是utf-8
     * 参数2指明了字符集，具体使用哪个字符集，取决于文件dbcp.txt保存时使用的字符集
     */
    @Test
    public void transformStreamTest(){
        InputStreamReader isr = null;
        OutputStreamWriter osr = null;

        try {
            //stream实例化
            isr = new InputStreamReader(new FileInputStream("dbcp.txt"),"utf-8");
            osr = new OutputStreamWriter(new FileOutputStream("dbcp_gbk.txt"),"gbk");
            //数据处理
            char[] cbuf = new char[1024];
            int len;
            while ( (len = isr.read(cbuf)) != -1){
                osr.write(cbuf,0,len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //关闭流
            if( osr != null){
                try {
                    osr.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if( isr != null){
                try {
                    isr.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }

}
```

### 五.字符编码

#### 1.编码表由来

计算机只能识别二进制数据，早期由来是电信号。为了方便应用计算机，让它可以识别各个国家的文字。就将各个国家的文字用数字来表示，并一一对应，形成一张表。这就是编码表。

#### 2.常见编码表

**ASCII**：美国标准信息交换码。用一个字节的7位可以表示。
**ISO8859-1**：拉丁码表。欧洲码表。用一个字节的8位表示。
**GB2312**：中国的中文编码表。最多两个字节编码所有字符
**GBK**：中国的中文编码表升级，融合了更多的中文文字符号。最多两个字节编码
**Unicode**：国际标准码，融合了目前人类使用的所有字符。为每个字符分配唯一的字符码。所有的文字都用两个字节来表示。
**UTF-8**：变长的编码方式，可用1-4个字节来表示一个字符。

#### 3.图例
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124163209239.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)
ANSI与Unicode
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124163239366.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)


#### 4.Unicode问题

1.  Unicode不完美，这里就有三个问题，一个是，我们已经知道，英文字母只用一个字节表示就够了，第二个问题是如何才能区别Unicode和ASCII？计算机怎么知道两个字节表示一个符号，而不是分别表示两个符号呢？第三个，如果和GBK等双字节编码方式一样，用最高位是1或0表示两个字节和一个字节，就少了很多值无法用于表示字符，不够表示所有字符。Unicode在很长一段时间内无法推广，直到互联网的出现。
2.  面向传输的众多 UTF（UCS Transfer Format）标准出现了，顾名思义，UTF-8就是每次8个位传输数据，而UTF-16就是每次16个位。这是为传输而设计的编码，并使编码无国界，这样就可以显示全世界上所有文化的字符了。
3.  Unicode只是定义了一个庞大的、全球通用的字符集，并为每个字符规定了唯一确定的编号，具体存储成什么样的字节流，取决于字符编码方案。推荐的Unicode编码是UTF-8和UTF-16
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124163352851.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

#### 5.编码与解码

编码：字符数组、**字符串** ---> 字节、**字节数组**      **OutputStreamWriter**(写入)

解码：字节、**字节数组**  --->字符数组、**字符串**      **InputStreamReader**(读取)

### 六.其他流(处理流)

#### 1.标准输入、输出流

##### 1.概念

1.  **System.in**和**System.out**分别代表了系统标准的输入和输出设备

2.  默认输入设备是：键盘，输出设备是：显示器

3.  System.in的类型是**InputStream**

4.  System.out的类型是**PrintStream**，其是OutputStream的子类FilterOutputStream 的子类

5.  重定向：通过System类的**setIn**，**setOut**方法对默认设备进行改变。

    `public static void setIn(InputStream in)`
    `public static void setOut(PrintStream out)`

##### 2.示例

```java
/**
     * 1.标准的输入，输出流
     * System.in:标准的输入流，默认从键盘输入     返回 InputStream
       System.out:标准的输出流，默认从控制台输出   返回 PrintStream
       System类的setIn(InputStream is) / setOut(PrintStream ps)方式重新指定输入和输出的流。
     *
     * 从键盘输入字符串，要求将读取到的整行字符串转成大写输出。然后继续进行输入操作，
        直至当输入“e”或者“exit”时，退出程序。
     方法一：使用Scanner实现，调用next()返回一个字符串
     	Scanner s = new Scanner(System.in);
		s.nextInt();
		s.next();
     方法二：使用System.in实现。System.in  --->  转换流 ---> BufferedReader的readLine()
     */
    public static void main(String[] args) {
        BufferedReader br = null;
        try {
            // 把"标准"输入流(键盘输入)这个字节流包装成字符流,再包装成缓冲流
            br = new BufferedReader(new InputStreamReader(System.in));

            while (true){
                System.out.println("请输入字符串：");
                String data = br.readLine();   // 读取用户输入的一行数据 --> 阻塞程序
                if ("e".equalsIgnoreCase(data) || "exit".equalsIgnoreCase(data)) {
                    System.out.println("程序结束");
                    break;
                }
			   // 将读取到的整行字符串转成大写输出
                String str = data.toUpperCase();
                System.out.println(str);

            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (br != null) {
                try {
                    br.close();		// 关闭过滤流时,会自动关闭它包装的底层节点流
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

#### 2.打印流

##### 1.概念

1.  实现将 **基本数据类型**的数据格式转化为 **字符串**输出

2.  打印流：**`PrintStream`**和**`PrintWriter`**，都是输出流

    提供了一系列重载的print()和println()方法，用于多种**数据类型的输出**
    `PrintStream`和`PrintWriter`的输出**不会抛出IOException异常**
    `PrintStream`和`PrintWriter`有**自动flush功能**
    `PrintStream` 打印的所有字符都使用平台的默认字符编码转换为**字节**。
    在需要写入**字符**而不是写入字节的情况下，应该使用 `PrintWriter` 类。
    System.out返回的是**PrintStream的实例**

##### 2.示例

```java
/**
     * 打印流：PrintStream 和PrintWriter ：都是输出流
       提供了一系列重载的print() 和 println()
     * 练习：使用打印输出流将ASCII码写入到文本中
     */
    @Test
    public void printStreamTest(){
        PrintStream ps = null;
        try {
            // 创建打印输出流,设置为自动刷新模式(写入换行符或字节 '\n' 时都会刷新输出缓冲区)
            ps = new PrintStream(new FileOutputStream("text.txt"),true);
            //重新指定打印流
            if (ps != null) {   // 把标准输出流(控制台输出)改成文件
                System.setOut(ps);
            }
            //循环写入ASCII
            for (int i=0;i<256;i++){
                System.out.print((char)i);
                if (i % 50 == 0) {  //每50个数据一行
                    System.out.println();
                }
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } finally {
            if (ps != null) {
                ps.close();
            }
        }
    }
```

#### 3.数据流

##### 1.概念

1.  为了方便地操作Java语言的**基本数据类型**和**String**的数据，可以使用数据流。

2.  数据流有两个类：(用于读取和写出基本数据类型、String类的数据）

    `**DataInputStream**` 和 **`DataOutputStream`**
    在 分别“套接”在 `InputStream` 和 和 `OutputStream`  子类的流 上

3.  DataInputStream 中的方法

    `boolean readBoolean()` 		`byte readByte()`
    `char readChar()` 			`float readFloat()`
    `double readDouble()` 		`short readShort()`
    `long readLong()` 			`int readInt()`
    `String readUTF()` 			`void readFully(byte[] b)`

4.  DataOutputStream 中的方法

    将上述的方法的read改为相应的write即可。

##### 2.示例

```java
/**
     * 数据流
     3.1 DataInputStream 和 DataOutputStream
     3.2 作用：用于读取或写出基本数据类型的变量或字符串

     练习：将内存中的字符串、基本数据类型的变量写出到文件中。
     */
    @Test
    public void dataOutputStreamTest(){
        DataOutputStream dos = null;
        try {
          	// 创建连接到指定文件的数据输出流对象
            dos = new DataOutputStream(new FileOutputStream("data.txt"));
			// 写UTF字符串
            dos.writeUTF("小明");
            dos.flush();    //刷新操作，将内存中的数据写入文件
            dos.writeInt(18);
            dos.flush();
            dos.writeBoolean(true);
            dos.flush();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (dos != null) {
                try {
                    dos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    /**
     * 数据流
      将文件中存储的基本数据类型变量和字符串读取到内存中，保存在变量中。
      注意点：读取不同类型的数据的顺序要与当初写入文件时，保存的数据的顺序一致！
     */
    @Test
    public void dataInputStreamTest(){
        DataInputStream dis = null;
        try {
            dis = new DataInputStream(new FileInputStream("data.txt"));

            String name = dis.readUTF();
            int age = dis.readInt();
            boolean isMale = dis.readBoolean();
            System.out.println("name=" + name + ", age=" + age + ", isMale=" + isMale);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (dis != null) {
                try {
                    dis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

### 七.对象流(处理流)

#### 1.概念

1.  `ObjectInputStream`和`OjbectOutputSteam`

    用于存储和读取 **基本数据类型**数据或 **对象**的处理流。它的强大之处就是可以把Java中的对象写入到数据源中，也能把对象从数据源中还原回来。

2.  **序列化**：用`ObjectOutputStream`类 **保存**基本类型数据或对象的机制 (**数据转为二进制流**)

3.  **反序列化**：用`ObjectInputStream`类 **读取**基本类型数据或对象的机制 (**二进制流还原为数据**)

4.  `ObjectOutputStream`和`ObjectInputStream`不能序列化**`static`**和**`transient`**修饰的成员变量

#### 2.对象序列化

1.  对象序列化机制允许把内存中的**Java对象**转换成平台无关的**二进制流**，从而允许把这种二进制流持久地**保存在磁盘**上，或通过网络将这种二进制流**传输到另一个网络节点**。(反序列化)当其它程序获取了这种二进制流，就可以恢复成原来的Java对象

2.  序列化的好处在于可将任何实现了`Serializable`接口的**对象转化为 字节数据**，使其在**保存和传输**时可被还原

3.  序列化是 RMI（Remote Method Invoke – 远程方法调用）过程的参数和返回值都必须实现的机制，而 RMI 是 JavaEE 的基础。因此序列化机制是JavaEE 平台的基础

4.  如果需要让某个对象支持序列化机制，则必须让对象所属的**类及其属性**是可序列化的，为了让某个类是可序列化的，该类必须实现如下两个接口之一。否则，会抛出**`NotSerializableException`**异常

    **`Serializable`**
    `Externalizable`

5.  实现了`Serializable` 接口的对象，可将它们转换成**一系列字节**，并可在以后完全恢复回原来的样子。 **这一过程亦可通过网络进行。这意味着序列化机制能自动补偿操作系统间的差异。**在 换句话说，可以先在Windows 机器上创台 建一个对象，对其序列化，然后通过网络发给一台Unix 机器，然后在那里准确无误地重新“装配”。不必关心数据在不同机器上如何表示，也不必关心字节的顺序或者其他任何细节。(序列化接口的理解)

6.  由于大部分作为参数的类如`String` 、`Integer` 等都实现了`java.io.Serializable` 的接口，也可以利用多态的性质，作为参数使接口更灵活。(序列化接口是空方法接口)

#### 3.序列化版本标识符

1.  凡是实现Serializable接口的类都有一个表示**序列化版本标识符的静态变量**：

    `private static final long serialVersionUID`;
    `serialVersionUID`用来表明类的不同版本间的兼容性。 简言之，其目的是以序列化对象进行版本控制，有关各版本反序列化时是否兼容。
    如果类没有显示定义这个静态常量，它的值是Java运行时环境根据**类的内部细节自动生成**的。若类的**实例变量做了修改**，`serialVersionUID` 可能发生变化。故建议，**显式声明**。

2.  简单来说，Java的序列化机制是通过在运行时判断类的`serialVersionUID`来验证**版本一致性**的。在进行反序列化时，JVM会把传来的**字节流**中的`serialVersionUID`与**本地相应实体**`serialVersionUID`进行比较，如果相同就认为是一致的，可以进行**反序列化**，否则就会出现序列化版本不一致的异常。(`InvalidCastException`)

#### 4.应用

1.  若某个类实现了 `Serializable` 接口，该类的对象就是可序列化的：

    创建一个 `ObjectOutputStream`
    调用 `ObjectOutputStream`  对象的 `writeObject`( 对象)  方法输出可 序列化对象
    注意写出一次，操作`flush()` 一次

2.  反序列化

    创建一个 `ObjectInputStream`
    调用 `readObject()`  方法读取流中的对象

3.  强调：如果某个类的属性不是基本数据类型或 String 类型，而是另一个引用类型，那么这个引用类型必须是**可序列化**的，否则拥有该类型的`Field` 的类也不能序列化

#### 5.示例

##### 1.测试类

```java
/**
 * 对象流的使用
 * 1.ObjectInputStream 和 ObjectOutputStream
 * 2.作用：用于存储和读取基本数据类型数据或对象的处理流。
 * 它的强大之处就是可以把Java中的对象写入到数据源中，也能把对象从数据源中还原回来。
 *
 * 3.要想一个java对象是可序列化的，需要满足相应的要求。见Person.java
 *
 * 4.序列化机制：
 * 对象序列化机制允许把内存中的Java对象转换成平台无关的二进制流，从而允许把这种
 * 二进制流持久地保存在磁盘上，或通过网络将这种二进制流传输到另一个网络节点。
 * 当其它程序获取了这种二进制流，就可以恢复成原来的Java对象。
 *
 * @author zlg
 * @create 2019-10-13 23:25
 */
public class ObjectStreamTest {

    /**
     * 序列化过程：将内存中的java对象保存到磁盘中或通过网络传输出去
     * 使用ObjectOutputStream实现
     */
    @Test
    public void objectOutputStreamTest(){

        ObjectOutputStream oos = null;
        try {
            oos = new ObjectOutputStream(new FileOutputStream("person.dat"));

            oos.writeObject( new Person("jack",18,new Card(1278234L)));
            oos.flush();    //刷新操作

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (oos != null) {
                try {
                    oos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    /**
     * 反序列化：将磁盘文件中的对象或网络中传输的二进制流还原为内存中的一个java对象
     * 使用ObjectInputStream来实现
     */
    @Test
    public void objectInputStreamTest(){

        ObjectInputStream ois = null;
        try {
            ois = new ObjectInputStream(new FileInputStream("person.dat"));

            Person person = (Person) ois.readObject();
            System.out.println(person);

        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } finally {
            if (ois != null) {
                try {
                    ois.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

}
```

##### 2.Person类

```java
/**
 * Person需要满足如下的要求，方可序列化
 * 1.需要实现接口：Serializable
 * 2.当前类提供一个全局常量：serialVersionUID
 * 3.除了当前Person类需要实现Serializable接口之外，还必须保证其内部所有属性
 *   也必须是可序列化的。（默认情况下，基本数据类型及包装类，String都已实现序列化）
 *
 * 注意：ObjectOutputStream和ObjectInputStream不能序列化static和transient修饰的成员变量
 * static不是类所私有的，transient是修饰不用序列化的成员变量
 *
 * @author zlg
 * @create 2019-10-14 0:20
 */
public class Person implements Serializable{
	//序列化的类需要显式声明序列化版本标识符
    private static final long serialVersionUID = 12421351542L;

    private String username;
    private Integer age;
    private Card card;

    public Card getCard() {
        return card;
    }

    public void setCard(Card card) {
        this.card = card;
    }

    public Person(String username, Integer age, Card card) {
        this.username = username;
        this.age = age;
        this.card = card;
    }

    @Override
    public String toString() {
        return "Person{" +
                "username='" + username + '\'' +
                ", age=" + age +
                ", card=" + card +
                '}';
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Person(String username, Integer age) {

        this.username = username;
        this.age = age;
    }

    public Person() {

    }
}
```

##### 3.Card类

```java
public class Card implements Serializable{
	//序列化的类需要显式声明序列化版本标识符
    private static final long serialVersionUID = 54635742L;

    private Long ID;

    @Override
    public String toString() {
        return "Card{" +
                "ID=" + ID +
                '}';
    }

    public Long getID() {
        return ID;
    }

    public void setID(Long ID) {
        this.ID = ID;
    }

    public Card() {

    }

    public Card(Long ID) {

        this.ID = ID;
    }
}
```

### 八.随机存取文件流

#### 1.概念

1.  RandomAccessFile声明在`java.io`包下，但直接继承**Object类**。并实现了`DataInput，DataOutput`这两个接口，意味着该类既可以读也可以写。
2.  RandomAccessFile类支持"**随机访问**"的方式，程序可直接跳到文件的任意位置来读，写文件。
    1.  支持只访问文件的部分内容
    2.  可以向已存在的文件后追加内容
3.  RandomAccessFile对象包含一个记录指针，用以标示当前读写的位置。RandomAccessFile类对象可以自由移动记录指针：
    1.  `long getFilePointer()`：获取文件记录指针的**当前位置**
    2.  `void seek(long pos)`：将文件记录指针**定位**到pos位置

#### 2.构造器

1.  构造器
    1.  `public RandomAccessFile(File file, String mode)`
    2.  `public RandomAccessFile(String name, String mode)`
2.  创建 RandomAccessFile 类实例需要指定一个 mode 参数，该参数指定 RandomAccessFile 的访问模式：
    1.  r:  以**只读方式**打开
    2.  rw ：打开以**读取和写入**
    3.  rwd: 打开以读取和 写入；同步文件内容的更新
    4.  rws: 打开以便读取和 写入； 同步文件内容和元数据的更新
3.  如果模式为**只读r**。则不会创建文件，而是会去读取一个已经存在的文件，如果读取的文件不存在则会出现异常。 如果模式为**rw读写**。如果文件不存在则会去创建文件，如果存在则不会创建。

#### 3.应用

##### 1.实现对图片的复制操作

```java
		RandomAccessFile raf = null;
        RandomAccessFile raf2 = null;
        try {
            raf = new RandomAccessFile("爱情与友情.jpg", "r");
            raf2 = new RandomAccessFile("爱情与友情raf.jpg", "rw");

            byte[] buffer = new byte[1024];
            int len;
            while ( (len = raf.read(buffer)) !=-1){
                raf2.write(buffer,0,len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (raf != null) {
                try {
                    raf.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (raf2 != null) {
                try {
                    raf2.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
```

##### 2.在原有内容某处进行数据插入操作

```java
		RandomAccessFile raf = null;
        ByteArrayOutputStream baos = null;
        try {
            raf = new RandomAccessFile("hello.txt", "rw");
            // 将从4个字符开始后的内容复制
            raf.seek(3);
            // 方式一
//            保存指针3后面的所有数据到StringBuilder中
            /*StringBuilder sb = new StringBuilder((int)new File("hello.txt").length());
            byte[] buffer = new byte[1024];
            int len;
            while ( (len = raf.read(buffer)) != -1){
                sb.append(new String(buffer,0,len));
            }*/
            // 方式二
            baos = new ByteArrayOutputStream();
            byte[] buffer = new byte[10];
            int len;
            while((len = raf.read(buffer)) != -1){
                baos.write(buffer, 0, len);
            }

            //再次将指针调到角标为3的位置
            raf.seek(3);
            //在原有内容上进行覆盖操作，并不会覆盖原文件的所有内容
            raf.write("xyz".getBytes());
            //写入复制的内容
            // 方式一
            //将StringBuilder中的数据写入到文件中
//            raf.write(sb.toString().getBytes());
            // 方式二
            raf.write(baos.toString().getBytes());

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (baos != null) {
                try {
                    baos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (raf != null) {
                try {
                    raf.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
```

### 九.NIO.2概念及相关类

#### 1.NIO概述

1.  Java NIO (New IO，Non-Blocking IO)是从Java 1.4版本开始引入的一套新的IO API，可以替代标准的Java IO API。NIO与原来的IO有同样的作用和目的，但是使用的方式完全不同，**NIO支持面向缓冲区的(IO是面向流的)、基于通道的IO操作。**NIO将以更加高效的方式进行文件的读写操作。
2.  Java API中提供了两套NIO，**一套是针对标准输入输出NIO，另一套就是网络编程NIO。**
    1.  `java.nio.channels.Channel`
        1.  FileChannel: 处理本地文件  (标准输入输出NIO)
        2.  SocketChannel: TCP网络编程的客户端的Channel
        3.  ServerSocketChannel: TCP网络编程的服务器端的Channel
        4.  DatagramChannel：UDP网络编程中发送端和接收端的Channel
3.  随着 **JDK 7 的发布**，Java对NIO进行了极大的扩展，增强了对文件处理和文件系统特性的支持，以至于我们称他们为 NIO.2。

#### 2.核心API

1.  早期java只提供了File类访问文件系统,File类功能有限,提供的性能也不高,并且**大多数方法在出错时仅返回失败，并不会提供异常信息**.
2.  NIO.2为了弥补不足,引入了**Path接口**,代表一个平台无关的平台路径,扫描了目录结构中文件的置.**Path可以看成是File类的升级版本，实际引用的资源也可以不存在。**
3.  NIO.2在java.nio.file包下还提供了**Files、Paths工具类**，**Files包含了大量静态的工具方法来操作文件；Paths则包含了两个返回Path的静态工厂方法。**
4.  Paths 类提供的静态 get() 方法用来获取 Path 对象：
    1.  `static Path get(String first, String … more)` : 用于将多个字符串串连成路径
    2.  `static Path get(URI uri)`: 返回指定uri对应的Path路径
5.  示例:
    1.  `import java.io.File;`
        `File file = new File("index.html");`
    2.  `import java.nio.file.Path;`
        `import java.nio.file.Paths;`
        `Path path = Paths.get("index.html");`

#### 3.Path接口

```java
// Path常用方法
String toString() ： 返回调用 Path 对象的字符串表示形式
boolean startsWith(String path) : 判断是否以 path 路径开始
boolean endsWith(String path) : 判断是否以 path 路径结束
boolean isAbsolute() : 判断是否是绝对路径
Path getParent() ：返回Path对象包含整个路径，不包含 Path 对象指定的文件路径
Path getRoot() ：返回调用 Path 对象的根路径
Path getFileName() : 返回与调用 Path 对象关联的文件名
int getNameCount() : 返回Path 根目录后面元素的数量
Path getName(int idx) : 返回指定索引位置 idx 的路径名称
Path toAbsolutePath() : 作为绝对路径返回调用 Path 对象	(***)
Path resolve(Path p) :合并两个路径，返回合并后的路径对应的Path对象
File toFile(): 将Path转化为File类的对象	(***)
```

#### 4.Files类

```java
// java.nio.file.Files  用于操作文件或目录的工具类。
// Files 常用方法：
Path copy(Path src, Path dest, CopyOption … how) : 文件的复制
Path createDirectory(Path path, FileAttribute<?> … attr) : 创建一个目录	(***)
Path createFile(Path path, FileAttribute<?> … arr) : 创建一个文件	(***)
void delete(Path path) : 删除一个文件/目录，如果不存在，执行报错
void deleteIfExists(Path path) : Path对应的文件/目录如果存在，执行删除	(***)
Path move(Path src, Path dest, CopyOption…how) : 将 src 移动到 dest 位置
long size(Path path) : 返回 path 指定文件的大小	(***)

// Files 常用方法：用于判断
boolean exists(Path path, LinkOption … opts) : 判断文件是否存在
boolean isDirectory(Path path, LinkOption … opts) : 判断是否是目录
boolean isRegularFile(Path path, LinkOption … opts) : 判断是否是文件
boolean isHidden(Path path) : 判断是否是隐藏文件
boolean isReadable(Path path) : 判断文件是否可读
boolean isWritable(Path path) : 判断文件是否可写
boolean notExists(Path path, LinkOption … opts) : 判断文件是否不存在

// Files 常用方法：用于操作内容
SeekableByteChannel newByteChannel(Path path, OpenOption…how) : 获取与指定文件的连接，how 指定打开方式。
DirectoryStream<Path> newDirectoryStream(Path path) : 打开 path 指定的目录
InputStream newInputStream(Path path, OpenOption…how):获取 InputStream 对象
OutputStream newOutputStream(Path path, OpenOption…how) : 获取 OutputStream 对象
```

#### 5.示例

##### 1.Path接口示例

```java
import java.io.File;
import java.nio.file.Path;
import java.nio.file.Paths;

/**
 * 1.jdk 7.0 时，引入了 Path、Paths、Files三个类。
 * 2.此三个类声明在：java.nio.file包下。
 * 3.Path可以看做是java.io.File类的升级版本。也可以表示文件或文件目录，与平台无关
 * 4.如何实例化Path:使用Paths.
 * static Path get(String first, String … more) : 用于将多个字符串串连成路径
 * static Path get(URI uri): 返回指定uri对应的Path路径
 *
 * @author zlg
 * @create 2019 下午 2:44
 */
public class PathTest {

    //如何使用Paths实例化Path
    @Test
    public void test1() {
        Path path1 = Paths.get("d:\\nio\\hello.txt");//new File(String filepath)

        Path path2 = Paths.get("d:\\", "nio\\hello.txt");//new File(String parent,String filename);

        System.out.println(path1);
        System.out.println(path2);

        Path path3 = Paths.get("d:\\", "nio");
        System.out.println(path3);
    }

    //Path中的常用方法
    @Test
    public void test2() {
        Path path1 = Paths.get("d:\\", "nio\\nio1\\nio2\\hello.txt");
        Path path2 = Paths.get("hello.txt");

//		String toString() ： 返回调用 Path 对象的字符串表示形式
        System.out.println(path1);

//		boolean startsWith(String path) : 判断是否以 path 路径开始
        System.out.println(path1.startsWith("d:\\nio"));
//		boolean endsWith(String path) : 判断是否以 path 路径结束
        System.out.println(path1.endsWith("hello.txt"));
//		boolean isAbsolute() : 判断是否是绝对路径
        System.out.println(path1.isAbsolute() + "~");
        System.out.println(path2.isAbsolute() + "~");
//		Path getParent() ：返回Path对象包含整个路径，不包含 Path 对象指定的文件路径
        System.out.println(path1.getParent());
        System.out.println(path2.getParent());
//		Path getRoot() ：返回调用 Path 对象的根路径
        System.out.println(path1.getRoot());
        System.out.println(path2.getRoot());
//		Path getFileName() : 返回与调用 Path 对象关联的文件名
        System.out.println(path1.getFileName() + "~");
        System.out.println(path2.getFileName() + "~");
//		int getNameCount() : 返回Path 根目录后面元素的数量
//		Path getName(int idx) : 返回指定索引位置 idx 的路径名称
        for (int i = 0; i < path1.getNameCount(); i++) {
            System.out.println(path1.getName(i) + "*****");
        }

//		Path toAbsolutePath() : 作为绝对路径返回调用 Path 对象
        System.out.println(path1.toAbsolutePath());
        System.out.println(path2.toAbsolutePath());
//		Path resolve(Path p) :合并两个路径，返回合并后的路径对应的Path对象
        Path path3 = Paths.get("d:\\", "nio");
        Path path4 = Paths.get("nioo\\hi.txt");
        path3 = path3.resolve(path4);
        System.out.println(path3);

//		File toFile(): 将Path转化为File类的对象
        File file = path1.toFile();//Path--->File的转换

        Path newPath = file.toPath();//File--->Path的转换
    }
}
```

##### 2.Files类示例

```java
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.nio.channels.SeekableByteChannel;
import java.nio.file.*;
import java.util.Iterator;

/**
 * Files工具类的使用：操作文件或目录的工具类
 *
 * @author zlg
 * @create 2019 下午 2:44
 */
public class FilesTest {

	@Test
	public void test1() throws IOException{
		Path path1 = Paths.get("d:\\nio", "hello.txt");
		Path path2 = Paths.get("atguigu.txt");
		
//		Path copy(Path src, Path dest, CopyOption … how) : 文件的复制
		//要想复制成功，要求path1对应的物理上的文件存在。path1对应的文件没有要求。
//		Files.copy(path1, path2, StandardCopyOption.REPLACE_EXISTING);
		
//		Path createDirectory(Path path, FileAttribute<?> … attr) : 创建一个目录
		//要想执行成功，要求path对应的物理上的文件目录不存在。一旦存在，抛出异常。
		Path path3 = Paths.get("d:\\nio\\nio1");
//		Files.createDirectory(path3);
		
//		Path createFile(Path path, FileAttribute<?> … arr) : 创建一个文件
		//要想执行成功，要求path对应的物理上的文件不存在。一旦存在，抛出异常。
		Path path4 = Paths.get("d:\\nio\\hi.txt");
//		Files.createFile(path4);
		
//		void delete(Path path) : 删除一个文件/目录，如果不存在，执行报错
//		Files.delete(path4);
		
//		void deleteIfExists(Path path) : Path对应的文件/目录如果存在，执行删除.如果不存在，正常执行结束
		Files.deleteIfExists(path3);
		
//		Path move(Path src, Path dest, CopyOption…how) : 将 src 移动到 dest 位置
		//要想执行成功，src对应的物理上的文件需要存在，dest对应的文件没有要求。
//		Files.move(path1, path2, StandardCopyOption.ATOMIC_MOVE);
		
//		long size(Path path) : 返回 path 指定文件的大小
		long size = Files.size(path2);
		System.out.println(size);

	}

	@Test
	public void test2() throws IOException{
		Path path1 = Paths.get("d:\\nio", "hello.txt");
		Path path2 = Paths.get("atguigu.txt");
//		boolean exists(Path path, LinkOption … opts) : 判断文件是否存在
		System.out.println(Files.exists(path2, LinkOption.NOFOLLOW_LINKS));

//		boolean isDirectory(Path path, LinkOption … opts) : 判断是否是目录
		//不要求此path对应的物理文件存在。
		System.out.println(Files.isDirectory(path1, LinkOption.NOFOLLOW_LINKS));

//		boolean isRegularFile(Path path, LinkOption … opts) : 判断是否是文件

//		boolean isHidden(Path path) : 判断是否是隐藏文件
		//要求此path对应的物理上的文件需要存在。才可判断是否隐藏。否则，抛异常。
//		System.out.println(Files.isHidden(path1));

//		boolean isReadable(Path path) : 判断文件是否可读
		System.out.println(Files.isReadable(path1));
//		boolean isWritable(Path path) : 判断文件是否可写
		System.out.println(Files.isWritable(path1));
//		boolean notExists(Path path, LinkOption … opts) : 判断文件是否不存在
		System.out.println(Files.notExists(path1, LinkOption.NOFOLLOW_LINKS));
	}

	/**
	 * StandardOpenOption.READ:表示对应的Channel是可读的。
	 * StandardOpenOption.WRITE：表示对应的Channel是可写的。
	 * StandardOpenOption.CREATE：如果要写出的文件不存在，则创建。如果存在，忽略
	 * StandardOpenOption.CREATE_NEW：如果要写出的文件不存在，则创建。如果存在，抛异常
	 *
	 * @author shkstart 邮箱：shkstart@126.com
	 * @throws IOException
	 */
	@Test
	public void test3() throws IOException{
		Path path1 = Paths.get("d:\\nio", "hello.txt");

//		InputStream newInputStream(Path path, OpenOption…how):获取 InputStream 对象
		InputStream inputStream = Files.newInputStream(path1, StandardOpenOption.READ);

//		OutputStream newOutputStream(Path path, OpenOption…how) : 获取 OutputStream 对象
		OutputStream outputStream = Files.newOutputStream(path1, StandardOpenOption.WRITE,StandardOpenOption.CREATE);

      
//		SeekableByteChannel newByteChannel(Path path, OpenOption…how) : 获取与指定文件的连接，how 指定打开方式。
		SeekableByteChannel channel = Files.newByteChannel(path1, StandardOpenOption.READ,StandardOpenOption.WRITE,StandardOpenOption.CREATE);

//		DirectoryStream<Path>  newDirectoryStream(Path path) : 打开 path 指定的目录
		Path path2 = Paths.get("e:\\teach");
		DirectoryStream<Path> directoryStream = Files.newDirectoryStream(path2);
		Iterator<Path> iterator = directoryStream.iterator();
		while(iterator.hasNext()){
			System.out.println(iterator.next());
		}
	}
}
```

##### 3.使用jar包

```java
import org.apache.commons.io.FileUtils;
import java.io.File;
import java.io.IOException;

/**
 * 使用commons-io.jar包中的工具类FileUtils类
 *
 * @author zlg
 * @create 2019 上午 11:58
 */
public class FileUtilsTest {

    public static void main(String[] args) {

        // 使用FailUtils类实现非文本的复制
        File srcFile = new File("day10\\爱情与友情.jpg");
        File destFile = new File("day10\\爱情与友情2.jpg");

        try {
            FileUtils.copyFile(srcFile,destFile);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```



















