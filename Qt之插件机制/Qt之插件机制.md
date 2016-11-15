Qt之插件机制
==================
####1. Qt提供了两种API用来创建插件

+ 一种`高级 API` 用来拓展Qt自身，譬如custom database drivers, image formats, text codecs, custom styles等
+ 一种`低级 API` 用来拓展Qt的应用程序


####2.高级API插件：拓展Qt自身

|Base Class	|Directory Name	|Qt Module	|Key Case Sensitivity|
| :-------- | --------:| :--: | :--: |
|QAccessibleBridgePlugin|	`accessiblebridge`|	Qt GUI|	Case Sensitive|
|QImageIOPlugin	|`imageformats`	|Qt GUI	|Case Sensitive|
|QPictureFormatPlugin|	`pictureformats`|	Qt GUI	|Case Sensitive|
|QAudioSystemPlugin	|`audio`	|Qt Multimedia|	Case Insensitive|
|QDeclarativeVideoBackendFactoryInterface|	`video/declarativevideobackend`|	Qt Multimedia	|Case Insensitive|
|QGstBufferPoolPlugin|	`video/bufferpool`|	Qt Multimedia|	Case Insensitive|
|QMediaPlaylistIOPlugin|	`playlistformats`	|Qt Multimedia	|Case Insensitive|
|QMediaResourcePolicyPlugin|	`resourcepolicy`|	Qt Multimedia	|Case Insensitive|
|QMediaServiceProviderPlugin|	`mediaservice`|	Qt Multimedia	|Case Insensitive|
|QSGVideoNodeFactoryPlugin	|`video/videonode`|	Qt Multimedia	|Case Insensitive|
|QBearerEnginePlugin	|`bearer`	|Qt Network|	Case Sensitive|
|QPlatformInputContextPlugin|	`platforminputcontexts`	|Qt Platform Abstraction	|Case Insensitive|
|QPlatformIntegrationPlugin	|`platforms`	|Qt Platform Abstraction	|Case Insensitive|
|QPlatformThemePlugin|	`platformthemes`	|Qt Platform Abstraction	|Case Insensitive|
|QGeoPositionInfoSourceFactory|	`position`	|Qt Positioning|	Case Sensitive|
|QPlatformPrinterSupportPlugin|	`printsupport`	|Qt Print Support|	Case Insensitive|
|QSGContextPlugin|	`scenegraph`	|Qt Quick	|Case Sensitive|
|QScriptExtensionPlugin|	`script`	|Qt Script	|Case Sensitive|
|QSensorGesturePluginInterface|	`sensorgestures`	|Qt Sensors|	Case Sensitive|
|QSensorPluginInterface|	`sensors`|	Qt Sensors|	Case Sensitive|
|QSqlDriverPlugin|	`sqldrivers`	|Qt SQL|	Case Sensitive|
|QIconEnginePlugin|	`iconengines`	|Qt SVG	|Case Insensitive
|QAccessiblePlugin|	`accessible`|	Qt Widgets	|Case Sensitive|
|QStylePlugin|	`styles`|	Qt Widgets	|Case Insensitive|

继承上面的这些插件类,继承重写一些方法，安装到指定的`插件标准目录`，即可拓展Qt自身


####3.低级API:拓展基于Qt的应用程序


1. 按照下面4个步骤即可在程序中使用插件:

+ Define a set of interfaces (classes with only pure virtual functions) used to talk to the plugins.
+ Use the Q_DECLARE_INTERFACE() macro to tell Qt's meta-object system about the interface.
		
			class MenuInterface
			{
			public:
			    virtual ~MenuInterface() {}
			
			    virtual QStringList keys() = 0;
			};
			
			#define MenuInterface_iid "com.deepin.dde-file-manager.MenuInterface"
			
			Q_DECLARE_INTERFACE(MenuInterface, MenuInterface_iid)

