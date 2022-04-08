# CMake 学习

CMake（Cross platform Make）是一个开源、跨平台的自动化构建工具，旨在构建、测试和打包软件。CMake 通过使用简单的平台和编译器独立的配置文件来控制软件编译过程，并根据不同的编译环境生成标准构建文件（如 Unix 的 Makefile 或 Windows Visual C++ 的 projects/workspaces）。

目录：

- [CMake 学习](#cmake-学习)
  - [基本语法](#基本语法)
  - [CMake 命令](#cmake-命令)
    - [cmake_minimum_required：指定 CMake 最低支持版本](#cmake_minimum_required指定-cmake-最低支持版本)
    - [project：设置工程名](#project设置工程名)
    - [set：定义变量](#set定义变量)
    - [message：打印信息](#message打印信息)
    - [add_compile_options：添加编译参数](#add_compile_options添加编译参数)
    - [add_subdirectory：添加存放源文件的子目录](#add_subdirectory添加存放源文件的子目录)
    - [add_library：生成库文件](#add_library生成库文件)
    - [add_executable：生成可执行文件](#add_executable生成可执行文件)
    - [include_directories：添加头文件搜索路径](#include_directories添加头文件搜索路径)
    - [target_include_directories：为 target 添加头文件搜索路径](#target_include_directories为-target-添加头文件搜索路径)
    - [link_directories：添加库文件搜索路径](#link_directories添加库文件搜索路径)
    - [target_link_libraries：为 target 添加需要链接的共享库](#target_link_libraries为-target-添加需要链接的共享库)
    - [aux_source_directory：发现一个目录下所有的源文件](#aux_source_directory发现一个目录下所有的源文件)
  - [常用变量](#常用变量)
    - [CMAKE_\<LANG\>_STANDARD：用于指定 \<LANG\> 标准](#cmake_lang_standard用于指定-lang-标准)
    - [CMAKE_\<LANG\>_FLAGS：设置编译选项](#cmake_lang_flags设置编译选项)
    - [CMAKE_BUILD_TYPE：设置编译类型](#cmake_build_type设置编译类型)
    - [XXX_SOURCE_DIR：顶层源码的路径](#xxx_source_dir顶层源码的路径)
    - [XXX_BINARY_DIR：顶层构建树的路径](#xxx_binary_dir顶层构建树的路径)
    - [CMAKE_\<LANG\>_COMPILER：指定 \<LANG\> 编译器完整路径](#cmake_lang_compiler指定-lang-编译器完整路径)
    - [EXECUTABLE_OUTPUT_PATH：可执行文件输出的存放路径](#executable_output_path可执行文件输出的存放路径)
    - [LIBRARY_OUTPUT_PATH：库文件输出的存放路径](#library_output_path库文件输出的存放路径)
  - [编译工程](#编译工程)
    - [编译流程](#编译流程)
    - [构建方式](#构建方式)
  - [工程实践](#工程实践)
    - [Hello World](#hello-world)
    - [包含头文件](#包含头文件)
    - [包含静/动态库](#包含静动态库)
    - [包含第三方库](#包含第三方库)
  - [参考链接](#参考链接)

## 基本语法

**基本语法格式：**

```txt
command(arg1 arg2 ${VAL} ...)
```

* 命令 `command` 是大小写无关的，而参数 `arg` 和变量 `VAL` 是大小写相关的。
* 命令后接 `()` 括起，参数、变量之间使用 空格 或 `;` 分隔。
* 变量使用 `${}` 方式取值，但在 `IF` 等控制语句中直接使用变量名。

## CMake 命令

### cmake_minimum_required：指定 CMake 最低支持版本

```txt
cmake_minimum_required(VERSION <min>[...<policy_max>] [FATAL_ERROR])

# 指定 CMake 最低支持版本 <min> 为 2.8.3
cmake_minimum_required(VERSION 2.8.3)
```

### project：设置工程名

```txt
project(<PROJECT-NAME> [<language-name>...])

# 设置工程名为 HELLOWORLD
project(HELLOWORLD)
```

该命令会同时设置以下变量：

* `PROJECT_NAME`
  * 保存工程名。如果定义于最高层的 `CMakeLists.txt` 则还会保存工程名到 `CMAKE_PROJECT_NAME` 变量。
* `PROJECT_SOURCE_DIR`, `<PROJECT-NAME>_SOURCE_DIR`
  * 项目源代码目录（source directory）的绝对路径，即顶层 `CMakeLists.txt` 所在的目录。
* `PROJECT_BINARY_DIR`, `<PROJECT-NAME>_BINARY_DIR`
  * 项目二进制目录（binary directory）的绝对路径，即 CMake 实际构建并生成相关文件的目录。

示例：

```
[root@VM-8-14-centos project]# pwd
/root/project
[root@VM-8-14-centos project]# tree
.
|-- build
|-- CMakeLists.txt
`-- src
    `-- main.cpp

2 directories, 2 files
```

当采用当采用外部构建，即在 `build` 目录中执行 `cmake ..`：

* `XXX_SOURCE_DIR` 值为 `/root/project`。
* `XXX_BINARY_DIR` 值为 `/root/project/build`。

### set：定义变量

```txt
set(VAR [VALUE] [CACHE TYPE DOCSTRING [FORCE]])

# 定义 SRC 变量，其值为 main.cpp
set(SRC main.cpp)
```

### message：打印信息

```txt
General messages
  message([<mode>] "message text" ...)
```

`<mode>` 参数决定信息类型与处理方式：

* `FATAL_ERROR`：CMake 出错，停止处理和生成。
* `WARNING`：CMake 警告，继续处理。
* `STATUS`：用户所感兴趣的一些信息。
* `DEBUG`：针对项目本身的开发人员而不是只想构建项目的用户的详细信息消息。

### add_compile_options：添加编译参数

```txt
add_compile_options(<option> ...)

# 添加编译参数 -Wall -std=c++11
add_compile_options(-Wall -std=c++11 -O2)
```

### add_subdirectory：添加存放源文件的子目录

```txt
add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL])

# 添加 src 子目录，src 中需有一个 CMakeLists.txt
add_subdirectory(src)
```

### add_library：生成库文件

```txt
add_library(<name> [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            [<source>...])

# 通过变量 SRC 生成变量名为 hello 的共享库（libhello.so）
add_library(hello SHARED ${SRC})

# 通过变量 SRC 生成变量名 world 的静态库（libworld.a）
add_library(world STATIC ${SRC})
```

### add_executable：生成可执行文件

```txt
add_executable(<name> [WIN32] [MACOSX_BUNDLE]
               [EXCLUDE_FROM_ALL]
               [source1] [source2 ...])

# 编译 main.cpp 生成可执行文件 main
add_executable(main main.cpp)
```

### include_directories：添加头文件搜索路径

```txt
include_directories([AFTER|BEFORE] [SYSTEM] dir1 dir2 …)

# 将 /usr/include/my_include 和 ./include 添加到头文件搜索路径
include_directories(/usr/include/my_include ./include)
```

等价于编译器指定 `-I` 参数。

### target_include_directories：为 target 添加头文件搜索路径

```txt
target_include_directories(<target> [SYSTEM] [AFTER|BEFORE]
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])

target_include_directories(mylib PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/mylib>
  $<INSTALL_INTERFACE:include/mylib>  # <prefix>/include/mylib
)
```

* `<target>` 必须由 `add_executable()` 或 `add_library()` 等命令创建，且不能为别名。

### link_directories：添加库文件搜索路径

```txt
link_directories([AFTER|BEFORE] directory1 [directory2 ...])

# 将 /usr/lib/my_lib 和 ./lib 添加到库文件搜索路径
link_directories(/usr/lib/my_lib ./lib)
```

等价于编译器指定 `-L` 参数。

### target_link_libraries：为 target 添加需要链接的共享库

```txt
target_link_libraries(<target> ... <item>... ...)

# 将 hello 动态库文件链接到可执行文件 main
target_link_libraries(main hello)
```

* `<target>` 必须由 `add_executable()` 或 `add_library()` 等命令创建，且不能为别名。
* 等价于编译器指定 `-l` 参数。

### aux_source_directory：发现一个目录下所有的源文件

```txt
aux_source_directory(dir VARIABLE)

# 定义 SRC 变量，其值为当前目录下所有的源代码文件
aux_source_directory(. SRC)

# 编译 SRC 变量所代表的源代码文件，生成 main 可执行文件
add_executable(main ${SRC})
```

## 常用变量

### CMAKE_\<LANG\>_STANDARD：用于指定 \<LANG\> 标准

主要包含以下变量：

* `CMAKE_C_STANDARD`：C 语言标准
* `CMAKE_CXX_STANDARD`：C++ 语言标准

其中 `CMAKE_CXX_STANDARD` 支持值：

* `98`：C++ 98
* `11`：C++ 11
* `14`：C++ 14
* `17`：C++ 17（New in version 3.8）
* `20`：C++ 20（New in version 3.12）
* `23`：C++ 23（New in version 3.20）

可通过 `set` 命令设置：`set(CMAKE_CXX_STANDARD 11)`。

### CMAKE_\<LANG\>_FLAGS：设置编译选项

主要包含以下变量：

* `CMAKE_C_FLAGS`：gcc 编译选项，由 `CFLAGS` 环境变量初始化。
* `CMAKE_CXX_FLAGS`：g++ 编译选项，由 `CXXFLAGS` 环境变量初始化。

示例：

```txt
# 在 CMAKE_CXX_FLAGS 编译选项后追加 -std=c++11
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
```

该变量会在 `add_compile_options()` 或 `target_compile_options()` 命令执行后传给编译器。

### CMAKE_BUILD_TYPE：设置编译类型

主要有以下类型：

* `Release`：程序开发完成后的发行版本，占的体积小，不可以打断点调试。其对代码做了优化，因此速度会非常快。
  * 等价于编译器指定 `-O3 -DNDEBUG` 参数。
* `Debug`：调试的版本，体积大。
  * 等价于编译器指定 `-g` 参数。
* `MinSizeRel`：
  * 等价于编译器指定 `-Os -DNDEBUG` 参数。
* `RelWithDebInfo`：
  * 等价于编译器指定 `-O2 -g -DNDEBUG` 参数。

```txt
# 设定编译类型为 Debug，调试时需要选择 Debug，否则无法打断点
set(CMAKE_BUILD_TYPE Debug) 
# 设定编译类型为 Release，发布时需要选择 Release
set(CMAKE_BUILD_TYPE Release) 
```

### XXX_SOURCE_DIR：顶层源码的路径

主要包含以下变量：

* `CMAKE_SOURCE_DIR`
* `PROJECT_SOURCE_DIR`
* `<projectname>_SOURCE_DIR`

以上三个变量指代的内容相同，均是指顶层 `CMakeLists.txt` 所在目录。

### XXX_BINARY_DIR：顶层构建树的路径

主要包含以下变量：

* `CMAKE_BINARY_DIR`
* `PROJECT_BINARY_DIR`
* `<projectname>_BINARY_DIR`

以上三个变量指代的内容相同，均是指 CMake 实际构建并生成相关文件的目录。

### CMAKE_\<LANG\>_COMPILER：指定 \<LANG\> 编译器完整路径

主要包含以下变量：

* `CMAKE_C_COMPILER`：指定 C 编译器
* `CMAKE_CXX_COMPILER`：指定 C++ 编译器

### EXECUTABLE_OUTPUT_PATH：可执行文件输出的存放路径

### LIBRARY_OUTPUT_PATH：库文件输出的存放路径

## 编译工程

CMake 工程目录结构：主目录存在一个 `CMakeLists.txt` 文件。

两种编译规则：

* 包含源文件的子文件夹包含 `CMakeLists.txt` 文件，主目录的 `CMakeLists.txt` 通过 add_subdirectory 添加子目录即可。
* 包含源文件的子文件夹未包含 `CMakeLists.txt` 文件，子目录编译规则体现在主目录的 `CMakeLists.txt` 中。

### 编译流程

在 Linux 下使用 CMake 构建 C/C++ 工程的流程如下:

* 手动编写 `CMakeLists.txt`。
* 执行命令 `cmake PATH` 生成 `Makefile`（`PATH` 是顶层 `CMakeLists.txt` 所在的目录）。
* 在 `Makefile` 所在目录，执行命令 `make` 进行编译。

### 构建方式

CMake 构建分为内部构建和外部构建：

* **内部构建：** 在源代码同级目录执行 cmake 命令构建。
  * 该方式会在源代码同级目录下产生大量中间文件，导致工程源文件显得杂乱无章。不推荐使用。
* **外部构建：** 新建目录，并在该目录下执行 `cmake PATH` 命令构建（`PATH` 是顶层 `CMakeLists.txt` 所在的目录）。
  * 该方式可将 CMake 生成的中间文件和 `Makefile` 与工程源文件分开，较为管理，并且如果需要重新构建，只需清空新建的 `build` 目录，重新执行 `cmake PATH` 命令即可。

**推荐使用外部构建（out-of-source build）：**

```txt
# 1. 在当前目录下，创建 build 文件夹
mkdir build

# 2. 进入 build 目录
cd build

# 3. 编译上级目录的 CMakeLists.txt，生成 `Makefile` 和其他文件
cmake ..

# 4. 执行 make 命令，生成 target
make
```

## 工程实践

### Hello World

**文件树：**

```txt
├── CMakeLists.txt

├── main.cpp
```

**CMakeLists.txt：**

```txt
# 设置 CMake 最小版本
cmake_minimum_required(VERSION 3.21)

# 设置工程名
project(test)

# 设置语法标准
set(CMAKE_CXX_STANDARD 11)

# 生成可执行文件
add_executable(${PROJECT_NAME} ${SOURCES})
```

### 包含头文件

**文件树：**

```txt
├── CMakeLists.txt
├── include
│   └── Hello.h
└── src
    ├── Hello.cpp
    └── main.cpp
```

**CMakeLists.txt：**

```txt
cmake_minimum_required(VERSION 3.5) 
project(hello_headers)

# 创建变量 SOURCE，其包含了所有的 cpp 源文件
set(SOURCES
    src/Hello.cpp
    src/main.cpp
)

# 通过所有的源文件生成一个可执行文件 hello_headers
add_executable(hello_headers ${SOURCES})
# 等价于：add_executable(hello_headers src/Hello.cpp src/main.cpp)

# 设置这个可执行文件 hello_headers 需要包含的库的路径
target_include_directories(hello_headers
    PRIVATE 
        ${PROJECT_SOURCE_DIR}/include
)
# 此处也可以使用 include_directories() 包含库路径
```

有关 `target_include_directories` 和 `include_directories` 区别与 PRIVATE 关键词作用，可参见 [cmake：target_** 中的 PUBLIC，PRIVATE，INTERFACE](https://zhuanlan.zhihu.com/p/82244559)。

### 包含静/动态库

**文件树：**

```txt
├── CMakeLists.txt
├── include
│   └── static(shared)
│       └── Hello.h
└── src
    ├── Hello.cpp
    └── main.cpp
```

**CMakeLists.txt：**

```txt
cmake_minimum_required(VERSION 3.5)
project(hello_library)

# 创建静/动态库
add_library(hello_library STATIC(SHARED)
    src/Hello.cpp
)

target_include_directories(hello_library
    PUBLIC 
        ${PROJECT_SOURCE_DIR}/include
)

add_executable(hello_binary 
    src/main.cpp
)

# 链接可执行文件和静/动态库
target_link_libraries(hello_binary
    PRIVATE 
        hello_library
)
```

### 包含第三方库

**文件树：**

```txt
├── CMakeLists.txt
├── main.cpp
```

**CMakeLists.txt：**

```txt
cmake_minimum_required(VERSION 3.5)
project (third_party_include)

# 查找第三方 boost 库
find_package(Boost 1.46.1 REQUIRED)

# 判断该库是否找到
if(Boost_FOUND)
    message ("boost found")
else()
    message (FATAL_ERROR "Cannot find Boost")
endif()

add_executable(third_party_include main.cpp)

# 链接 boost 库
target_link_libraries(third_party_include
    PRIVATE
        Boost::filesystem
)
```

## 参考链接

* [CMake Reference Documentation](https://cmake.org/cmake/help/latest/index.html)
* [CMake - wiki](https://zh.wikipedia.org/wiki/CMake)
* [基于 VSCode 和 CMake 进行 C/C++ 开发「第六讲」CMake](https://mp.weixin.qq.com/s/nmjQIezDb44a089YMQvNyw)
* [cmake-examples-Chinese](https://sfumecjf.github.io/cmake-examples-Chinese/)
