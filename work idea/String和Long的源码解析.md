# String和Long的源码解析

## String

### 不变性

所谓不可变性是指一旦被初始化，就不能再改变了，如果修改将会是全新的类或者是值

``String s = "hello"``

``s = "world"``

对上面的来看，s的值好像会被修改，但是如果在idea里debug可以看出来s的内存地址已经被修改，其实s的引用指向了一个新的地址了。

可以从源码上找到点端倪

- 类结构

- ```java
  public final class String
      implements java.io.Serializable, Comparable<String>, CharSequence {
  ```

  String的类是final修改，就说明String这个类无法被覆盖继承的，其实这样设计不仅说明String是不可变之外，还存在Java的整个生态体系的复杂性和完整性

  - 常量

    ```java
    /** The value is used for character storage. */
    private final char value[];
    
    /** Cache the hash code for the string */
    private int hash; // Default to 0
    ```

    这两个常量很容易看出 value[]的常量说明String是对char的数组操作,value也是被final修饰，一旦被赋值，就不能修改，内存地址也无法修改，而且更值得注意的是，value是private修饰的外部绝对访问不到（利用反射可以）

    hash相当于String的缓存hashCode

    从类接口和常量两点可以说明一点String的绝大部分的方法都会返回新的String。

    #### equals相等判断

    String提供了2种相等的判断的方法，equals和equalsIgnoreCase后者忽略大小写

    ```java
    public boolean equals(Object anObject) {
      //this指向本地地址 判断内存地址是否相同
        if (this == anObject) {
            return true;
        }
      //如果比较的对象不是String类型的直接返回false
        if (anObject instanceof String) {
          //下面将长度拆分，对char对比
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
    ```

    从equals的源码可以总结出String底层数据结构其实就是char的数组一样，判断相等时，就是挨个比较char

    从设计角度考虑，判断两者是否相等可以参考String#equals方法，这是一个比较经典的代码

    下面都不赘述String的方法实现了，其实就是简单的chars数组操作

    

    #### 反射摧毁String的不可变性

    ```java
    String testContent = "Hello, Wold";
    
    String otherContent = "yaocc";
    System.out.println("反射修改前的 testContent: " + testContent);
    // private final char value[];
    Field valueField = String.class.getDeclaredField("value");
    //设置访问检查
    valueField.setAccessible(true);
    //替换
    valueField.set(testContent, otherContent.toCharArray());
    System.out.println("反射后的 testContent: " + testContent);
    ```

  ## Long

  其实很容易忽视这个Long的，而且Long的存在缓存机制，值得借鉴设计思想

  ### 缓存

  Long自己实现了一套 独特的缓存机制，缓存了-128-127的Long值，果然Long的值在这个范围内，就不会初始化

  而从缓存中拿，源码如下

  ```java
  private static class LongCache {
      private LongCache(){}
  	//+1是因为存在一个0值
      static final Long cache[] = new Long[-(-128) + 127 + 1];
  	//初始化 放在static块进行加载 JVM字节码提升
      static {
          for(int i = 0; i < cache.length; i++)
              cache[i] = new Long(i - 128);
      }
  }
  ```

  如果命中缓存就会减少开销，但是parseLong没有这个机制

