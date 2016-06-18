---
layout: blog_item
title: '抽象工厂'
date: 2016-06-14 00:58:14 -0700
author: christiein
version: 1.0.0
categories: [原创]
---

# 抽象工厂(Abstract Factory)

## 1. 意图

提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。

## 2. 别名

Kit

## 3. 动机

考虑一个支持多种视感（look-and-feel）标准的用户界面工具包，例如Motif和Presentation Manager。不同的视感风格为诸如滚动条、窗口和按钮等用户界面“窗口组件”定义不同的外观和行为。为保证视感风格标准间的可移植性，一个应用不应该为一个特定的视感外观硬编码它的窗口组件。在整个应用中实例化特定视感风格的窗口组件类将使得以后很难改变视感风格。

为解决这一问题我们可以定义一个抽象的WidgetFactory类，这个类声明了一个用来创建每一类基本窗口组件的接口。每一类窗口组件都有一个抽象类，而具体子类则实现了窗口组件的特定视感风格。对于每一个抽象窗口组件类，WidgetFactory接口都有一个返回新窗口组件对象的操作。客户调用这些操作以获得窗口组件实例，但客户并不知道他们正在使用的是哪些具体类。这样客户就不依赖于一般的视感风格，如下图所示。

图略<font color=red>（请增加图）</font>

每一种视感标准都对应于一个具体的WidgetFactory子类。每一子类实现那些用于创建合适视感风格的窗口组件的操作。例如，MotifWidgetFactory的CreateScrollBar操作实例化并返回一个Motif滚动条，而相应的PMWidgetFactory操作返回一个Presentation Manager的滚动条。客户仅通过WidgetFactory接口创建窗口组件，他们并不知道哪些类实现了特定视感风格的窗口组件。换言之，客户仅与抽象类定义的接口交互，而不使用特定的具体类的接口。

WidgetFactory也增强了具体窗口组件类之间依赖关系。一个Motif的滚动条应该与Motif按钮、Motif正文编辑器一起使用，这一约束条件作为使用MotifWidgetFactory的结果被自动加上。

## 4. 适用性

在以下情况可以使用Abstract Factory模式

* 一个系统要独立于它的产品的创建、组件和表示时。
* 一个系统要由多个产品系列中的一个来配置时。
* 当你要强调一系列相关的产品对象的设计以便进行联合使用时。
* 当你提供一个产品类库，而只想显示它们的接口而不是实现时。

## 5. 结构

此模式的结构如下图所示。

图略<font color=red>（请增加图）</font>

## 6. 参与者

* AbstractFactory(WidgetFactory)
  —— 声明一个创建抽象产品对象的操作接口。
* ConcreteFactory(MotifWidgetFactory，PMWidgetFactory)
  —— 实现创建具体产品对象的操作。
* AbstractProduct（Windows，ScrollBar）
  —— 为一类产品对象声明一个接口。
* ConcretePruduct（MotifWindow，MotifScrollBar）
  —— 定义一个将被相应的具体工厂创建的产品对象。
  —— 实现AbstractProduct接口。
* Client
  —— 仅使用由AbstractFactory和AbstractProduct类声明的接口。

## 7. 协作

* 通常在运行时刻创建一个ConcreteFactory类的实例。这一具体的工厂创建具体特定实现的产品对象。为创建不同的产品对象，客户应使用不同的具体工厂。
* AbstractFactory将产品对象的创建延迟到它的ConcreteFactory子类。

## 8. 效果

AbstractFactory模式有下面的一些优点和缺点：

1. **它分离了具体的类。** Abstract Factory模式帮助你控制一个应用创建的对象的类。因为一个工厂封装创建产品对象的责任和过程，它将客户与类的实现分离。客户通过它们的抽象接口操作实例。产品的类名也在具体工厂的实现中被分离；他们不出现在客户代码中。
2. **它使得易于交换产品系列。** 一个具体工厂类在一个应用中仅出现一次————即在它初始化的时候。这使得改变一个应用的具体工厂变得很容易。它只需改变具体的工厂即可使用不同的产品配置，这是因为一个抽象工厂创建了一个完整的产品系列，所以整个产品系列会立刻改变。在我们的用户界面的例子中，我们仅需转换到相应的工厂对象并重新创建接口，就可实现从Motif窗口组件转换为Presentation Manager窗口组件。
3. **它有利于产品的一致性。** 当一个系列中的产品对象被设计成一起工作时，一个应用一次只能使用同一个系列中的对象，这一点很重要。而AbstractFactory很容易实现这一点。
4. **难以支持新种类的产品。** 难以扩展抽象工厂以生产新种类的产品。这是因为AbstractFactory接口确定了可以被创建的产品集合。支持新种类的产品就需要扩展该工厂接口，这将涉及AbstractFactory类及其所有子类的改变。我们会在实现一节讨论这个问题的一个解决办法。

## 9. 实现

下面是实现Abstract Factory模式的一些有用技术：

