## java序列化作战指南

##### 序列化动机

​	1:持久化对象内容	2:传输对象内容

##### 序列化准备

```java
public class User implements Serializable
或
public class User implements Externalizable
/**
辨析：
	后者有两个抽象方法：
		public void writeExternal(ObjectOutput out) throws IOException
		public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException
		作用之后再讲
*/
```

##### 一般序列化与反序列化方法

```java
//序列化对象到文件中,由ObjectOutputStream执行，“template”为持久化文件，user为一个对象
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("template"));
        oos.writeObject(user);	
//反序列化,由ObjectInputStream执行，
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file));
        User newUser = (User)ois.readObject();
```

##### 定制化序列与反序列过程

```java
//法1:加修饰
private static T attr1;	//静态变量不会序列化
private Transient T attr2;	//Transient修饰的变量也不会序列化，反序列化后会变为初值
//法2:类内实现以下方法。此时，ObjectOutputStream和ObjectInputStream会改变默认实现，而采用该方法的实现
//通常出现在collation包的实现里
private void writeObject(ObjectOutputStream oos) throws IOException {...}
private void readObject(ObjectInputStream ois) throws IOException, ClassNotFoundException {...}
//法3:继承Externalizable并实现抽象方法，效果同上。用此法，则类必有无参构造
public class User implements Externalizable{
  @Override public void writeExternal(ObjectOutput out) throws IOException{}
	@Override public void readExternal(ObjectInput in) throwsIOException,ClassNotFoundException{}
}

```

### 进阶

##### 待序列化类重写writeObject和readObject，是两个private的方法。并且它们既不存在于java.lang.Object，也没有在Serializable中声明。那么ObjectOutputStream如何使用它们的呢

```java
以ObjectInputStream 为例：调用链为：
readObject() -> readObject0(boolean unshared) -> checkResolve(readOrdinaryObject(unshared)) -> readSerialData(obj, desc) -> slotDesc.invokeReadObject(obj, this) -> readObjectMethod.invoke(obj, new Object[]{ in })
这显然是一个通过反射进行的方法调用。
再看 :
ObjectStreamClass(final Class<?> cl) {
  ...
		readObjectMethod = getPrivateMethod(cl, "readObject",new Class<?>[] { ObjectInputStream.class },Void.TYPE)
   ...
}
getPrivateMethod意在获取 non-static private method，可知通过私有重写writeObject和readObject方法会被ObjectOutputStream通过反射的方式优先调用
```

##### 关于collection包中的序列化方式

```
该包中大多数类如HashSet、ArrayList等都被Transient修饰了，可依旧能正常序列化，原因是其重新实现了writeObject和readObject（见上），遍历集合内容单独进行序列化
```

##### 序列化出来的二进制内容

```java
//例
aced 0005 7372 0018 636f 6d2e 7973 6c2e
5365 7269 616c 697a 6162 6c65 5465 7374
ffff ffff ffff ffff 0200 0149 0003 6e75
6d78 7000 0007 e2

第一部分是序列化文件头
AC ED ：STREAM_MAGIC声明使用了序列化协议
00 05 ：STREAM_VERSION序列化协议版本
73 ：TC_OBJECT声明这是一个新的对象

第二部分是序列化的类的描述，在这里是SerializableTest
72 ：TC_CLASSDESC声明这里开始一个新的class
00 18：class名字的长度是24个字节
636f 6d2e 7973 6c2e 5365 7269 616c 697a 6162 6c65 5465 7374：SerializableTest的完整类名
ffff ffff ffff ffff：serialVersionUID，序列化ID，如果没有指定，则会由算法随机生成一个8字节的ID
02 ：标记号，声明该类支持序列化
00 01：该类所包含的域的个数为1

第三部分是对象中各个属性的描述
49：域类型，49代表I，也就是int类型
00 03：域名字的长度为3
6e 75 6d：num属性的名称

第四部分为对象的父类信息描述
SerializableTest没有父类，如果有，和第二部分的描述相同
78 ：TC_ENDBLOCKDATA，对象块的结束标志
70：TC_NUL：说明没有其他超类的标志

第五部分为对象属性的实际值
如果属性是一个对象，那么这里还将序列化这个对象，规则和第二部分一样
00 0007 e2：数值2018

结论：
	1:类的信息都在二进制文件里，反序列化时就是根据这些信息定位类和属性的
		主要包括类名、属性类型、属性名
	2:反序列化时根据类名和serialVersionUID相校验，不一致则序列化失败
	3:待补充
```

##### 序列化与反序列化运作机制

```java
/*序列化：以ObjectOutputStream 为例，它在序列化的时候会依次调用 writeObject()→writeObject0()→
writeOrdinaryObject()→writeSerialData()→invokeWriteObject()→defaultWriteFields()*/
private void defaultWriteFields(Object obj, ObjectStreamClass desc) throws IOException {
        Class<?> cl = desc.forClass();
        desc.checkDefaultSerialize();

        int primDataSize = desc.getPrimDataSize();
        desc.getPrimFieldValues(obj, primVals);
        bout.write(primVals, 0, primDataSize, false);

        ObjectStreamField[] fields = desc.getFields(false);
        Object[] objVals = new Object[desc.getNumObjFields()];
        int numPrimFields = fields.length - objVals.length;
        desc.getObjFieldValues(obj, objVals);
        for (int i = 0; i < objVals.length; i++) {
            try {
                writeObject0(objVals[i], fields[numPrimFields + i].isUnshared());
            }
        }
}
/*反序列化：以 ObjectInputStream 为例，它在反序列化的时候会依次调用 readObject()→readObject0()→
readOrdinaryObject()→readSerialData()→defaultReadFields()。*/
private void defaultReadFields(Object obj, ObjectStreamClass desc) throws IOException {
        Class<?> cl = desc.forClass();
        if (cl != null && obj != null && !cl.isInstance(obj)) {
            throw new ClassCastException();
        }
	...
        int objHandle = passHandle;
        ObjectStreamField[] fields = desc.getFields(false);
        Object[] objVals = new Object[desc.getNumObjFields()];
        int numPrimFields = fields.length - objVals.length;
        for (int i = 0; i < objVals.length; i++) {
            ObjectStreamField f = fields[numPrimFields + i];
            objVals[i] = readObject0(f.isUnshared());
            if (f.getField() != null) {
                handles.markDependency(objHandle, passHandle);
            }
        }
        if (obj != null) {
            desc.setObjFieldValues(obj, objVals);
        }
        passHandle = objHandle;
}
//这部分知识看详细 https://mp.weixin.qq.com/s/7sRNnY7ZFklCW38_fEjA9g
```

##### 其他事项

1、当父类继承Serializable接口时，所有子类都可以被序列化

2、子类实现了该接口，父类没有，父类的属性不能序列化（不报错，数据会丢失），但是子类可以正常序列化

3、如果序列化的属性是对象，则这个对象也必须实现该接口，否则会报错

4、在反序列化的时候，如果对象的属性有修改或删减，则修改或删减的部分属性会丢失。但不会报错