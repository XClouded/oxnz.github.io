---
layout: post
title: public、protected和private继承
date: 2014-04-23 19:59:20.000000000 +08:00
type: post
published: true
status: publish
categories:
- C++
tags:
- inherit
meta:
  _edit_last: '1'
author:
  login: oxnz
  email: yunxinyi@gmail.com
  display_name: Will Z
  first_name: Will
  last_name: Z
---
<p>网上找了很多关于c++中这三者在继承方面的文章，貌似都不够全面。将几篇我觉得不错的文章整合了一下，方便大家查阅。</p>
<p><!--more--></p>
<p>首先贴一个转载比较多的帖子：</p>
<p><a href="http://blog.csdn.net/wuliming_sc/archive/2009/01/18/3827743.aspx">http://blog.csdn.net/wuliming_sc/archive/2009/01/18/3827743.aspx</a></p>
<p>&nbsp;</p>
<p>在C++ 中继承主要有三种关系：public 、protected 和private 。这三种继承关系中public 继承是最为常用的一种继承关系，private 继承是最少见的继承关系。<br />
1. public<br />
从语义角度上来说，<strong>public 继承是一种接口继承</strong> ，根据面向对象中的关系而言就是，派生类可以代替基类完成基类接口所声明的行为，也就是必须符合“Liskov 替换原则（LSP ）” ，此时派生类可以自动转换成为基类的接口，完成接口转换。</p>
<p>从语法角度上来说，<strong>public 继承会保留基类中成员（包括函数和变量等）的可见性不变，也就是说，基类的public 成员为派生类的public 成员，基类的protected 成员为派生类的protected 成员</strong> 。</p>
<p>2. protected<br />
从语义角度上来说，<strong>protected 继承是一种实现继承</strong> ，根据面向对象中的关系而言就是，派生类不能代替基类完成基类接口所声明的行为，也就是不符合“Liskov 替换原则（LSP ）” ，此时派生类不能自动转换成为基类的接口，就算通过类型转换（static_cast 和dynamic_cast ）也会得到一个空指针。</p>
<pre> error: cannot cast 'Derived' to its protected base class
      'Base'
        Base *b = static_cast(d);</pre>
<p>从语法角度上来说，<strong>protected 继承会将基类中的public 可见性的成员修改成为protected 可见性</strong> ，相当于在派生类中引入了protected 成员，这样一来在派生类中同样还是可以调用基类的protected 和public成员，派生类的派生类就也可以调用被protected 继承的基类的protected 和public 成员。例如：</p>
<pre>
class CSample1 {

protected:

void printProtected() {}

public:

void printPublic() {}

};

class CSample2 : protected CSample1 {};

class CSample3 : public CSample2 {

void print3() {

printProtected();

printPublic();

}

};

</pre>
<p>3. private<br />
从语义角度上来说，<strong>private 继承是一种实现继承</strong> ，根据面向对象中的关系而言就是，派生类不能代替基类完成基类接口所声明的行为，也就是不符合“Liskov 替换原则（LSP ）” ，此时派生类不能自动转换成为基类的接口，就算通过类型转换（static_cast 和dynamic_cast ）也会得到一个空指针。<br />
从语法角度上来说，<strong>private 继承会将基类中的public 和protected 可见性的成员修改成为private 可见性</strong> ，这样一来虽然派生类中同样还是可以调用基类的protected 和public 成员，但是在派生类的派生类就不可以再调用被private 继承的基类的成员了。</p>
<pre>
class CSample1 {
protected:
void printProtected() {}
public:
void printPublic() {}
};
class CSample2 : private CSample1 {};
class CSample3 : public CSample2 {
void print3() {
printProtected(); // 编译错误，不可以调用该函数
printPublic();    // 编译错误，不可以调用该函数
}
};

</pre>
<p>4. using 声明</p>
<p>如果进行 private 或 protected 继承，则基类成员的访问级别在派生类中比在基类中更受限：</p>
<pre>
class <strong>Base</strong> {
public:
std::size_t size() const {return n;}
protected:
std::size_t n;
}
class <strong>Derived</strong> : private Base{...};
</pre>
<p>在这一继承层次中， size 在 Base 中为 public ，但在 Derived 中为 private 。为了使 size 在Derived 中成为 public ，可以在 Derived 的 public 部分增加一个 <strong>using </strong><strong>声明 </strong>。如下这样改变Derived 的定义，可以使 size 成员能够被用户访问，并使 n 能够被从 Derived 派生的类访问：</p>
<pre>
class Derived : private Base{
public:
using Base::size;
privated:
using Base::n;
//...
}
</pre>
<p>用 struct 和 class 保留字定义的类具有不同的默认访问级别。同样，默认继承访问级别根据使用哪个保留字定义派生类也不相同。使用 class 保留字定义的派生类默认具有 private 继承，而用 struct 保留字定义的类默认具有 public 继承：</p>
<pre class="lang:default decode:true ">class Base{/*...*/}
struct D1 : Base{/*...*/};  //public  inheritance by default
class  D2 : Base{/*...*/};  //private inheritance by default</pre>
<p>有一种常见的误解认为用 struct 保留字定义的类与用 class 定义的类有更大的区别。实际上它们唯一的不同只是默认的成员保护级别和默认的派生保护级别，除此之外，再也没有其他的区别。</p>
<p>&nbsp;</p>
<p>总的来说上面这个帖子写的挺不错的，可欠缺的是它隐藏了一些关系，如public的继承中，对父类各变量的访问权限及总的继承关系表。下面转载了一篇来自csdn上的帖子，对这方面加了一些说明：</p>
<p><code>class b:public a{}</code>继承的含义是b可以访问a中public, protected成员，但是不能访问a中private成员。</p>
<p><code>class b:protected a{}</code>继承的含义是b可以访问a中public, protected成员，并将a中public, protected成员转成protected成员，同时也不能访问a中private成员。</p>
<p><code>class b:private a{}</code>继承的含义是b可以访问a中public, protected成员，将a中public, protected成员转成private成员，不能访问a中private成员，同时如果b类再派生出子类的话(假设为c)，那么c类将不可访问a类中 的任何成员。</p>
<p>关系表如下：</p>
<table style="width:100%">
<tbody>
<tr>
<th>继承方式</th>
<td>public</td>
<td>private</td>
<td>protected</td>
</tr>
<tr>
<td>public</td>
<td>public</td>
<td>private</td>
<td>protected</td>
</tr>
<tr>
<td>private</td>
<td>private</td>
<td>private</td>
<td>private</td>
</tr>
<tr>
<td>protected</td>
<td>protected</td>
<td>private</td>
<td>protected</td>
</tr>
</tbody>
</table>
