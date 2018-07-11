---
layout:    post
title:    "关于 ADT 的一点笔记"
subtitle:    "introduction to ADT"
date:    2018-06-19 18:00:00
catalog:    true
tags:
    - java
    - 学习记录
---

#### **1、ADT 是什么？**
##### **1.1、ADT 的基本概念**

ADT，即抽象数据型（Abstract Data Type）包括两层含义，一层含义是**“抽象（abstract）”**这个概念：
* **抽象（Abstraction）**：用更简单，更高层次的想法来省略或隐藏低级细节；
* **模块（Modularity）**：将系统分成组件或模块，每个组件可以单独进行设计、实现、测试和复用；
* **封装（Encapsulation）**：在模块的内部和外部之间建立防火墙，外部不能直接访问模块内部，模块对内部行为负责；
* **信息隐藏（Information hiding）**：对系统的其他部分隐藏模块实现的细节，以便稍后可以更改这些细节而不更改系统的其他部分；

另一层含义就是抽象的数据：
* 由一组操作所刻画的数据类型，强调在这种数据上能进行什么样子的操作。

这个描述可能有点模糊。但实际上，现实生活中有很多这种操作性的定义。例如，对于物理学中的重力，我们对其本质并不了解，于是我们通过描述它表现出什么样子的行为来定义它本身。这和我们通过数据表现出的行为（能对数据进行操作）来定义数据是一样的。

##### **1.2、ADT 的一些性质**

前面说到，ADT 是由一组操作定义的，但这不意味着不需要数据的表示。实际上，操作性的定义只是给了 ADT 内部表示的自由，或者说，ADT 的使用者不必了解其内部是如何表示的。ADT 通过操作性的定义，描述了 ADT 具有的一些属性，ADT 的开发者就需要合理设计 ADT 的数据表示，来描述这些属性。

###### **1.2.1、表示独立性（Representation Independence）与表示泄露（representation exposure）**

**表示独立性**是说 ADT 的使用者使用 ADT 时无需考虑其内部如何实现。ADT 的实现者负责设计能够满足操作性定义的内部的表示，这些表示对 ADT 的使用者来说是不可见的，这就是所谓的“信息隐藏”。通过这种方式，实现者可以对 ADT 的内部表示进行随意的调整，而无需通知其使用者，因为这些表示会表现出相同的行为。

**表示泄露**就是说 ADT 的使用者通过某种方式得到了 ADT 的内部表示。若是使用者在代码中利用了 ADT 的内部表示，那么当实现者修改 ADT 的内部表示，使用者设计的这些代码就可能失去意义。

###### **1.2.2、不变量（Invariant）**

不变量，是指无论怎么操作 ADT 后均满足的一些属性，这是由 ADT 内部来维护的一些属性。例如，如果 ADT 描述了一个人，这个人包括一个身份证号的属性，那“身份证号不发生改变”就是这个 ADT 的一个不变量，ADT 必须保证它的操作不会改变其省份证号这个属性。

###### **1.2.3、抽象函数（Abstraction Function）和表示不变量（Rep Invariant）**

首先给出一 ADT，然后介绍两个辅助的概念。
{% highlight java %}
// 这是一个字符的集合（通常意义上的集合），包括集合的一些基本操作：交、并、补。
class CharSet {
	
	// 内部表示
	char[] characters;
	
	public void union(CharSet that) {
		// 集合的并操作
	}
	
	public void intersection(CharSet that) {
		// 集合的交操作
	}
	
	public void complete() {
		// 集合的补操作
	}
}
{% endhighlight %}

第一个概念就是**表示空间（the space of representation values）**。是指 ADT 的内部表示方式能够表示的全部数据组成的集合。在上面的`CharSet`这个 ADT 中，内部使用一个字符数组来表示，那么表示空间就是所有的字符数组组成的集合。

第二个概念就是**抽象空间（the space of abstract values）**。ADT 的使用者能够观察到的数据全体组成的集合。在上面的`CharSet`这个 ADT 中，用户能够观测到的应该是一个个由字符组成的无序、没有重复元素的集合，那么表示空间就是由这些集合组成的集合。

**抽象函数（Abstraction Function）**就是指一个从表示空间（R）到抽象空间（A）的映射。一般而言，这个映射不是单射，上面的`CharSet`这个 ADT 中，抽象函数的部分映射可能如下图：
![抽象函数的部分表示](/img/post/abstract-function.png)

