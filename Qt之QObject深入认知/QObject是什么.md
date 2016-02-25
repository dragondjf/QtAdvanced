QObject是什么
=================
###Detailed Description 基础描述

The QObject class is the base class of all Qt objects.
QObject是所有Qt对象的基类

QObject is the heart of the Qt Object Model. The central feature in this model is a very powerful mechanism for  object communication called signals and slots. You can connect a signal to a slot with connect() and destroy the connection with disconnect(). To avoid never ending notification loops you can temporarily block signals with blockSignals(). The protected functions connectNotify() and disconnectNotify() make it possible to track connections.

QObejct是Qt对象模型的核心。Qt对象模型最大的特色就是提供了强大的信号和槽通信机制。你可以使用connect()和disconnect()进行信号和槽的连接与断开。为了规避无止境的消息循环，可以使用blockSignals()临时性的隔断信号。受保护方法connectNotify() 和 disconnectNotify()可以追踪信号和槽的连接。 

QObjects organize themselves in object trees. When you create a QObject with another object as parent, the object will automatically add itself to the parent's children() list. The parent takes ownership of the object; i.e., it will automatically delete its children in its destructor. You can look for an object by name and optionally type using findChild() or findChildren().

QObject对象按对象树的方式组织。当创建一个带parent的QObject对象时，这个对象将会自动的添加到这个parent的children()列表中。parent父对象拥有子对象，自动负责在析构函数中删除children.可以使用findChild() 或findChildren()接口根据对象的名字或者类型进行对象查询。

Every object has an objectName() and its class name can be found via the corresponding metaObject() (see QMetaObject::className()). You can determine whether the object's class inherits another class in the QObject inheritance hierarchy by using the inherits() function.

每个QObject对象拥有一个objectName()，类名可以通过metaObject() (详情查看QMetaObject::className())获取。可以通过inherits()方法判断一个类是否继承自另一个类。

When an object is deleted, it emits a destroyed() signal. You can catch this signal to avoid dangling references to QObjects.

当一个QObjetc对象实例被删除时，会发射destroyed() 信号; 通过捕获这个信号可以避免悬空的对象引用。

QObjects can receive events through event() and filter the events of other objects. See installEventFilter() and eventFilter() for details. A convenience handler, childEvent(), can be reimplemented to catch child events.

QObejct可以通过event()接口接受事件，和过滤其他对象的一些事件。详情参加installEventFilter() 和 eventFilter() ; 可以通过重写快捷接口childEvent()用捕获子对象的事件。

Last but not least, QObject provides the basic timer support in Qt; see QTimer for high-level support for timers.
最后但同样重要的是，QObject提供了基本的时钟支持，详情参加高级别的timer支持类QTimer;


Notice that the Q_OBJECT macro is mandatory for any object that implements signals, slots or properties. You also need to run the Meta Object Compiler on the source file. We strongly recommend the use of this macro in all subclasses of QObject regardless of whether or not they actually use signals, slots and properties, since failure to do so may lead certain functions to exhibit strange behavior.

请注意，Q_OBJECT宏是所有类实现信号的基础。同时，必须使用moc元对象编译系统对源码进行编译。强烈建议，无论是否需要使用信号、槽和属性，所有继承自QObect的对象都需要使用Q_OBJECT宏。如果不这样使用，可能会出现一系列奇怪的行为。

All Qt widgets inherit QObject. The convenience function isWidgetType() returns whether an object is actually a widget. It is much faster than qobject_cast<QWidget *>(obj) or obj->inherits("QWidget").
所有的控件类都继承自QObejct，可以简单使用isWidgetType() 来判断一个对象是否是一个控件类。这个接口相比qobject_cast<QWidget *>(obj) 或者obj->inherits("QWidget").=要快很多。

Some QObject functions, e.g. children(), return a QObjectList. QObjectList is a typedef for QList<QObject *>.

一些QObject接口，比如children()接口返回一个对象列表，QObjectList 是QList<QObject *>别名。


###Thread Affinity 线程关联

A QObject instance is said to have a thread affinity, or that it lives in a certain thread. When a QObject receives a queued signal or a posted event, the slot or event handler will run in the thread that the object lives in.

每个QObejct实例对象都拥有线程关联，或者说存在某一个线程。当一个对象接受一个队列信号或者是一个posted事件，对应的槽函数或者是事件处理器会在对象生存的线程中执行。


Note: If a QObject has no thread affinity (that is, if thread() returns zero), or if it lives in a thread that has no running event loop, then it cannot receive queued signals or posted events.