1. **将工厂作为单件。** 一个应用中一般每个产品系列只需要一个ConcreteFactory的实例。因此工厂通常最好实现为一个[Singleton][1]。
2. **创建产品。** AbstractFactory仅声明一个创建产品的接口，真正创建产品是由ConcreteProduct子类实现的。最通常的一个办法是为每一个产品定义一个工厂方法（参见[Factory Method][2]）。一个具体的工厂将为每个产品重定义该工厂方法以指定产品。虽然这样的实现很简单，但它却要求每个产品系列都要有一个新的具体工厂子类，即使这些产品系列的差别很小。
如果有多个可能的产品系列，具体工厂也可以使用Prototype模式来实现。具体工厂使用产品系列中每一个产品的原型实例来初始化，且它通过复制它的原型来创建新的产品。在基于原型的方法中，使得不是每个新的产品系列都需要一个新的具体工厂类。
此外是Smalltalk是实现一个基于原型的工厂的方法。具体工厂在一个呗称为partCatalog的字典中存储将复制的原型。方法make:检索该原型并复制它：
 >```Smalltalk
 >make : partName
 >     ^ (partCatalog at : partName) copy
 >```
具体工厂有一个方法用来向该目录中增加部件。
>```Smalltalk
>addPart : partTemplate named : partName
>   partCatalog at : partName put : partTemplate
>```
原型通过用一个符号标识它们，从而被增加到工厂中：
>```Smalltalk
>aFactory addPart : aPrototype named : #ACMEWidget
>```
在将类作为第一类对象的语言中（例如Smalltalk和ObjectiveC），这个基于原型的方法可能有所变化。你可以将这些语言中的一个类看成是一个退化的工厂，它仅创建一种产品。你可以将类存储在一个具体工厂中，这个具体工厂在变量中创建多个具体的产品，这很像原型。这些类代替具体工厂创建了新的实例。你可以通过使用产品的类而不是子类初始化一个具体工厂的实例，来定义一个新的工厂。这一方法利用了语言的特点，而纯基于原型的方式是与语言无关的。
像刚讨论的Smalltalk中的基于原型的工厂一样，基于类的版本将有一个唯一的实例变量partCatalog，它是一个字典，它的主键是各部分的名字。partCatalog存储产品的类而不是存储被复制的原型。方法make：现在是这样：
>```Smalltalk
>make : partName
>     ^ (partCatalog at : partName) new
>```
3. **定义可扩展的工厂。** AbstractFactory通常为每一种它可以生产的产品定义一个操作。产品的种类被编码在操作型构中。增加一种新的产品要求改变AbstractFactory的接口以及所有与它相关的类。一个更灵活但不太安全的设计是给创建对象的操作增加一个参数。该参数指定了将被创建的对象的种类。它可以是一个类标识符、一个整数、一个字符串，或其他任何可以标识这种产品的东西。实际上使用这种方法，AbstractFactory只需要一个“Make”操作和一个指示要创建对象的种类的参数。这是前面已经讨论过的基于类的抽象工厂的技术。
C++这样的静态类型语言与相比，这一变化更容易用在类似于Smalltalk这样的动态类型语言中。仅当所有对象都有相同的抽象基类，或者当产品对象可以被请求它们的客户安全的强制转换成正确类型时，你才能够在C++中使用它。Factory Method的实现部分说明了怎样在C++中实现这样的参数化操作。
该方法即使不需要类型强制转换，但仍有一个本质的问题：所有的产品将返回类型所给定的相同的抽象接口返回给客户。客户将不能区分或对一个产品的类别进行安全的假定。如果一个客户需要进行与特定子类相关的操作，而这些操作却不能通过抽象接口得到。虽然客户可以实施一个向下类型转换（downcast）（例如在C++中用dynamic_cast），但这并不是可行或安全的，因为向下类型转换可能会失败。这是一个典型的高度灵活和可扩展接口的权衡折衷。 

## 10. 代码示例

1. [Java代码示例](? "示例代码在哪？")
2. [？](? "还有什么？")

## 11. 已知应用

InterView使用“Kit”后缀[Lin92]来表示AbstractFactory类。它定义WidgetKit和DialogKit抽象工厂来生成与特定视感风格相关的用户界面对象。InterView还包括一个LayoutKit，它根据所需要的布局生成不同的组成（composition）对象。例如，一个概念上是水平的布局根据文档的定位（画像或是风景）可能需要不同的组成对象。

ET++[WGM88]使用Abstract Factory模式以达到在不同窗口系统（例如，X Windows和SunView）间的可移植性。WindowSystem抽象基类定义一些接口，来创建表示窗口系统资源的对象（例如MakeWindow、MakeFont、MakeColor）。具体的子类为某个特定的窗口系统实现这些接口。运行时刻，ET++创建一个具体WindowSystem子类的实例，以创建具体的系统资源对象。

## 12. 相关模式

AbstractFactory类通常用工厂方法（Factory Method）实现，但它们也可以用Prototype实现。

一个具体的工厂通常是一个单件（Singleton）。

[1]:./Singleton.md "Singleton"
[2]:./FactoryMethod.md "Factory Method"