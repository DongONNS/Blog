---
title: SerialVersionUid的作用
date: 2021-01-26 22:00:00
tags:
categories: java
---

序列化运行时与每个可序列化的类关联一个版本号，称为`serialVersionUID`，在反序列化期间使用该版本号来验证序列化对象的发送者和接收者是否已加载了该对象的与序列化兼容的类。如果接收者为对象加载的类`serialVersionUID`与相应的发送者的类不同，则反序列化将导致 [`InvalidClassException`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/InvalidClassException.html)。可序列化的类可以`serialVersionUID`通过声明一个`serialVersionUID`必须为static，final和type 的字段来显式声明其自身`long`：

```java
ANY-ACCESS-MODIFIER static final long serialVersionUID = 42L;
```

如果可序列化的类未显式声明a `serialVersionUID`，则序列化运行时将根据`serialVersionUID`该类的各个方面为该类计算默认值，如Java™对象序列化规范中所述。但是，*强烈建议*所有可序列化的类显式声明`serialVersionUID`值，因为默认的情况下`serialVersionUID`计算对类详细信息高度敏感，而类详细信息可能会根据编译器的实现而有所不同，因此可能导致`InvalidClassExceptions`反序列化期间发生意外情况。因此，为了保证`serialVersionUID`不同Java编译器实现之间的值一致，可序列化的类必须声明一个显式`serialVersionUID`值。还强烈建议明确`serialVersionUID`声明尽可能使用private修饰符，因为此类声明仅适用于立即声明的类`serialVersionUID`字段，不能用作继承成员。