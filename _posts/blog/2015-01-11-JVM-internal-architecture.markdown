---
layout: post
title:  "JVM Internal Architecture"
date:   2015-01-11
categories: blog
tags: [java,memory]
author: shobhit_garg
share: true
comments: true
excerpt:
---

__Java APIs :__ Set of run time libraries that give you standard way to access the system resources of host computer.When you run a java program JVM loads the JAVA API class files that are referred by your program's class file. They are platform dependent.

Java Virtual Machine(JVM) uses two types of class loaders:

__1.Bootstrap ClassLoader:__ It's a primary class loader which loads the main method class, Java APIs etc.It's the part of JVM itself. It is written in __native language__.Native langague is the language other than Java which can be executed without using JVM.


__2.User Defined ClassLoader:__ They usually are written in Java.There can be many user defined class loaders.They are the instance of subclass of java.lang.ClassLoader.

![Diagram2]({{ site.url }}/assets/fig1-6.gif)

When a java application starts a runtime instance of JVM is created.__Each java application runs inside it's own JVM__.JVM instance starts running it's app by invoking the main method of some initial class.There are two types of of threads in JVM:

1. __Daemon Thread:__ Ordinary thread used by JVM like thread used by garbage collector.

2. __Non-Daemon Thread:__ Initial thread of app that starts main method.

Each JVM has a __Class loader subsystem__ and a __Execution engine__.The job of former is to loading classes and later provides a mechanism responsible for executing instructions.

![Diagram2]({{ site.url }}/assets/screenshot2.jpg)


JVM organizes the memory it needs to execute the program into several run time data areass like pc Register,method area,java stack,heap,run-time constant pool(allocated from method area) and native methods stacks.


Each instance of the Java virtual machine has one __method area__ and one __heap__.These two areas are shared by all threads running inside the virtual machine.

__Method Area__ : It basically stores the class data for the loaded type:

* The fully qualified name of the type

* The fully qualified name of the typeʹs direct superclass (unless the type is an interface or class java.lang.Object, neither of which have a superclass)

* Whether or not the type is a class or an interface

* The typeʹs modifiers ( some subset of` public, abstract, final)

* An ordered list of the fully qualified names of any direct superinterfaces

* The constant pool for the type(A constant pool is an ordered set of constants used by the type, including literals (string, integer, and floating point
constants) and symbolic references to types, fields, and methods.)

* Field information

* Method information

* All class (static) variables declared in the type, except constants

* A reference to class ClassLoader who loads the class.

* A reference to class Class.(With each object JVM creates a object of class Class)

(JVM can get info like class is loaded or not through method area.Class datas are linked with each other through references.)

__Heap__ : Heap stores the run time object data.





Every new thread which JVM starts has a __PC register__ and a __Java stack__.No other thread can access it.Java stack stores the state of Java non native method invocation like local variables,parameters,return value,intermediate calculations etc.(Stack of native method invocation is stored in an implementation dependent way in native methods stacks as well as in registers.).JVM __pushes and pops Stacks frames in Java stacks__.A frame stores state of one java method invocation.Each frame has three parts: local variables, operand stack(stores data during operations) and frame data(data to support normal method run,constant pool resolution ,exception dispatch etc.).The size of these frames are defined in terms of word size which is implementation specific.When a thread invokes a native method JVM simply dynamically link to and directly invoke the native method.(not pushing the frame to java stack.)

__Method Table__ : Some implementation uses this data structure.Its basically a array of direct references to all instance methods.It's included as a part of class data stored in method area.

For more detailed into please read [artima blog][ab].

[ab]: http://www.artima.com/insidejvm/ed2/jvm.html
