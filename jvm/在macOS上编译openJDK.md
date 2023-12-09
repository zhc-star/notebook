### 1. 绕不开的五个步骤

1. 获取完整的源代码
2. 执行configure
3. 执行make
4. 验证新构建的jdk
5. 执行基础的测试



### 2. 执行configure时遇到的问题

- 第一次尝试

```sh
bash configure --with-debug-level=fastdebug --with-jvm-variants=server --with-freetype=/opt/homebrew/Cellar/freetype/2.13.2 --with-sysroot="/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk" --with-freetype=bundled

configure: The tested number of bits in the target (64) differs from the number of bits expected to be found in the target (32)
configure: error: Cannot continue.
/Users/zhanghaocheng/jdk12/jdk12-06222165c35f/build/.configure-support/generated-configure.sh: line 84: 5: Bad file descriptor
configure exiting with result code 1
```

解决方案:

```sh
# 由提示可知, generated-configure.sh在执行的时候, 判断改设备为32位, 但是在验证时, 又得出改设备为64位, # # 导致该脚本无法继续执行.打开generated-configure.sh文件可以看到有两处进行了判断, 如下:
# First argument is the cpu name from the trip/quad
17385   case "$build_cpu" in
17386     x86_64*x32)
17387       VAR_CPU=x32
17388       VAR_CPU_ARCH=x86
17389       VAR_CPU_BITS=32
17390       VAR_CPU_ENDIAN=little
17391       ;;
17392     x86_64)
17393       VAR_CPU=x86_64
17394       VAR_CPU_ARCH=x86
17395       VAR_CPU_BITS=64
17396       VAR_CPU_ENDIAN=little
17397       ;;
17398     i?86)
17399       VAR_CPU=x86
17400       VAR_CPU_ARCH=x86
17401       VAR_CPU_BITS=32
17402       VAR_CPU_ENDIAN=little
17403       ;;
17404			alpha*)
17405       VAR_CPU=alpha
17406       VAR_CPU_ARCH=alpha
17407       VAR_CPU_BITS=64
17408       VAR_CPU_ENDIAN=little
17409       ;;
17410     arm*)
17411       VAR_CPU=arm
17412       VAR_CPU_ARCH=arm
17413       VAR_CPU_BITS=32
17414       VAR_CPU_ENDIAN=little
17415       ;;
17416     aarch64)
17417       VAR_CPU=aarch64
17418       VAR_CPU_ARCH=aarch64
17419       VAR_CPU_BITS=64
17420       VAR_CPU_ENDIAN=little
17421       ;;
17422     ia64)
17423       VAR_CPU=ia64
17424       VAR_CPU_ARCH=ia64
17425       VAR_CPU_BITS=64
17426       VAR_CPU_ENDIAN=little
17427       ;;
17428     m68k)
17429       VAR_CPU=m68k
17430       VAR_CPU_ARCH=m68k
17431       VAR_CPU_BITS=32
17432       VAR_CPU_ENDIAN=big
17433       ;;
17434     mips)
17435       VAR_CPU=mips
17436       VAR_CPU_ARCH=mips
17437       VAR_CPU_BITS=32
17438       VAR_CPU_ENDIAN=big
17439       ;;
17440     mipsel)
17441       VAR_CPU=mipsel
17442       VAR_CPU_ARCH=mipsel
17443       VAR_CPU_BITS=32
17444       VAR_CPU_ENDIAN=little
17445       ;;
17446     mips64)
17447       VAR_CPU=mips64
17448       VAR_CPU_ARCH=mips64
17449       VAR_CPU_BITS=64
17450       VAR_CPU_ENDIAN=big
17451       ;;
17452     mips64el)
17453       VAR_CPU=mips64el
17454       VAR_CPU_ARCH=mips64el
17455       VAR_CPU_BITS=64
17456       VAR_CPU_ENDIAN=little
17457       ;;
17458     powerpc)
17459       VAR_CPU=ppc
17460       VAR_CPU_ARCH=ppc
17461       VAR_CPU_BITS=32
17462       VAR_CPU_ENDIAN=big
17463       ;;
17464     powerpc64)
17465       VAR_CPU=ppc64
17466       VAR_CPU_ARCH=ppc
17467       VAR_CPU_BITS=64
17468       VAR_CPU_ENDIAN=big
17469       ;;
17470     powerpc64le)
17471       VAR_CPU=ppc64le
17472       VAR_CPU_ARCH=ppc
17473       VAR_CPU_BITS=64
17474       VAR_CPU_ENDIAN=little
17475       ;;
17476     s390)
17477       VAR_CPU=s390
17478       VAR_CPU_ARCH=s390
17479       VAR_CPU_BITS=32
17480       VAR_CPU_ENDIAN=big
17481       ;;
17482     s390x)
17483       VAR_CPU=s390x
17484       VAR_CPU_ARCH=s390
17485       VAR_CPU_BITS=64
17486       VAR_CPU_ENDIAN=big
17487       ;;
17488     sh*eb)
17489       VAR_CPU=sh
17490       VAR_CPU_ARCH=sh
17491       VAR_CPU_BITS=32
17492       VAR_CPU_ENDIAN=big
17493       ;;
17494     sh*)
17495       VAR_CPU=sh
17496       VAR_CPU_ARCH=sh
17497       VAR_CPU_BITS=32
17498       VAR_CPU_ENDIAN=little
17499       ;;
17500     sparc)
17501       VAR_CPU=sparc
17502       VAR_CPU_ARCH=sparc
17503       VAR_CPU_BITS=32
17504       VAR_CPU_ENDIAN=big
17505       ;;
17506     sparcv9|sparc64)
17507       VAR_CPU=sparcv9
17508       VAR_CPU_ARCH=sparc
17509       VAR_CPU_BITS=64
17510       VAR_CPU_ENDIAN=big
17511       ;;
17512     *)
17513       as_fn_error $? "unsupported cpu $build_cpu" "$LINENO" 5
17514       ;;

# 另一处与这里类似, 可以看到17413行, 如果时arm开头的cpu架构, 则认为是32位. 把这里改成64即可.
# 找到另一处也进行修改, 并保存退出.



```

