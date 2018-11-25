### 枚举及其常见的使用

---

#### 看源码得到的内容

``` java
package java.lang;

/*
 * ......
 * @since   1.5
 */
public abstract class Enum<E extends Enum<E>>
        implements Comparable<E>, Serializable {
        ......
}

```

1. java.lang.Enum 是 java.lang 包的一个类，java.lang 包里面的类不用显示 import 。
2. 它是一个抽象类，从 jdk 1.5 版本开始提供。
3. 它是一个泛型类，实现了 Comparable 接口和 Serializable 接口。 

#### 提出问题

##### 这个泛型到底有什么用？

可以看到这个泛型 E ，使用到它的地方是 Comparable 接口，即需要实现的 compareTo 方法会使用到这个泛型。

##### 这个泛型的类型要是 Enum\<E\> 的子类，看起来有点递归的意思，那么怎么赋值这个泛型的类型？

我们平时使用 enum 关键字定义枚举，底层会自动赋值这个泛型的类型就是我们自定义的这个枚举的类型，这里编写一个示例，让我们反编译来看看它的怎么实现的。
```  java
package test;

public enum EnumTest {
	INSTANCE;

	String instance;

	private EnumTest() {
		System.out.println("lazy load");
		instance = "instance";
	}

	public String getInstance() {
		return instance;
	}

}

```

我们使用 javap 命令进行反编译看看它的实现。
``` java
// >javap EnumTest
// 警告: 二进制文件EnumTest包含test.EnumTest
// Compiled from "EnumTest.java"
public final class test.EnumTest extends java.lang.Enum<test.EnumTest> {
  public static final test.EnumTest INSTANCE;
  java.lang.String instance;
  static {};
  public java.lang.String getInstance();
  public static test.EnumTest[] values();
  public static test.EnumTest valueOf(java.lang.String);
}

```

这里可以看到很多内容：

1. 可以看到我们定义的枚举 EnumTest 本质上还是一个类，继承自 Enum 类，泛型被赋值为 EnumTest。这样就找到了我们怎么样去设置这个泛型的类型。很明显这样赋值符合 \<E extends Enum\<E\>\> 的方式。

2. 我们定义的枚举类被 final 修饰，他不会被其他子类继承。

3. 我们自定义的一个泛型元素被设置成一个静态常量，它会在 java 内存的方法区有唯一实例，它是 EnumTest 类的一个实例。


#### 枚举的使用

1. switch 中可以使用枚举，防止越界等。

2. **单例模式中使用，懒汉模式，线程安全，可序列化。 **

常见的单例模式有饿汉式，懒汉式。

饿汉式可以直接设置成静态常量，静态代码块，保证线程安全，并且直接初始化。

懒汉式可以使用 volatile 加上双重检查，静态内部类的方式，实现延迟加载和线程安全。

但是上述方式没有实现序列化，需要自己进行序列化，我们可以利用 Enum 实现支持序列化的单例模式，对于线程安全和懒汉模式有些类似与静态变量和静态内部类结合的方式。

代码：
```  java
package test;

public enum EnumTest {
	INSTANCE;

	String singleton;

	private EnumTest() {
		System.out.println("lazy load");
		singleton = "singleton";
	}

	public String getInstance() {
		return instance;
	}

}

```
测试类：
```  java
package test;

public class ATest {
	public static void main(String[] args) throws InterruptedException {
		System.out.println("waiting");
		Thread.sleep(3000);
		String singleton = EnumTest.INSTANCE.getInstance();
		System.out.println(singleton);
		System.out.println("over");
	}
	
}

```
结果：
```
waiting
lazy load
instance
over
```


