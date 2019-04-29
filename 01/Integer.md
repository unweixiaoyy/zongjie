### Integer

##### Integer和int区别

```java
1.int是基本数据类型 Integer是封装类
2.int的初始值是0，Integer是 null .

应用场景的区别：
比如要体现出 考试成绩为0和缺考的区别的时候 用Integer可以 int不行
比如用容器的时候 ，ArrayList等只能放对象，不能放基本数据类型。
```

##### 比较Integer==int和Integer==Integer

Integer直接赋值和valueOf会缓存-128到127的对象引用，
但如果赋值和valueOf超过这个范围或者是new的Integer，那便是新的对象了，2个新的对象使用==比较内存地址肯定不相同。
Integer和int比的就是值。

Integer静态内部类IntegerCache，维护了一个-128到127的Integer数组。当为Integer直接赋值或者valueOf时去缓存里面取。

示例代码
```java
public class TestInteger {

    public static void main(String[] args) {
        Integer i1 = 1;                     
        Integer i2 = 1;                     
        Integer i3 = Integer.valueOf(1);    
        Integer i4 = new Integer(1);        
        Integer i5 = new Integer(1);        

        Integer i11 = 128;                  
        Integer i12 = 128;                  
        Integer i13 = Integer.valueOf(128); 
        Integer i14 = new Integer(128);     
        Integer i15 = new Integer(128);     

        System.out.println(i1 == i2);           true
        System.out.println(i1 == i3);           true
        System.out.println(i1 == i4);           false
        System.out.println(i3 == i5);           false
        System.out.println(i4 == i5);           false
        System.out.println("--------------");
        System.out.println(i11 == i12);         false
        System.out.println(i11 == i13);         false
        System.out.println(i11 == i14);         false
        System.out.println(i13 == i14);         false
        System.out.println(i14 == i15);         false
        //Integer直接赋值和valueOf会缓存-128到127的对象引用，
        //但如果赋值和valueOf超过这个范围或者是new的Integer，那便是新的对象了，新的对象使用==比较内存地址肯定不相同
        System.out.println("--------------");
        System.out.println(i1 == 1);            true
        System.out.println(i2 == 1);            true
        System.out.println(i3 == 1);            true
        System.out.println(i4 == 1);            true
        System.out.println(i5 == 1);            true
        System.out.println(i11 == 128);         true
        System.out.println(i12 == 128);         true
        System.out.println(i13 == 128);         true
        System.out.println(i14 == 128);         true
        System.out.println(i15 == 128);         true
        //Integer和int比的就是值。
        System.out.println("--------------");
    }

}
```

##### 打印结果

```java
true
true
false
false
false
--------------
false
false
false
false
false
--------------
true
true
true
true
true
true
true
true
true
true
--------------
```


















