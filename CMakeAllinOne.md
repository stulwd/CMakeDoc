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
## 循环结构
```
foreach(<loop_var> <items>)
   <commands>
 endforeach()
```
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
- 添加动态库：`add_library(xxx_shared SHARED xxx)`相当于`gcc --shared -o xxx_shared.so xxx.o `,或者直接由源文件编译成动态库：`gcc --shared -o xxx_shared.so xxx.c`对应的cmake为 `add_library(xxx_shared SHARED xxx.c)`
- 添加目标库: `add_library(xxx OBJECT xxx.c)`目标库是一种中间文件, 是由`gcc -c xxx.c -o xxx.o`命令所生成
- 添加导入库：`add_library(<name> <SHARED|STATIC|MODULE|OBJECT|UNKNOWN> IMPORTED [GLOBAL])`
  - 功能是添加一个本工程之外的库文件
  - 导入后IMPORTED目标属性置位TRUE
  - 可以像任何本工程创建的库一样被引用
- 添加接口库：`add_library(<name> INTERFACE])`
  - 添加INTERFACE库,一般用于导出头文件
  - 添加完之后必须用
  ```
      target_include_directories(<name> INTERFACE 
                                  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                                  $<INSTALL_INTERFACE:include>)
  ```
  命令来设置头文件路径，然后用`install(TARGETS name EXPORT <export-name>)`将安装的库与export-name进行关联，
  最后install(EXPORT export-name)就安装完成，生成cmake文件，供其他工程使用include命令来导入
