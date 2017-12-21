---
layout:     post
title:      Anomalies in Java
date:       2017-12-21 14:00:00
summary:    Anomalies in Java
categories: Java
commentIssueId: 2
---


In this post, we present a catalog of anomalies that developers
perform in Java. For each anomaly, we include name, description,
benefits, code example, and tools that include the anomaly in their
catalog (if any). Each anomaly could be associated to one or more code
metrics such as bug fixing, clarity, performance.

Field, Parameter, Local Variable Could Be Final
-----------------------------------------------

Besides classes and methods, developers can use the `final` modifier in fields,
parameters, and local variables. The semantic differs for each one of
these usages. A final class cannot be extended, a final method cannot be
overridden, and final fields, parameters, and local variables cannot
change their value. Thus, a final modifier guarantees that fields,
parameters, and local variables cannot be re-assigned. A re-assignment
generates an error at compile-time. Final modifier improves clarity,
helps developers to debug the code showing constructors that change
state and are more likely to break the code. In addition, final modifier
allows the compiler and virtual machine to optimize the code. This
anomaly is included in tools such as PMD. IDEs such as Eclipse and
Netbeans can be configured to add final modifiers to fields, parameters,
and local variables automatically on saving. Fig. \[fig:final\] shows a
code example of adding the final modifier to a parameter. Variable `a` is
assigned a single time. Thus, it can be declared final such as variable `b`.

```java
public class Bar {
 public void foo() {
  String a = "a"; //if a is not assigned again it is better to do this:
  final String b = "b";
 }
```

Allows Type Inference for Generic Instance Creation
---------------------------------------------------

Since Java 7, developers can replace the type arguments required to
invoke the constructor of a generic class with an empty set of type
parameters (&lt;&gt;)[^6]. The empty set of type
parameters, also known as diamond, allows the compiler to infer type
arguments from the context. By using diamond construction, developers
make clear the use of generic instead of the deprecated raw types, the
version of a generic type without type arguments. Java allows raw types
only to ensure compatibility with pre-generics code. The benefit of the
diamond constructor, in this context, is clarity since it is more
concise. Fig. \[fig:removediamond\] shows the use of the diamond
operator in a variable declaration. Instead of using the type parameter `<String, List<String>>`
, developers can use the diamond to invoke the constructor of generic `HashMap`
class.

```java
//Bad
Map < String, List < String >> myMap = new HashMap < String, List < String >> ();
//Good
Map < String, List < String >> myMap = new HashMap < > ();   
```

Remove Raw Type
---------------

Java discourages the use raw types. A raw type denotes a generic type
without type arguments, which was used in the out-dated version of Java
and is allowed to ensure compatibility with pre-generics code. Since
type arguments of raw types are unchecked, they can cause compiler error
at run-time. Java compiler generates warning to indicate the use of raw
types into the source code. Fig. \[fig:rawtype\] shows the use of a raw
type. Developers can pass any type of collection to the constructor of a
raw type since it is unchecked.

```java
List<String> strings = ... // some list that contains some strings
// Totally legal since you used the raw type and lost all type checking!
List<Integer> integers = new LinkedList(strings);
// Not legal since the right side is actually generic!
List<Integer> integers = new LinkedList<>(strings);
```

Prefer `Class<?>`
-----------------

Java prefers `Class<?>` over plain `Class` although these constructions are
equivalent[^2]. The benefit of `Class<?>` is clarity since developers
explicitly indicates that they are aware of not using an out-dated Java
construction. The Java compiler generates warning on the use of `Class`.
Fig. \[fig:class\] exemplifies the use of `Class<?>`.

```java
    //Any class
    Class anyType = String.class;
    //Unknown Type
    Class <?> unknownType = String.class;
```

Use Variadic Functions
----------------------

Variadic functions denote functions that use variable-length arguments
(varargs). This feature was introduced in Java 5 to indicate that the
method receives zero or more arguments. Prior to Java 5, if a method
receives a variable number of arguments, developers have to create
overload method for each number of arguments or to pass an array of
arguments to the method. The benefit of using varargs is simplicity
since developers do not need to create overload methods and use the same
notation independently of the number of arguments. For the compiler
perspective, the method receives an array as parameter.
Fig. \[fig:varargs\] shows the use of the Variadic functions.

```java
//...
myMethod("foo", "bar");
myMethod("foo", "bar", "baz");
myMethod(new String[] {
 "foo",
 "var",
 "baz"
}); // you can even pass an array
//...
public void myMethod(String...strings) {
 for (String whatever: strings) {
  // do what ever you want
 }
 // the code above is is equivalent to
 for (int i = 0; i < strings.length; i++) {
  // classical for. In this case you use strings[i]
 }
}
```

String to Character
-------------------

In Java, we can represent a character both as a string or a character.
For some operations such as concatenating or appending a value to a / it
is better to represent the value as a character if the value itself is a
character. Representing the value as a character improves performance.
For instance, this edit improves from 10-25% the performance at the
Guava project[^7]. This transformation is included in the catalog of
anomalies of tools such as PMD.

```java
public class Foo {
 void bar() {
  StringBuffer sb = new StringBuffer();
  // Avoid this
  sb.append("a");

  // use instead something like this
  StringBuffer sb = new StringBuffer();
  sb.append('a');
 }
}
```
StringBuffer to StringBuilder
-----------------------------

