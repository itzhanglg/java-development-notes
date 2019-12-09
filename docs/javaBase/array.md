### 一.概述

-   数组(Array)，是多个**相同类型数据**按**一定顺序排列**的集合，并使用**一个名字命名**，并通过**编号**的方式对这些数据进行统一管理
-   数组常见概念: 数组名, 下标(或索引), 元素, 数组的长度
-   数组本身是**引用数据类型**，而数组中的元素可以是**任何数据类型**，包括基本数据类型和引用数据类型
-   创建数组对象会在内存中开辟一整块**连续的空间**，而数组名中引用的是这块连续空间的首地址
-   数组的**长度一旦确定，就不能修改**
-   可以直接通过下标(或索引)的方式调用指定位置的元素，速度很快
-   数组的分类:
    -   按照维度: 一维数组、二维数组、三维数组、…
    -   按照元素的数据类型分: 基本数据类型元素的数组、引用数据类型元素的数组(即对象数组)

### 二.一维数组的使用

#### 1.声明

`type var[]; ` 或 `type[] var; `

例如: int a[]; int[] a; String[] b;

 Java语言中声明数组时不能指定其长度(数组中元素的数)， 例如： int a[5];   // 非法

#### 2.初始化

1.  动态初始化 ：**数组声明且为数组元素分配空间与赋值的操作分开进行**

    ```java
    int[] arr = new int[3];
    arr[0] = 3;
    arr[0] = 6;
    arr[0] = 9;
    ```

2.  静态初始化 ：**在定义数组的同时就为数组元素分配空间并赋值**

    ```java
    int arr[] = new int[]{3,6,9};
    // 或
    int[] arr = {3,6,9};

    // 定义数组长度为0, 数组的长度可以为0
    int[] a = new int[]{};
    int[] a = {};
    ```

#### 3.数组元素的引用

-   定义并用**运算符new为之分配空间**后，才可以引用数组中的每个元素
-   数组元素的引用方式：**数组名[数组元素下标]**
    -   数组元素下标可以是整型常量或整型表达式。如a[3] , b[i] , c[6*i]
    -   **数组元素下标从0开始；长度为n的数组合法下标取值范围: 0 —> n-1**
-   每个数组都有一个属性**length**指明它的长度: **数组一旦初始化, 其长度是不可变的**

#### 4.元素默认初始化值

数组是引用类型，它的元素**相当于类的成员变量**，因此数组**一经分配空间**，其中的每个元素也被按照成员变量同样的方式被**隐式初始化**

| 数组元素类型  | 元素默认初始值               |
| ------- | --------------------- |
| byte    | 0                     |
| short   | 0                     |
| int     | 0                     |
| long    | 0L                    |
| float   | 0.0F                  |
| double  | 0.0                   |
| char    | 0 或写为: '\u0000'(表现为空) |
| boolean | false                 |
| 引用类型    | null                  |

#### 5.示例

数组遍历:

```java
// 数组的赋值和遍历输出
public class Test{
    public static void main(String args[]){
        int[] s;	// 局部变量:栈空间
        s = new int[10];	// 在堆内存为int[]对象分配空间,并隐式初始化为默认值
        //int[] s=new int[10];
        // 基本数据类型数组在显式赋值之前，自动赋默认值
        for ( int i=0; i<10; i++ ) {
            s[i] =2*i+1;	//显示初始化
            System.out.println(s[i]);	//输出
        }
    }
}
```

对象数组:

声明和初始化:  `类名称[] 对象数组名称 = new 类名称[对象个数];`

```java
// 对象数组的使用
Student[] stu = new Student[10];
for(int i=0; i<stu.length; i++){
  	Student s = new Student("Jack"+i,18+i);		//实例化对象
  	stu[i] = s;    //将对象放入数组
}
for(int i=0; i<stu.length; i++){
  	System.out.println(stu[i].name+" "+stu[i].age);		//打印输出每个对象的属性
}
```

### 三.二维数组的使用

对于二维数组(**数组中的数组**)的理解，我们可以看成是一维数组array1又作为另一个一维数组array2的元素而存在。其实，**从数组底层的运行机制来看，其实没有多维数组**

#### 1.初始化

1.动态初始化: `int[][] arr = new int[3][2];`

-   定义了名称为arr的二维数组
-   二维数组中有3个一维数组
-   每一个一维数组中有2个元素
-   一维数组的名称分别为arr[0], arr[1], arr[2]
-   给第一个一维数组1脚标位赋值为78写法是： `arr[0][1] = 78;`

2.动态初始化: `int[][] arr = new int[3][]`

-   二维数组中有3个一维数组
-   每个一维数组都是默认初始化值null
-   可以对这个三个一维数组分别进行初始化
-   `arr[0] = new int[3]; arr[1] = new int[1]; arr[2] = new int[2];`

3.静态初始化: `int[][] arr = new int[][]{{1,3,5},{2,4},{6,8,5,6}};`或 `int[][] a = {(1,2,3),(4,5,6),(7,8,9)};`