- 第二次尝试

```sh
bash configure --with-debug-level=slowdebug --with-jvm-variants=server --with-freetype=/opt/homebrew/Cellar/freetype/2.13.2

====================================================
A new configuration has been successfully created in
/Users/zhanghaocheng/jdk12/jdk12-06222165c35f/build/macosx-arm-server-slowdebug
using configure arguments '--with-debug-level=slowdebug --with-jvm-variants=server --with-freetype=/opt/homebrew/Cellar/freetype/2.13.2 --with-sysroot=/Library/Developer/CommandLineTools/SDKs/MacOSX12.3.sdk --with-freetype=bundled'.

Configuration summary:
* Debug level:    slowdebug
* HS debug level: debug
* JVM variants:   server
* JVM features:   server: 'cds cmsgc compiler1 compiler2 dtrace epsilongc g1gc jfr jni-check jvmti management nmt parallelgc serialgc services vm-structs' 
* OpenJDK target: OS: macosx, CPU architecture: arm, address length: 64
* Version string: 12-internal+0-adhoc.zhanghaocheng.jdk12-06222165c35f (12-internal)

Tools summary:
* Boot JDK:       openjdk version "11.0.20.1" 2023-08-24 OpenJDK Runtime Environment Homebrew (build 11.0.20.1+0) OpenJDK 64-Bit Server VM Homebrew (build 11.0.20.1+0, mixed mode)  (at /opt/homebrew/Cellar/openjdk@11/11.0.20.1/libexec/openjdk.jdk/Contents/Home)
* Toolchain:      clang (clang/LLVM)
* C Compiler:     Version 14.0.3 (at /usr/bin/clang)
* C++ Compiler:   Version 14.0.3 (at /usr/bin/clang++)

Build performance summary:
* Cores to use:   10
* Memory limit:   16384 MB
```

成功!



### 3. 执行make images

- 第一次执行

