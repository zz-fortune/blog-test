---
layout: post
title: "关于HashSet和ArrayList的一点记录"
subtitle: "hashCode and Equals in HashSet and ArrayList"
date: 2018-05-21 18:00:00
catalog: false
tags:
    - collection
    - java
    - 学习记录
---

在实验中，经常用到集合类（[Collection](https://docs.oracle.com/javase/10/docs/api/java/util/Collections.html)）
，对于集合类的一些基本的操作如`add`、`remove`、`contain`等有了一点自己的经验。不过关于集合类整个框架我也并非特别了解
，若是有疏漏之处，望包含指正。  

集合类主要是定义了一组接口，一组操作，至于实现倒是有很多种，可以根据不同的场景下来选择使用相应的实现，我这里不会一一比较
每一种实现的优劣。有句话是这么讲的：“使用`Set`时，不知道用什么就选择`HashSet`，使用`List`时，不知道用什么就用`ArrayList`”，
我今天就这两种常用的集合类上做点笔记。从某种意义上来讲，这两种实现可以将集合类的实现分为两种：一种是采用 hash 表的思想，
一种不是采用的 hash 表的思想（事实上，是采用的数组的思想）。两者在几个操作的具体实现上存在一个明显的区别（这也就是
使用 hash 表与否的区别）：在`HashSet`中，这几个方法都涉及到`hashCode`和`equals`这个方法，而另外一类实现中并不关心`hashCode`
这个方法。  

具体来说，`HashSet`讲数据结构维护成一个 hash 表，在添加元素时，首先会计算元素的 hash 值，根据 hash 值的不同将元素存放在不同的
 hash 桶中，存放在同一个 hash 桶中的元素会维护成一个顺序表；在搜索或者删除某个元素时，会先计算出来删除的元素的 hash 值，
根据这个 hash 值找到相应的 hash 桶，再在该桶的顺序表中按顺序检查时候有特定的元素（看到这里，想必也容易看出`HashSet`的效率
跟元素的`hashCode`的设计好坏是密切相关的，hash 值重复概率越小，检索元素时的代价就越小）。而`ArrayList`则是将数据结构维护成
一个数组结构的顺序表，效率比较固定。  

对于`ArrayList`来说，存储的元素无论是 mutable 的，还是 immutable 的，均会有相同的表现，而对于`HashSet`来说，由于其相当依赖于
 hash 值，而元素的 hash 值往往依赖于元素本身的属性，所以对于一个 mutable 对象，在放入`HashSet`中时，一个不慎就会引发不好的结果。
下面看一个实例：
 
{% highlight java %}

public class test {
	public static void main(String[] args) {
		Set<Human> set = new HashSet<>();
		List<Human> list = new ArrayList<>();
		Human human = new Human("zz", "M", 20);
		
		set.add(human);
		list.add(human);
		
		System.out.println("size of set: "+set.size()+", size of list: "+list.size());
		human.setName("test");
		System.out.println("size of set: "+set.size()+", size of list: "+list.size());
		System.out.println("set contains human? "+set.contains(human));
		System.out.println("list contains human? "+list.contains(human));
		
	}
}

class Human{
	
	String name;
	String sex;
	int age;
	
	public Human(String name, String sex, int age) {
		this.name = name;
		this.sex = sex;
		this.age = age;
	}

	
	public String getSex() {
		return sex;
	}
	
	public boolean setSex(String sex) {
		
		if (!sex.equals("M") && !sex.equals("F")) {
			return false;
		}
		this.sex = sex;
		return true;
	}
	
	public int getAge() {
		return age;
	}
	
	public boolean setAge(int age) {
		if (age<0) {
			return false;
		}
		this.age = age;
		return true;
	}

	public String getName() {
		return name;
	}
	
	public void setName(String name) {
		this.name = name;
	}
	
	@Override
	public boolean equals(Object obj) {
		if (obj==this) {
			return true;
		}
		
		if (!(obj instanceof Human)) {
			return false;
		}
		
		Human that = (Human)obj;
		if (!this.name.equals(that.name)) {
			return false;
		}
		
		if (!this.sex.equals(that.sex)) {
			return false;
		}
		
		if (this.age!=that.age) {
			return false;
		}
		return true;
	}
	
	@Override
	public int hashCode() {
		int hash = this.name.hashCode();
		hash = 31*hash + this.sex.hashCode();
		hash = 31*hash + this.age;
		return hash;
	}
	

}
{% endhighlight %}

程序的运行结果如下：


> size of set: 1, size of list: 1  
> size of set: 1, size of list: 1  
> set contains human? false  
> list contains human? true  

可见，在`HashSet`中，查找（或删除）元素时只会在元素对应的 hash 桶去寻找，只要元素的 hash 值发生了改变，
哪怕是两个元素指向同一个内存引用，也无法找到初始存入的元素。

