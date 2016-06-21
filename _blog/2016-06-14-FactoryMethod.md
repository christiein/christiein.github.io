---
layout: blog_item
title: '工厂方法'
date: 2016-06-14 06:58:14 -0700
author: christiein
version: 1.0.0
categories: [原创]
---
#工厂方法(Factory Method)

## 1. 意图

定义一个用于创建对象的接口，让子类决定实例化哪一个类。Factory Method使一个类的实例化延迟到其子类。

## 2.别名

虚构造器（Virtual Constructor）

## 3. 动机

框架使用抽象类定义和维护对象之间的关系。这些对象的创建通常也由框架负责。

考虑这样一个应用框架，它可以向用户显示多个文档。在这个框架中，两个主要的抽象是类Application和Document。这两个类都是抽象的，客户必须通过它们的子类来做与具体应用相关的实现。例如，为创建一个绘图应用，我们定义类DrawingApplication和DrawingDocument。Application类负责管理Document并根据需要创建它们——例如，当用户从菜单中选择Open或New的时候。

因为被实例化得特定Document子类是与特定应用相关的，所以Application类不可能预测到哪个Document子类将被实例化——Application类仅知道一个新的文档***何时***被创建，而不知道***哪一种***Document将被创建。这就产生了一个尴尬的局面：框架必须实例化类，但是它只知道不能被实例化的抽象类。

Factory Method模式提供了一个解决办法。它封装了哪一个Document子类将被创建的信息并将这些信息从该框架中分离出来，如下图所示。

![图](/image/image.jpg "图略") 图略

Application的子类重定义Application的抽象操作CreateDocument以返回适当的Document子类对象。一旦一个Application子类实例化以后，它就可以实例化与应用相关的文档，而无需知道这些文档的类。我们城CreateDocument市一个工厂方法（factory method），因为它负责“生产”一个对象。

## 4. 适用性

在下列情况下可以使用Factory Method模式：

* 当一个类不知道它所必须创建的对象的类的时候。
* 当一个类希望由它的子类来指定它所创建的对象的时候。
* 当类将创建对象的职责委托给多个帮助子类中的某一个，并且你希望将哪一个帮助子类是代理者这一信息局部化的时候。

## 5. 结构

图略 ![图](/images/image.jpg "结构图")

## 6. 参与者

* **Product**(Document)
 —— 定义工厂方法所创建的对象的接口。
* **ConcreteProduct**(MyDocument)
 —— 实现Product接口
* **Creator**(Application)
 —— 声明工厂方法，该方法返回一个Product类型的对象。Creator也可以定义一个工厂方法的缺省实现，它返回一个缺省的ConcreteProduct对象。
* **ConcreteCreator**(MyApplication)
 —— 重定义工厂方法以返回一个ConcreteProduct实例。

## 7. 协作

* Creator依赖于它的子类来定义工厂方法，所以它返回一个适当的ConcreteProduct实例。

## 8. 效果

略。。。。

下面是Factory Method模式的另外两种效果：

1. 为子类提供挂钩（hook）
2. 连接平行的类层次

## 9. 实现

略。。。。

当应用Factory Method模式时要考虑下面一些问题：

1. 主要有两种不同的情况
2. 参数化工厂方法
3. 特定语言的变化和问题
4. 使用魔板以避免创建子类
5. 命名预定

## 10. 代码示例

函数CreateMaze建造并返回一个迷宫。这个函数存在的一个问题是它对迷宫、房间、门和墙壁的类进行了硬编码。我们引入工厂方法以使子类可以选择这些构件。首先我们将在MazeGame中定义工厂方法以创建迷宫、房间、墙壁和门对象：

```c++
class MazeGame {
public:
    Maze* CreateMaze();

// factory methods:

    virtual Maze* MakeMaze() const
        { return new Maze; }
    virtual Room* MakeRoom(int n) const
        { return new Room(n); }
    virtual Wall* MakeWall() const
        { return new Wall; }
    virtual Door* MakeDoor(Room* r1, Room* r2) const
        { return new Door(r1, r2); }
};
```

每一个工厂方法返回一个给定类型的迷宫构件。MazeGame提供一些缺省的实现，它们返回最简单的迷宫、房间、墙壁和门。

现在我们可以用这些工厂方法重写CreateMaze：

```c++
Maze* MazeGame::CreateMaze() {
    Maze* aMaze = MakeMaze();

    Room* r1 = MakeRoom(1);
    Room* r2 = MakeRoom(2);

    Door* theDoor = MakeDoor(r1, r2);

    aMaze->AddRoom(r1);
    aMaze->AddRoom(r2);

    r1->SetSide(North, MakeWall());
    r1->SetSide(East, theDoor);
    r1->SetSide(South, MakeWall());
    r1->SetSide(West, MakeWall());

    r2->SetSide(North, MakeWall());
    r2->SetSide(East, MakeWall());
    r2->SetSide(South, MakeWall());
    r2->SetSide(West, theDoor);
};
```

不同的游戏可以创建MazeGame的子类以特别指明一些迷宫的部件。MazeGame子类可以重定义一些或所有的工厂方法以指定产品中的变化。例如，一个BombedMazeGame可以重定义产品Room和Wall以返回操作爆炸后的变体：

```c++
class BombedMazeGame ： public MazeGame {
public:
    BombedMazeGame();

    virtual Wall* MakeWall() const
        { return new BombedWall; }

    virtual Room* MakeRoom(int n) const
        { return new RoomWithABomb(n); }
};
```

一个EnchantedMazeGame变体可以像这样定义：

```c++
class EnchantedMazeGame : public MazeGame {
public:
    EnchantedMazeGame();

    virtual Room* MakeRoom(int n) const
        { return new EnchantedRoom(n, CastSpell()); }

    virtual Door* MakeDoor(Room* r1, Room* r2) const
        { return new DoorNeedingSpell(r1, r2); }
protected:
    Spell* CastSpell() const;
};
```

## 11. 已知应用

工厂方法主要用于工具包和框架中。前面的文档例子是MacApp和ET++[WGM88]中的一个典型应用。操纵器的例子来自Unidraw。

Smalltalk-80 Model/View/Controller框架中的类视图（Class View）有一个创建控制器的方法defaultController，它有点类似于一个工厂方法[Par90]。但是View的子类通过定义defaultControllerClass来指定它们默认的控制器。defaultControllerClass返回defaultController所创建实例的类，因此它才是真正的工厂方法，即子类应该重定义它。

Smalltalk-80中一个更为深奥的例子是Behavior（用来表示类的所有对象的超类）定义的工厂方法parseClass。这使得一个雷可以对它的源代码使用一个定制的语法分析器。例如，一个客户可以定义一个类SQLParser来分析嵌入了SQL语句的类的源代码。Behavior类实现了parserClass，返回一个标准的Smalltalk Parser类。一个包含嵌入SQL语句的类重定义了该方法（以类方法的形式）并返回SQLParser类。

IONA Technologies的Orbix ORB系统[ION94]在对象给一个远程对象应用发送请求时，使用Factory Method生成一个适当类型的代理（参见Proxy）。Factory Method使得易于替换缺省代理。比如说，可以用一个使用客户端高速缓存的代理来替换。

## 12. 相关模式

Abstract Factory经常用工厂方法来实现。Abstract Factory模式中动机一节的例子也对Factory Method进行了说明。

工厂方法通常在Template Method中被调用。在上面的文档例子中，NewDocument就是一个模板方法。

Prototype不需要创建Creator的子类。但是，它们通常要求一个针对Product类的Initialize操作。Creator使用Initialize来初始化对象。而Factory Method不需要这样的操作。
