Qt中QThread的使用方式
=====================

#### 1. QThread使用简介
The QThread class provides a platform-independent way to manage threads.
>**QThread提供了一个平台独立的方式去管理线程操作**

A QThread object manages one thread of control within the program. QThreads begin executing in run(). By default, run() starts the event loop by calling exec() and runs a Qt event loop inside the thread.
>**一个QThread实例对象在程序中管理着线程的控制逻辑。QThreads在run()中开始执行属于自己线程的逻辑。默认，QThread在run()方法内调用exec()开启了一个属于自己线程内部的事件循环。**

You can use worker objects by moving them to the thread using QObject::moveToThread.
>**你可以写一个继承自QObjet的worker对象来实现你的业务逻辑，然后利用 QObject::moveToThread()方法将worker实例对象移动到你期望它被执行的线程对象中去, 实现线程关联，下面称这种方式为`work-object`模式**

    
    class Worker : public QObject
    {
        Q_OBJECT
    
    public slots:
        void doWork(const QString &parameter) {
            QString result;
            /* ... here is the expensive or blocking operation ... */
            emit resultReady(result);
        }
    
    signals:
        void resultReady(const QString &result);
    };
    
    class Controller : public QObject
    {
        Q_OBJECT
        QThread workerThread;
    public:
        Controller() {
            Worker *worker = new Worker;
            worker->moveToThread(&workerThread);
            connect(&workerThread, &QThread::finished, worker, &QObject::deleteLater);
            connect(this, &Controller::operate, worker, &Worker::doWork);
            connect(worker, &Worker::resultReady, this, &Controller::handleResults);
            workerThread.start();
        }
        ~Controller() {
            workerThread.quit();
            workerThread.wait();
        }
    public slots:
        void handleResults(const QString &);
    signals:
        void operate(const QString &);
    };

The code inside the Worker's slot would then execute in a separate thread. However, you are free to connect the Worker's slots to any signal, from any object, in any thread. It is safe to connect signals and slots across different threads, thanks to a mechanism called queued connections.
>**the Worker's slot doWork()逻辑将在一个独立的线程中去执行；这样你就可以自由的连接在 任何线程内 任意对象 的信号到 the Worker's slots；得益于Qt信号与槽的queued connections机制，跨线程连接信号和槽是安全的**


Another way to make code run in a separate thread, is to subclass QThread and reimplement run(). For example:
>另外一种让代码运行在一个独立线程内的方法就是，继承QThread，重写run()方法，例子如下：

    class WorkerThread : public QThread
    {
        Q_OBJECT
        void run() Q_DECL_OVERRIDE {
            QString result;
            /* ... here is the expensive or blocking operation ... */
            emit resultReady(result);
        }
    signals:
        void resultReady(const QString &s);
    };
    
    void MyObject::startWorkInAThread()
    {
        WorkerThread *workerThread = new WorkerThread(this);
        connect(workerThread, &WorkerThread::resultReady, this, &MyObject::handleResults);
        connect(workerThread, &WorkerThread::finished, workerThread, &QObject::deleteLater);
        workerThread->start();
    }

In that example, the thread will exit after the run function has returned. There will not be any event loop running in the thread unless you call exec().
> **在上面的例子中，线程将在run()方法执行返回后退出，这个线程内没有事件循环，除非你在run()函数内部最后调用exec()开启一个事件循环。**

It is important to remember that a QThread instance lives in the old thread that instantiated it, not in the new thread that calls run(). This means that all of QThread's queued slots will execute in the old thread. Thus, a developer who wishes to invoke slots in the new thread must use the worker-object approach; new slots should not be implemented directly into a subclassed QThread.

> **记住，一个线程实例对象存活在创建它的老线程内，不是run()启动后的新线程内，但是在run()里构建的对象是属于子线程的。这就意味着所有线程的槽函数都将在老线程中执行；因此，开发者如果希望槽函数在新线程中执行，就必须使用work-object方式；注意：槽函数定义不应该在继承QThread的子类中实现。**

When subclassing QThread, keep in mind that the constructor executes in the old thread while run() executes in the new thread. If a member variable is accessed from both functions, then the variable is accessed from two different threads. Check that it is safe to do so.

>**当继承QThread时，记住这个线程构建是在老的线程中，而run()执行在新的线程中。如果一个变量在两个线程中访问都被使用，需要确保这样操作是否安全。**

####2. 总结：
+ QThread使用，请使用`work-object`这种模式，逻辑操作在worker中实现，利用movetoThread与新的线程实现线程关联，调用start方法开启新的线程即可.
+ 如果非要使用继承QThread重写run方法来实现你的逻辑，请认真了解QThread的实现机制和注意事项,遵循如下原则：

    When to subclass and when not to?

    + If you do not really need an event loop in the thread, you should subclass.
    + If you need an event loop and handle signals and slots within the thread, you may not need to subclass.
    
    什么时候使用继承QThread，什么时候不使用？

    + 如果你不需要事件循环，你应当采用继承；
    + 如果你需要事件循环，并希望在新线程内处理信号与槽，你不需要继承。

+ 使用QThread的注意事项：
    + 1. 明确你的槽是在主线程还是子线程中执行
    + 2. 继承QThread，并在继承类中定义槽方法，然后使用movetoThread(this)/movetoThread(self)是一种糟糕的实践

如果你想了解更多，请参考下面这些文章，并认真研读qt官方文档，**`Qt的官方文档永远是解决你问题的最好帮手`**。

参考：

+ [You’re doing it wrong…](http://blog.qt.digia.com/blog/2010/06/17/youre-doing-it-wrong/)
+ [You were not doing so wrong.](http://woboq.com/blog/qthread-you-were-not-doing-so-wrong.html)
+ [When a QThread isn't a thread...](http://ilearnstuff.blogspot.jp/2012/08/when-qthread-isnt-thread.html)
+ [How to Use QThread in the Right Way (Part 1)](http://blog.debao.me/2013/08/how-to-use-qthread-in-the-right-way-part-1/)
+ [How to Use QThread in the Right Way (Part 2)](http://blog.debao.me/2013/08/how-to-use-qthread-in-the-right-way-part-2/)