- <font color=green>*需要使用的依赖项为obj，而非源文件时，要用生成表达式？不太确定，下周再查*</font>
- 当`BUILD_SHARED_LIBS`global flag变量为1时，不指定STATIC或SHARED目标时，默认编译为SHARED库
---
## target_include_directories
给目标添加包含目录
- target_include_directories(<target> [SYSTEM] [BEFORE]
   <INTERFACE|PUBLIC|PRIVATE> [items1...]
   [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
- 当指定为PUBLIC时，设置INCLUDE_DIRECTORIES和INTERFACE_INCLUDE_DIRECTORIES属性
- 当指定为PRIVATE时，设置INCLUDE_DIRECTORIES
- 当指定为INTERFACE时，设置INTERFACE_INCLUDE_DIRECTORIES

---
## add_subdirectory()
为工程添加一个build的子目录
- 用法: add_subdirectory(source_dir [binary_dir])
- source_dir指定包含CmakeLists.txt源码路径.
- binary_dir指定编译路径，要是没有指定，则默认值为构建树/source_dir，实际上在构建树下新建了一个同名的文件夹用于存放结果 
---
## target_compile_options()
添加编译flags
- 用法：`target_compile_options(<target> [BEFORE]
   <INTERFACE|PUBLIC|PRIVATE> [items1...]
   [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])`
- 当指定为PUBLIC时，设置COMPILE_OPTIONS和INTERFACE_COMPILE_OPTIONS属性
- 当指定为PRIVATE时，设置COMPILE_OPTIONS
- 当指定为INTERFACE时，设置INTERFACE_COMPILE_OPTIONS

---
## target_compile_definition()
给目标添加编译定义, 当添加xxx后，源文件中`#ifdef xxx`为TRUE
- 用法：target_compile_definitions(<target> <INTERFACE|PUBLIC|PRIVATE> [items1...])
- 当指定为INTERFACE时，设置target的INTERFACE_COMPILE_DEFINITIONS属性
- 当指定为PRIVATE时，设置target的COMPILE_DEFINITIONS属性
- 当指定为PUBLIC时，设置target的COMPILE_DEFINITIONS和INTERFACE_COMPILE_DEFINITIONS属性

# 安装命令
## install()
- 目标安装
  ```
    install(TARGETS tar1 tar2
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
    )
  ```
  这一条命令可以在目标后面加上EXPORT <export-name>,将安装的库与export-name进行关联
- export安装
  ```
    install(EXPORT export-name DESTINATION PATH)
  ```
  这条命令用于安装一个export，这个export由在安装目标时加上EXPORT <export-name>来指定

# 引入其他工程命令
## include()
- 用法：include(xxx.cmake)
- 读取并执行cmake文件或module
- 可以用来引入其他工程导出的module，include其他工程通过export安装命令生成的module即可，之后，便可直接使用target_link_libraries()进行库的链接。

# 生成一个package
## step1：创建包的配置文件
- 首先include(CMakePackageConfigHelpers)
- 其次调用configure_package_config_file函数创建包描述文件
```
  configure_package_config_file(cmake/HelloConfig.cmake.in
                                ${CMAKE_CURRENT_BINARY_DIR}/HelloConfig.cmake
                                # path must be the destination where the helloConfig.camke will be installed into.
                                INSTALL_DESTINATION ${CMAKE_INSTALL_PREFIX}/cmake)
```
helloConfig.cmake.in里面一般就是用include命令引入export安装的module
## step2：生成包版本文件(可选)
- 生成版本文件
```
write_basic_package_version_file(
	# <filename> is the output filename
	${CMAKE_CURRENT_BINARY_DIR}/HelloConfigVersion.cmake
	VERSION 1.0.0
	# the major version number must be the same as requested.
	COMPATIBILITY SameMajorVersion
)
```
- <filename>的路径最好和创建的包配置文件的路径一致
## step3：将上面生成的两个文件安装到指定目录下
```
install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/HelloConfig.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/HelloConfigVersion.cmake
  DESTINATION cmake
)
```
## step4: 在其他工程里导入package
- 设置CMAKE_PREFIX_PATH变量来进入安装HelloConfig.cmake的目录  
  set(CMAKE_PREFIX_PATH ${CMAKE_CURRENT_SOURCE_DIR}/../install/cmake)
- 导入pkg  
  find_package(Hello 1.0.0 REQUIRED)
- 至此，就可以正常使用target_link_libraries()链接库

# externalProject
导入外部工程，一般使用`ExternalProject_Add(<name> [<option>...])`导入外部工程，**当使用make命令时，externalProject就会依次执行config，make（biuld），make install（install）命令，除非按照下列options给出特殊的初始化、构建、安装命令，cmake一律把externalProject当成makefile工程**

**dir option：**
- SOURCE_DIR：指定源文件目录
- CMAKE_ARGS: 指定传入的参数，一般前面加-D符号，如`-DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}/hello`指定外部工程的安装树前缀为主工程安装树下的hello文件
- BUILD_ALWAYS：当指定为TRUE时，每次在代码改变时，重新编译
- BINARY_DIR：构建树
- STAMP_DIR：存放log文件的地方
- *_DIR默认情况下是主目录的构建树/<name>-prefix下的各个目录，具体可以查文档

**download options:**
- 如果使用了SOURCE_DIR option指定了一个非空的目录，则download method可以省略。否则，下面几种download method必须被指定
- URL下载：
  - URL <url1> [<url2>...]：外部工程的路径（可以是本地文件路径）或者地址，当超过一个url给出时，会一个接着一个尝试下载，直到某一个成功，
  - URL_HASH <algo>=<hashValue>:将要下载的archive file的哈希值，algo指定哈希算法，hashValue指定值
  - URL_MD5 <md5>：等同于URL_HASH MD5=<md5>
  - DOWNLOAD_NAME <fname>：下载的文件的名称，如果不指定，则URL的末尾指定为文件名，这个option不太需要
  - DOWNLOAD_NO_EXTRACT <bool>：是否需要解压，当为true时，则下载步骤中的解压环节被省略。
  - DOWNLOAD_NO_PROGRESS <bool>：是否要禁用下载日志。
  - TIMEOUT <seconds>：下载的最长时间
  - HTTP_USERNAME <username>：下载操作的用户名，如果下载前需要验证，则要提供
  - HTTP_PASSWORD <password>：下载操作的密码。
- git下载：
  - 使用git下载时，git的最低版本为1.6.5
  - GIT_REPOSITORY <url>：git代码仓库的url
  - GIT_TAG <tag>：git的分支名，标签或者commit的哈希值。
  - GIT_REMOTE_NAME <name>：远程的可选名，默认情况是origin
  - GIT_SUBMODULES <module>...：指定应该被更新的git子模块，默认情况，所有的子模块会被更新
  - GIT_SHALLOW <bool>：指定浅拷贝，如果设置为TRUE，则git clone将会被给入--depth 1选项，避免下载所有的历史
  - GIT_PROGRESS <bool>：当设置为TRUE，则会给git clone命令传入--prograss选项让其报告进度
  - GIT_CONFIG <option1> [<option2>...]：指定config options并传给git clone，每一个option都会被转换为--config <option>
- svn下载：
- mercurial下载：
- cvs：

**update/patch：**
每一次cmake重新运行的时候，如果下载方法支持更新，默认情况下外部工程将会被更新（例如，如果Git tag不一致）
- UPDATE_COMMAND <cmd>...：用一个自定义的命令去更新，会覆盖掉下载方法中的更新步骤
- UPDATE_DISCONNECTED <bool>:当设置为TRUE，更新步骤将会被跳过，不会阻止下载步骤，更新步骤仍可以被添加为一个step target来手动调用，
  （具体查阅ExternalProject_Add_StepTargets方法），这个方法允许开发者在没有网的时候进行构建
- PATCH_COMMAND <cmd>...:指定在更新后要打补丁的命令，默认情况下，没有补丁命令。设计一个鲁棒性强的补丁命令是比较难的，不常用。

**configure step option：**
在下载和更新之后，就开始配置，默认情况下，外部工程是一个cmake工程，但是可以改变。
- CONFIGURE_COMMAND <cmd>...：默认情况下，配置命令会运行cmake工程，options和主工程一致，但是对于非cmake外部工程，需要用此命令设置配置运行命令
  对于不需要配置的工程，<cmd>为""空字符串（引号一定要写），但是要写出CONFIGURE_COMMAND
- CMAKE_COMMAND /.../cmake：要是外部工程主目录里没有CMakeLists.txt,则指定一个可以替代的cmake配置命令路径。要是CONFIGURE_COMMAND已经给出，则这个
  option会被忽略
- CMAKE_GENERATOR <gen>：为外部工程指定cmake generator，要是CONFIGURE_COMMAND已经给出，则这个option会被忽略
- CMAKE_GENERATOR_PLATFORM <platform>：传递一个generator-specific的平台名称到cmake命令(查阅CMAKE_GENERATOR_PLATFORM变量)，
  如果CMAKE_GENERATOR没有给出，则给出这个命令会出错
- CMAKE_GENERATOR_TOOLSET <toolset>：传递一个generator-specific的toolset名称到cmake命令(查阅CMAKE_GENERATOR_TOOLSET变量)，
  如果CMAKE_GENERATOR没有给出，则给出这个命令会出错
- CMAKE_GENERATOR_INSTANCE <instance>：传递一个generator-specific的instance selection到cmake命令(查阅CMAKE_GENERATOR_INSTANCE变量)，
  如果CMAKE_GENERATOR没有给出，则给出这个命令会出错
- CMAKE_ARGS: 指定传入给cmake的参数，一般前面加-D符号，如`-DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}/hello`指定外部工程的安装树前缀为主工程安装树下的hello文件
- CMAKE_CACHE_ARGS  <arg>...
- CMAKE_CACHE_DEFAULT_ARGS <arg>...
- SOURCE_SUBDIR <dir>：要是CONFIGURE_COMMAND没有指定，则默认为cmake工程，会去执行根目录的CMakeLists.txt来配置，但是要是cmakelist在其他路径下，则用此option来指明

**Build step options：**
默认情况下，如果配置步骤使用cmake，则构建步骤同样使用cmake。其他情况下，构建步骤会把外部工程认为是makefile工程，并直接执行make来构建。构建步骤可以指定具体命令来改变这一行为。
- BUILD_COMMAND <cmd>...：指定构建命令，如果没给出此option，则会用make或者cmake --build来构建，要是不需要构建，则cmd为空
- BUILD_IN_SOURCE <bool>：为true时，build将会直接在source目录下进行构建，当在源码下构建时，BINARY_DIRs不应该给出。
- BUILD_ALWAYS <bool>：当设置为true时，则每次都运行构建，当开发者修改外部工程的依赖文件时，并且依赖文件不受到监控时，开启这个选项保证每次都执行构建，一般不需要开启
- BUILD_BYPRODUCTS <file>...：

**Install Step Options:**
默认情况下，如果配置步骤使用cmake，则安装步骤同样使用cmake。其他情况下，安装步骤会把外部工程认为是makefile工程，并直接执行make install来构建。安装步骤可以指定具体命令来改变这一行为。
- INSTALL_COMMAND <cmd>...：

**test step Options:**
测试选项，只当下列任何一个options被定义时，才能开启test功能
- TEST_COMMAND <cmd>...:
- TEST_BEFORE_INSTALL <bool>:当设置为TRUE时，test step将会在install之前执行，默认是在install之后执行
- TEST_AFTER_INSTALL <bool>：当设置为TRUE时，test step将会在install之后执行
- TEST_EXCLUDE_FROM_MAIN <bool>: 
**Miscellaneous Options:**
- COMMAND <cmd> ... : 任何*_COMMAND命令要是有多条语句，可以用此命令加在后面补充

# 链接相关命令
## target_link_directories()
为特定目标添加需要链接的库目录
- 用法：
``` target_link_directories(<target> [BEFORE]
                            <INTERFACE|PUBLIC|PRIVATE> [items1...]
                            [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
```
- 当指定为PUBLIC时，设置LINK_DIRECTORIES和INTERFACE_LINK_DIRECTORIES属性
- 当指定为PRIVATE时，设置LINK_DIRECTORIES
- 当指定为INTERFACE时，设置INTERFACE_LINK_DIRECTORIES
---
## target_link_libraries()
添加需要链接的库
- 链接静态库：`target_link_libraries(main test)` main必须提前被add_executable或者add_library
  所创建，test必须是add_library()创建的静态库目标。对应gcc命令：`gcc main.c -L. libtest.a -o main`或者`gcc main.c -L. -static -ltest -o main`,
  -L参数指定链接库路径。因为此处直接生成了可执行文件（没有指定-shared或-c参数），所以对应的main目标是由add_executable()所创建。
- 链接动态库：`target_link_libraries(main dyLib)` main必须提前被add_executable创建，dyLib必须是被add_library所创建的动态库。
  对应gcc命令：`gcc main.c -L /path/of/sharedLib -ldyLib.so -o main`
---
## link_directories()
为所有目标添加需要链接的库目录
- 用法：`link_directories([AFTER|BEFORE] directory1 [directory2 ...])`
- 可以添加多个目录
- 此命令实际对LINK_DIRECTORIES目录属性进行修改
- 当指定为相对路径，则前缀为当前的源文件目录
- 仅仅对在其之后创建的target有效
---
## link_libraries()
链接库到后面创建的所有对象
- 用法`link_libraries([item1 [item2 [...]]]`
- 优先使用和`link_libraries()`和`target_link_libraries()`进行库的链接

# 自定义目标和命令
## add_custom_target()
创建一个没有输出的目标
```
 add_custom_target(Name [ALL] [command1 [args1...]]
                   [COMMAND command2 [args2...] ...]
                   [DEPENDS depend depend depend ... ]
                   [BYPRODUCTS [files...]]
                   [WORKING_DIRECTORY dir]
                   [COMMENT comment]
                   [JOB_POOL job_pool]
                   [VERBATIM] [USES_TERMINAL]
                   [COMMAND_EXPAND_LISTS]
                   [SOURCES src1 [src2...]])

```
**OPTIONS:**
- ALL :添加到默认构建目标，让其每一次都构建
- WORKING_DIRECTORY：设置工作目录
- DEPENDS：依赖的文件或者add_custom_command输出的文件
- COMMAND: 每一次运行的命令

# 属性
## set_target_properties()
为targets**们**设置属性
- 用法: set_target_properties(target1 target2 PROPERTIES prop1 val1 prop2 val2 ...)
- `POSITION_INDEPENDENT_CODE`属性: 设置1相当于在编译时加上-fPIC参数，但是默认情况下shared或module的target，其值默认为1，其他类型目标默认为0
- `OUTPUT_NAME`属性：设置生成的target base名称，若不指定，则使用逻辑名
- 常用属性
  - `CXX_STANDARD`:设置C++标准
  - `CXX_EXTENSIONS`：
  - `CXX_STANDARD_REQUIRED`
  - `POSITION_INDEPENDENT_CODE`
  - 
---
## set_source_files_properties()
设置源文件属性
- 用法：
```
set_source_files_properties([file1 [file2 [...]]]
                                    PROPERTIES prop1 value1
                                     [prop2 value2 [...]])
```
- 常用属性
  - `COMPILE_FLAGS`:设置某一文件编译flags，如-O2,这时，便会覆盖掉用set_source_files_properties设置的属性

---
## get_source_file_property()
获取源文件属性
- 用法：`get_source_file_property(VAR file property)`

---
## get_property
- 用法 
  ```
  get_property(<variable> 
              <GLOBAL             |
               DIRECTORY [<dir>]  | 
               TARGET    <target> | 
               SOURCE    <source> |
               INSTALL   <file>   |
               TEST      <test>   |
               CACHE     <entry>  |
               VARIABLE           >
              PROPERTY <name>
              [SET | DEFINED | BRIEF_DOCS | FULL_DOCS])
  ```

# 变量
- `CMAKE_CXX_COMPILER_LOADED` C++编译器是否加载
- `CMAKE_CXX_COMPILER_ID`   C++编译器版本
- `CMAKE_COMPILER_IS_GNUCXX`    C++是否GUN
- `CMAKE_CXX_COMPILER_VERSION`  C++编译器版本
- C编译器相关信息将CXX替换为C
- `CMAKE_BUILD_TYPE`    编译类型：release或者debug

# 导入相关指令
## find_package()
查找包并载入环境
### **module mode**
```
find_package(<PackageName> [version] [EXACT] [QUIET] [MODULE]
              [REQUIRED] [[COMPONENTS] [components...]]
              [OPTIONAL_COMPONENTS components...]
              [NO_POLICY_SCOPE])
```
- <PackageName>_FOUND变量将会置位，如果发现包
- 如果给出REQUIRED，则如果找不到包，将会产生错误
- 特定于包的组件在COMPONENTS选项后列出
- 可选组件在OPTIONAL_COMPONENTS选项后列出
- version参数指定包版本，找到的包版本必须与其兼容
- 有两种模式：module和config模式，如果没有找到MODULE模块，则回退到config模式，当指定了MODULE选项时，不会回退到config模式
- module模式下，cmake查找名为Find<PackageName>.cmake的文件，首先在CMAKE_MODULE_PATH路径下查找,这个路径初始情况下为空，需要去设置，然后在CMAKE安装目录下提供的Find modules中找，如果找到，cmake就会读取并执行此文件，如果没找到，就会进入config模式
- 设置CMAKE_FIND_PACKAGE_PREFER_CONFIG为TRUE让cmake首先进入config模式，找不到再进入module模式
### **config mode**
```
find_package(<PackageName> [version] [EXACT] [QUIET]
              [REQUIRED] [[COMPONENTS] [components...]]
              [CONFIG|NO_MODULE]
              [NO_POLICY_SCOPE]
              [NAMES name1 [name2 ...]]
              [CONFIGS config1 [config2 ...]]
              [HINTS path1 [path2 ... ]]
              [PATHS path1 [path2 ... ]]
              [PATH_SUFFIXES suffix1 [suffix2 ...]]
              [NO_DEFAULT_PATH]
              [NO_PACKAGE_ROOT_PATH]
              [NO_CMAKE_PATH]
              [NO_CMAKE_ENVIRONMENT_PATH]
              [NO_SYSTEM_ENVIRONMENT_PATH]
              [NO_CMAKE_PACKAGE_REGISTRY]
              [NO_CMAKE_BUILDS_PATH] # Deprecated; does nothing.
              [NO_CMAKE_SYSTEM_PATH]
              [NO_CMAKE_SYSTEM_PACKAGE_REGISTRY]
              [CMAKE_FIND_ROOT_PATH_BOTH |
               ONLY_CMAKE_FIND_ROOT_PATH |
               NO_CMAKE_FIND_ROOT_PATH])

```
- 使用CONFIG或者NO_MODULE选项，或者未出现在module模式下的选项都会迫使其首先进入config模式
- 名为<PackageName>_DIR的缓存项被创建用于存放包含文件的目录。
- 默认情况下，config将会查找PackageName的包，若NAMES选项给出，则会查找以后面项命名的packages
- cmake将会查找名为<PackageName>Config.cmake或<lower-case-package-name>-config.cmake的文件对于给出的每一个names
- 配置文件名用CONFIGS来给出
- 一旦查找到包，即xxxconfig.cmake文件，就知道了包内容的路径。配置文件的路径将存放在<PackageName>_CONFIG变量

## execute_process()
执行一个或多条子程序
- 用法：
```
execute_process(COMMAND <cmd1> [<arguments>]
                 [COMMAND <cmd2> [<arguments>]]...
                 [WORKING_DIRECTORY <directory>]
                 [TIMEOUT <seconds>]
                 [RESULT_VARIABLE <variable>]
                 [RESULTS_VARIABLE <variable>]
                 [OUTPUT_VARIABLE <variable>]
                 [ERROR_VARIABLE <variable>]
                 [INPUT_FILE <file>]
                 [OUTPUT_FILE <file>]
                 [ERROR_FILE <file>]
                 [OUTPUT_QUIET]
                 [ERROR_QUIET]
                 [COMMAND_ECHO <where>]
                 [OUTPUT_STRIP_TRAILING_WHITESPACE]
                 [ERROR_STRIP_TRAILING_WHITESPACE]
                 [ENCODING <name>])

```
- command与command之间与pipeline相连
- 使用``INPUT_*``, ``OUTPUT_*``, 和 ``ERROR_*`` 选项重定向stdin，stdout，stderr
- WORKING_DIRECTORY指定子进程的工作目录
- RESULTS_VARIABLE存放子程序运行的返回码，要是多个command，则返回码用";"隔开
- OUTPUT_VARIABLE存放子程序标准输出
