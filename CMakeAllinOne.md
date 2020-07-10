# cmake命令使用方法

## project()
设置工程名
- 可指定编译语言，如`project(recipe-01 LANGUAGES CXX)`
# 语法相关命令
## list()
- `list(APPEND <list> [<element>...])`:为一个list增加元素，可以时新的变量
---
## set()
设置变量
- set(var val)设置变量var的值为val
- 可以用来设置global flags变量，如`set(BUILD_SHARED_LIBS ON)`，将add_library()默认编译类型设置为动态
---
## 分支结构
if(expr)...elseif(expr)...else()...endif()

---
## 选项开关option()
提供给用户在构建时可改变的变量
- 用法：`option(<variable> "<help_text>" [value])`
- value一般为ON或OFF
---
## cmake_dependent_option()
依赖选项的选项
- 使用前要include(CMakeDependentOption)加载
- 用法：```cmake_dependent_option(
  OPTION1 "show desc of option" OFF
  "OPTION2" ON
  )```
  当OPTION2值为ON时，OPTION1取前值OFF, 当OPTION2值为OFF, OPTION1值为ON
# 编译相关命令
## add_executable()
添加可执行文件目标，可以指定多个源文件一起编译

---
## add_library()
添加库文件目标
- 添加静态库：`add_library(xxx_static STATIC xxx)`相当于`ar r xxx_static.a xxx.o`，或者由源文件生成`gcc -c -o xxx.o xxx.c`   `ar r xxx_static.a xxx.o`
  对应的cmake为 `add_library(xxx_static STATIC xxx.c)`
- 添加动态库：`add_library(xxx_shared SHARED xxx)`相当于`gcc --shared -o xxx_shared.so xxx.o `,或者直接由源文件编译成动态库：`gcc --shared -o xxx_shared.so xxx.c` 
  对应的cmake为 `add_library(xxx_shared SHARED xxx.c)`
- 添加目标库: `add_library(xxx OBJECT xxx.c)`目标库是一种中间文件, 是由`gcc -c xxx.c -o xxx.o`命令所生成
- <font color=green>*需要使用的依赖项为obj，而非源文件时，要用生成表达式？不太确定，下周再查*</font>
- 当`BUILD_SHARED_LIBS`global flag变量为1时，不指定STATIC或SHARED目标时，默认编译为SHARED库
---
## target_link_libraries()
添加需要链接的目标
- 链接静态库：`target_link_libraries(main test)` main必须提前被add_executable或者add_library
  所创建，test必须是add_library()创建的静态库目标。对应gcc命令：`gcc main.c -L. libtest.a -o main`或者`gcc main.c -L. -static -ltest -o main`,
  -L参数指定链接库路径。因为此处直接生成了可执行文件（没有指定-shared或-c参数），所以对应的main目标是由add_executable()所创建。
- 链接动态库：`target_link_libraries(main dyLib)` main必须提前被add_executable创建，dyLib必须是被add_library所创建的动态库。
  对应gcc命令：`gcc main.c -L /path/of/sharedLib -ldyLib.so -o main`<font color=green>这里如何在cmake语法里指定dyLib的路径呢=cmake不允许build到其他路径？不需要指定：dylib装在了其他路径下，那么会自动为其检测并指定路径：人工指定</font>
---
## add_target_properties()
为targets**们**设置属性
- 用法: add_target_properties(target1 target2 PROPERTIES prop1 val1 prop2 val2 ...)
- `POSITION_INDEPENDENT_CODE`属性: 设置1相当于在编译时加上-fPIC参数，但是默认情况下shared或module的target，其值默认为1，其他类型目标默认为0
- `OUTPUT_NAME`属性：设置生成的target base名称，若不指定，则使用逻辑名
## add_subdirectory()
为工程添加一个build的子目录
- 用法: add_subdirectory(source_dir [binary_dir])
- source_dir指定包含CmakeLists.txt源码路径.
- binary_dir指定编译路径，要是没有指定，则默认值为构建树/source_dir，实际上在构建树下新建了一个同名的文件夹用于存放结果 
