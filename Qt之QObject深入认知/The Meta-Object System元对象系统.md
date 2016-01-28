The Meta-Object System元对象系统
===========================
####1. 什么Qt的元对象系统
Qt's meta-object system provides the signals and slots mechanism for inter-object communication, run-time type information, and the dynamic property system.

Qt的元对象系统提供了对象间的信号和槽通信机制，运行时类型信息和动态属性系统。

The meta-object system is based on three things:

+ The QObject class provides a base class for objects that can take advantage of the meta-object system.
+ The Q_OBJECT macro inside the private section of the class declaration is used to enable meta-object features, such as dynamic properties, signals, and slots.
+ The Meta-Object Compiler (moc) supplies each QObject subclass with the necessary code to implement meta-object features.

元对象系统的实现依赖下面三个技术要点：

+ **`QObject类`**是元对象系统的基类；
+ **`Q_OBJECT宏`**，定义在类的私有声明部分，拥有Q_OBJECT才具备元对象系统特性，如动态属性，信号和槽；
+ **`Meta-Object Compiler (moc)`**元对象编译器对每个继承自QObject的子类添加必要的代码来实现元对象特性。

The moc tool reads a C++ source file. If it finds one or more class declarations that contain the Q_OBJECT macro, it produces another C++ source file which contains the meta-object code for each of those classes. This generated source file is either #include'd into the class's source file or, more usually, compiled and linked with the class's implementation.

moc工具遍历每一个c++源文件，如果发现某个或多个文件的类型声明中包含Q_OBJECT宏，将会对应生成一个对应每个类的元对象代码的moc_filename的文件；生成的代码文件将会被qmake自动的引入、编译并链接到类的实现中去。


####2.元对象系统的特性
In addition to providing the signals and slots mechanism for communication between objects (the main reason for introducing the system), the meta-object code provides the following additional features:

+ QObject::metaObject() returns the associated meta-object for the class.
+ QMetaObject::className() returns the class name as a string at run-time, without requiring + native run-time type information (RTTI) support through the C++ compiler.
+ QObject::inherits() function returns whether an object is an instance of a class that inherits a specified class within the QObject inheritance tree.
+ QObject::tr() and QObject::trUtf8() translate strings for internationalization.
+ QObject::setProperty() and QObject::property() dynamically set and get properties by name.
+ QMetaObject::newInstance() constructs a new instance of the class.

除了提供对象间的信号和槽通信机制（主要特性）外，元对象代码还提供了以下额外的特性：
+ QObject::metaObject() QObject的静态方法metaObject()返回与类对应的元类meta-object；
+ QMetaObject::className() QMetaObject的静态方法className()返回一个对象的运行时的字符串类名，不需要依赖c++编译器原生提供的RTTI(运行时类型识别)；
+  QObject::inherits() QObject的静态方法inherits，​bool inherits(const char * className)返回这个对象是否属于它QObject类型继承树中某个类的实例；
+   QObject::tr() and QObject::trUtf8()这两个静态方法为国际化字符串提供了支持；
+  QObject::setProperty() and QObject::property()这两个静态方法为根据名字动态设置和读取属性提供了支持；
+  QMetaObject::newInstance()可以通过这个静态方法构建一个属于这个类的新实例。


It is also possible to perform dynamic casts using qobject_cast() on QObject classes. The qobject_cast() function behaves similarly to the standard C++ dynamic_cast(), with the advantages that it doesn't require RTTI support and it works across dynamic library boundaries. It attempts to cast its argument to the pointer type specified in angle-brackets, returning a non-zero pointer if the object is of the correct type (determined at run-time), or 0 if the object's type is incompatible.

**`同时，可以对基于QObject的类使用qobject_cast() 进行动态类型转换`**；qobject_cast()的功能和标准c++类型转换工具 dynamic_cast()相似，但是它的优点在于不需要RTTI的支持，并且可以跨动态库边界。T qobject_cast(QObject * object)，如果运行时转换成功，将返回一个非零的指针；如果转换失败，将返回0;

####3. 举例
For example, let's assume MyWidget inherits from QWidget and is declared with the Q_OBJECT macro:

    QObject *obj = new MyWidget;
The obj variable, of type QObject *, actually refers to a MyWidget object, so we can cast it appropriately:

    QWidget *widget = qobject_cast<QWidget *>(obj);
The cast from QObject to QWidget is successful, because the object is actually a MyWidget, which is a subclass of QWidget. Since we know that obj is a MyWidget, we can also cast it to MyWidget *:

    MyWidget *myWidget = qobject_cast<MyWidget *>(obj);
The cast to MyWidget is successful because qobject_cast() makes no distinction between built-in Qt types and custom types.

    QLabel *label = qobject_cast<QLabel *>(obj);
    // label is 0
The cast to QLabel, on the other hand, fails. The pointer is then set to 0. This makes it possible to handle objects of different types differently at run-time, based on the type:

    if (QLabel *label = qobject_cast<QLabel *>(obj)) {
        label->setText(tr("Ping"));
    } else if (QPushButton *button = qobject_cast<QPushButton *>(obj)) {
        button->setText(tr("Pong!"));
    }

####4. 友情提示
Qt strongly recommend that all subclasses of QObject use the Q_OBJECT macro regardless of whether or not they actually use signals, slots, and properties.

**`Qt强烈建议，无论你是否需要使用信号和槽机制还是动态属性，所有继承自QObject的类最好都加上Q_OBJECT宏声明。`**