---
title: 面向对象的五大基本原则
date: 2023-08-02 07:44:56
tags:
  - 设计模式
---

#### 1. 单一职责原则（SRP）
其核心思想为：一个类，最好只做一件事，只有一个引起它的变化。
一个类，最好有且仅有一个引起它变化的原因。
举个栗子，职员类里包括了普通员工、经理、老板，那类中势必需要用if else来区分判断，而且无论是这三种职员的需求发生变化，都会影响到整个职员类。
按照“单一职责原则”，将普通员工、经理、老板分别建一个类，既不用if else加以区分，也不会在修改某个职员类别的时候影响另一个。
#### 2. 开放封闭原则（OCP）
其核心思想是：软件实体应该是可扩展的，而不可修改的。
一个类，可以扩展（添加属性和功能），但是不要修改已经写好的属性和方法。
实现开开放封闭原则的核心思想就是对抽象编程，而不对具体编程，因为抽象相对稳定。
打个简单的比方，X的大舅二舅都是他舅，是有血缘关系的舅舅，如果突然冒出来一个跟他有血缘关系的三舅，那也是他舅舅。同时也不能改变他大舅和二舅的亲缘关系。
#### 3.里氏替换原则（LSP)
其核心思想是：子类必须能够替换其基类。
类A是类B的父类，那么在进行调用的时候，类A可以引用类B，但是反过来不行。
其实可以粗糙地理解为，类A就是对外提供一个接口，具体的实现在类B中。
实现的方法是面向接口编程：将公共部分抽象为基类接口或抽象类，通过Extract Abstract Class，在子类中通过覆写父类的方法实现新的方式支持同样的职责。
也就是说，其实里氏替换原则是继承和多态的综合体现。
#### 4. 依赖倒置原则（DIP)
其核心思想是：依赖于抽象。具体而言就是高层模块不依赖于底层模块，二者都同依赖于抽象；抽象不依赖于具体，具体依赖于抽象。
在对客观事物抽象成逻辑实体时，可以先思考，同类事物的共性是什么，将这个共性作为这类事物的“高层模块”，若干不同的客观事物作为“底层模块”在依赖”高层“之后，对共性进行特定描述。
举个栗子，苹果跟西瓜都是水果，水果的共同属性是水分、糖分。在这里，”水果“作为高层模块，其属性可以在描述“苹果”和“西瓜”的时候使用，所以“苹果”“西瓜”在此是“底层模块”。
#### 5. 接口隔离原则
其核心思想是：使用多个小的专门的接口，而不要使用一个大的总接口。
接口中定义属性和需要子类实现的方法，实现类必须完全实现接口的所有方法、属性。为什么要接口隔离呢？目的有二：

- 避免引用接口的类，需要实现其实用不到的接口方法、属性。
- 避免当接口修改的时候，有一连串的实现类需要更改。

分离的手段主要有以下两种：1、委托分离，通过增加一个新的类型来委托客户的请求，隔离客户和接口的直接依赖，但是会增加系统的开销。2、多重继承分离，通过接口多继承来实现客户的需求，这种方式是较好的。

- 委托分离，不直接使用原先的接口，可以用另外增加一个新的接口或类来实现需求。
- 多重继承分离，JDK源码、Spring框架使用了这种方式，后续的JDK源码解析系列会提及，好奇的朋友可以查看集合类的结构。