`StringBuffer` and `StringBuilder` denote a mutable sequence of characters. These two types are
compatible, but `StringBuilder` provides no guarantee of synchronizations. Since
synchronization is rarely used, `StringBuilder` offers right performance over its
counterpart. If developers want to synchronize a `StringBuilder`, they can
surround the code block with a synchronized operator `synchronized(sb){}`. This class is
designed to replace `StringBuffer` in places where was being used by a
single thread[^5]. Java recommends the use of `StringBuilder` in
preference to `StringBuffer` due to performance. Fig. \[fig:stringbuilder\] shows an
example of the use of the `StringBuilder` class.

```java
public class Bar {
 public void foo() {
  StringBuffer a = new StringBuffer(); //In a single thread, prefer the following:
  StringBuilder b = new StringBuilder();
 }
}
```

Use Collection `isEmpty`
-----------------------

The use of `isEmpty` is encouraged to verify whether the list contains no elements
instead of verifying the size of a collection. Although in the majority
of collections, these two constructions are equivalent, for other
collections computing the size of an arbitrary list could be expensive.
For instance, in the class `ConcurrentSkipListSet`, the size method is not a constant-time
operation[^4]. This transformation is
included in the catalog of anomalies of tools such as PMD.
Fig. \[fig:isempty\] shows an example of use of the `isEmpty` method.

```java
public class Foo {
 void good() {
  List foo = getList();
  if (foo.isEmpty()) {
   // blah
  }
 }

 void bad() {
  List foo = getList();
  if (foo.size() == 0) {
   // blah
  }
 }
}
```

Prefer String Literal `equals` Method
-------------------------------------

The equals method is widely used in software development. Some usages
can cause due to the right-hand side of the method object reference
being null. When using the `equals` method to compare some variable to a String
Literal, developers could overcome null point errors by allowing the
string literal to call the `equals` method because a string literal is never
null. Since Java string literal equals method checks for null, we do not
need to check for null explicitly when calling the equals method of a
string literal.

```java
public class Foo {
  //Good
  void good(String str) {
    if ("string".equals(str)) {
     // blah
    }
   }
 // Causes error if str is null
 void bad(String str) {
   if (str.equals("string") {
     // blah
    }
 }
 // Do not need to check for null since equals evaluate it.
 void bad2(String str) {
   if (str != null && str.equals("string") {
     // blah
   }
 }
}
 ```

Prefer String Constant `equals` Method
--------------------------------------

The equals method is widely used in software development. Some usages
can cause due to the right-hand side of the `equals` method object reference
being null. When using the `equals` method to compare some variable to a string
constant, developers could overcome null point errors by allowing the
string constant to call the `equals` method because a constant is rarely null.
Since Java string literal equals method checks for null, we do not need
to check for null explicitly when calling the equals method of a string
literal.

```java
public class Foo {
  //Good
  void good(String str) {
    if (CONSTANT.equals(str)) {
      // blah
    }
  }
  // Causes error if str is null
  void bad(String str) {
    if (str.equals(CONSTANT) {
      // blah
    }
  }
  // Do not need to check for null since equals evaluate it.
  void bad2(String str) {
    if (str != null && str.equals(CONSTANT) {
      // blah
    }
  }
}
```

Use `valueOf` instead Wrapper Constructor
-----------------------------------------

Java allows to use the method `valueOf` or the constructor to create wrapper
objects of primitive types. Java recommends the use of `valueOf` for
performance purpose since `valueOf` method caches some values. This checker is
included in the catalog of anomalies of tools such as Sonar.
Fig. \[fig:isempty\] shows an example of the use of the `valueOf` method.

```java
Integer a = new Integer(1); //Instead of using the Integer constructor, use the valueOf
Integer b = Integer.valueOf(1);
```

Avoid using `FileInputStream`/`FileOutputStream`
------------------------------------------------

`FileInputStream` and `FileOutputStream` override `finalize()`. As a result, objects of these classes go to a category
that is cleaned only when a full clearing is performed by the Garbage
Collector[^1]. Since Java 7, developers can use
counterpart to improve program performance. This anomaly is described as
a bug by Java JDK[^3].

```java
//Bad
public void writeToFile(String fileName, byte[] content) throws IOException {
  try (FileOutputStream os = new FileOutputStream(fileName)) {
   os.write(content);
  }
 }
 //Good
public void writeToFile(String fileName, byte[] content) throws IOException {
 try (OutputStream os = Files.newOutputStream(Paths.get(fileName))) {
  os.write(content);
 }
}
```

##References



[^1]:	https://dzone.com/articles/ fileinputstream-fileoutputstream-considered-harmful

[^2]:	Bruce Eckel. 2005. Thinking in Java (4th Edition). Prentice Hall PTR, Upper Saddle River, NJ, USA.

[^3]:	Java JDK. 2017. Relax FileInputStream/FileOutputStream requirement to use finalize. (2017). At https://bugs.openjdk. java.net/browse/JDK-8187325. Accessed in 2017, December 19.

[^4]:	Oracle. 2017. Class ConcurrentSkipListSet<E>. (2017). At https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ ConcurrentSkipListSet.html. Accessed in 2017, December 19.
  
[^5]:	Oracle. 2017. Class StringBuilder. (2017). At https://docs.oracle.com/javase/8/docs/api/java/lang/StringBuilder.html. Accessed in 2017, December 19.

[^6]:	Oracle. 2017. Type Inference for Generic Instance Creation. (2017). At https://docs.oracle.com/javase/7/docs/technotes/ guides/language/type-inference-generic-instance-creation.html. Accessed in 2017, December 19.

[^7]: https://github.com/google/guava/commit/8f48177132547cee2943c93837d76b898154d722

