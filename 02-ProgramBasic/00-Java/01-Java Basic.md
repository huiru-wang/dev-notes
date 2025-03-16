


# 关键字

## final

声明数据为常量，可以是编译时常量，也可以是在运行时被初始化后不能被改变的常量。

- 对于基本类型，final 使数值不变；
- 对于引用类型，final 使引用不变，也就不能引用其它对象，但是被引用的对象本身是可以修改的。
- 对于方法：声明方法不能被子类重写；
- 对于类：声明类不允许被继承；如：String；


# 隐式类型转换

只能将变量<font color="#f79646">向上隐式类型转换</font>，不可向下，因为向下存在数据丟失的可能

# 重写与重载

**重写（Override）**：存在于继承体系中，指子类实现了一个与父类在方法声明上完全相同的一个方法；
- 子类方法的访问权限必须大于等于父类方法；
- 子类方法的返回类型必须是父类方法返回类型或为其子类型。
- 子类方法抛出的异常类型必须是父类抛出异常类型或为其子类型。
使用 @Override 注解，可以让编译器帮忙检查是否满足上面的三个限制条件；

**重载（Overload）**：存在于同一个类中，指一个方法与已经存在的方法名称上相同，但是参数类型、个数、顺序至少有一个不同；
- 返回值不同，其它都相同不算是重载


# Object

```java
public native int hashCode()

public boolean equals(Object obj)

protected native Object clone() throws CloneNotSupportedException

public String toString()

public final native Class<?> getClass()

protected void finalize() throws Throwable {}

public final native void notify()

public final native void notifyAll()

public final native void wait(long timeout) throws InterruptedException

public final void wait(long timeout, int nanos) throws InterruptedException

public final void wait() throws InterruptedException
```

## equals

判断2个对象是否等价（非相等，相等代表同一个对象）

equals通常用来判断对象的各项字段是否相等，也可以自定义重写equals；





# String / StringBuffer / StringBuilder

**1. 可变性**
- `String` 不可变
- `StringBuffer` 和 `StringBuilder` 可变

**2. 线程安全**
- `String` 不可变，因此是线程安全的
- `StringBuilder` 不是线程安全的
- `StringBuffer` 是线程安全的，内部使用 synchronized 进行同步

## String

```java
// s1 和 s2 采用 new String() 的方式新建了两个不同字符串（前提是 String Pool 中还没有 "abc" 字符串对象）
String s1 = new String("aaa");
String s2 = new String("aaa");
System.out.println(s1 == s2);           // false

// 通过 s1.intern() 和 s2.intern() 方法取得同一个字符串引用
String s3 = s1.intern();
String s4 = s2.intern();
System.out.println(s3 == s4);           // true

// 字面量的形式创建字符串，会自动地将字符串放入 String Pool 中
String s5 = "bbb";
String s6 = "bbb";
System.out.println(s5 == s6);  // true
```

不可变性：
- String 不可变性可以保证参数不可变；
- String 不可变性天生具备线程安全，可以在多个线程中安全地使用；
- String 是不可变的，才可能使用 String Pool（缓存池）；



# 法参数传递机制（Pass by value）

Java 的参数是以<font color="#f79646">值传递</font>的形式传入方法中，而不是引用传递。

无论是基本数据类型、还是引用类型：
1. 基本数据类型：传递值的拷贝；
2. 引用数据类型：可以理解为传递引用的拷贝；指向同一个内存对象；

# 浅拷贝和深拷贝

