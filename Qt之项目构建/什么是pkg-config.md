什么是`pkg-config`
=====================================
####1. pkg-config介绍
　　pkg-config用来检索系统中安装库文件的信息。典型的是用作库的编译和连接。如在Makefile中：

		program: program.c
			cc program.c `pkg-config --cflags --libs gnomeui`

#####2. pkg-config功能
　　一般来说，如果库的头文件不在/usr/include目录中，那么在编译的时候需要用-I参数指定其路径。由于同一个库在不同系统上可能位于不同的目录下，用户安装库的时候也可以将库安装在不同的目录下，所以即使使用同一个库，由于库的路径的不同，造成了用-I参数指定的头文件的路径和在连接时使用-L参数指定lib库的路径都可能不同，其结果就是造成了编译命令界面的不统一。可能由于编译，连接的不一致，造成同一份程序从一台机器copy到另一台机器时就可能会出现问题。
　　`pkg-config 就是用来解决编译连接界面不统一问题的一个工具。`
　　它的基本思想：**pkg-config是通过库提供的一个.pc文件获得库的各种必要信息的，包括版本信息、编译和连接需要的参数等。需要的时候可以通过pkg-config提供的参数(–cflags, –libs)，将所需信息提取出来供编译和连接使用。这样，不管库文件安装在哪，通过库对应的.pc文件就可以准确定位,可以使用相同的编译和连接命令，使得编译和连接界面统一。**
　　它提供的主要功能有:
+   检查库的版本号。如果所需库的版本不满足要求，打印出错误信息，避免连接错误版本的库文件。
+  获得编译预处理参数，如宏定义，头文件的路径。
+  获得编译参数，如库及其依赖的其他库的位置，文件名及其他一些连接参数。
+  自动加入所依赖的其他库的设置。

#####3. 常用命令
| 命令      |    说明 |  
| :-------- | --------:|
| --list-all  | list all known packages`列出支持pkg-config的库` |
| --cflags| output all pre-processor and compiler flags`列出所有预编译和编译标志`|
| --libs| output all linker flags`列出所有的链接标志`| 
|--validate|validate a package's .pc file `校验pc文件` |


	 pkg-config --cflags --libs dtklog         
	 -I/usr/include/libdtk-1.0/DLog -ldtklog

#####4.  pc文件

	prefix=/usr
	exec_prefix=${prefix}
	libdir=${prefix}/lib/x86_64-linux-gnu
	includedir=${prefix}/include/libdtk-1.0/DLog
	
	
	Name: DTK_LOG
	Description: Deepin Tool Kit Log Module
	Version: 1.0
	Libs: -ldtklog
	Libs.private: -lQt5Core -lpthread
	Cflags: -I${includedir}

#####5. pkg-config查找pc路径 和 环境变量PKG_CONFIG_PATH
   > pkg-config一般在`/usr/lib/pkgconfig` 或者是`/usr/lib/x86_64-linux-gnu/pkgconfig/`目录下寻找*.pc文件;
 
 环境变量PKG_CONFIG_PATH是用来设置.pc文件的搜索路径的，pkg-config按照设置路径的先后顺序进行搜索，直到找到指定的.pc 文件为止。这样，库的头文件的搜索路径的设置实际上就变成了对.pc文件搜索路径的设置。
　　在安装完一个需要使用的库后，比如Glib，一是将相应的.pc文件，如glib-2.0.pc拷贝到/usr/lib/pkgconfig目录下，二是通过设置环境变量PKG_CONFIG_PATH添加glib-2.0.pc文件的搜索路径。
　　添加环境变量PKG_CONFIG_PATH，在bash中应该进行如下设置：
		
		$ export PKG_CONFIG_PATH=/opt/gtk/lib/pkgconfig:$PKG_CONFIG_PATH
　　
　　可以执行下面的命令检查是否 /opt/gtk/lib/pkgconfig 路径已经设置在PKG_CONFIG_PATH环境变量中：
				
		$ echo $PKG_CONFIG_PATH
　　
　　这样设置之后，使用Glib库的其它程序或库在编译的时候pkg-config就知道首先要到/opt/gtk/lib/pkgconfig这个目录中去寻找glib-2.0.pc了(GTK+和其它的依赖库的.pc文件也将拷贝到这里，也会首先到这里搜索它们对应的.pc文件)。之后，通过pkg-config就可以把其中库的编译和连接参数提取出来供程序在编译和连接时使用。

**注意**：
　　环境变量的设置只对当前的终端窗口有效。如果到了没有进行上述设置的终端窗口中，pkg-config将找不到新安装的glib-2.0.pc文件、从而可能使后面进行的安装(如Glib之后的Atk的安装)无法进行。
　　在我们采用的安装方案中，由于是使用环境变量对GTK+及其依赖库进行的设置，所以当系统重新启动、或者新开一个终端窗口之后，如果想使用新安装的GTK+库，需要如上面那样重新设置PKG_CONFIG_PATH和LD_LIBRARY_PATH环境变量。
　　这种使用GTK+的方法，在使用之前多了一个对库进行设置的过程。虽然显得稍微繁琐了一些，但却是一种最安全的使用GTK+库的方式，不会对系统上已经存在的使用了GTK+库的程序(比如GNOME桌面)带来任何冲击。