可以看到，由`a, b, c`这三个元素组成的集合在`CharSet`内部有多个表示，具体的对应规则由设计者决定。比如我们可以规定`a, b, c`这三个元素组成的集合在`CharSet`内部只能由字符数组"abc"表示。于是，这里出现了一个新的问题：`CharSet`内部的表示"acb"等在抽象空间中就没有对应的元素。这就必须提到 ADT 的另一个属性表示不变量。

**表示不变量（Rep Invariant）**，用以限制 ADT 的内部表示，也是一个映射，是一个由表示空间（R）到布尔值的映射。其中在 RI 这个映射关系下，表示空间中的一个值 r 映射为 true 当且仅当 r 在抽象函数下映射为抽象空间中的一个值。表示空间中映射为 true 的值被称为合法的表示值，ADT 有义务保证其内部表示总是合法的表示值。

##### **1.3、ADT 的可变性**

我们将 ADT 分为两种类型：
* **可变的（Mutable）ADT**：是指 ADT 提供了能够修改其内部表示，使其映射为抽象空间中的另一个值的操作。这种 ADT 的一个实例可以通过其提供的操作改变为不同的可见值（抽象空间中的值）。
* **不可变的（Immutable）ADT**：是指没有提供上述的那种操作。这种 ADT 的一个实例总是表示同一个可见值。


##### **1.4、ADT 的操作类型**

通过操作对 ADT 的不同的效果，将 ADT 的操作分为四种类型：
* **构造器（Creators）**：（用其它类型的数据实例作为参数），返回一个当前类型的数据实例。
* **生产器（Producers）**：以当前类型的数据实例（和其它类型的数据实例）作为参数，返回一个当前类型的数据示例。
* **观察器（Observers）**：以当前类型的数据实例（和其它类型的数据实例）作为参数，返回一个其它类型的数据实例，用于观察当前类型的数据实例的状态。
* **变值器（Mutators）**：以当前类型的数据实例（和其它类型的数据实例）作为参数，通常不返回值，用于修改当前类型的数据实例的状态。

#### **2、ADT 的一种实现方式：OOP**

OOP，即面向对象程序设计（Object-oriented programming），以类（class）为中心概念，是 ADT 的一种实现技术。OOP 中有一些基本的概念：
* **类（class）**： 一个类即对应于一个 ADT。
* **属性（attribute）**：对应于 ADT 的内部表示，可以映射成抽象空间。
* **方法（method）**：对应于 ADT 的操作。
* **对象（object）**：一个对象即是某一个类的实例，拥有一组具体的状态。

类和 ADT 是高度相似的，可以说一个类就是一个 ADT 的程序化表示。因此，类的实现也需要遵循 ADT 的设计哲理。除此，OOP 的具体内容其实是很丰富的，包括但不限于多态、继承等，具体细节不在这里描述。

但是，有一条比较重要。在 java 中，我们总是持有某一个对象的引用。考虑如下的情况：
{% highlight java %}
public class Test {

	public static void main(String[] args) {
		Person person = new Person();
		List<Person> list = person.getFriends();
	}
}

// 这是对一个人的抽象
class Person {
	private List<Person> friends;
	
	// 返回某个人的朋友的列表
	public List<Person> getFriends() {
		return friends;  // 直接返回了内部表示的引用
	}
}
{% endhighlight %}

通过指令`List<Person> list = person.getFriends();`客户端会获得`person`这个对象的内部表示的引用，此时客户端就可以通过这个引用直接对`person`的内部表示进行修改，破坏了 ADT 的表示独立性。为了解决这个问题，提出一个新的概念：

* **防御式拷贝（defensive copy）**：指在需要返回内部表示时，将内容复制一份返回；或者是在需要将客户端提交的数据作为内部表示存储起来前，将内容复制一份，存储复制的内容。如下：
{% highlight java %}
public class Test {

	public static void main(String[] args) {
		Person person = new Person();
		List<Person> list = person.getFriends();
	}
}

// 这是对一个人的抽象
class Person {
	private List<Person> friends;
	
	// 返回某个人的朋友的列表
	public List<Person> getFriends() {
		List<Person> list = new ArrayList<>();
		list.addAll(friends);
		return list;   // 返回复制的副本
	}
}
{% endhighlight %}

这样，客户端就无法获得内部表示的引用了。



