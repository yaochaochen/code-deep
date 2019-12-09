# Java序列化引发的血案

## 规范

【强制】 当序列化新增属性时，请不要修改serialVersionUID字段，以避免反序列失败；如果完全不兼容升级，

避免反序列化混乱，那么修改serialVersionUID值

**说明**：注意serialVersionUID值不一致会抛出序列化运行时异常

在规范中给出上面一段话，我们应该思考几个问题

- 序列化和反序列化是什么？
- 它的主要使用场景是那些？
- Java序列化常见的方案有那些？
- 常见序列化的区别是什么？
- 实际开发中的那些坑？

## 序列化和反序列化是什么？为什么需要它？

**序列化**是将内存的对象信息转化成可以存储或者传输的数据到临时永久存储的过程

**反序列化**是从临时或者永久存储中读取序列化的数据并转化成内存对象的过程

对象状态--序列化-->字节流

对象状态<--反序列化--字节流

### 那么为什么需要序列化和反序列化？

从本质上讲，文本文件图片视频等文件底层都是被转化为二进制字节流来传输的。对方得到文件进行解析，因为需要将文件对象等序列化成字节流传输。

实现java远程方法调用，就需要将调用结果通过网络传输给调用方，如果调用方和服务提供不在一台机器上就很难共享内存，就需要Java对象进行传输，而想Java的对象进行网络传输到文件中，就需要对象转化为二进制字节流。这就所谓的**Java序列化**

- 将远程方法调用的框架会用到序列化
- 将对象储存到文件时，需要序列化
- 将文件对象储存到缓存数据库需要序列化
- 通过序列化和反序列化的方法实现对象深浅拷贝

## 常见的序列化方式

- java常见原生序列化
- Hessian序列化
- Kryo序列化
- JSON序列化

### Java原生序列化

Serializable的源码非常简单只有声明 没有属性和方法

```java
/**
 * Serializability of a class is enabled by the class implementing the
 * java.io.Serializable interface. Classes that do not implement this
 * interface will not have any of their state serialized or
 * deserialized.  All subtypes of a serializable class are themselves
 * serializable.  The serialization interface has no methods or fields
 * and serves only to identify the semantics of being serializable. <p>
 *
 * To allow subtypes of non-serializable classes to be serialized, the
 * subtype may assume responsibility for saving and restoring the
 * state of the supertype's public, protected, and (if accessible)
 * package fields.  The subtype may assume this responsibility only if
 * the class it extends has an accessible no-arg constructor to
 * initialize the class's state.  It is an error to declare a class
 * Serializable if this is not the case.  The error will be detected at
 * runtime. <p>
 *
 * During deserialization, the fields of non-serializable classes will
 * be initialized using the public or protected no-arg constructor of
 * the class.  A no-arg constructor must be accessible to the subclass
 * that is serializable.  The fields of serializable subclasses will
 * be restored from the stream. <p>
 *
 * When traversing a graph, an object may be encountered that does not
 * support the Serializable interface. In this case the
 * NotSerializableException will be thrown and will identify the class
 * of the non-serializable object. <p>
 *
 * Classes that require special handling during the serialization and
 * deserialization process must implement special methods with these exact
 * signatures:
 *
 * <PRE>
 * private void writeObject(java.io.ObjectOutputStream out)
 *     throws IOException
 * private void readObject(java.io.ObjectInputStream in)
 *     throws IOException, ClassNotFoundException;
 * private void readObjectNoData()
 *     throws ObjectStreamException;
 * </PRE>
 *
 * <p>The writeObject method is responsible for writing the state of the
 * object for its particular class so that the corresponding
 * readObject method can restore it.  The default mechanism for saving
 * the Object's fields can be invoked by calling
 * out.defaultWriteObject. The method does not need to concern
 * itself with the state belonging to its superclasses or subclasses.
 * State is saved by writing the individual fields to the
 * ObjectOutputStream using the writeObject method or by using the
 * methods for primitive data types supported by DataOutput.
 *
 * <p>The readObject method is responsible for reading from the stream and
 * restoring the classes fields. It may call in.defaultReadObject to invoke
 * the default mechanism for restoring the object's non-static and
 * non-transient fields.  The defaultReadObject method uses information in
 * the stream to assign the fields of the object saved in the stream with the
 * correspondingly named fields in the current object.  This handles the case
 * when the class has evolved to add new fields. The method does not need to
 * concern itself with the state belonging to its superclasses or subclasses.
 * State is saved by writing the individual fields to the
 * ObjectOutputStream using the writeObject method or by using the
 * methods for primitive data types supported by DataOutput.
 *
 * <p>The readObjectNoData method is responsible for initializing the state of
 * the object for its particular class in the event that the serialization
 * stream does not list the given class as a superclass of the object being
 * deserialized.  This may occur in cases where the receiving party uses a
 * different version of the deserialized instance's class than the sending
 * party, and the receiver's version extends classes that are not extended by
 * the sender's version.  This may also occur if the serialization stream has
 * been tampered; hence, readObjectNoData is useful for initializing
 * deserialized objects properly despite a "hostile" or incomplete source
 * stream.
 *
 * <p>Serializable classes that need to designate an alternative object to be
 * used when writing an object to the stream should implement this
 * special method with the exact signature:
 *
 * <PRE>
 * ANY-ACCESS-MODIFIER Object writeReplace() throws ObjectStreamException;
 * </PRE><p>
 *
 * This writeReplace method is invoked by serialization if the method
 * exists and it would be accessible from a method defined within the
 * class of the object being serialized. Thus, the method can have private,
 * protected and package-private access. Subclass access to this method
 * follows java accessibility rules. <p>
 *
 * Classes that need to designate a replacement when an instance of it
 * is read from the stream should implement this special method with the
 * exact signature.
 *
 * <PRE>
 * ANY-ACCESS-MODIFIER Object readResolve() throws ObjectStreamException;
 * </PRE><p>
 *
 * This readResolve method follows the same invocation rules and
 * accessibility rules as writeReplace.<p>
 *
 * The serialization runtime associates with each serializable class a version
 * number, called a serialVersionUID, which is used during deserialization to
 * verify that the sender and receiver of a serialized object have loaded
 * classes for that object that are compatible with respect to serialization.
 * If the receiver has loaded a class for the object that has a different
 * serialVersionUID than that of the corresponding sender's class, then
 * deserialization will result in an {@link InvalidClassException}.  A
 * serializable class can declare its own serialVersionUID explicitly by
 * declaring a field named <code>"serialVersionUID"</code> that must be static,
 * final, and of type <code>long</code>:
 *
 * <PRE>
 * ANY-ACCESS-MODIFIER static final long serialVersionUID = 42L;
 * </PRE>
 *
 * If a serializable class does not explicitly declare a serialVersionUID, then
 * the serialization runtime will calculate a default serialVersionUID value
 * for that class based on various aspects of the class, as described in the
 * Java(TM) Object Serialization Specification.  However, it is <em>strongly
 * recommended</em> that all serializable classes explicitly declare
 * serialVersionUID values, since the default serialVersionUID computation is
 * highly sensitive to class details that may vary depending on compiler
 * implementations, and can thus result in unexpected
 * <code>InvalidClassException</code>s during deserialization.  Therefore, to
 * guarantee a consistent serialVersionUID value across different java compiler
 * implementations, a serializable class must declare an explicit
 * serialVersionUID value.  It is also strongly advised that explicit
 * serialVersionUID declarations use the <code>private</code> modifier where
 * possible, since such declarations apply only to the immediately declaring
 * class--serialVersionUID fields are not useful as inherited members. Array
 * classes cannot declare an explicit serialVersionUID, so they always have
 * the default computed value, but the requirement for matching
 * serialVersionUID values is waived for array classes.
 *
 * @author  unascribed
 * @see java.io.ObjectOutputStream
 * @see java.io.ObjectInputStream
 * @see java.io.ObjectOutput
 * @see java.io.ObjectInput
 * @see java.io.Externalizable
 * @since   JDK1.1
 */
public interface Serializable {
}
```

