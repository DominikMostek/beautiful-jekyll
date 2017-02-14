---
layout: post
title: Classes as Map keys in Java
permalink: 2016-02-12-class-value
---
By accident I came across this [interesting class](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassValue.html). I suggest you to read its source code, its very unusual. But I struggled to understand the purpose of this class. The reason for its existence is even more interesting. 

If you need to store the relation between two objects you use a [Map](https:u//docs.oracle.com/javase/8/docs/api/java/util/Map.html), right? But when the key instance is of [class](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html) type it gets more complicated. Best explanation can be found here [JDK-6389107](https://bugs.openjdk.java.net/browse/JDK-6389107). 
**My TLDR version of it:**
When the key in a Map is Class instance, this class instance now cannot be garbage collected. No big deal you might add. But class holds reference to its class loader and this class loader holds references to all classes it has loaded. Now that might be a problem. It can also lead to locking the jar files containing the class files. There are some solutions but they all suck.

And here comes the ClassValue utility. It maps classes to some other representation. It does it lazily and caches the result. 
ClassValue is also thread safe. Thread safety is implemented in very unusual way. Read [get method](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassValue.html#get-java.lang.Class-) and [remove method](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassValue.html#remove-java.lang.Class-) javadoc to grasp the idea. Reading the source code is not that easy though. 

**Be careful when using Classes as keys in your maps!**

Some links: <br />
[SO answer with some info abou Class value](http://stackoverflow.com/a/32146848/694677) <br />
[Is using the Class instance as a Map key a best practice?](http://stackoverflow.com/questions/2625546/is-using-the-class-instance-as-a-map-key-a-best-practice)