+ Use QPluginLoader in the application to load the plugins.
+ Use qobject_cast() to test whether a plugin implements a given interface.

			QDir pluginsDir(QDir::currentPath() + QDir::separator() +  "customplugins");
		    qDebug() << pluginsDir.absolutePath();
		    foreach (QString fileName, pluginsDir.entryList(QDir::Files)) {
		        QPluginLoader loader(pluginsDir.absoluteFilePath(fileName));
		        QObject *plugin = loader.instance();
		        if (plugin) {
		            qDebug() << fileName << loader.isLoaded() << loader.metaData();
		            qDebug() << qobject_interface_iid<MenuInterface*>();
		            qDebug() << qobject_cast<MenuInterface*>(plugin)->keys();
		        }else{
		            qDebug() << fileName << loader.isLoaded() << loader.metaData();
		            qDebug() << loader.errorString();
		        }
		    }


2.实现一个插件遵循下面4个步骤:

+ Declare a plugin class that inherits from QObject and from the interfaces that the plugin wants to provide.
+ Use the Q_INTERFACES() macro to tell Qt's meta-object system about the interfaces.
+ Export the plugin using the Q_PLUGIN_METADATA() macro.
+ Build the plugin using a suitable .pro file.

		class NormalPlugin :
		        public QObject,
		        public MenuInterface
		{
		    Q_OBJECT
		
		    Q_PLUGIN_METADATA(IID MenuInterface_iid FILE "NormalPlugin.json")
		    Q_INTERFACES(MenuInterface)
		
		public:
		    NormalPlugin(QObject *parent = 0);
		
		    QStringList keys();
		};


####4.定位Qt插件

+ 基于高级API定制的插件，Qt默认会到【QTDIR/plugins/`style dirname`】（`QTDIR为Qt的安装目录`）中寻找插件；如果使用QCoreApplication::addLibraryPath(`custompath`) 增加了自定义插件路径，那么Qt会优先寻找【`custompath`/`style dirname`】中的插件。
+ 基于低级API定制的插件，使用QPluginLoader(const QString & fileName, QObject * parent = 0)加载
> + Constructs a plugin loader with the given parent that will load the plugin specified by fileName.
+ To be loadable, the file's suffix must be a valid suffix for a loadable library in accordance with the platform, e.g. .so on Unix, - .dylib on Mac OS X, and .dll on Windows. The suffix can be verified with QLibrary::isLibrary().


####5.静态插件

创建 Static Plugins遵循下面三个步骤即可:
+ Add CONFIG += static to your plugin's .pro file.
+ Use the Q_IMPORT_PLUGIN() macro in your application.
+ Link your application with your plugin library using LIBS in the .pro file.


####6. Qt内部是如何使用PluginFactory去加载插件的

+  QFactoryLoader的头文件：

		class Q_CORE_EXPORT QFactoryLoader : public QObject
		{
		    Q_OBJECT
		    Q_DECLARE_PRIVATE(QFactoryLoader)
		
		public:
		    explicit QFactoryLoader(const char *iid,
		                   const QString &suffix = QString(),
		                   Qt::CaseSensitivity = Qt::CaseSensitive);
		    ~QFactoryLoader();
		
		    QList<QJsonObject> metaData() const;
		    QObject *instance(int index) const;
		
		#if defined(Q_OS_UNIX) && !defined (Q_OS_MAC)
		    QLibraryPrivate *library(const QString &key) const;
		#endif
		
		    QMultiMap<int, QString> keyMap() const;
		    int indexOf(const QString &needle) const;
		
		    void update();
		
		    static void refreshAll();
		};
		
		template <class PluginInterface, class FactoryInterface>
		    PluginInterface *qLoadPlugin(const QFactoryLoader *loader, const QString &key)
		{
		    const int index = loader->indexOf(key);
		    if (index != -1) {
		        QObject *factoryObject = loader->instance(index);
		        if (FactoryInterface *factory = qobject_cast<FactoryInterface *>(factoryObject))
		            if (PluginInterface *result = factory->create(key))
		                return result;
		    }
		    return 0;
		}
		
		template <class PluginInterface, class FactoryInterface, class Parameter1>
		PluginInterface *qLoadPlugin1(const QFactoryLoader *loader,
		                              const QString &key,
		                              const Parameter1 &parameter1)
		{
		    const int index = loader->indexOf(key);
		    if (index != -1) {
		        QObject *factoryObject = loader->instance(index);
		        if (FactoryInterface *factory = qobject_cast<FactoryInterface *>(factoryObject))
		            if (PluginInterface *result = factory->create(key, parameter1))
		                return result;
		    }
		    return 0;
		}
