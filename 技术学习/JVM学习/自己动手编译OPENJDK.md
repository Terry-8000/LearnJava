## 自己编译JDK（macOS）

### 1. 获取JDK源码

官网提供了两种下载方式：

1. 通过Mercurial版本控制工具下载\([http://hg.openjdk.java.net/\)；](http://hg.openjdk.java.net/%29；)
2. 通过官网提供的压缩包下载并解压（目前只提供了jdk6的版本）：[http://download.java.net/openjdk/jdk6。](http://download.java.net/openjdk/jdk6。)

由于一些版本问题，踩过坑后，笔者毅然采取了第一种方式，下载了jdk7u版本（ hg clone [http://hg.openjdk.java.net/jdk7u/jdk7u-dev/](http://hg.openjdk.java.net/jdk7u/jdk7u-dev/) ），如果没有安装Mercurial，请先安装一下。

下载好后，可以参考README-builds.html文档，里面有很多需要注意的细节。官方文档称编译jdk需要很高的专业技术，Sun提供了jdk源码是为了技术专家进行研究之用。虽然没有这么夸张，但还是要注意细节。

### 2. 构建编译环境

1. 需要安装最新版本的XCode和Command Line Tools for XCode。这两个SDK包提供了OpenJDK所需要的编译器以及Makefile中用到的外部命令。
2. 需要准备一个6u14以上版本的JDK，因为OpenJDK的各个组成部分有的使用C++编写，更多代码则是使用Java自身实现的，因此编译这些代码需要一个可用的JDK，官方称这个SDK为“Bootstrap JDK”。

## 3. 进行编译

下面直接贴上Shell脚本，读者可参考注释内容。

```shell
#语言选项，这个必须要配置，否则编译后会出现一个Hashtable的NPE错误。
export LANG=C
#Bootstrap JDK的安装路径。
export ALT_BOOTDIR=/Library/Java/JavaVirtualMachines/jdk1.8.0_144.jdk/Contents/Home
#允许自动下载依赖
export ALLOW_DOWNLOAD=true
#并行编译的线程数，设置为和CPU内核数量一致即可，通过 关于本机-系统报告-核总数 可查看
export HOSTPOT_BUILD_JOBS=2
export ALT_PARALLEL_COMPILE_JOBS=2

#比较本次bulid出来的映像和之前版本的差异。这对我们第一次编译来说没有意义
#因此设置为true，否则sanity检查会报缺少先前版本JDK映像的错误提示
export SKIP_COMPARE_IMAGES=true

#使用预编译头文件，不加这个编译会更慢
export USE_PRECOMPILED_HEADER=true

#要编译的内容
export BUILD_LANGTOOLS=true
#export BUILD_JAXP=false
#export BUILD_JAXWS=false
#export BUILD_CORBA=false
export BUILD_HOTSPOT=true
export BUILD_JDK=true

#要编译的版本
#export SKIP_DEBUG_BUILD=false
#export SKIP_FASTDEBUG=true
#export DEBUG_NAME=debug

#将它设置为fasle，可以避开javawa和浏览器Java插件之类的部分的build
BUILD_DEPLOY=false

#将它设置为false就不会build出安装包，因为安装包里面有奇怪的依赖
#但即使不build它也已经能得到完整的jdk映像了
BUILD_INSTALL=false

#编译结果存放的路径
export ALT_OUTPUTDIR=/Users/wangjun/Downloads

#这两个环境变量必须去掉，不然会有很诡异的事情发生
unset JAVA_HOME
unset CLASSPATH

#先执行make sanity，显示Sanity check passed.成功后再执行make 2>&1 | tee $ALT_OUTPUTDIR/build.log
#make sanity
#make 2>&1 | tee $ALT_OUTPUTDIR/build.log
```

## 4. 可能遇到的错误以及解决办法

**1. make sanity的时候报错：**

```shell
   jdk/make/common/shared/Defs-utils.gmk:79: *** "\"ant\" not found; please set ANT_HOME or put \"ant\" on your PATH".  Stop.
```

解决方案：这个错误是由于忘记安装ant了，安装ant即可

**2. make sanity的时候报错：**

```shell
  ERROR: The Compiler version is undefined. 

  ERROR: FreeType version  2.3.0  or higher is required. 
   /bin/mkdir -p /Users/wangjun/Downloads/jdk7u-dev/build/macosx-x86_64/btbins
  rm -f /Users/wangjun/Downloads/jdk7u-dev/build/macosx-x86_64/btbins/freetype_versioncheck
  Failed to build freetypecheck.  

  Exiting because of the above error(s). 

  make: *** [post-sanity] Error 1
```

解决方案：

**The Compiler version is undefined.** 原因是Xcode5.0之后不再提供llvm-gcc与llvm-g++这两样东西，编译jdk是需要这两个所以一直出错。

解决方法是在XCode的安装目录下/Applications/Xcode.app/Contents/Developer/usr/bin 做一个ln -s的链接连到/usr/bin的 llvm-g++ llvm-gcc中

例子：

```shell
sudo ln -s /usr/bin/llvm-g++ /Applications/Xcode.app/Contents/Developer/usr/bin/llvm-g++
sudo ln -s /usr/bin/llvm-gcc /Applications/Xcode.app/Contents/Developer/usr/bin/llvm-gcc
```

**FreeType version  2.3.0  or higher is required.** 的原因是freetype的版本太低或者没有安装freetype。

使用brew重新安装freetype最新版本：

```shell
brew install freetype
```

然后更新环境变量：

```shell
export ALT_FREETYPE_HEADERS_PATH=/usr/local/Cellar/freetype/2.9/include/freetype2
```

重新make sanity就可以了。

**3. 编译时报错**

```shell
BUILD FAILED
/Users/wangjun/Downloads/jdk7u-dev/langtools/make/build.xml:452: The following error occurred while executing this line:
/Users/wangjun/Downloads/jdk7u-dev/langtools/make/build.xml:795: Compile failed; see the compiler error output for details.
```

解决方案：

安装jdk7版本，并更新

```shell
#Bootstrap JDK的安装路径。
export ALT_BOOTDIR=/Library/Java/JavaVirtualMachines/jdk1.7.0_80.jdk/Contents/Home
export ALT_JDK_IMPORT_PATH=/Library/Java/JavaVirtualMachines/jdk1.7.0_80.jdk/Contents/Home
```

**4. 编译时报错**

```shell
jdk7u-dev/hotspot/src/share/vm/adlc/adlparse.cpp:3217:71: error: equality comparison with extraneous parentheses [-Werror,-Wparentheses-equality]
    if ( ((primary = get_ident_or_literal_constant("primary opcode")) == NULL) ) {
          ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~^~~~~~~
/Users/wangjun/Downloads/jdk7u-dev/hotspot/src/share/vm/adlc/adlparse.cpp:3226:77: error: equality comparison with extraneous parentheses [-Werror,-Wparentheses-equality]
      if ( ((secondary = get_ident_or_literal_constant("secondary opcode")) == NULL) ) {
            ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~^~~~~~~
/Users/wangjun/Downloads/jdk7u-dev/hotspot/src/share/vm/adlc/adlparse.cpp:3235:77: error: equality comparison with extraneous parentheses [-Werror,-Wparentheses-equality]
        if ( ((tertiary = get_ident_or_literal_constant("tertiary opcode")) == NULL) ) {
              ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~^~~~~~~
/Users/wangjun/Downloads/jdk7u-dev/hotspot/src/share/vm/adlc/adlparse.cpp:4476:21: error: '&&' within '||' [-Werror,-Wlogical-op-parentheses]
  while ((c >= '0') && (c <= '9')
         ~~~~~~~~~~~^~~~~~~~~~~~~
/Users/wangjun/Downloads/jdk7u-dev/hotspot/src/share/vm/adlc/adlparse.cpp:4515:17: error: equality comparison with extraneous parentheses [-Werror,-Wparentheses-equality]
    if( (second == '=') ) {
         ~~~~~~~^~~~~~
5 errors generated.
```

解决方案：

因为一开始没有在环境变量面去设置，所以要在环境变量里面加上

```shell
#解决严格检查导致的错误
export COMPILER_WARNINGS_FATAL=false
```

**5. 编译时报错**

```shell
clang: error: unknown argument: '-fpch-deps'
```

解决方案:

clang不支持参数-fpch-deps

首先查到哪个文件用到了

```shell
find . -type f ! -name "*.java" | xargs grep -r "\-fpch\-deps"

./hotspot/make/bsd/makefiles/gcc.make:DEPFLAGS = -fpch-deps -MMD -MP -MF $(DEP_DIR)/$(@:%=%.d)
./hotspot/make/linux/makefiles/gcc.make:DEPFLAGS = -fpch-deps -MMD -MP -MF $(DEP_DIR)/$(@:%=%.d)
./hotspot/make/solaris/makefiles/gcc.make:DEPFLAGS = -fpch-deps -MMD -MP -MF $(DEP_DIR)/$(@:%=%.d)
Binary file ./hotspot/.hg/store/data/make/bsd/makefiles/gcc.make.i matches
Binary file ./hotspot/.hg/store/data/make/linux/makefiles/gcc.make.i matches
Binary file ./hotspot/.hg/store/data/make/solaris/makefiles/gcc.make.i matches

#在网上搜到的答案都是，Mac选择bsd，修改./hotspot/make/bsd/makefiles/gcc.make
1）注释217-219行
# Flags for generating make dependency flags.
# ifneq ("${CC_VER_MAJOR}", "2")
# DEPFLAGS = -fpch-deps -MMD -MP -MF $(DEP_DIR)/$(@:%=%.d)
# endif
2）在219行下添加下面代码
DEPFLAGS = -MMD -MP -MF $(DEP_DIR)/$(@:%=%.d)
ifeq ($(USE_CLANG),)
  ifneq ($(CC_VER_MAJOR), 2)
    DEPFLAGS += -fpch-deps
  endif
endif

##发现还是报错，后来干脆都注释了，虽然不知道这个参数的作用是啥
DEPFLAGS = -MMD -MP -MF $(DEP_DIR)/$(@:%=%.d)
#ifeq ($(USE_CLANG),)
#  ifneq ($(CC_VER_MAJOR), 2)
#    DEPFLAGS += -fpch-deps
#  endif
#endif
```

**6. 编译失败，形参默认值问题**

报错

```shell
/Users/wangjun/Downloads/jdk7u-dev/hotspot/src/share/vm/code/relocInfo.hpp:469:18: error: friend
      declaration specifying a default argument must be the only declaration
inline relocInfo prefix_relocInfo(int datalen) {
                 ^
/Users/wangjun/Downloads/jdk7u-dev/hotspot/src/share/vm/code/relocInfo.hpp:374:27: note: previous
      declaration is here
  inline friend relocInfo prefix_relocInfo(int datalen = 0);
                          ^
/Users/wangjun/Downloads/jdk7u-dev/hotspot/src/share/vm/code/relocInfo.hpp:471:59: error: 'RAW_BITS' is a
      protected member of 'relocInfo'
  return relocInfo(relocInfo::data_prefix_tag, relocInfo::RAW_BITS, relocInfo::datalen_tag | datalen);
                                                          ^
/Users/wangjun/Downloads/jdk7u-dev/hotspot/src/share/vm/code/relocInfo.hpp:279:23: note: declared
      protected here
  enum RawBitsToken { RAW_BITS };
                      ^
/Users/wangjun/Downloads/jdk7u-dev/hotspot/src/share/vm/code/relocInfo.hpp:471:10: error: calling a
      protected constructor of class 'relocInfo'
  return relocInfo(relocInfo::data_prefix_tag, relocInfo::RAW_BITS, relocInfo::datalen_tag | datalen);
         ^
/Users/wangjun/Downloads/jdk7u-dev/hotspot/src/share/vm/code/relocInfo.hpp:280:3: note: declared
      protected here
  relocInfo(relocType type, RawBitsToken ignore, int bits)
  ^
```

解决方法

```
# 解决办法
# 修改relocInfo.hpp（路径：hotspot/src/share/vm/code/relocInfo.hpp）
修改374行
inline friend relocInfo prefix_relocInfo(int datalen);
修改469行
inline relocInfo prefix_relocInfo(int datalen = 0) {
   assert(relocInfo::fits_into_immediate(datalen), "datalen in limits");
   return relocInfo(relocInfo::data_prefix_tag, relocInfo::RAW_BITS, relocInfo::datalen_tag | datalen);
}
```

**7. 编译失败**

```shell
/Users/wangjun/Downloads/jdk7u-dev/hotspot/src/share/vm/runtime/virtualspace.cpp:527:14: error: ordered
      comparison between pointer and zero ('char *' and 'int')
  if (base() > 0) {
      ~~~~~~ ^ ~
/Users/wangjun/Downloads/jdk7u-dev/hotspot/src/share/vm/runtime/virtualspace.cpp:545:23: warning: 
      implicit conversion changes signedness: 'int' to 'size_t' (aka 'unsigned long') [-Wsign-conversion]
                  lcm(os::vm_page_size(), prefix_align) : 0) {
                  ~~~ ^~~~~~~~~~~~~~~~~~
/Users/wangjun/Downloads/jdk7u-dev/hotspot/src/share/vm/runtime/virtualspace.cpp:546:14: error: ordered
      comparison between pointer and zero ('char *' and 'int')
  if (base() > 0) {
      ~~~~~~ ^ ~
```

解决方案：

```shell
jdk7u-dev/hotspot/src/share/vm/runtime/virtualspace.cpp文件
527行改成
if (base() != NULL) {
546行改成
if (base() != NULL) {
```

**8.编译失败，找不到对应的类库和方法**

```shell
cd bsd_amd64_compiler2/product && ./test_gamma
Using java runtime at: /Library/Java/JavaVirtualMachines/jdk1.7.0_80.jdk/Contents/Home/jre
Error occurred during initialization of VM
Unable to load native library: dlopen(/Library/Java/JavaVirtualMachines/jdk1.7.0_80.jdk/Contents/Home/jre/lib/libjava.dylib, 1): Symbol not found: __cg_jpeg_resync_to_restart
  Referenced from: /System/Library/Frameworks/ImageIO.framework/Versions/A/Resources/libTIFF.dylib
  Expected in: /Library/Java/JavaVirtualMachines/jdk1.7.0_80.jdk/Contents/Home/jre/lib/libJPEG.dylib
 in /System/Library/Frameworks/ImageIO.framework/Versions/A/Resources/libTIFF.dylib
```

解决方案：

将

/System/Library/Frameworks/ImageIO.framework/Versions/A/Resources

下面的libJPEG.dylib拷贝到

/Library/Java/JavaVirtualMachines/jdk1.7.0\_80.jdk/Contents/Home/jre/lib

下面，覆盖掉之前的libjpeg.dylib。

**9.编译失败，**

```shell
../../../src/solaris/native/sun/awt/utility/rect.h:31:10: fatal error: 'X11/Xlib.h' file not found
#include <X11/Xlib.h>
         ^~~~~~~~~~~~
1 error generated.
make[5]: *** [/Users/wangjun/Downloads/myjdk7/tmp/sun/sun.awt/awt/obj64/Region.o] Error 1
make[5]: *** Waiting for unfinished jobs....
In file included from ../../../src/share/native/sun/awt/image/BufImgSurfaceData.c:31:
In file included from ../../../src/solaris/native/sun/awt/img_util_md.h:26:
In file included from ../../../src/solaris/native/sun/awt/color.h:28:
../../../src/solaris/native/sun/awt/awt.h:38:10: fatal error: 'X11/Intrinsic.h' file not found
#include <X11/Intrinsic.h>
         ^~~~~~~~~~~~~~~~~
1 error generated.
```

解决方案：

