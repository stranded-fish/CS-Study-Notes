# Java 常用 API 总结

该文总结了 Java 在算法题练习过程中所常用的 API。

目录：

- [Java 常用 API 总结](#java-常用-api-总结)
  - [字符串](#字符串)
  - [数组](#数组)
  - [集合框架](#集合框架)
    - [List 等 待完成](#list-等-待完成)
  - [待完成](#待完成)
  - [参考链接](#参考链接)

## 字符串

1\. 获取指定索引位置的字符

`charAt()` 方法返回指定索引处的字符。

```java
String str = "hello world";
char myChar = str.charAt(4); // myChar = 'o'
```

注：索引下标从 0 开始。当目标索引超过字符串长度时，会抛出 `java.lang.StringIndexOutOfBoundsException: String index out of range: ***` 错误。

## 数组

## 集合框架

**1\. 遍历 Map 对象**

**eg 1. 在 for-each 循环中使用 entries 来遍历**

```java
Map<Integer, Integer> map = new HashMap<Integer, Integer>();

for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
  System.out.println("Key = " + entry.getKey() + ", Value = " + entry.getValue());
}
```

**eg 2. 在 for-each 循环中遍历 keys**

```java
Map<Integer, Integer> map = new HashMap<Integer, Integer>();
 
//遍历map中的键
for (Integer key : map.keySet()) {
  System.out.println("Key = " + key);
}
```

**eg 3. 在 for-each 循环中遍历 values**

```java
Map<Integer, Integer> map = new HashMap<Integer, Integer>();

for (Integer value : map.values()) {
  System.out.println("Value = " + value);
}
```

**eg 4. 通过 Iterator 遍历**

使用泛型：

```java
Map<Integer, Integer> map = new HashMap<Integer, Integer>();
Iterator<Map.Entry<Integer, Integer>> entries = map.entrySet().iterator();
while (entries.hasNext()) {
  Map.Entry<Integer, Integer> entry = entries.next();
  System.out.println("Key = " + entry.getKey() + ", Value = " + entry.getValue());
} 
```

不使用泛型：

```java
Map map = new HashMap();
Iterator entries = map.entrySet().iterator();
while (entries.hasNext()) {
  Map.Entry entry = (Map.Entry) entries.next();
  Integer key = (Integer)entry.getKey();
  Integer value = (Integer)entry.getValue();
  System.out.println("Key = " + key + ", Value = " + value);
}
```

**eg 5. 通过键值遍历 （效率低、不推荐）**

```java
Map<Integer, Integer> map = new HashMap<Integer, Integer>();
for (Integer key : map.keySet()) {
  Integer value = map.get(key);
  System.out.println("Key = " + key + ", Value = " + value);
}
```

注意：如果想要通过下标访问该集合，可通过将其赋值到 List 集合中实现。

```java
ArrayList<Map.Entry<Integer, Integer>> arrayList = new ArrayList<>();

for (Map.Entry<Integer, Integer> entry : linkedHashMap.entrySet()) {
  arrayList.add(entry);
}

for (int i = 0; i < arrayList.size(); i++) {
  System.out.println("key: " + arrayList.get(i).getKey() + " value: " + arrayList.get(i).getValue());
}
```

### List 等 待完成

## 待完成

## 参考链接

* [Java中如何遍历Map对象的4种方法](https://blog.csdn.net/tjcyjd/article/details/11111401)