+ QFactoryLoader的实现
	
			Q_GLOBAL_STATIC(QList<QFactoryLoader *>, qt_factory_loaders)
		
		Q_GLOBAL_STATIC_WITH_ARGS(QMutex, qt_factoryloader_mutex, (QMutex::Recursive))
		
		namespace {
		
		// avoid duplicate QStringLiteral data:
		inline QString iidKeyLiteral() { return QStringLiteral("IID"); }
		inline QString versionKeyLiteral() { return QStringLiteral("version"); }
		inline QString metaDataKeyLiteral() { return QStringLiteral("MetaData"); }
		inline QString keysKeyLiteral() { return QStringLiteral("Keys"); }
		
		}
		
		class QFactoryLoaderPrivate : public QObjectPrivate
		{
		    Q_DECLARE_PUBLIC(QFactoryLoader)
		public:
		    QFactoryLoaderPrivate(){}
		    ~QFactoryLoaderPrivate();
		    mutable QMutex mutex;
		    QByteArray iid;
		    QList<QLibraryPrivate*> libraryList;
		    QMap<QString,QLibraryPrivate*> keyMap;
		    QString suffix;
		    Qt::CaseSensitivity cs;
		    QStringList loadedPaths;
		
		    void unloadPath(const QString &path);
		};
		
		QFactoryLoaderPrivate::~QFactoryLoaderPrivate()
		{
		    for (int i = 0; i < libraryList.count(); ++i) {
		        QLibraryPrivate *library = libraryList.at(i);
		        library->unload();
		        library->release();
		    }
		}
		
		QFactoryLoader::QFactoryLoader(const char *iid,
		                               const QString &suffix,
		                               Qt::CaseSensitivity cs)
		    : QObject(*new QFactoryLoaderPrivate)
		{
		    moveToThread(QCoreApplicationPrivate::mainThread());
		    Q_D(QFactoryLoader);
		    d->iid = iid;
		    d->cs = cs;
		    d->suffix = suffix;
		
		
		    QMutexLocker locker(qt_factoryloader_mutex());
		    update();
		    qt_factory_loaders()->append(this);
		}
		
		
		
		void QFactoryLoader::update()
		{
		#ifdef QT_SHARED
		    Q_D(QFactoryLoader);
		    QStringList paths = QCoreApplication::libraryPaths();
		    for (int i = 0; i < paths.count(); ++i) {
		        const QString &pluginDir = paths.at(i);
		        // Already loaded, skip it...
		        if (d->loadedPaths.contains(pluginDir))
		            continue;
		        d->loadedPaths << pluginDir;
		
		        QString path = pluginDir + d->suffix;
		
		        if (qt_debug_component())
		            qDebug() << "QFactoryLoader::QFactoryLoader() checking directory path" << path << "...";
		
		        if (!QDir(path).exists(QLatin1String(".")))
		            continue;
		
		        QStringList plugins = QDir(path).entryList(QDir::Files);
		        QLibraryPrivate *library = 0;
		
		#ifdef Q_OS_MAC
		        // Loading both the debug and release version of the cocoa plugins causes the objective-c runtime
		        // to print "duplicate class definitions" warnings. Detect if QFactoryLoader is about to load both,
		        // skip one of them (below).
		        //
		        // ### FIXME find a proper solution
		        //
		        const bool isLoadingDebugAndReleaseCocoa = plugins.contains(QStringLiteral("libqcocoa_debug.dylib"))
		                && plugins.contains(QStringLiteral("libqcocoa.dylib"));
		#endif
		        for (int j = 0; j < plugins.count(); ++j) {
		            QString fileName = QDir::cleanPath(path + QLatin1Char('/') + plugins.at(j));
		
		#ifdef Q_OS_MAC
		            if (isLoadingDebugAndReleaseCocoa) {
		#ifdef QT_DEBUG
		               if (fileName.contains(QStringLiteral("libqcocoa.dylib")))
		                   continue;    // Skip release plugin in debug mode
		#else
		               if (fileName.contains(QStringLiteral("libqcocoa_debug.dylib")))
		                   continue;    // Skip debug plugin in release mode
		#endif
		            }
		#endif
		            if (qt_debug_component()) {
		                qDebug() << "QFactoryLoader::QFactoryLoader() looking at" << fileName;
		            }
		            library = QLibraryPrivate::findOrCreate(QFileInfo(fileName).canonicalFilePath());
		            if (!library->isPlugin()) {
		                if (qt_debug_component()) {
		                    qDebug() << library->errorString;
		                    qDebug() << "         not a plugin";
		                }
		                library->release();
		                continue;
		            }
		
		            QStringList keys;
		            bool metaDataOk = false;
		
		            QString iid = library->metaData.value(iidKeyLiteral()).toString();
		            if (iid == QLatin1String(d->iid.constData(), d->iid.size())) {
		                QJsonObject object = library->metaData.value(metaDataKeyLiteral()).toObject();
		                metaDataOk = true;
		
		                QJsonArray k = object.value(keysKeyLiteral()).toArray();
		                for (int i = 0; i < k.size(); ++i)
		                    keys += d->cs ? k.at(i).toString() : k.at(i).toString().toLower();
		            }
		            if (qt_debug_component())
		                qDebug() << "Got keys from plugin meta data" << keys;
		
		
		            if (!metaDataOk) {
		                library->release();
		                continue;
		            }
		
		            int keyUsageCount = 0;
		            for (int k = 0; k < keys.count(); ++k) {
		                // first come first serve, unless the first
		                // library was built with a future Qt version,
		                // whereas the new one has a Qt version that fits
		                // better
		                const QString &key = keys.at(k);
		                QLibraryPrivate *previous = d->keyMap.value(key);
		                int prev_qt_version = 0;
		                if (previous) {
		                    prev_qt_version = (int)previous->metaData.value(versionKeyLiteral()).toDouble();
		                }
		                int qt_version = (int)library->metaData.value(versionKeyLiteral()).toDouble();
		                if (!previous || (prev_qt_version > QT_VERSION && qt_version <= QT_VERSION)) {
		                    d->keyMap[key] = library;
		                    ++keyUsageCount;
		                }
		            }
		            if (keyUsageCount || keys.isEmpty())
		                d->libraryList += library;
		            else
		                library->release();
		        }
		    }
		#else
		    Q_D(QFactoryLoader);
		    if (qt_debug_component()) {
		        qDebug() << "QFactoryLoader::QFactoryLoader() ignoring" << d->iid
		                 << "since plugins are disabled in static builds";
		    }
		#endif
		}
		
		QFactoryLoader::~QFactoryLoader()
		{
		    QMutexLocker locker(qt_factoryloader_mutex());
		    qt_factory_loaders()->removeAll(this);
		}
		
		QList<QJsonObject> QFactoryLoader::metaData() const
		{
		    Q_D(const QFactoryLoader);
		    QMutexLocker locker(&d->mutex);
		    QList<QJsonObject> metaData;
		    for (int i = 0; i < d->libraryList.size(); ++i)
		        metaData.append(d->libraryList.at(i)->metaData);
		
		    foreach (const QStaticPlugin &plugin, QPluginLoader::staticPlugins()) {
		        const QJsonObject object = plugin.metaData();
		        if (object.value(iidKeyLiteral()) != QLatin1String(d->iid.constData(), d->iid.size()))
		            continue;
		        metaData.append(object);
		    }
		    return metaData;
		}
		
		QObject *QFactoryLoader::instance(int index) const
		{
		    Q_D(const QFactoryLoader);
		    if (index < 0)
		        return 0;
		
		    if (index < d->libraryList.size()) {
		        QLibraryPrivate *library = d->libraryList.at(index);
		        if (library->instance || library->loadPlugin()) {
		            if (!library->inst)
		                library->inst = library->instance();
		            QObject *obj = library->inst.data();
		            if (obj) {
		                if (!obj->parent())
		                    obj->moveToThread(QCoreApplicationPrivate::mainThread());
		                return obj;
		            }
		        }
		        return 0;
		    }
		
		    index -= d->libraryList.size();
		    QVector<QStaticPlugin> staticPlugins = QPluginLoader::staticPlugins();
		    for (int i = 0; i < staticPlugins.count(); ++i) {
		        const QJsonObject object = staticPlugins.at(i).metaData();
		        if (object.value(iidKeyLiteral()) != QLatin1String(d->iid.constData(), d->iid.size()))
		            continue;
		
		        if (index == 0)
		            return staticPlugins.at(i).instance();
		        --index;
		    }
		
		    return 0;
		}
		
		#if defined(Q_OS_UNIX) && !defined (Q_OS_MAC)
		QLibraryPrivate *QFactoryLoader::library(const QString &key) const
		{
		    Q_D(const QFactoryLoader);
		    return d->keyMap.value(d->cs ? key : key.toLower());
		}
		#endif
		
		void QFactoryLoader::refreshAll()
		{
		    QMutexLocker locker(qt_factoryloader_mutex());
		    QList<QFactoryLoader *> *loaders = qt_factory_loaders();
		    for (QList<QFactoryLoader *>::const_iterator it = loaders->constBegin();
		         it != loaders->constEnd(); ++it) {
		        (*it)->update();
		    }
		}
		
		QMultiMap<int, QString> QFactoryLoader::keyMap() const
		{
		    QMultiMap<int, QString> result;
		    const QString metaDataKey = metaDataKeyLiteral();
		    const QString keysKey = keysKeyLiteral();
		    const QList<QJsonObject> metaDataList = metaData();
		    for (int i = 0; i < metaDataList.size(); ++i) {
		        const QJsonObject metaData = metaDataList.at(i).value(metaDataKey).toObject();
		        const QJsonArray keys = metaData.value(keysKey).toArray();
		        const int keyCount = keys.size();
		        for (int k = 0; k < keyCount; ++k)
		            result.insert(i, keys.at(k).toString());
		    }
		    return result;
		}
		
		int QFactoryLoader::indexOf(const QString &needle) const
		{
		    const QString metaDataKey = metaDataKeyLiteral();
		    const QString keysKey = keysKeyLiteral();
		    const QList<QJsonObject> metaDataList = metaData();
		    for (int i = 0; i < metaDataList.size(); ++i) {
		        const QJsonObject metaData = metaDataList.at(i).value(metaDataKey).toObject();
		        const QJsonArray keys = metaData.value(keysKey).toArray();
		        const int keyCount = keys.size();
		        for (int k = 0; k < keyCount; ++k) {
		            if (!keys.at(k).toString().compare(needle, Qt::CaseInsensitive))
		                return i;
		        }
		    }
		    return -1;
		}

