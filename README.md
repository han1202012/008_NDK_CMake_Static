# 008_NDK_CMake_Static

<br>

**CSDN 博客地址 :** [【Android NDK 开发】Android Studio 使用 CMake 导入静态库 ( CMake 简介 | 构建脚本路径配置 | 引入静态库 | 指定静态库路径 | 链接动态库 )](https://hanshuliang.blog.csdn.net/article/details/104337399)

**博客资源下载地址 :** [https://download.csdn.net/download/han1202012/12162132](https://download.csdn.net/download/han1202012/12162132)

**示例代码 GitHub 地址 :** [https://github.com/han1202012/008_NDK_CMake_Static](https://github.com/han1202012/008_NDK_CMake_Static)



<br>
<br>

#### I . CMake 简介

---

<br>

**1 . CMake 简介 :**

<br>

**① 构建工具 :** <font color=blue>CMake 是 Android 中使用 C/C++ 构建原生库的默认工具 ; 

**② 跨平台 :** <font color=purple>CMake 是跨平台的构建工具 , 其可以根据不同类型的平台 , 不同类型的编译器 , 生成对应的 Makefile ; 

**③ 本质 :** <font color=green>CMake 不是直接编译项目的 , 而是生成 make 对应的构建脚本 Makefile 文件 , 还是使用 make 进行构建项目 ; 

**③ Android 中生成的脚本 :** <font color=orange>Android Studio 中 , CMake 生成 ninja 脚本 , ninja 是一种轻量级快速构建工具 ; ( 仅做参考 )

<br>

**2 . CMake 与 Android.mk :** <font color=red>Google 逐渐放弃了对 Android.mk 的支持 , 目前新项目推荐使用 CMake 构建本地库 , <font color=blue>旧的项目建议将 Android.mk 转为 CMake 构建 , 以获取更好的代码维护 ; 




<br>
<br>

#### II . Android Studio 中 CMake 引入静态库流程

---

<br>

**Android Studio 中 CMake 引入静态库流程 :** 

<br>

**1 . build.gradle 配置 CMake 编译选项 :** <font color=purple>在 Module 级别的 build.gradle 脚本中配置 CMake 编译选项 ; 

```java
        // I . NDK 配置 1 : 配置 AS 工程中的 C/C++ 源文件的编译


        //     defaultConfig 内部的 externalNativeBuild 配置的是配置 AS 工程的 C/C++ 源文件编译参数
        //     defaultConfig 外部的 externalNativeBuild 配置的是 CMakeList.txt 或 Android1.mk 构建脚本的路径
        externalNativeBuild {
            cmake {
                cppFlags ""

                //配置编译 C/C++ 源文件为哪几个 CPU 指令集的函数库 (arm , x86 等)
                abiFilters "armeabi-v7a"
            }
            /*ndkBuild{
                abiFilters "armeabi-v7a" *//*, "arm64-v8a", "x86", "x86_64"*//*
            }*/
        }
```

<br>

**2 . build.gradle 配置 NDK 打包选项 :** <font color=blue>在 Module 级别的 build.gradle 脚本中配置 NDK 打包选项 ; 

```java
        // II . NDK 配置 2 : 配置 AS 工程中的 C/C++ 源文件的编译


        //配置 APK 打包 哪些动态库
        //  示例 : 如在工程中集成了第三方库 , 其提供了 arm, x86, mips 等指令集的动态库
        //        那么为了控制打包后的应用大小, 可以选择性打包一些库 , 此处就是进行该配置
        ndk{
            // 打包生成的 APK 文件指挥包含 ARM 指令集的动态库
            abiFilters "armeabi-v7a" /*, "arm64-v8a", "x86", "x86_64"*/
        }
```

<br>

**3 . build.gradle 配置 CMake 构建脚本 CMakeList.txt 路径 :** <font color=red>在 Module 级别的 build.gradle 脚本中配置 Android.mk 构建脚本的路径 ;

```java
    // III . NDK 配置  : 配置 AS 工程中的 C/C++ 源文件的编译构建脚本


    // 配置 NDK 的编译脚本路径
    // 编译脚本有两种 ① CMakeList.txt ② Android1.mk
    //     defaultConfig 内部的 externalNativeBuild 配置的是配置 AS 工程的 C/C++ 源文件编译参数
    //     defaultConfig 外部的 externalNativeBuild 配置的是 CMakeList.txt 或 Android1.mk 构建脚本的路径
    externalNativeBuild {

        // 配置 CMake 构建脚本 CMakeLists.txt 脚本路径
        cmake {
            path "src/main/cpp/CMakeLists.txt"
            version "3.10.2"
        }

        // 配置 Android1.mk 构建脚本路径
        /*ndkBuild{
            //path "src/main/ndkBuild_Shared/Android.mk"
            path "src/main/ndkBuild_Static/Android.mk"
        }*/
    }
```

<br>

**4 . CMake 构建脚本 CMakeList.txt 引入静态库 :** 

```shell
# 引入静态库
#       ① 参数 1 ( add ) : 设置引入的静态库名称
#       ② 参数 2 ( SHARED ) : 设置引入的函数库类型 : ① 静态库 STATIC ② 动态库 SHARED
#       ③ 参数 3 ( IMPORTED ) : 表示引入第三方静态库 , 导入静态库 , 相当于预编译静态库
#                                   后续还需要设置导入路径 , 配合该配置使用
add_library(
        # 设置引入的静态库名称
        add

        # 设置引入的函数库类型为静态库
        STATIC

        # 表示引入第三方静态库
        IMPORTED)
```

<br>

**5 . CMake 构建脚本 CMakeList.txt 设置静态库路径 :** 

```shell
# 设置上述静态库的导入路径
#       设置目标属性参数 :
#           ① 参数 1 ( add ) : 要设置哪个函数库的属性
#           ② 参数 2 ( PROPERTIES ) : 设置目标属性
#           ③ 参数 3 ( IMPORTED_LOCATION ) : 设置导入路径
#           ④ 参数 4 : 配置静态库的文件路径
set_target_properties(
        # 设置目标
        add

        # 设置属性
        PROPERTIES

        # 导入路径
        IMPORTED_LOCATION

        # ${CMAKE_SOURCE_DIR} 是本 CMakeList.txt 构建脚本的路径 , 是 CMake 工具内置的变量
        #       Android CMake 也内置了一些变量 , 如 ANDROID_ABI
        ${CMAKE_SOURCE_DIR}/../jniLibs/armeabi-v7a/libadd.a)
```

<br>

**6 . CMake 构建脚本 CMakeList.txt 链接静态库 :** 

```shell
# 链接函数库
#       参数 1 : 本构建脚本要生成的动态库目 标
#       参数 2 ~ ... : 后面是之前预编译的动态库或静态库 , 或引入的动态库
target_link_libraries(
        native-lib

        # 表示 编译 native-lib 模块, 要链接 add 模块
        add

        ${log-lib})
```









<br>
<br>

#### III . 指定 CMake 最小版本号 

---

<br>

**指定 CMake 最低版本 :** 在 CMake 构建脚本 CMakeList.txt 文件中 , 第一行一定要先指定 CMake 最小版本号 ; 

```shell
cmake_minimum_required(VERSION 3.4.1)
```


<br>
<br>

#### IV . 导入函数库 ( 静态库 / 动态库 ) 编译配置

---

<br>


**函数库 ( 静态库 / 动态库 ) 编译配置 :** <font color=red>函数库编译需要传入 3 个参数 ; 

<br>

**① 参数 1 :** <font color=blue>设置生成的动态库名称 ; 

**② 参数 2 :** <font color=purple>设置生成的函数库类型 : a . 静态库 STATIC , b . 动态库 SHARED ; 

**③ 参数 3 :** <font color=blue>配置要编译的源文件 ;


```shell
# 引入静态库
#       ① 参数 1 ( add ) : 设置引入的静态库名称
#       ② 参数 2 ( SHARED ) : 设置引入的函数库类型 : ① 静态库 STATIC ② 动态库 SHARED
#       ③ 参数 3 ( IMPORTED ) : 表示引入第三方静态库 , 导入静态库 , 相当于预编译静态库
#                                   后续还需要设置导入路径 , 配合该配置使用
add_library(
        # 设置引入的静态库名称
        add

        # 设置引入的函数库类型为静态库
        STATIC

        # 表示引入第三方静态库
        IMPORTED)
```

<br>

**2 . 特别注意 :** <font color=red>使用这种方法引入动态库 , 在 6.0 以上的系统是无法使用的 , <font color=blue>推荐使用 set() 设置 -L 参数的方式引入动态库 ; 

```shell
# 设置变量
# CMAKE_CXX_FLAGS 表示会将 C++ 的参数传给编译器
# CMAKE_C_FLAGS 表示会将 C 参数传给编译器

# 参数设置 : 传递 CMAKE_CXX_FLAGS C+= 参数给编译器时 , 在 该参数后面指定库的路径
#   CMAKE_SOURCE_DIR 指的是当前的文件地址
#   -L 参数指定动态库的查找路径
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -L${CMAKE_SOURCE_DIR}/../jniLibs/armeabi-v7a")
```






<br>
<br>

#### V . 导入第三方函数库路径配置

---

<br>


**导入第三方函数库路径配置 :** <font color=red>通过调用 set_target_properties () 设置第三方库路径 ; 

<br>

**① 参数 1 ( add ) :** <font color=blue>要设置哪个函数库的属性 ; 

**② 参数 2 ( PROPERTIES ) :** <font color=green>设置目标属性 ;

**③ 参数 3 ( IMPORTED_LOCATION ) :** <font color=purple>设置导入路径 ;

**④ 参数 4 :** 配置静态库的文件路径 ;

```shell
# 设置上述静态库的导入路径
#       设置目标属性参数 :
#           ① 参数 1 ( add ) : 要设置哪个函数库的属性
#           ② 参数 2 ( PROPERTIES ) : 设置目标属性
#           ③ 参数 3 ( IMPORTED_LOCATION ) : 设置导入路径
#           ④ 参数 4 : 配置静态库的文件路径
set_target_properties(
        # 设置目标
        add

        # 设置属性
        PROPERTIES

        # 导入路径
        IMPORTED_LOCATION

        # ${CMAKE_SOURCE_DIR} 是本 CMakeList.txt 构建脚本的路径 , 是 CMake 工具内置的变量
        #       Android CMake 也内置了一些变量 , 如 ANDROID_ABI
        ${CMAKE_SOURCE_DIR}/../jniLibs/armeabi-v7a/libadd.a)
```


<br>
<br>

#### VI . 输出日志信息

---

<br>

**调用 message() 方法可以输出日志信息 :** 

```shell
# 打印日志信息
#       ${ANDROID_ABI} 的作用是获取当前的 CPU 指令集架构
#           当本次编译 armeabi-v7a CPU 架构时 , ${ANDROID_ABI} 值为 armeabi-v7a
#           当本次编译 x86 CPU 架构时 , ${ANDROID_ABI} 值为 x86
message("CMAKE_SOURCE_DIR : ${CMAKE_SOURCE_DIR}, ANDROID_ABI : ${ANDROID_ABI}")
```


<br>
<br>

#### VII . 链接函数库

---

<br>

**链接函数库 :** <font color=blue>这里注意第一个参数必须是要生成的动态库模块 ; 

```shell
# 链接函数库
#       参数 1 : 本构建脚本要生成的动态库目标
#       参数 2 ~ ... : 后面是之前预编译的动态库或静态库 , 或引入的动态库
target_link_libraries(
        native-lib

        # 表示 编译 native-lib 模块, 要链接 add 模块
        add

        ${log-lib})
```


<br>
<br>

#### VIII . Module 级别的 build.gradle 完整配置代码 

---

<br>

```java
apply plugin: 'com.android.application'

android {
    compileSdkVersion 29
    buildToolsVersion "29.0.0"
    defaultConfig {
        applicationId "kim.hsl.cmake"
        minSdkVersion 15
        targetSdkVersion 29
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"



        // I . NDK 配置 1 : 配置 AS 工程中的 C/C++ 源文件的编译


        //     defaultConfig 内部的 externalNativeBuild 配置的是配置 AS 工程的 C/C++ 源文件编译参数
        //     defaultConfig 外部的 externalNativeBuild 配置的是 CMakeList.txt 或 Android1.mk 构建脚本的路径
        externalNativeBuild {
            cmake {
                cppFlags ""

                //配置编译 C/C++ 源文件为哪几个 CPU 指令集的函数库 (arm , x86 等)
                abiFilters "armeabi-v7a"
            }
            /*ndkBuild{
                abiFilters "armeabi-v7a" *//*, "arm64-v8a", "x86", "x86_64"*//*
            }*/
        }



        // II . NDK 配置 2 : 配置 AS 工程中的 C/C++ 源文件的编译


        //配置 APK 打包 哪些动态库
        //  示例 : 如在工程中集成了第三方库 , 其提供了 arm, x86, mips 等指令集的动态库
        //        那么为了控制打包后的应用大小, 可以选择性打包一些库 , 此处就是进行该配置
        ndk{
            // 打包生成的 APK 文件指挥包含 ARM 指令集的动态库
            abiFilters "armeabi-v7a" /*, "arm64-v8a", "x86", "x86_64"*/
        }

    }



    // III . NDK 配置  : 配置 AS 工程中的 C/C++ 源文件的编译构建脚本


    // 配置 NDK 的编译脚本路径
    // 编译脚本有两种 ① CMakeList.txt ② Android1.mk
    //     defaultConfig 内部的 externalNativeBuild 配置的是配置 AS 工程的 C/C++ 源文件编译参数
    //     defaultConfig 外部的 externalNativeBuild 配置的是 CMakeList.txt 或 Android1.mk 构建脚本的路径
    externalNativeBuild {

        // 配置 CMake 构建脚本 CMakeLists.txt 脚本路径
        cmake {
            path "src/main/cpp/CMakeLists.txt"
            version "3.10.2"
        }

        // 配置 Android1.mk 构建脚本路径
        /*ndkBuild{
            //path "src/main/ndkBuild_Shared/Android.mk"
            path "src/main/ndkBuild_Static/Android.mk"
        }*/
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'androidx.appcompat:appcompat:1.1.0'
    implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'androidx.test:runner:1.2.0'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.2.0'
}

```


<br>
<br>

#### IX . CMakeList.txt 完整配置代码 

---

<br>

```shell

# 指定 CMake 最低版本
cmake_minimum_required(VERSION 3.4.1)

# 设置函数库编译
add_library( # 参数 1 : 设置生成的动态库名称
        native-lib

        # 参数 2 : 设置生成的函数库类型 : ① 静态库 STATIC ② 动态库 SHARED
        SHARED

        # 参数 3 : 配置要编译的源文件
        native-lib.cpp)


# 引入静态库
#       ① 参数 1 ( add ) : 设置引入的静态库名称
#       ② 参数 2 ( SHARED ) : 设置引入的函数库类型 : ① 静态库 STATIC ② 动态库 SHARED
#       ③ 参数 3 ( IMPORTED ) : 表示引入第三方静态库 , 导入静态库 , 相当于预编译静态库
#                                   后续还需要设置导入路径 , 配合该配置使用
add_library(
        # 设置引入的静态库名称
        add

        # 设置引入的函数库类型为静态库
        STATIC

        # 表示引入第三方静态库
        IMPORTED)

# 设置上述静态库的导入路径
#       设置目标属性参数 :
#           ① 参数 1 ( add ) : 要设置哪个函数库的属性
#           ② 参数 2 ( PROPERTIES ) : 设置目标属性
#           ③ 参数 3 ( IMPORTED_LOCATION ) : 设置导入路径
#           ④ 参数 4 : 配置静态库的文件路径
set_target_properties(
        # 设置目标
        add

        # 设置属性
        PROPERTIES

        # 导入路径
        IMPORTED_LOCATION

        # ${CMAKE_SOURCE_DIR} 是本 CMakeList.txt 构建脚本的路径 , 是 CMake 工具内置的变量
        #       Android CMake 也内置了一些变量 , 如 ANDROID_ABI
        ${CMAKE_SOURCE_DIR}/../jniLibs/armeabi-v7a/libadd.a)

# 打印日志信息
#       ${ANDROID_ABI} 的作用是获取当前的 CPU 指令集架构
#           当本次编译 armeabi-v7a CPU 架构时 , ${ANDROID_ABI} 值为 armeabi-v7a
#           当本次编译 x86 CPU 架构时 , ${ANDROID_ABI} 值为 x86
message("CMAKE_SOURCE_DIR : ${CMAKE_SOURCE_DIR}, ANDROID_ABI : ${ANDROID_ABI}")


# 到预设的目录查找 log 库 , 将找到的路径赋值给 log-lib
#   这个路径是 NDK 的 ndk-bundle\platforms\android-29\arch-arm\usr\lib\liblog.so
#   不同的 Android 版本号 和 CPU 架构 需要到对应的目录中查找 , 此处是 29 版本 32 位 ARM 架构的日志库
find_library(
        log-lib

        log)


# 设置变量
# CMAKE_CXX_FLAGS 表示会将 C++ 的参数传给编译器
# CMAKE_C_FLAGS 表示会将 C 参数传给编译器

# 参数设置 : 传递 CMAKE_CXX_FLAGS C+= 参数给编译器时 , 在 该参数后面指定库的路径
#   CMAKE_SOURCE_DIR 指的是当前的文件地址
#   -L 参数指定动态库的查找路径
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -L${CMAKE_SOURCE_DIR}/../jniLibs/armeabi-v7a")

# 链接函数库
#       参数 1 : 本构建脚本要生成的动态库目 标
#       参数 2 ~ ... : 后面是之前预编译的动态库或静态库 , 或引入的动态库
target_link_libraries(
        native-lib

        # 表示 编译 native-lib 模块, 要链接 add 模块
        add

        ${log-lib})
```



<br>
<br>

#### X . 博客资源

---

<br>

**CSDN 博客地址 :** [【Android NDK 开发】Android Studio 使用 CMake 导入静态库 ( CMake 简介 | 构建脚本路径配置 | 引入静态库 | 指定静态库路径 | 链接动态库 )](https://hanshuliang.blog.csdn.net/article/details/104337399)

**博客资源下载地址 :** [https://download.csdn.net/download/han1202012/12162132](https://download.csdn.net/download/han1202012/12162132)

**示例代码 GitHub 地址 :** [https://github.com/han1202012/008_NDK_CMake_Static](https://github.com/han1202012/008_NDK_CMake_Static)

