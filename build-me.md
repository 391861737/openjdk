## 0. 预备 / Preparation
* 下载并安装`VS2017`
* 下载并安装`JDK1.8`

## 1. 获取并配置Cygwin / Get and configure the Cygwin
* 下载 <http://www.cygwin.com/setup-x86_64.exe> 并安装  
* 然后通过`cygwin`包管理器添加如下包  

> Binary-Name&nbsp;|Category&nbsp;|Package&nbsp;|Description
:---|:---|:---|:---
ar.exe|Devel|binutils|The GNU assembler, linker and binary utilities
make.exe|Devel|make|The GNU version of the 'make' utility built for CYGWIN
m4.exe|Interpreters&nbsp;|m4|GNU implementation of the traditional Unix macro processor
cpio.exe|Utils|cpio|A program to manage archives of files
gawk.exe|Utils|awk|Pattern-directed scanning and processing language
file.exe|Utils|file|Determines file type using 'magic' numbers
zip.exe|Archive|zip|Package and compress (archive) files
unzip.exe|Archive|unzip|Extract compressed files in a ZIP archive
free.exe|System|procps|Display amount of free and used memory in the system

## 2. 获取源码 / Get the source code
* 在`cygwin`命令行中，通过`mkdir`创建目录`<work dir>`，并进入该目录
* 在`cygwin`命令行中，执行如下命令  
	```
	hg clone http://hg.openjdk.java.net/jdk8/jdk8 <name>
	cd <name>
	bash ./get_source.sh
## 3. 规避路径中的空格 / Avoid blank spaces in paths
* 对路径中含有空格的目录创建软链接
	```
	D:\VS        <---> D:\Program Files (x86)\Microsoft Visual Studio
	D:\WinKits   <---> D:\Windows Kits
	D:\Java      <---> D:\Program Files\Java
	D:\Mercurial <---> D:\Program Files (x86)\Mercurial
	```

* 配置`JAVA_HOME`指向不带空格的软链接路径
* 修改`D:\Program Files (x86)\Microsoft Visual Studio\2017\Community\Common7\Tools\vsdevcmd\core\winsdk.bat`，
   使其获取到的`WindowsSdkDir`为不含空格的软链接路径

## 4. 获取并编译Freetype / Get and compile Freetype
* 下载最新版freetype源码 <https://sourceforge.net/projects/freetype/files/freetype2/>，进入目录`freetype-x.x.x\builds\windows\vc2010`，分别编译动态链接库dll及静态库lib
* 在`freetype-x.x.x`下创建目录`lib`
* 编译好的`freetype.dll`及`freetype.lib`在`freetype-x.x.x\objs`目录下，将它们复制到刚刚创建的`freetype-x.x.x\lib`目录下
* 配置环境变量`FREETYPE_CFLAGS=...\freetype-x.x.x\include`，`FREETYPE_LIBS=...\freetype-x.x.x\lib`，并在环境变量`Path`中引入它们（`%FREETYPE_CFLAGS%;%FREETYPE_LIBS%`）

## 5. 执行configure脚本 / Configure JDK8

* 用`cygwin bash`打开jdk8目录，执行`configure`脚本，指定`Freetype`目录、目标位元、`Visual Studio`目录，并启用调试，例如:  
	```
	./configure --with-freetype=/cygdrive/e/Coding/opensource/freetype-2.10.0 --enable-debug --with-target-bits=64 --with-tools-dir="/cygdrive/d/VS/2017/Community"
	```
	
* 按照`configure`所提示的错误信息修改—— 

	+ 指定正确的vcvars32/64.bat脚本位置，并将所有的$with_toolsdir改为$with_tools_dir，因为$with_toolsdir获取不到参数--with-tools-dir的值
	```
	@@ -16773,17 +16773,17 @@ $as_echo "no" >&6; }
	   # First-hand choice is to locate and run the vsvars bat file.

	   if test "x$OPENJDK_TARGET_CPU_BITS" = x32; then
	-    VCVARSFILE="vc/bin/vcvars32.bat"
	+    VCVARSFILE="VC/Auxiliary/Build/vcvars32.bat"
	   else
	-    VCVARSFILE="vc/bin/amd64/vcvars64.bat"
	+    VCVARSFILE="VC/Auxiliary/Build/vcvars64.bat"
	   fi

	   VS_ENV_CMD=""
	   VS_ENV_ARGS=""
	-  if test "x$with_toolsdir" != x; then
	+  if test "x$with_tools_dir" != x; then

	   if test "x$VS_ENV_CMD" = x; then
	-    VS100BASE="$with_toolsdir/../.."
	+    VS100BASE="$with_tools_dir"
		 METHOD="--with-tools-dir"

	   windows_path="$VS100BASE"
	————————
	@@ -16811,7 +16811,7 @@ $as_echo "$as_me: Warning: $VCVARSFILE is missing, this is probably Visual Studi

	   fi

	-  if test "x$with_toolsdir" != x && test "x$VS_ENV_CMD" = x; then
	+  if test "x$with_tools_dir" != x && test "x$VS_ENV_CMD" = x; then
		 # Having specified an argument which is incorrect will produce an instant failure;
		 # we should not go on looking
		 { $as_echo "$as_me:${as_lineno-$LINENO}: The path given by --with-tools-dir does not contain a valid Visual Studio installation" >&5
	```
	+ 修改用来判断VS编译器版本的正则表达式
	```
	@@ -20037,9 +20037,9 @@ $as_echo "$as_me: The result from running with -V was: \"$COMPILER_VERSION_TEST\
		 # First line typically looks something like:
		 # Microsoft (R) 32-bit C/C++ Optimizing Compiler Version 16.00.40219.01 for 80x86
		 COMPILER_VERSION_TEST=`$COMPILER 2>&1 | $HEAD -n 1 | $TR -d '\r'`
	-    COMPILER_VERSION=`$ECHO $COMPILER_VERSION_TEST | $SED -n "s/^.*Version \([1-9][0-9.]*\) .*/\1/p"`
	+    COMPILER_VERSION=`$ECHO $COMPILER_VERSION_TEST | $SED -n "s/^.* \([1-9][0-9.]*\) .*/\1/p"`
		 COMPILER_VENDOR="Microsoft CL.EXE"
	-    COMPILER_CPU_TEST=`$ECHO $COMPILER_VERSION_TEST | $SED -n "s/^.* for \(.*\)$/\1/p"`
	+    COMPILER_CPU_TEST=`$ECHO $COMPILER_VERSION_TEST | $SED -n "s/^.* \([x8064]*\) .*/\1/p"`
		 if test "x$OPENJDK_TARGET_CPU" = "xx86"; then
		   if test "x$COMPILER_CPU_TEST" != "x80x86"; then
			 as_fn_error $? "Target CPU mismatch. We are building for $OPENJDK_TARGET_CPU but CL is for \"$COMPILER_CPU_TEST\"; expected \"80x86\"." "$LINENO" 5
	————————
	@@ -21616,9 +21616,9 @@ $as_echo "$as_me: The result from running with -V was: \"$COMPILER_VERSION_TEST\
		 # First line typically looks something like:
		 # Microsoft (R) 32-bit C/C++ Optimizing Compiler Version 16.00.40219.01 for 80x86
		 COMPILER_VERSION_TEST=`$COMPILER 2>&1 | $HEAD -n 1 | $TR -d '\r'`
	-    COMPILER_VERSION=`$ECHO $COMPILER_VERSION_TEST | $SED -n "s/^.*Version \([1-9][0-9.]*\) .*/\1/p"`
	+    COMPILER_VERSION=`$ECHO $COMPILER_VERSION_TEST | $SED -n "s/^.* \([1-9][0-9.]*\) .*/\1/p"`
		 COMPILER_VENDOR="Microsoft CL.EXE"
	-    COMPILER_CPU_TEST=`$ECHO $COMPILER_VERSION_TEST | $SED -n "s/^.* for \(.*\)$/\1/p"`
	+    COMPILER_CPU_TEST=`$ECHO $COMPILER_VERSION_TEST | $SED -n "s/^.* \([x8064]*\) .*/\1/p"`
		 if test "x$OPENJDK_TARGET_CPU" = "xx86"; then
		   if test "x$COMPILER_CPU_TEST" != "x80x86"; then
			 as_fn_error $? "Target CPU mismatch. We are building for $OPENJDK_TARGET_CPU but CL is for \"$COMPILER_CPU_TEST\"; expected \"80x86\"." "$LINENO" 5
	```
	+ 搜索`D_STATIC_CPPLIB`，去除`-D_STATIC_CPPLIB -D_DISABLE_DEPRECATE_STATIC_CPPLIB`参数选项
	```
	@@ -29530,8 +29530,8 @@ fi
		   LDFLAGS_CXX_JDK="$LDFLAGS_CXX_JDK -norunpath -xnolib"
		   ;;
		 cl )
	      CCXXFLAGS_JDK="$CCXXFLAGS $CCXXFLAGS_JDK -Zi -MD -Zc:wchar_t- -W3 -wd4800 \
	-      -D_STATIC_CPPLIB -D_DISABLE_DEPRECATE_STATIC_CPPLIB -DWIN32_LEAN_AND_MEAN \
	+      -DWIN32_LEAN_AND_MEAN \
		   -D_CRT_SECURE_NO_DEPRECATE -D_CRT_NONSTDC_NO_DEPRECATE \
		   -DWIN32 -DIAL"
	```  
## 6. 编译 / Make

* 添加VC编译工具的`Path`环境变量，例如：  
`D:\VS\2017\Community\VC\Tools\MSVC\14.16.27023\bin\Hostx64\x64`
* 修改`jdk8\hotspot\make\windows\get_msc_ver.sh`，在脚本开头添加:  
	```
	FORCE_MSC_VER=1700
	FORCE_LD_VER=1100
	```
* 通过VS自带的脚本启动VS命令行（例如: `D:\VS\2017\Community\Common7\Tools\LaunchDevCmd.bat`），`cd`进入jdk根目录下的`\hotspot\make\windows`目录，设置变量`HOTSPOTMKSHOME`为`Cygwin`的bin目录（例如: `set HOTSPOTMKSHOME=D:\Lang\cygwin64\bin`），然后执行脚本`create.bat`
* `create.bat`脚本会在jdk根目录下的`E:\Coding\opensource\openjdk\jdk8\hotspot\build\vs-amd64`生成对应版本的vs项目，通过`Visual Studio`打开它，然后对弹出的升级提示框选择"确定"升级，然后修改`项目属性 -> 配置属性 -> C/C++ -> 将警告视为错误`为否。对项目进行`Build`（或`生成`），针对错误信息进行处理——

* 针对错误`error C3688`，参见<https://msdn.microsoft.com/en-us/library/bb531344.aspx>`§ String literals followed by macros`，在字符串与宏之间添加空格

* 针对错误`error C2065 “timezone”: 未声明的标识符`，将`...\hotspot\src\share\vm\runtime\os.cpp` `129行` 附近的下述代码
	```
	#if defined(_ALLBSD_SOURCE)
	  const time_t zone = (time_t) time_struct.tm_gmtoff;
	#else
	  const time_t zone = timezone;
	#endif
	```
	修改为
	```
	#if defined(_ALLBSD_SOURCE)
	  const time_t zone = (time_t) time_struct.tm_gmtoff;
	#else
	  #if _MSC_VER < 1900
		const time_t zone = timezone;
	  #else
		const time_t zone = 0;
		_get_timezone((long *)&zone);
	  #endif
	#endif
	```
	
* VS中编译成功，重新运行`make images`，对错误信息进行处理——
* 针对错误`error C2956`，参见<https://msdn.microsoft.com/en-us/library/bb531344.aspx>`§ Placement new and delete`， 将`...\hotspot\src\share\vm\adlc\arena.hpp` `70行` 附近的下述代码
	```
	// Linked list of raw memory chunks
	class Chunk: public CHeapObj {
	 public:
	  void* operator new(size_t size, size_t length) throw();
	  void  operator delete(void* p, m_size_t length);
	  Chunk(size_t length);
	...  
	```
	修改为
	```
	enum class m_size_t : size_t {};

	// Linked list of raw memory chunks
	class Chunk: public CHeapObj {
	 public:
	  void* operator new(size_t size, m_size_t length) throw();
	  void  operator delete(void* p, m_size_t length);
	  Chunk(size_t length);
	```
* 针对错误`error C2220`，修改`...\hotspot\make\windows\makefiles\compile.make` `56行`，去掉`/WX`的编译参数