注意：如果一个QObject对象没没有线程关联（就是 thread()接口返回空），或者是对象生存的线程没有事件循环，这样无法接受队列信号和posted事件。

By default, a QObject lives in the thread in which it is created. An object's thread affinity can be queried using thread() and changed using moveToThread().

一般而言，QObejct对象生存在起创建的线程中。可以通过thread()查询对象所处的线程和moveToThread()改变线程关联性。

All QObjects must live in the same thread as their parent. Consequently:

所有对象必须生存在父对象的线程中。

setParent() will fail if the two QObjects involved live in different threads.
When a QObject is moved to another thread, all its children will be automatically moved too.

如果两个QObject对象生存在不同的线程中，那么setParent()接口将返回失败。当一个QObejct对象被移动到另一个线程中时，所有的子对象会自动的移动到父对象的生存线程中去。

moveToThread() will fail if the QObject has a parent.
如果QObejct已经拥有parent,那么调用moveToThread()接口将会返回失败。

If QObjects are created within QThread::run(), they cannot become children of the QThread object because the QThread does not live in the thread that calls QThread::run().
Note: A QObject's member variables do not automatically become its children. The parent-child relationship must be set by either passing a pointer to the child's constructor, or by calling setParent(). Without this step, the object's member variables will remain in the old thread when moveToThread() is called.

如果OQbejct对象在QThread::run()函数内创建，这些对象无法成为QThreadd对象的子对象，因为QThread不生存在QThread::run().对应的新线程中。

注意：一个QObect的成员变量不会自动的成为其子对象。父子关系可以通过两种方式设定：一种是传递一个父对象指针，第二种是调用setParent()。如果没有这一步，子对象的成员变量将继续生存在老的线程中直至 moveToThread()接口被调用。 


###No Copy Constructor or Assignment Operator 无复制构造器和赋值操作

QObject has neither a copy constructor nor an assignment operator. This is by design. Actually, they are declared, but in a private section with the macro Q_DISABLE_COPY(). In fact, all Qt classes derived from QObject (direct or indirect) use this macro to declare their copy constructor and assignment operator to be private. The reasoning is found in the discussion on Identity vs Value on the Qt Object Model page.

QObejct没有复制构造器和赋值构造器，这是设计。实际上，在私有声明部分，Q_DISABLE_COPY()宏进行了声明。所有的Qt类直接或间接继承QObject都具备这一特性。具体的原因可以参考Qt Object Model 设计理念id和value的区别。

The main consequence is that you should use pointers to QObject (or to your QObject subclass) where you might otherwise be tempted to use your QObject subclass as a value. For example, without a copy constructor, you can't use a subclass of QObject as the value to be stored in one of the container classes. You must store pointers.

没有复制构造器和赋值构造器的设计，导致最好的结果就是使用指针来操作QObject。

###Auto-Connection 自动连接

Qt's meta-object system provides a mechanism to automatically connect signals and slots between QObject subclasses and their children. As long as objects are defined with suitable object names, and slots follow a simple naming convention, this connection can be performed at run-time by the QMetaObject::connectSlotsByName() function.

Qt的元对象系统为QObejct和QObjetc子类和子对象提供了自动信号和槽连接的机制。

uic generates code that invokes this function to enable auto-connection to be performed between widgets on forms created with Qt Designer. More information about using auto-connection with Qt Designer is given in the Using a Designer UI File in Your Application section of the Qt Designer manual.


###Dynamic Properties 动态属性

From Qt 4.2, dynamic properties can be added to and removed from QObject instances at run-time. Dynamic properties do not need to be declared at compile-time, yet they provide the same advantages as static properties and are manipulated using the same API - using property() to read them and setProperty() to write them.
自Qt4.2后，程序运行时属性可以动态的添加和移除。动态属性不需要在编译的时候声明。动态属性是通过property() 和setProperty() 接口来进行读写的。

From Qt 4.3, dynamic properties are supported by Qt Designer, and both standard Qt widgets and user-created forms can be given dynamic properties.

自从Qt4.3之后， Qt Designer、标准的Qt widget和用户自定义的ui表单都支持动态属性。

####Internationalization (I18n) 国际化

All QObject subclasses support Qt's translation features, making it possible to translate an application's user interface into different languages.

所有QObejct子类对象都支持Qt的翻译机制，让应用程序可以翻译成不同的语言成为可能。

To make user-visible text translatable, it must be wrapped in calls to the tr() function. This is explained in detail in the Writing Source Code for Translation document.

具体翻译参加Writing Source Code for Translation document.
