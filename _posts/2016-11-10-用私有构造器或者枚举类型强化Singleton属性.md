---
layout: post
title: "用私有构造器或者枚举类型强化Singleton属性"
description: "实现单例模式通常有3种方法:   1.静态成员   2.静态工厂方法   3.单元素枚举类型"
category: 技术
tags: [技术, java]
---
{% include JB/setup %}


实现单例模式通常有3种方法

### 1.静态成员

```java
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();

  private Elvis() {
  }

  public void leaveTheBuilding() {
    System.out.println("Whoa baby, I'm outta here!");
  }

  // This code would normally appear outside the class!
  public static void main(String[] args) {
    Elvis elvis = Elvis.INSTANCE;
    elvis.leaveTheBuilding();
  }
}
```

### 2.静态工厂方法

```
public class Elvis {
  private static final Elvis INSTANCE = new Elvis();

  private Elvis() {
  }

  public static Elvis getInstance() {
    return INSTANCE;
  }

  public void leaveTheBuilding() {
    System.out.println("Whoa baby, I'm outta here!");
  }

  // This code would normally appear outside the class!
  public static void main(String[] args) {
    Elvis elvis = Elvis.getInstance();
    elvis.leaveTheBuilding();
  }
}
```

**但是这两种方法不能保证全局只有一个对象，例如可以通过反射机制，设置`AccessibleObject.setAccessible(true)`，改变构造器的访问属性，调用构造器生成新的实例。**

```
Constructor<?> constructor = Elvis.class.getDeclaredConstructors()[0];
constructor.setAccessible(true);
Elvis instance = (Elvis) constructor.newInstance();
```

要抵御这总攻击，可以修改构造器，让它在第二次创建实例的时候抛出异常。

```
public class Elvis {
    private static int count = 0;
    private static final Elvis INSTANCE = new Elvis();

    private Elvis() {
        if(count > 0) {
            throw new IllegalArgumentException("Cannot create Elvis twice");
        }
        count++;
    }

    public static Elvis getInstance() {
        return INSTANCE;
    }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    public static void main(String[] args) throws InstantiationException, InvocationTargetException, IllegalAccessException {
        Elvis instance = Elvis.getInstance();
        instance.leaveTheBuilding();

        //use reflection to instantiate a object
        Constructor<?> constructor = Elvis.class.getDeclaredConstructors()[0];
        constructor.setAccessible(true);
        Elvis instance2 = (Elvis) constructor.newInstance();
        instance2.leaveTheBuilding();
    }
}
```

异常：

```
Whoa baby, I'm outta here!
Exception in thread "main" java.lang.reflect.InvocationTargetException
  at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
  at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
  at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
  at java.lang.reflect.Constructor.newInstance(Constructor.java:422)
  at effective.singleton.Client.main(Client.java:17)
Caused by: java.lang.IllegalArgumentException: Cannot create Elvis twice
  at effective.singleton.Elvis.<init>(Elvis.java:12)
  ... 5 more
```

如果上面两种方法实现的Singleton是可以序列化的，加上 implements Serializable只保证它可以序列化，为了保证反序列化的时候，实例还是Singleton，必须声明所有的实例域都是transient的，并且提供 readResolve方法，否则，每次反序列化都会生成新的实例。

反例，下面的例子每次反序列化都会生成新的实例：

```
public class Elvis implements Serializable{
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() {
    }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    // This code would normally appear outside the class!
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        Elvis elvis = Elvis.INSTANCE;
        elvis.leaveTheBuilding();

        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("elvis.ser"));
        oos.writeObject(elvis);
        oos.close();

        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("elvis.ser"));
        Elvis o = (Elvis) ois.readObject();
        ois.close();
        System.out.println(elvis == o);
    }
}
```

输出：

```
Whoa baby, I'm outta here!
false
```

加入readResolve方法后：

```
public class Elvis implements Serializable{
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() {
    }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    private Object readResolve() {
        // Return the one true Elvis and let the garbage collector
        // take care of the Elvis impersonator.
        return INSTANCE;
    }

    // This code would normally appear outside the class!
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        Elvis elvis = Elvis.INSTANCE;
        elvis.leaveTheBuilding();

        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("elvis.ser"));
        oos.writeObject(elvis);
        oos.close();

        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("elvis.ser"));
        Elvis o = (Elvis) ois.readObject();
        ois.close();
        System.out.println(elvis == o);
    }
}
```

输出：

```
Whoa baby, I'm outta here!
true
```

### 3.单元素枚举类型

```
public enum Elvis {
  INSTANCE;

  public void leaveTheBuilding() {
    System.out.println("Whoa baby, I'm outta here!");
  }

  // This code would normally appear outside the class!
  public static void main(String[] args) {
    Elvis elvis = Elvis.INSTANCE;
    elvis.leaveTheBuilding();
  }
}
```

通过枚举实现Singleton更加简洁，同时枚举类型无偿地提供了序列化机制，可以防止反序列化的时候多次实例化一个对象。枚举类型也可以防止反射攻击，当你试图通过反射去实例化一个枚举类型的时候会抛出`IllegalArgumentException`异常：

```
Exception in thread "main" java.lang.IllegalArgumentException: Cannot reflectively create enum objects
  at java.lang.reflect.Constructor.newInstance(Constructor.java:416)
  at org.effectivejava.examples.chapter02.item03.enumoration.Elvis.main(Elvis.java:21)
```


**所以单元素的枚举类型是实现Singleton的最佳方法**