+  QGenericPluginFactory的具体实现：

			QT_BEGIN_NAMESPACE
			
			#if !defined(Q_OS_WIN32) || defined(QT_SHARED)
			#ifndef QT_NO_LIBRARY
			
			Q_GLOBAL_STATIC_WITH_ARGS(QFactoryLoader, loader,
			    (QGenericPluginFactoryInterface_iid,
			     QLatin1String("/generic"), Qt::CaseInsensitive))
			
			#endif //QT_NO_LIBRARY
			#endif //QT_SHARED
			
			/*!
			    \class QGenericPluginFactory
			    \ingroup plugins
			    \inmodule QtGui
			
			    \brief The QGenericPluginFactory class creates plugin drivers.
			
			    \sa QGenericPlugin
			*/
			
			/*!
			    Creates the driver specified by \a key, using the given \a specification.
			
			    Note that the keys are case-insensitive.
			
			    \sa keys()
			*/
			QObject *QGenericPluginFactory::create(const QString& key, const QString &specification)
			{
			#if (!defined(Q_OS_WIN32) || defined(QT_SHARED)) && !defined(QT_NO_LIBRARY)
			    const QString driver = key.toLower();
			    if (QObject *object = qLoadPlugin1<QObject, QGenericPlugin>(loader(), driver, specification))
			        return object;
			#else // (!Q_OS_WIN32 || QT_SHARED) && !QT_NO_LIBRARY
			    Q_UNUSED(key)
			    Q_UNUSED(specification)
			#endif
			    return 0;
			}
			
			/*!
			    Returns the list of valid keys, i.e. the available mouse drivers.
			
			    \sa create()
			*/
			QStringList QGenericPluginFactory::keys()
			{
			    QStringList list;
			
			#if !defined(Q_OS_WIN32) || defined(QT_SHARED)
			#ifndef QT_NO_LIBRARY
			    typedef QMultiMap<int, QString> PluginKeyMap;
			    typedef PluginKeyMap::const_iterator PluginKeyMapConstIterator;
			
			    const PluginKeyMap keyMap = loader()->keyMap();
			    const PluginKeyMapConstIterator cend = keyMap.constEnd();
			    for (PluginKeyMapConstIterator it = keyMap.constBegin(); it != cend; ++it)
			        if (!list.contains(it.value()))
			            list += it.value();
			#endif //QT_NO_LIBRARY
			#endif
			    return list;
			}

