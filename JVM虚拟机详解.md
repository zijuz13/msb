## JVM虚拟机讲义
### 1.
![](./assets/1.png)
只要某种语言可以生成class文件，就可以交给jvm虚拟机解释执行
![](./assets/2.png)
![](./assets/3.png)
![](./assets/4.png)
![](./assets/5.png)
### 2.Class File Format
- Class文件16进制格式:
![](./assets/6.png)
- 二进制字节流
- 数据类型：u1 u2 u4 u8和_info（表类型）
_info的来源是hotspot源码中的写法
- 查看16进制格式的ClassFile
sublime/notepad
IDEA插件-BinEd
- 观察ByteCode的方法：
  1.javap
  2.JBE：可以直接修改
  3.JClassLib IDEA插件之一
- Class文件形式：
![](./assets/7.jpg)
![](./assets/8.jpg)
![](./assets/9.jpg)
![](./assets/10.jpg)
### 3,ClassLoader
![](./assets/11.png)
- verification判断class文件是否符合装入标准
- preparation是为class中的静态变量赋默认值，int为0，不是指定值
- 类加载器的层次图：
![](./assets/12.png)
### 4.DCL为什么非要用volatile
Double Check Locking双重检查锁：
```
private volatile Singleton singleton;
public Singleton getInstance(){
 if(null==singleton){
  synchronized(Singleton.class){
   if(null==singleton){
     singleton=new Singleton();
   }
  }
 }
 return singleton;
}
```
因为volatile可以防止指令重排序,具体原因如下：
![](./assets/13.png)
![](./assets/133.png)
对象创建过程中的汇编new是和c语言中的malloc一样先给对象进行内存分配，m值此时为0，然后invokespecial调用构造方法进行变量初始化，这时m变为8，最后astore_i把这个对象的地址赋给t变量，在这个过程中如果没有volatile的禁止指令重排序的功能，很有可能，astore_i和invokespecial的指令执行顺序进行了调换，最后返回了一个未进行初始化的对象，这个可能性很小，但是大厂容易问，因此双重检查锁需要对变量volatile修饰

### 5.Java对象内存布局
查看工具，使用maven添加Java Object Layout
![](./assets/15.png)
打印代码：
```
Stinrg c=ClassLayout.parseInstance(o).toPrintable();
```
分布:
![](./assets/16.png)
数组：
![](./assets/18.png)
普通对象中markword占8字节，class pointer指向这个类的字节码对象的地址，实例数据指向相应的数据，对齐是为了让所有的字节数可以被8整除
下面是Object对象的内存：
![](./assets/17.png)
markword包括
1.锁信息
2.gc信息
3.hashcode信息
### 6.对象的定位：
![](./assets/20.png)
句柄方式优点：方便进行gc，gc时不用改变t的值，而直接指针需要改变t的值

### 7.对象的分配：
能够在栈上分配的对象很高效，因为当方法调用完毕，栈弹出，就会自动对象没了,在栈上进行分配的时候要先进行逃逸分析
如果对象比较大需要分配到老年代中去
小对象直接分配在线程本地在eden区
![](./assets/22.png)
![](./assets/23.png)

Object obj=new Object();
占用16个字节，markword 8个字节
classpointer 4个字节 padding 4个字节
一共16个
### 8.Java的方法区
java的方法区可以看作是接口，而1.8以前的实现是permanent generation永久代以及1.8之后的meta space元空间
而class字节码对象是位于堆中的，并不是对象的class指针直接指向，虽然也可以这样说，很多教材都这样说，也可以当作是,在堆里的原因是，方便我们拿出来作java的反射使用
直接指向class对象位于堆的地址，但实际如下:
![](./assets/134.png)