```sh
=== Output from failing command(s) repeated here ===
* For target hotspot_variant-server_tools_adlc_objs_adlparse.o:
/Users/zhanghaocheng/jdk12/jdk12-06222165c35f/src/hotspot/share/adlc/adlparse.cpp:214:11: error: 'sprintf' is deprecated: This function is provided for compatibility reasons only.  Due to security concerns inherent in the design of sprintf(3), it is highly recommended that you use snprintf(3) instead. [-Werror,-Wdeprecated-declarations]
          sprintf(buf, "%s_%d", instr->_ident, match_rules_cnt++);
          ^
/Users/zhanghaocheng/Desktop/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX13.3.sdk/usr/include/stdio.h:188:1: note: 'sprintf' has been explicitly marked deprecated here
__deprecated_msg("This function is provided for compatibility reasons only.  Due to security concerns inherent in the design of sprintf(3), it is highly recommended that you use snprintf(3) instead.")
^
/Users/zhanghaocheng/Desktop/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX13.3.sdk/usr/include/sys/cdefs.h:215:48: note: expanded from macro '__deprecated_msg'
        #define __deprecated_msg(_msg) __attribute__((__deprecated__(_msg)))
                                                      ^
/Users/zhanghaocheng/jdk12/jdk12-06222165c35f/src/hotspot/share/adlc/adlparse.cpp:2862:3: error: 'sprintf' is deprecated: This function is provided for compatibility reasons only.  Due to security concerns inherent in the design of sprintf(3), it is highly recommended that you use snprintf(3) instead. [-Werror,-Wdeprecated-declarations]
  sprintf(ec_name, "%s%s", prefix, inst._ident);
  ^
   ... (rest of output omitted)
* For target hotspot_variant-server_tools_adlc_objs_archDesc.o:
/Users/zhanghaocheng/jdk12/jdk12-06222165c35f/src/hotspot/share/adlc/archDesc.cpp:813:5: error: 'sprintf' is deprecated: This function is provided for compatibility reasons only.  Due to security concerns inherent in the design of sprintf(3), it is highly recommended that you use snprintf(3) instead. [-Werror,-Wdeprecated-declarations]
    sprintf(regMask,"%s%s()", rc_name, mask);
    ^
/Users/zhanghaocheng/Desktop/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX13.3.sdk/usr/include/stdio.h:188:1: note: 'sprintf' has been explicitly marked deprecated here
__deprecated_msg("This function is provided for compatibility reasons only.  Due to security concerns inherent in the design of sprintf(3), it is highly recommended that you use snprintf(3) instead.")
^
/Users/zhanghaocheng/Desktop/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX13.3.sdk/usr/include/sys/cdefs.h:215:48: note: expanded from macro '__deprecated_msg'
        #define __deprecated_msg(_msg) __attribute__((__deprecated__(_msg)))
                                                      ^
/Users/zhanghaocheng/jdk12/jdk12-06222165c35f/src/hotspot/share/adlc/archDesc.cpp:906:3: error: 'sprintf' is deprecated: This function is provided for compatibility reasons only.  Due to security concerns inherent in the design of sprintf(3), it is highly recommended that you use snprintf(3) instead. [-Werror,-Wdeprecated-declarations]
  sprintf(result,"%s%s", stack_or, reg_mask_name);
  ^
   ... (rest of output omitted)
* For target hotspot_variant-server_tools_adlc_objs_dfa.o:
/Users/zhanghaocheng/jdk12/jdk12-06222165c35f/src/hotspot/share/adlc/dfa.cpp:218:5: error: 'sprintf' is deprecated: This function is provided for compatibility reasons only.  Due to security concerns inherent in the design of sprintf(3), it is highly recommended that you use snprintf(3) instead. [-Werror,-Wdeprecated-declarations]
    sprintf(Expr::buffer(), "_kids[0]->_cost[%s]", lchild_to_upper);
    ^
/Users/zhanghaocheng/Desktop/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX13.3.sdk/usr/include/stdio.h:188:1: note: 'sprintf' has been explicitly marked deprecated here
__deprecated_msg("This function is provided for compatibility reasons only.  Due to security concerns inherent in the design of sprintf(3), it is highly recommended that you use snprintf(3) instead.")
^
/Users/zhanghaocheng/Desktop/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX13.3.sdk/usr/include/sys/cdefs.h:215:48: note: expanded from macro '__deprecated_msg'
        #define __deprecated_msg(_msg) __attribute__((__deprecated__(_msg)))
                                                      ^
/Users/zhanghaocheng/jdk12/jdk12-06222165c35f/src/hotspot/share/adlc/dfa.cpp:224:5: error: 'sprintf' is deprecated: This function is provided for compatibility reasons only.  Due to security concerns inherent in the design of sprintf(3), it is highly recommended that you use snprintf(3) instead. [-Werror,-Wdeprecated-declarations]
    sprintf(Expr::buffer(), "_kids[1]->_cost[%s]", rchild_to_upper);
    ^
   ... (rest of output omitted)

* All command lines available in /Users/zhanghaocheng/jdk12/jdk12-06222165c35f/build/macosx-arm-server-slowdebug/make-support/failure-logs.
=== End of repeated output ===

No indication of failed target found.
Hint: Try searching the build log for '] Error'.
Hint: See doc/building.html#troubleshooting for assistance.

make[1]: *** [main] Error 2
make: *** [images] Error 2

```

失败, 通过添加参数 --disable-warnings-as-errors 可以解决该问题.