+ 增加LibraryPath后插件自动加载是如何实现的

			/*!
		
		    Sets the list of directories to search when loading libraries to
		    \a paths. All existing paths will be deleted and the path list
		    will consist of the paths given in \a paths.
		
		    The library paths are reset to the default when an instance of
		    QCoreApplication is destructed.
		
		    \sa libraryPaths(), addLibraryPath(), removeLibraryPath(), QLibrary
		 */
		void QCoreApplication::setLibraryPaths(const QStringList &paths)
		{
		    QMutexLocker locker(libraryPathMutex());
		
		    // setLibraryPaths() is considered a "remove everything and then add some new ones" operation.
		    // When the application is constructed it should still amend the paths. So we keep the originals
		    // around, and even create them if they don't exist, yet.
		    if (!coreappdata()->app_libpaths)
		        libraryPaths();
		
		    if (coreappdata()->manual_libpaths)
		        *(coreappdata()->manual_libpaths) = paths;
		    else
		        coreappdata()->manual_libpaths.reset(new QStringList(paths));
		
		    locker.unlock();
		    QFactoryLoader::refreshAll();
		}
		
		/*!
		  Prepends \a path to the beginning of the library path list, ensuring that
		  it is searched for libraries first. If \a path is empty or already in the
		  path list, the path list is not changed.
		
		  The default path list consists of a single entry, the installation
		  directory for plugins.  The default installation directory for plugins
		  is \c INSTALL/plugins, where \c INSTALL is the directory where Qt was
		  installed.
		
		  The library paths are reset to the default when an instance of
		  QCoreApplication is destructed.
		
		  \sa removeLibraryPath(), libraryPaths(), setLibraryPaths()
		 */
		void QCoreApplication::addLibraryPath(const QString &path)
		{
		    if (path.isEmpty())
		        return;
		
		    QString canonicalPath = QDir(path).canonicalPath();
		    if (canonicalPath.isEmpty())
		        return;
		
		    QMutexLocker locker(libraryPathMutex());
		
		    QStringList *libpaths = coreappdata()->manual_libpaths.data();
		    if (libpaths) {
		        if (libpaths->contains(canonicalPath))
		            return;
		    } else {
		        // make sure that library paths are initialized
		        libraryPaths();
		        QStringList *app_libpaths = coreappdata()->app_libpaths.data();
		        if (app_libpaths->contains(canonicalPath))
		            return;
		
		        coreappdata()->manual_libpaths.reset(libpaths = new QStringList(*app_libpaths));
		    }
		
		    libpaths->prepend(canonicalPath);
		    locker.unlock();
		    QFactoryLoader::refreshAll();
		}
		
		/*!
		    Removes \a path from the library path list. If \a path is empty or not
		    in the path list, the list is not changed.
		
		    The library paths are reset to the default when an instance of
		    QCoreApplication is destructed.
		
		    \sa addLibraryPath(), libraryPaths(), setLibraryPaths()
		*/
		void QCoreApplication::removeLibraryPath(const QString &path)
		{
		    if (path.isEmpty())
		        return;
		
		    QString canonicalPath = QDir(path).canonicalPath();
		    if (canonicalPath.isEmpty())
		        return;
		
		    QMutexLocker locker(libraryPathMutex());
		
		    QStringList *libpaths = coreappdata()->manual_libpaths.data();
		    if (libpaths) {
		        if (libpaths->removeAll(canonicalPath) == 0)
		            return;
		    } else {
		        // make sure that library paths is initialized
		        libraryPaths();
		        QStringList *app_libpaths = coreappdata()->app_libpaths.data();
		        if (!app_libpaths->contains(canonicalPath))
		            return;
		
		        coreappdata()->manual_libpaths.reset(libpaths = new QStringList(*app_libpaths));
		        libpaths->removeAll(canonicalPath);
		    }
		
		    locker.unlock();
		    QFactoryLoader::refreshAll();
		}
通过上述源码可以大致明白Qt是如何利用PluginFactory去创建插件实例的。