-   定义一个名称为arr的二维数组，二维数组中有三个一维数组
-   每一个一维数组中具体元素也都已初始化
-   第一个一维数组`arr[0] = {1,3,5};`
-   第二个一维数组`arr[1] = {2,4};`
-   第三个一维数组`arr[2] = {6,8,5,6};`
-   第三个一维数组的长度表示方式`arr[2].length;`
-   注意特殊写法情况：int[] x,y[]; x是一维数组，y是二维数组

#### 2.示例

二位数组的遍历:

```java
String[][] s = new String[2][3];
for(int i=0; i<s.length; i++){    //s.length表示二维数组的行数
    for(int j=0; j<s[i].length; j++){    //s[i].length表示二维数组的列数
      	s[i][j] = "第"+i+"行,第"+j+"列";    //将字符串对象存入数组
    }
}
//遍历输出二维数组的值  --嵌套循环
for(int i=0; i<s.length; i++){    
    for(int j=0; j<s[i].length; j++){    //遍历每个一维数组
      	System.out.print(s[i][j]+" ");
    }
    System.out.println();
}
```

不规则二维数组遍历:

```java
// a.定义
int[][] a = new int[5][];    //定义行,即a[0]-a[4],a[i]都是数组对象的引用
a[0] = new int[4];    //定义a[0]所实际引用的数组对象,即第一行由4个元素组成
a[1] = new int[3];	  
a[2] = new int[1];
a[3] = new int[2];
//没有定义a[4],即第五行开辟的空间,没有显示初始化,a[4]默认值为null

// b.数组的遍历
for(int i=0; i<a.length; i++){
    System.out.print("第"+(i+1)+"行:");
    if(a[i] == null){	//判断数组对象的引用是否是null
      	System.out.print(a[i]);
      	continue;
    }
    for(int j=0; j<a[i].length; j++){
      	System.out.print(a[i][j]+" ");
    }
    System.out.println();
}
```

### 四.常用两种排序

#### 1.冒泡排序

![](../../media/pictures/003.gif)

```java
// 定右边数字
int[] a = {1,3,2,6,4,8};
for(int i=0; i<a.length-1; i++){	//轮数
    for(int j=0; j<a.length-1-i; j++){	//次数
        if(a[j]>a[j+1]){
          	int temp = a[j];
          	a[j] = a[j+1];
          	a[j+1] = temp;
        }
    }
}
```

#### 2.选择排序

![](../../media/pictures/004.gif)

```java
// 定左边数字
int[] flag = new int[2];    //中间容器
for(int i=0; i<a.length-1; i++){    //a.length轮只需要比较a.length-1轮
  	flag[0] = a[i];
  	flag[1] = i;
    for(int j=i+1; j<a.length; j++){	//比较最小值,放入中间容器
      	if(a[i]>a[j]){		
        	flag[0] = a[j];
        	flag[1] = j;
     	}
	}
  	if(flag[1] != i){    //换位置
    	int temp = a[i];
    	a[i] = flag[0];
    	a[flag[1]] = temp;
  	}
}

// 简单写法
int[] a = {1,3,2,6,4,8};
for(int i=0; i<a.length-1; i++){	//轮数
    for(int j=i+1; j<a.length; j++){	//次数
        if(a[i]>a[j]){
          	int temp = a[i];
          	a[i] = a[j];
          	a[j] = temp
        }
    }
}
```

### 五.其它使用

#### 1.Arrays工具类的使用

java.util.Arrays类即为操作数组的工具类，包含了用来操作数组（比如排序和搜索）的各种方法

| 方法                                | 描述                 |
| --------------------------------- | ------------------ |
| boolean equals(int[] a,int[] b)   | 判断两个数组是否相等         |
| String toString(int[] a)          | 输出数组信息             |
| void fill(int[] a,int val)        | 将指定值填充到数组之中        |
| void sort(int[] a)                | 对数组进行排序            |
| int binarySearch(int[] a,int key) | 对排序后的数组进行二分法检索指定的值 |

示例:

```java
// 利用sort()方法对数组进行排序
import java.util.Arrays;
public class SortTest {
    public static void main(String[] args) {
        int[] numbers = {5,900,1,5,77,30,64,700};
        Arrays.sort(numbers);
        for(int i = 0; i < numbers.length; i++){
        	System.out.println(numbers[i]);
        }
    }
}
```

#### 2.数组使用中的常见异常

1.  **数组脚标越界**异常(ArrayIndexOutOfBoundsException)

    ```java
    // 访问到了数组中的不存在的脚标时发生
    int[] arr = new int[2];
    System.out.println(arr[2]);
    ```

2.  **空指针**异常(NullPointerException)

    ```java
    // arr引用没有指向实体，却在操作实体中的元素时
    int[] arr = null;
    System.out.println(arr[0]);
    ```

    ​