类结构变化还能保证正确的反序列化吗？

那么如果判断文件被修改过？ 通常可以通过加密算法进行签名，文件作为任何修改签名就不一致 但是Java的序列化的场景并不适应使用上述的方案，因为类文件的某些位置加上一个空格呢？这个签名是不应该改变的

 实现序列化接口 如果开发者不手动指定版本号ID怎么办？

既然Java序列化场景下的签名根据类的特点生成 我们是否可以不指定序列化版本号，就默认根据类名属性和函数计算呢？

#### 注释解析

- Java原生序列化需要实现Serializable接口，序列化接口不包含任何方法和属性等，它只起序列化标识作用

- 一个类实现序列化接口则其子类会继承序列化的能力，但是实现序列化接口的类中有其他对象引用，则其他对象也要实现序列化接口，序列化如果抛出 NotSerializableException异常说明该对象没有实现Serializable接口

- 每个序列化类都有SerializableUID的版本号，反序列化会校验带反射的类的序列化版本号和加载的序列化字节流版本号是否一致，如果版本号不一致会抛InvaidClassException异常

  强烈推荐每一个序列化类都手动指定其SerializableUID如果不指定的话，Java会默认生成序列版本号因为这个序列化版本号和类的特征编译器的实现都有关系，很容易抛InvaidClassException

-  提供了自定义序列化方法和反序列化方法writeObject和readObject

  

### Hessian序列化

Hessian 是一个动态类型，二进制序列化，也是一个基于对象传输的网络协议，Hessian 是一种跨语言的序列化方案，序列化后的字节数变少，效率更高。

### Kryo 序列化

Kryo 是一个快速高效的Java序列化和克隆工具。Kryo的目标是快速 字节少和易用 Kryo还可以自动实现深拷贝或者浅拷贝 Kryo的拷贝是对象到对象的拷贝，而不是对象到字节拷贝。

### JSON序列化

​	JSON序列化是将对象转成JSON字符串，JSON反序列化就是将JSON字符串转为对象





