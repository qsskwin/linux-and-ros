# Linux 的 ros 笔记


# 第一章 启程
apt = Advanced Package Tool  
dpkg -i xxx.deb   # 安装  -i = i-install  
dpkg -l           # 列出已装包  
这种安装方式只能安装已经有的deb安装包，不能处理已经依赖关系等，不会更新软件源，不会联网。  
sudo apt install xxx 这种会联网从软件源下载(这就涉及到了我们最开始的linux换源)，apt可以管理软件源的升级，更新，实际上其底层还是调用的dpkg -i这种。   
```bash
apt update        # 更新源信息    
apt install git   # 安装并自动解决依赖    
apt upgrade       # 升级所有包  
```
nano xxx.txt 可以使用linux自带的文本编辑器进行文本编辑，当然还有gedit等。  
cat /xxx 可以用于查看当前文件的内容，会在终端中进行显示  
cmake .是编译当前目录下的文件， cmake ..是上一级  
---
写最简单的代码进行调试：如果是python的话，随便写点，然后在终端相应目录使用python3 xxx.py即可运行，python3是linux22.04安装会自带的版本。是解释语言，也不用像c和cpp一样需要编译。不过这样也导致了效率没有c和cpp高。  
cpp的话则需要在对应目录终端使用g++ xxx.cpp进行编译，编译成可执行文件运行。一般后缀是a.out (linux) 运行的话是./a.out  
这种是最传统的编写方式，针对单文件来说当然可以，但如果文件一旦多起来，需要相互串联，那么就很复杂，一般是g++ a.cpp b.cpp c.cpp -o qss
这种就会生成一个qss.out的文件，使用./qss.out就可以进行输出。
想带调试信息就是 g++ -g a.cpp ...    -g代表着gdb，方便打断点调试  
g++ -Wall ... 则是显示所有警告  

因此这样做是不太好的，于是引入了makefile文件来做这个事情，将所有的命令通过makefile文件及其语法写好，然后使用make指令，就可以编译了，make指令调用的就是makefile文件，makefile文件里都是跟我们刚才说的g++ 类似的有关的语法，展开就形成了g++命令进行文件编译。

但makefile编写人们觉得还是很麻烦，很困难，因此在考虑有没有更简便的工具，于是就有了cmake工具，cmake工具可以使用cmake . 在当前目录下寻找指导其生成makefile文件的CMakeLists.txt，然后生成对应的makefile，最后再make就可以编译啦，cmakelist的相关语法自行进行查阅。

紧接着，make 可以在后面加入-j24 -j12等，启用多核编译，但人们还是觉得大型工程编译慢，后续又推出了cmake生成build-ninja，再ninja工具进行编译的工具，自行进行查阅。

---
ros2 中的ament前缀是有渊源的，ros1中是catkin ros2中是ament  
catkin 这个名字，有明确的官方来源和三层「深意」，不是随便取的。
ROS 文档与社区明确说明：
> The name catkin comes from the tail-shaped flower cluster found on willow trees — a reference to Willow Garage where catkin was created.
> （catkin 名字来自柳树上的尾状花簇 —— 致敬创造它的公司 **Willow Garage**）

- **Willow Garage**：ROS 1 的诞生地（“柳树车库”）
- **Catkin**：柳树的柔荑花序/柳絮/柳花

直接关联：柳树（Willow）→ 柳絮（Catkin）

---

### 二、词源深意（小彩蛋）
`catkin` 词源本身也很巧：
- 源自中古荷兰语 **katteken** = **little cat（小猫咪）**
- 因为柳絮柔软、毛茸茸、像**小猫尾巴**

**双关**：
- 表面：柳树的花（致敬 Willow Garage）
- 深层：**小猫尾巴** → 可爱、轻量、灵活（隐喻构建系统的设计哲学）

---

### 三、技术隐喻（架构深意）
catkin 作为构建系统，名字也暗合设计理念：
- **一串多花**：一个 catkin 花序 = 很多小花 → 对应 **一个工作空间 = 很多功能包**
- **有序下垂、结构清晰**：对应 **包依赖拓扑、有序编译、层次分明**
- **随风传播、易于扩散**：对应 **ROS 包生态、易于分发、跨平台**

---

### 四、和 ament 的呼应
- **catkin**：柳树的柔荑花序（ROS 1）
- **ament**：**同义词**，也是柔荑花序（ROS 2）

**寓意**：
- 一脉相承（构建系统思想）
- 全新独立（不冲突、不混淆）
- 植物学小彩蛋贯穿 ROS 两代

---
**catkin = 柳树的柳絮 → 致敬 Willow Garage；词源是“小猫尾巴”（可爱轻量）；隐喻多包有序、生态扩散。**

colcon build 构建命令 colcon: col + con  
col = collective  集体的 一起的
con = construction 构建

在ros1里 用的是 catkin_make catkin_build 分开的 现在被合并了

ament 定义包怎么写，例如cmakelists.txt package.xml
colcon 负责编译

ros2在安装过程中，会将自身需要的环境变量加入到AMENT_PREFIX_PATH中，因此用ros2 run 的时候会在该系统环境变量下找相应的路径或者资源。 为什么在后续运行节点的时候 需要先进行source install/setup.bash ，就是ros2为你写好了一个一键执行的bash指令，从而把colcon build生成的可执行文件等路径添加到系统变量中。

***ros2为什么能通过修改环境变量rcutils console修改输出的日志格式：***   
ROS 2 能通过 **RCUTILS_CONSOLE_OUTPUT_FORMAT** 等环境变量改日志格式，核心是：**日志系统底层是 rcutils，启动时主动读环境变量 → 全局格式化模板 → 所有日志宏共用 → 一设全进程生效**。

### 1. 架构：谁负责日志？
- **rcutils**：ROS 2 的底层 C 工具库，**提供最基础的日志宏与格式化引擎**
  - `RCUTILS_LOG_INFO` / `RCUTILS_LOG_ERROR` 等
  - 所有上层（`rcl`/`rclcpp`/`rclpy`）都基于它
- **全局单例**：进程内只有一套日志配置，**所有节点共享**

### 2. 环境变量如何生效（关键流程）
1. **进程启动 → 日志系统初始化**
   - `rcutils_logging_initialize()` 自动执行
   - 内部调用：**读取环境变量**
     - `RCUTILS_CONSOLE_OUTPUT_FORMAT`
     - `RCUTILS_COLORIZED_OUTPUT`
     - `RCUTILS_LOGGING_USE_STDOUT` 等

2. **解析格式字符串 → 生成模板**
   - 把 `[{severity}] [{time}] {message}` 这类字符串
   - 解析成**可替换的字段模板**
     - `{severity}` / `{time}` / `{name}` / `{message}` / `{function_name}` / `{file_name}` / `{line_number}`

3. **全局生效：所有日志都用这个模板**
   - 每次 `RCLCPP_INFO` / `RCLCPP_ERROR` 时
   - 底层调用 rcutils：**按模板拼接字符串 → 输出到控制台**

### 3. 为什么用环境变量（设计意图）
- **无需改代码、无需重新编译**
  - 调试时临时改格式：`export RCUTILS_CONSOLE_OUTPUT_FORMAT="..."`
  - 运行时生效，适合调试/部署/CI
- **进程全局、统一风格**
  - 一个设置影响**当前进程所有节点**
  - 避免每个节点单独配置
- **跨语言一致**
  - C++/Python/Java 都用同一套 rcutils 底层
  - 环境变量对所有语言节点有效

### 4. 极简原理一句话
**rcutils 在初始化时读取 RCUTILS_* 环境变量 → 保存全局格式模板 → 每条日志都按这个模板格式化 → 一设全进程生效。**

### 常用变量速查
- `RCUTILS_CONSOLE_OUTPUT_FORMAT`：格式字符串
- `RCUTILS_COLORIZED_OUTPUT`：0/1 强制关/开颜色
- `RCUTILS_LOGGING_USE_STDOUT`：1 改输出到 stdout
- `RCUTILS_LOG_LEVEL`：全局日志级别（DEBUG/INFO/WARN/ERROR/FATAL）
 


rclcpp.rclcpp.hpp头文件报错 添加/opt/ros/humble/include/**在includepath中 




# 第二章 节点

# 2.1 编写你的第一个节点
## 2.1.1 Python
`node.get_logger().info('xxxxxx')`这个代码应理解为，node这个对象，调用其成员函数get_logger() 这个成员函数的返回值又是一个对象，然后再.info，调用返回值对象的成员函数，该成员函数就是日志打印。 
## 2.1.2 C++
```cpp
#define RCLCPP_INFO(logger, ...) \
  logger->log(INFO, __FILE__, __LINE__, __VA_ARGS__)

  RCLCPP_INFO(node->get_logger(), "hello"); //我们使用的

  node->get_logger()->log(
    rcutils_log_severity_t::RCUTILS_LOG_SEVERITY_INFO,
    __FILE__,  // 自动帮你加文件名
    __LINE__,  // 自动帮你加行号
    "hello"
); //编译器实际看到的
宏展开时，编译器会将__FILE__自动替换成当前文件名，__LINE__自动替换成当前行。 跟if else这种一样，编译器天生就认识。
```
---
`int main(int argc, char **argv)`
其中的参数 一个是Argument Count = argc  参数的数量
还一个参数是 Argument vector = argv     参数的具体内容(字符串数组)  
argc 是一个整数，表示你在命令行输入的时候，一共敲了多少个单词，程序名本身也算一个  
argv 是一个字符串数组，存入的是你在命令行输入的每一个单词  
你在终端运行：
```bash
./my_app 123 hello world
```
argc = 4因为有 4 个单词：./my_app、123、hello、world  
argv 是这样的：
``` bash
argv[0] = "./my_app"
argv[1] = "123"
argv[2] = "hello"
argv[3] = "world"
```
在cmake中  
使用find_package(xxx REQUIRED)找依赖，还会嵌套找xxx所需的依赖  
使用target_include_directories(文件名，头文件路径)  
target_link_libraries(文件名 库路径)进行头文件和库文件的依赖添加  
在vscode里的标红只是插件的报错，并不代表着编译会不通过，插件和编译库这俩是分开的，可以编译插件的includepath进行添加报错文件的路径进行添加  

# 2.2 使用功能包组织Python节点
colcon build会生成三个文件夹，其中 build是构建过程中产生的中间文件，install是放置构建结果的，log则是生成的日志信息等。  
为什么要使用setup.bash修改环境变量？  
因为我们写的代码经colcon build编译后，生成的功能包，节点等代码文件，其目录都在install目录下，而ros2 run只会通过AMENT_PREFIX_PATH的路径查找功能包和节点，因此需要修改该环境变量的值来查找功能包，所以就要运行setup.bash。该命令是ros2官方为我们写好了，colcon build的时候会自动生成。 使用`source install/setup.bash`即可更改。  

包含__init__.py的文件夹，表示该文件夹是python的一个功能包.该文件默认为空。
resource文件夹提供功能包标识，可以不用管  
test用于存放代码单元测试的文件  
lincense是协议许可  
package.xml是功能包清单，声明了功能包名称，构建类型，依赖等信息，需要在里面添加依赖，在后续可以在创建包命令里填写  
```bash
 ros2 pkg create demo_python_topic --build-type ament_python --dependencies rclpy example_interfaces --license Apache-2.0
```
在创建包的时候就添加  
setup.cfg是普通的文本文件，就放包的配置选项，这些配置在构建的时候会被读取和处理。  
setup.py则是python包的构建脚本文件，其中包含一个setup()函数，用于指定如何构建和安装python包，有点类似cmakelist.txt

# 2.3 C++
同理 使用ros2 pkg create demo_cpp_pkg --build-type ament_cmake --license Apache-2.0 进行功能包的构建，也可以加入--denpendencies 无非是这个依赖被自动加入到了cmakelists和package.xml中。  
生成的目录和python差不多，代码我们一般写在src中  
构建类型是ament_cmake 其是cmake的超集，又加入了一些更方便的指令，因此我们可以不用麻烦的target添加头文件库文件路径，转而使用ament_target_dependencies(xxx 依赖文件路径)进行直接添加，然后再在install里写入最终生成的路径  
添加完毕之后需要在xml中加入rclcpp的声明。  




## 2.4 多功能包的最佳实践 Workspace

一般在总文件夹里创建一个新的两级文件夹，然后把包mv进这些文件夹中，再进行colcon build。如果有多个包，他们之间有依赖关系，需要在package.yml里写depend
```bash
mkdir -p ws/src
mv demo_cpp_pkg/ chapt2_ws/src/
mv demo_python_pkg/chapt2_ws/src/
```

## 2.5 ROS2基础之编程学习

### 2.5.1 面向对象编程
在python中，所有的python类内部的方法第一个参数默认都是self，代表其本身，类似于Cpp的this指针
#### Python

```python
class PersonNode:
    def __init__(self,name_value:str,age_value:int) -> None:
        print("personnode __init__被调用了,添加了两个属性,分别是name和age")
        self.name = name_value
        self.age = age_value

    def eat(self,food_name:str):
        """
        定义一个方法，表示吃东西
        """
        print(f"{self.name},{self.age}正在吃{food_name}")


def main():
    person_node = PersonNode('张三', 25)
    person_node1 = PersonNode('李四', 30)
    person_node.eat('苹果')
    person_node1.eat('香蕉')
```
![alt text](assets\README\image.png) 在src的对应的python_pkg创建文件，并编写，编写完毕后，在src下的setup.py中填写自己的路径（填写在entry_points）中  
![alt text](assets\README\image-2.png)   
第一个名字是后续colcon生成的节点名字，后面的是路径，：代表要运行的函数
注意，colcon build 命令的使用会产生三个文件，最好在ws的文件夹的这个目录下进行colcon build
关于继承等方面的，python的话建议去chapt2自行查看源码
注意，继承是没办法继承import的文件的，还是得import一下。    

Python的继承：这一章节可以看ros2的书，在第四十九页  
继承后要使用父类的话ua，需要加入super().xxx xxx是父类的构造函数，成员函数等.

#### C++

include <rclcpp/rclcpp.hpp>的原因是在opt/ros/humble/include下，rclcpp.hpp中嵌套了其他的例如node的文件，都在include这个文件夹下，因此如果把路径配置成include/rclcpp，这样确实可以只写 #include <rclcpp.hpp>，但其调用其他文件时会找不到嵌套依赖，仍然会报错。

```Cpp
#include <string>
#include <rclcpp/rclcpp.hpp>

class PersonNode : public rclcpp::Node
{
private:
    // 声明
    std::string name_;
    int age_;

public:
    PersonNode(const std::string &node_name, const std::string &name, const int &age)
        : Node(node_name) // 调用父类的构造函数,传递节点的名字作为参数。等同于python中的super.__init__()
    {
        this->name_ = name;
        this->age_ = age;
    };

    void eat(const std::string &food)
    {
        RCLCPP_INFO(this->get_logger(), "%s , %d age , is eating %s", this->name_.c_str(), this->age_, food.c_str());
    };
};

int main(int argc, char *argv[])
{
    rclcpp::init(argc, argv);
    auto person_node = std::make_shared<PersonNode>("person_node", "Tom", 18);
    person_node->eat("apple");
    rclcpp::spin(person_node);
    rclcpp::shutdown();
    return 0;
}  

```
说一下流程：
1. 首先在src下的demo_cpp_pkg下的src中新建cpp，然后写代码，这个写的就是节点。
2. 然后在cpp里导入rclcpp/rclcpp.hpp的库，写代码。
3. 写完后因为要使用colcon build命令，而colcon build涉及到cmakelist，因此先去cmakelist.txt添加编译的东西
4. ![alt text](assets\README\image-3.png)  
5. 这里find_pkg找工具包和核心库，然后添加可执行文件，意味着将把那些带cpp文件生成名字为前面的可执行文件，因为生成的这些可执行文件需要跟ros2链接起来(要用ros2的相关功能)，将生成后的可执行文件与rclcpp相链接，链接完之后把这些编译好的程序安装到install下，这个目录是ros2执行时能找到的路径。
6. 成功后再source install/setup.bash，这步骤是在配置ros2的运行环境，例如将colcon build后的需要的环境变量添加到系统，使得ros2在执行的时候可以找到.
7. 然后再colcon build 可以写`colcon build -packages-select demo_cpp_pkg` 就只会构建cpp的了，不然cpp和python在一个目录下，都会被生成。
8. 最后 ros2 run 找demo_cpp_pkg 再找节点名字进行运行即可。(有必要去回顾一下前面的关于用cmake生成，再用make的流程，这俩被colcon build合并了) 

### 2.5.2 用得到的C++新特性

1. 自动类型推导：智能指针代替了new和delete，防止内存泄漏，智能管理内存。
```cpp
#include <iostream>

int main()
{
    auto x = 5;     // x is deduced to be of type int
    auto y = 3.14f; // y is deduced to be of type float
    auto z = 'a';   // z is deduced to be of type char

    std::cout << "x: " << x << ", y: " << y << ", z: " << z << std::endl;
    return 0;
}
```
2. 智能指针
```cpp
#include <iostream>
#include <memory>

int main()
{
    auto p1 = std::make_shared<std::string>("Hello, World!");                                    // std::make_shared<数据类型/类>(参数) 返回值，
                                                                                // 对应类的共享指针 std::shared_ptr<std::string>  可以直接写成auto
    std::cout << "p1的引用计数" << p1.use_count() << ",指向的内存地址" << p1.get() << std::endl; // 输出p1的引用计数 1

    auto p2 = p1;                                                                                // 共享指针p2指向与p1相同的内存地址
    std::cout << "p1的引用计数" << p1.use_count() << ",指向的内存地址" << p1.get() << std::endl; // 输出p1的引用计数 2
    std::cout << "p2的引用计数" << p2.use_count() << ",指向的内存地址" << p2.get() << std::endl; // 输出p2的引用计数 2

    p1.reset();                                                                                  // 释放p1指向的内存，p1不再指向任何对象
    std::cout << "p1的引用计数" << p1.use_count() << ",指向的内存地址" << p1.get() << std::endl; // 输出p1的引用计数 0
    std::cout << "p2的引用计数" << p2.use_count() << ",指向的内存地址" << p2.get() << std::endl; // 输出p2的引用计数 1
    std::cout <<"p2指向的内存地址数据" << p2->c_str() << std::endl; // 输出p2指向的内存地址数据
    return 0;
}
```
3. Lambda 主要用来简化回调函数 需要加入algorithm头文件 其优势主要就在捕获变量不用传参。把外部变量打包带进函数里，让调用者无感知，而且是闭包，变量被打包进函数对象里。 也可以不写lambda表达式的返回值，会根据return自动推导
```cpp
#include <iostream>
#include <algorithm>

int main()
{
    auto add = [](int a, int b) -> int
    { return a + b; };      // 定义一个lambda表达式，参数为两个整数，返回它们的和
    int sum = add(200, 50); // 调用lambda表达式，传入200和50，得到结果250
    auto print_sum = [sum]() -> void
    { std::cout << "The sum is: " << sum << std::endl; }; // 定义一个lambda表达式，捕获sum变量，打印结果
    print_sum();                                          // 调用lambda表达式，打印结果
    return 0;
}
```
函数分为自由函数 成员函数(定义到类的内部，也叫做实现方法，如果要调用，得用对应的对象，加上.函数名(参数))   和Lambda函数


4. 函数包装器 可以统一函数的调用方式 需要functionnal头文件
```cpp
#include <iostream>
#include <functional> //函数包装器头文件

// 自由函数

void save_with_free_fun(const std::string &filename)
{
    std::cout << "自由函数: " << filename << std::endl;
}

// 成员函数

class FileSave
{
private:
    /* data */
public:

    void save_with_member_fun(const std::string &filename)
    {
        std::cout << "成员函数: " << filename << std::endl;
    }
};

int main()
{
    FileSave file_save; // 创建FileSave对象

    // Lambda函数

    auto save_with_lambda = [](const std::string &filename) -> void
    {
        std::cout << "Lambda函数: " << filename << std::endl;
    };

    save_with_free_fun("1.txt");             // 调用自由函数
    file_save.save_with_member_fun("3.txt"); // 调用成员函数
    save_with_lambda("2.txt");

    std::function<void(const std::string &)> save2 = save_with_lambda; // 将Lambda函数赋值给函数包装器
    std::function<void(const std::string &)> save1 = save_with_free_fun; // 将自由函数赋值给函数包装器
    std::function<void(const std::string &)> save3 = std::bind(&FileSave::save_with_member_fun, &file_save, std::placeholders::_1); // 将成员函数赋值给函数包装器
    
    save1("a.txt"); // 调用函数包装器，实际调用的是自由函数
    save2("b.txt"); // 调用函数包装器，实际调用的是Lambda函数
    save3("c.txt"); // 调用函数包装器，实际调用的是成员函数
    return 0;
}
```
函数包装器可以将成员函数单独进行包装，其他地方在调用成员函数时可以直接使用，不用将对象传递，一般都用在回调函数里 类似于C里的函数指针进行回调。
	成员函数用std::bind包装的原因：是因为成员函数自带一个this指针，但cpp自动隐藏了，在进行函数包装的时候必须要求接口严格匹配，这样就会导致函数包装不成功，因此需要通过bind将this指针提前绑定好，将函数接口抹平一致 例如原函数是void (A*,int)，bind后就变成了void(int)。 

例如上文中的save3的绑定，第一个传参传的是class成员函数的地址，不是普通的函数指针，而是成员函数指针，类型为 :

 void (FileSave::*)(int) 他不能单独调用，必须要搭配一个对象(this)才能调用，因此第二个参数传递一个实际的对象地址，此时就会将这个对象地址当作this传递给FileSave::save_with_member_fun。第三个则是占位符，表示还没确定的参数，留到以后等调用的时候再进行传递。  

#### 2.5.3 多线程与回调函数
1. Python

Python写好后 记得添加到 setup.py里,然后colcon build 再soruce改变环境变量
![alt text](assets\README\image-4.png)这里是打开服务器并下载其中内容的方法，类似于爬虫

```python
import threading
import requests
class download:
    def download(self,url,callback_word_count):
        print(f"线程{threading.get_ident()}开始下载:{url}")
        """
        模拟下载，实际是请求url并获取文本内容
        """
        response = requests.get(url)
        response.encoding = 'utf-8'
        callback_word_count(url,response.text)

    def start_download(self,url,callback_word_count):
        thread = threading.Thread(target=self.download,args=(url,callback_word_count))
        thread.start()


def world_count(url,result):
    """
    普通函数，后续用于回调
    """
    print(f"{url}:{len(result)}->{result[:]}")

def main():
    downloader = download()
    downloader.start_download("http://0.0.0.0:8000/novel1.txt", world_count)
    downloader.start_download("http://0.0.0.0:8000/novel2.txt", world_count)
    downloader.start_download("http://0.0.0.0:8000/novel3.txt", world_count)
```

2. C++

git clone https://gitee.com/fishros/cpp-httplib.git
这个是cpp的http库 用法在其readme里
函数包装器的声明是&，不用写具体变量名，是一种声明
使用额外的头文件，需要在cmakelist里添加，使用include_directories(/路径)来进行添加。 在这里我们把git的库放在include下，include与cmakelist并级，那就直接填include_directories(/include)即可,这个库只要头文件就可以用，因此没有传统的库文件

```cpp
#include <iostream>
#include <thread>                //多线程
#include <chrono>                //时间 计时的 adj. 类似于time.h
#include <functional>            //函数包装器
#include <cpp-httplib/httplib.h> //HTTP库

class Download
{
private:
    /* data */
public:
    void download(const std::string &host, const std::string &path, const std::function<void(const std::string &, const std::string &)> &callback)
    {

        std::cout << "线程" << std::this_thread::get_id() << std::endl;
        httplib::Client client(host);
        auto response = client.Get(path);
        if (response && response->status == 200)
        {
            std::cout << "下载成功！" << std::endl;
            callback(path, response->body);
        }
        else
        {
            std::cout << "下载失败！" << std::endl;
        }
        // httplib::Client cli(host.c_str());
        // auto res = cli.Get(path.c_str());
        // if(res && res->status == 200)
        // {
        //     std::cout << "下载成功！" << std::endl;
        //     std::cout << "内容：" << res->body << std::endl;
        // }
        // else
        // {
        //     std::cout << "下载失败！" << std::endl;
        // }
    };
    void start_download(const std::string &host, const std::string &path, const std::function<void(const std::string &, const std::string &)> &callback) {
        // std::thread t(&Download::download, this, host, path, callback);
        // t.detach(); //分离线程
        auto download_fun = std::bind(&Download::download, this, std::placeholders::_1, std::placeholders::_2, std::placeholders::_3); //绑定成员函数和参数
        std::thread thread(download_fun,host,path,callback);
        thread.detach();
    };
};

int main()
{
    auto d = Download();
    auto word_count = [](const std::string &path, const std::string &result) -> void
    {
        std::cout << "下载完成" << path << ':' << result.length() << "->" << result.substr() << std::endl;
    };
    d.start_download("http://0.0.0.0:8000", "/novel1.txt", word_count);
    d.start_download("http://0.0.0.0:8000", "/novel2.txt", word_count);
    d.start_download("http://0.0.0.0:8000", "/novel3.txt", word_count);

    std::this_thread::sleep_for(std::chrono::seconds(5)); // 主线程等待5秒
}
```

这里创建线程要分离的原因是因为，这个成员函数在运行完毕之后会被销毁，如果不分离detach，则线程也会没跑完就被跟着销毁了，因此需要分离线程，但这个线程被叫做子线程的原因是因为这是主线程运行下产生的线程，实际上还是向操作系统申请的新线程，他们和主线程的等级是并列的，但确实依托主线程的存活时间，主线程一结束，函数运行完毕，整个代码就会结束，此时子线程会被强制结束，因此需要主线程等待一段时间，等待子线程运行完毕后再结束。

 detach的原因则是函数内运行完毕后会类似栈直接销毁，析构的时候发现线程还在运行的话会报错，加入detach则是当析构的时候不报错，让线程继续运行。

join则是让主线程等待子线程运行完毕。理论上如果子线程是在main里没有被析构，运行时间远小于主线程，其实不写也可以，只是因为 C++ 这样规定，所以必须 join 或 detach，防呆，保证线程的绝对安全。

# 第三章 话题

# 3.1 话题通信介绍

```bash
ros2 run turtlesim turtlesim_node
```
创建一个小海龟节点
```bash
ros2 node list 
```
打印当前节点
当前节点名字会出现 /turtlesim
得分清 turtlesim是功能包名 turtlesim_node 是功能包下被编译出来的的可执行程序文件名 /turtlesim是该程序源码里定义的名字
```bash
ros2 node info /turtlesim
```
可以看见/turtlesim这个节点里的信息
![alt text](assets\README\image-5.png)
其中  /turtle1/cmd_vel: geometry_msgs/msg/Twist 是话题名称和话题接口 用于控制小海龟的

/turtle1/pose: turtlesim/msg/Pose 是小海龟的实时位置信息
```bash
ros2 topic -h
```
 可以查看命令帮助 关于topic的
 ![alt text](assets\README\image-6.png)
```bash
ros2 topic echo /turtle1/pose
```
可以输出该话题的内容

![alt text](assets\README\image-7.png)
theta是弧度制,是前向角度

```bash
ros2 topic info /turtle1/cmd_vel
```
可以查看该话题的接口类型
![alt text](assets\README\image-8.png)

```bash
ros2 interface show /xxx
```
可以查看消息接口的详细信息,/xxx即是消息接口的名字
![alt text](assets\README\image-9.png)

ros2的坐标系是右手坐标系,x轴朝上,y轴朝左 z朝屏幕向外  (二维平面下)

---
```bash
ros2 topic pub /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 1.0 , y: 0.0} , angular: {z: -1.0} }"
```
该命令是进行命令的pub发送 第四个参数是节点名称 第五个参数是节点接口 
请注意Yaml的严格发布格式,冒号后有空格等,默认发布频率是1Hz,可以使用ros2 pub --help 查看相关命令 例如 ros2 topic --r N 可以改变频率 

## 3.2 Python话题的订阅与发布
### 3.2.1 通过话题发布小说
```bash
 ros2 pkg create demo_python_topic --build-type ament_python --dependencies rclpy example_interfaces --license Apache-2.0
```
1. ros2 pkg create

    作用：ROS 2 官方提供的创建功能包工具
    含义：我要新建一个 ROS 2 包
    相当于：mkdir + 自动生成CMakeLists/package.xml + 模板代码

2. demo_python_topic

    这是自定义的【功能包名字】
    可以随便取，比如：my_talker, robot_comm, test_pkg
    这里取名 demo_python_topic 意思是：
        demo：示例
        python：用 Python 写
        topic：用于话题通信

3. --build-type ament_python

    --build-type：指定编译类型
    ament_python：表示这是 Python 版 ROS 2 包
>   ament_python = Python 项目
>
>   ament_cmake = C++ 项目

4. --dependencies rclpy example_interfaces 

自动给功能包添加依赖库 这里实际上就是自动将依赖填写进入package.xml里,跟以前手动写是一样的

interface:接口的意思

① rclpy

    ROS 2 Python 核心库
    所有 Python 节点必须依赖它
    相当于：import rclpy 

② example_interfaces

    ROS 2 官方提供的示例接口（消息 / 服务）
    里面有常用的：
        String 字符串消息
        AddTwoInts 加法服务
    做话题 / 服务通信一定会用到   到时候需要使用string

1. --license Apache-2.0

    指定代码开源许可证
    Apache-2.0 是 ROS 2 默认许可证
    不影响功能，只是代码规范

创建完功能包后,在工作空间进行colcon build,完毕后如果没问题就可以在/src/demo_python_topic里写代码了

代码写完,记住先去setup.py里添加路径,然后source环境变量,colcon build 再ros2 run 找相应的节点运行.

```python
import rclpy
from rclpy.node import Node
import requests
from example_interfaces.msg import String   
from queue import Queue



class NovelPubNode(Node):
    def __init__(self,node_name): 
        super().__init__(node_name)
        self.get_logger().info(f'{node_name} was created')
        self.novel_queue_ = Queue()  #创建队列
        self.novel_publisher_ = self.create_publisher(String, 'novel', 10)
        self.create_timer(5.0, self.timer_callback)

    def timer_callback(self):
        if not self.novel_queue_.empty():
            line = self.novel_queue_.get()
            msg = String()
            msg.data = line
            self.novel_publisher_.publish(msg)
            self.get_logger().info(f'Novel content published: {msg.data}')

    def download(self,url):
            response = requests.get(url)
            response.encoding = 'utf-8'
            text = response.text
            self.get_logger().info(f'Novel content:{text}')
            for line in text.splitlines( ): 
                self.novel_queue_.put(line)  #将每行文本放入队列
            self.get_logger().info('Novel download completed')  


def main():
    rclpy.init()
    node = NovelPubNode('novel_pub')
    node.download("http://0.0.0.0:8000/novel2.txt")
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()
```
打开服务器的命令:python3 -m http.server  
可以拆分终端,输入ros2 topic list -v 查看当前话题是否有我们注册的  
也可以使用 ros2 topic echo /novel 查看实时话题内容  
还有 ros2 topic hz /novel 来查看当前发送频率(我们这里是5s发送一次)

### 3.2.2 订阅小说并合成语音
需要安装:
sudo apt install python3-pip -y 安装包管理工具 
pip3 install espeakng  这是espeakng的python库
sudo apt install espeak-ng 安装这个 这个是语音合成引擎
都安装好
```python
import espeakng
import rclpy
from rclpy.node import Node
from example_interfaces.msg import String   
from queue import Queue
import threading
import time 

class NovelSubNode(Node):
    def __init__(self,node_name): 
        super().__init__(node_name)
        self.get_logger().info(f'{node_name} was created')
        self.noevels_queue_ = Queue()  #创建队列
        self.novel_subscriber = self.create_subscription(String, 'novel', self.novel_callback, 10)
        self.speech_thread_ = threading.Thread(target=self.speak_thread)
        self.speech_thread_.start()
    def novel_callback(self, msg):
        self.noevels_queue_.put(msg.data)  #将接收到的消息放入队列
    def speak_thread(self):
        speaker = espeakng.Speaker()
        speaker.voice = 'zh'
        while rclpy.ok(): # 检测当前ros上下文是否ok
            if not self.noevels_queue_.empty():
                text = self.noevels_queue_.get()
                self.get_logger().info(f'Novel content received: {text}')
                speaker.say(text)  # 使用espeak-ng库进行文本转语音
                speaker.wait()  # 等待语音播放完成
            else:
                time.sleep(1)  # 如果队列为空，稍微休眠一下，避免CPU占用过高
                # rclpy.sleep(0.1)  # 如果队列为空，稍微休眠一下，避免CPU占用过高


def main():
    rclpy.init()
    node = NovelSubNode('novel_sub')
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()
```
写完代码之后 去setup.py 添加路径 然后colcon build 
source 改变环境变量
再ros2 run 即可

---
# 3.3 C++话题订阅与发布

## 3.3.1 发布速度控制小海龟画圆

首先`ros2 run turtlesim turtlesim_node`,即运行turtlesim功能包下的turtlesim_node节点,启动小海龟模拟器  
然后使用`ros2 node list`可以显示当前节点有哪些,可以发现turtlesim这个节点,请分清,上面那个命令的turtlesim是功能包的名字,turtlesim_node是可执行的文件名,这个可执行文件里应该是初始化节点名为turtlesim,因此在这里可以看见节点的名字是turtlesim,不要误以为是功能包  
然后使用`ros2 topic list` 可以查看当前话题有哪些,命令后面可以跟-t 显示话题接口  
没运行小海龟模拟器之前,只有parameter_events和rosout两个话题,运行后会多出三个   
```bash
/turtle1/cmd_vel [geometry_msgs/msg/Twist]
/turtle1/color_sensor [turtlesim/msg/Color]
/turtle1/pose [turtlesim/msg/Pose]
```

使用`ros2 node info /turtlesim` 可以查看这个节点发布了哪些话题,话题接口是什么,订阅了哪些话题,订阅接口是什么，实际上可以把话题接口理解为结构体或者类。   
然后我们要自己编写节点,因此先创建功能包,回到工作空间,使用`ros2 pkg create demo_cpp_topic --build-typde ament_cmake --dependencies rclcpp geometry_msgs turtlesim --license Apache-2.0` 创建功能包 rclcpp geometry_msgs turtlesim 都是功能包的名字,因为我们创建的新功能包下写的节点需要依赖这三个功能包.分别是ros2的官方功能包, 提供C++核心库,geometry_msgs功能包,提供各种消息类型,turtlesim功能包,里面是小海龟,也就是我们的控制的对象.这样其实就加入到了我们cmakelist和xml里了   
这些包写入的含义就是我们写代码的时候需要引入头文件,这些功能包这样加入之后,我们就可以‵#include‵相关的头文件,来使用其他功能包里的代码,比如`rclcpp.init()`出自rclcpp功能包,`auto message = geometry_msgs::msg::Twist();`创建一个Twist消息对象,因为小海龟模拟器的turtlesim节点默认订阅的就是该话题接口,得使用这个(得理解一下,说的不够清晰，就是实例化一个对象，),还有turtlesim,可以获得小海龟发布的话题接口,获取其当前位置,可以做闭环  

```cpp
#include <rclcpp/rclcpp.hpp>
#include <geometry_msgs/msg/twist.hpp>
#include <chrono>
    
using namespace std::chrono_literals;

class TurtleCircleNode : public rclcpp::Node
{
private:
    rclcpp::Publisher<geometry_msgs::msg::Twist>::SharedPtr publisher_; // 发布者的智能指针
    rclcpp::TimerBase::SharedPtr timer_;

public:
    explicit TurtleCircleNode(const std::string &node_name) : Node(node_name)
    {
        publisher_ = this->create_publisher<geometry_msgs::msg::Twist>("turtle1/cmd_vel", 10); // 创建发布者，发布到/turtle1/cmd_vel话题，队列大小为10
        timer_ = this->create_wall_timer(
            1000ms,                                            // 定时器周期为100ms
            std::bind(&TurtleCircleNode::timer_callback, this) // 绑定定时器回调函数
        );
    }

    void timer_callback()
    {
        auto message = geometry_msgs::msg::Twist(); // 创建一个Twist消息对象
        message.linear.x = 1.0; // 设置线速度为1.0
        message.angular.z = 0.5; // 设置角速度为0.5
        publisher_->publish(message); // 发布消息 
    }
};

int main(int argc, char * argv[])
{
    rclcpp::init(argc, argv); // 初始化ROS 2
    auto node = std::make_shared<TurtleCircleNode>("turtle_circle_node"); // 创建节点实例
    rclcpp::spin(node); // 进入循环，等待回调函数被调用
    rclcpp::shutdown(); // 关闭ROS 2 
}
```

值得注意的是,`rclcpp::publisher`的这个publisher不在rclcpp.hpp里,但rclcpp.hpp里又包含了node.hpp,node.hpp里有publisher.hpp,最终完成了嵌套,从而找到了这个publisher,在publisher.hpp里,使用了using namespace rclcpp包含了这个publisher类,因此需要使用rclcpp::publisher  
`publisher<xx>`是一个模板类,xx传入的是什么,就是什么这个类就是xx类,::SharedPtr 是这个类里定义的一个智能指针别名,等于`std::shared_ptr< Publisher<T> >`,有点类似`typdef`  
所以
`rclcpp::Publisher<geometry_msgs::msg::Twist>::SharedPtr publisher_; // 发布者的智能指针`的意思就是声明了一个变量,变量名叫做publisher_,这个变量是一个智能指针变量,变量的类型是`rclcpp::Publisher<geometry_msgs::msg::Twist>::SharedPtr`,它里面以后可以存的以publisher实例化的一个对象的内存地址   

`publisher_ = this->create_publisher<geometry_msgs::msg::Twist>("turtle1/cmd_vel", 10); // 创建发布者，发布到/turtle1/cmd_vel话题，队列大小为10`这里是真的创建了一个发布者对象,使用create_publisher<>()函数,<>里填入要实例化的对象的类,然后将这个对象的智能指针返回出来,给publisher_   
`create_wall_timer`这个函数是父类里的成员函数,但其参数需要用到chrono里的东西,所以需要引入chrono的头文件  
auto message那句等同于`geometry_msgs::msg::Twist message;`    
## 3.3.2 订阅pose实现闭环控制  
`ros2 interface show 接口名称` 可以看见接口的定义 
```cpp
#include <rclcpp/rclcpp.hpp>
#include <geometry_msgs/msg/twist.hpp>
#include <chrono>
#include <turtlesim/msg/pose.hpp>


using namespace std::chrono_literals;

class TurtleControlNode : public rclcpp::Node
{
private:
    rclcpp::Publisher<geometry_msgs::msg::Twist>::SharedPtr publisher_; // 发布者的智能指针
    rclcpp::Subscription<turtlesim::msg::Pose>::SharedPtr subscriber_; // 订阅者的智能指针
    double target_x_ = 1.0; // 目标位置的x坐标
    double target_y_ = 1.0; // 目标位置的y坐标
    double k_ = 1.0; // 控制增益
    double max_speed_ = 3.0; // 最大速度
public:
    explicit TurtleControlNode(const std::string &node_name) : Node(node_name)
    {
        publisher_ = this->create_publisher<geometry_msgs::msg::Twist>("turtle1/cmd_vel", 10); // 创建发布者，发布到/turtle1/cmd_vel话题，队列大小为10
        subscriber_ = this->create_subscription<turtlesim::msg::Pose>(
            "turtle1/pose", 10, std::bind(&TurtleControlNode::on_pose_received, this, std::placeholders::_1));
    }

    void on_pose_received(const turtlesim::msg::Pose::SharedPtr pose) //收到数据的共享指针
    {
        auto current_x = pose->x; // 获取当前x坐标
        auto current_y = pose->y; // 获取当前y坐标
        RCLCPP_INFO(this->get_logger(), "Received pose: x=%.2f, y=%.2f", current_x, current_y); // 打印当前坐标

        // 计算误差
        auto distance = std::sqrt(std::pow(target_x_ - current_x, 2) + std::pow(target_y_ - current_y, 2)); // 计算与目标位置的距离
        auto angle_to_target = std::atan2(target_y_ - current_y, target_x_ - current_x); // 计算指向目标位置的角度
        auto angle_error = angle_to_target - pose->theta; // 计算角度误差

        auto message = geometry_msgs::msg::Twist(); // 创建一个Twist消息对象
        if (distance > 0.1)
        {
            if(fabs(angle_error) > 0.2)
            {
                message.angular.z = fabs(angle_error);
            }
            else
            {
                message.linear.x = k_ * distance;
            }
        }

        // 限制最大速度
        if (message.linear.x > max_speed_)
        {
            message.linear.x = max_speed_;
        }
        if (message.angular.z > max_speed_)
        {
            message.angular.z = max_speed_; 
        }
        publisher_->publish(message); // 发布消息


    }
};

int main(int argc, char * argv[])
{
    rclcpp::init(argc, argv); // 初始化ROS 2
    auto node = std::make_shared<TurtleControlNode>("turtle_control_node"); // 创建节点实例
    rclcpp::spin(node); // 进入循环，等待回调函数被调用
    rclcpp::shutdown(); // 关闭ROS 2
    return 0;
}
```

话题类型和话题接口都必须有,因为话题接口有可能是一样的含义,可以复用,(即结构体内的参数定义是一样的),例如无人机位置的xyz和云台角度的xyz,会混,但是订阅话题名称加上话题接口就不会错了.  
atan2(target_y - current_y , target_x - current_x) 可以求出目标点相对于乌龟的角度 这个值最终是由弧度制表示出来的,因此,假设目标点为1,1 初始点为5,5,求出来的在第三象限,-135度,即-2.356rad.对于theta来说,经测试,范围也是由弧度制表示的,为0到pi到-pi到0,因此在代码里,最开始angle_error是负数减去正数,会越来越大,然后再突然调变到-3.14,此时为误差为正数,最后慢慢逼近到误差绝对值为0.2左右.  
 

# 3.4 话题通信最佳实践
## 3.4.1 自定义通信接口  
在chapt3中新建一个工作空间,进入该工作空间,新建src文件夹,然后进入src,并创建功能包.python是没有办法定义接口的,只能用cpp的功能包, 使用`ros2 pkg create status_interfaces --dependencies builtin_interfaces rosidl_default_generators --license Apache-2.0`,这个Builtin会给我们提供时间戳,就在这个消息接口里面.而rosidl这个可以帮助我们把自定义的消息文件转换成cpp和python的源码.  
然后在status_interfaces下创建msg文件夹,这个名字必须是这个,是规定好了,再在msg里使用驼峰命名法,首字母一定大写,创建一个话题接口,以msg结尾.  
在msg里添加builtin_interfaces/Time stamp,但实际上当我们使用ros2 interface show | grep Time 的时候,会发现其接口名字为builtin_interfaces/msg/Time,我们需要把msg这个去掉再引用,这是ros的格式规定  
写完msg后,再在cmakelist里配置一下,再构建就可以转换了,配置如图![alt text](assets/README/image-10.png)   
例如rosidl这个package就是ros官方的,其提供了rosidl_generate_interfaces这个cmake指令(函数),这个里面要填消息接口的相对路径.    
cmakelist配置完后,再进入xml里添加命令,告诉ros,当前功能包是包含消息接口的功能包,`<member_of_group>rosidl_interface_packages</member_of_group>`,配置当前功能包属于什么组里   
全部做完之后,就可以colcon build了,记得回到ws工作空间里build  
后续能找到我们所编写的话题接口,全部都是因为我们使用了source 将其加入到了我们的环境变量当中,节点和话题都是临时存在的,只有你运行节点,节点才存在,只有节点订阅发布,或者有节点再使用的时候,话题才存在,当最后一个节点断开,该话题也就立刻消失了,但是话题接口不一样,是在编译的时候就一直存在的.  
另外SystemStatus能用却没办法msg跳转,是因为ros2自动生成的是隐式导入,python插件没办法识别到,rclpy是工程师写的可以直接获取  

## 3.4.2 系统信息获取与发布 Python

```python
import rclpy
from status_interfaces.msg import SystemStatus
from rclpy.node import Node
import psutil
import platform

class SystemStatusPub(Node):
    def __init__(self):
        super().__init__('system_status_pub')
        self.publisher_ = self.create_publisher(SystemStatus, 'system_status', 10)
        timer_period = 1.0  # seconds
        self.timer = self.create_timer(timer_period, self.timer_callback)

    def timer_callback(self):
        cpu_percent = psutil.cpu_percent()
        memory_info = psutil.virtual_memory()
        net_io_counters = psutil.net_io_counters()
        msg = SystemStatus()
        msg.stamp = self.get_clock().now().to_msg()
        msg.host_name = platform.node()
        msg.cpu_percent = cpu_percent
        msg.memory_percent = memory_info.percent
        msg.memory_total = memory_info.total /1024 /1024 # convert to MB
        msg.memory_available = memory_info.available /1024 /1024 # convert to MB 
        msg.net_sent = net_io_counters.bytes_sent /1024 /1024
        msg.net_recv = net_io_counters.bytes_recv /1024 /1024 # convert to MB
        self.get_logger().info(f'Published system status: {str(msg)}')
        self.publisher_.publish(msg) 

def main():
    rclpy.init()
    node = SystemStatusPub()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()
```

没啥说的,看得懂即可  注意要source,另外就是如`ros2 interface list`这类指令要熟悉.   

## 3.4.3 在功能包中使用qt

没啥说的,照样重新创建功能包,然后导入库,Qt的库是放在usr/include的,linux安装的时候会自动安装Qt的库,因此不需要自己安装,另外,以前usr是放用户文件的,后来硬盘变大了,就专门创建了home用于用户使用,把usr这个文件夹改成了专门存放系统资源,系统级软件,系统级库的地方,rclcpp,rclpy也都在这,这地方也是默认搜索路径的时候会被搜索的地方,因此cmake编译的时候能找到.我们经常看见的opt则是用来存放第三方大软件,大开发工具的, 因此vscode ros2等,都被安装在这里.    
另外,还要注意cmakelist等路径的编写  
```cpp
#include "QApplication"
#include "QLabel"
#include <QString>

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);
    QLabel* label = new QLabel();
    QString message = QString::fromStdString("Hello, Qt!");
    label->setText(message);
    label->show();
    app.exec(); //执行应用,阻塞代码
    return 0;
}
```
## 3.4.4 订阅数据并用qt显示

whereis xxx 可以查安装路径 例如 whereis qt5
本节代码
```cpp
#include "QApplication"
#include "QLabel"
#include <QString>
#include "rclcpp/rclcpp.hpp"
#include <status_interfaces/msg/system_status.hpp>

using SystemStatus = status_interfaces::msg::SystemStatus;  

class SysStatusDisplay : public rclcpp::Node
{
private:
    rclcpp::Subscription<SystemStatus>::SharedPtr subscriber_;
    QLabel *label_;
public:
  SysStatusDisplay() : Node("sys_status_display")
  {
    label_ = new QLabel();
    subscriber_ = this->create_subscription<SystemStatus>(
      "system_status",
      10,
      [&](const SystemStatus::SharedPtr msg) 
      {
        label_->setText(get_qstr_from_msg(msg));
      });
      label_->setText(get_qstr_from_msg(std::make_shared<SystemStatus>()));//构造函数里调用第一次,传入一个空的默认值对象,先显示出qt界面
      label_->show();
  }


  QString get_qstr_from_msg(const SystemStatus::SharedPtr msg)
  {
    std::stringstream show_str;
    show_str << "======系统状态可视化显示工具======\n";
    show_str << "数据时间:\t " << msg->stamp.sec << "s\t\n";
    show_str << "主机名字:\t " << msg->host_name << "\t\n";
    show_str << "CPU使用率:\t " << msg->cpu_percent << "%\t\n";
    show_str << "内存使用率:\t " << msg->memory_percent << "%\t\n";
    show_str << "内存总大小:\t " << msg->memory_total << "%\t\n";
    show_str << "剩余有效内存:\t " << msg->memory_available << "%\t\n";
    show_str << "网络发送量:\t " << msg->net_sent << "MB\t\n";
    show_str << "网络接收量:\t " << msg->net_recv << "MB\t\n";    
    return QString::fromStdString(show_str.str());
    
  }
};

int main(int argc, char *argv[])
{
    rclcpp::init(argc, argv);
    QApplication app(argc, argv);
    auto node = std::make_shared<SysStatusDisplay>();
    std::thread spin_thread([&](){rclcpp::spin(node);});
    spin_thread.detach();
    app.exec(); //执行应用,阻塞代码
    return 0;
}
```

写完代码后,去cmakelist 添加`find_package(),add_executalbe`对于qt类的,需要使用原生的target_link_libraries命令,和ros2有关的可以用ament_target_dependencies 然后install中写路径   
在代码中,使用了lambda表达式,[&]则是捕获当前作用域里所有用到的变量,在这个代码里,我们就捕获了this指针.  
因为 Lambda 里用到了：  
label_ → 它是类成员变量  
get_qstr_from_msg() → 它是类成员函数  
访问类成员 → 必须通过 this 指针。  
lambda只会捕获我们用到的变量  
编译器的工作流程非常简单：看到你写了 [&] 或 [=]  进入 Lambda 内部，逐行扫描代码.发现你使用了外部的变量或者成员,就捕获了.    
另外,也可以QString返回引用或者指针,不过qt已经做好了所谓的隐式共享,所以无所谓.
# 3.5 Git的使用 

```bash

git config --global user.name "Qss" 
git config --global user.email "xxx" 可以在github上显示头像啥的,得配置正确,错的也行,无所谓
git config --global init.defaultBranch master

git config -l


git add src/xxx
git add src
git add .
git reset
git commit -m "xxx"   --message

.gitignore 里使用通配符 *.xxx 可以忽略后缀为xxx的文件
```

# 第四章 服务和参数

服务是用来进行双向传递信息的,你可能会疑惑为什么不用双向订阅对方发布的话题完成这种情况.因为双向订阅话题这种本质上还是异步通信,对于某些必须要得到反馈结果的来说,就没办法做到,而服务可以可靠的执行命令,等待结果,处理错误,一对一调用,必须要有应答.    
ROS2底层用的是DDS,底层只有订阅发布这一个机制.服务通信实际上就是由两个话题构成的,参数通信和动作通信是多个服务和话题一起构成的,实际上都是话题的变种  

# 4.1 服务通信介绍
## 4.1.1 基础服务通信及其介绍
`ros2 service list -t` 显示出服务及其接口类型
![alt text](assets/README/image-11.png)   
`---`上面的4个是请求部分 下面是返回部分.
![alt text](assets/README/image-12.png)   
使用`ros2 service call /服务名 服务接口 "数据yaml格式"`可以调用服务,如图,成功后会给你反馈.   
除了使用这种命令行的请求方式外,还可以使用rqt进行请求服务,在plugins的service里 选择call 然后填写数据即可,可以再摸索摸索rqt,里面还有topic相关   
## 4.1.2 基于服务的参数通信
参数被视为节点的设置,基于服务通信实现     
`ros2 param list`可以展现参数列表    
`ros2 param describe /turtlesim background_r` 可以查看描述     
`ros2 param get /turtlesim background_r`可以查看值      
`ros2 param set /turtlesim background_r 255` 可以设置值     
这种单个配置方式比较繁琐,还有使用文件进行参数配置的方法,首先将节点的配置文件导出为yaml格式的文件:`ros2 param dump /节点名字 > /输出的文件名.yaml`,可以用cat查看         
然后`ros2 run turtlesim turtlesim_node --ros-args --params-file turtlesim_param.yaml `即可实现配置.后面的三个参数都是传给了argc和argv,根据我们的设置解析我们的命令.读取yaml,将参数应用到命令上去.    
使用`ros2 param -h` 可以查看相关命令,rqt里也可以通过可视化实现这些   
# 4.2 Python服务通信,实现人脸检测  
## 4.2.1 自定义服务接口  
大致流程:  
新开一个chapt4文件夹,文件夹下新开chapt4_ws,进入chapt4_ps,新开src,创建功能包`ros2 pkg create chapt4_interfaces --dependencies sensor_msgs rosidl_default_generators --license Apache-2.0`,用不到include和src,删掉.在功能包下新建srv,新建FaceDetector.srv,注意驼峰命名,进入FaceDetector.srv,添加`Sensor_msgs/Image image`,这是对应功能包名/消息接口名 定义该种类型变量  然后自定义类型,遵循ros2官方给的定义语法,例如 数组定义为int32[] a 而不是 int32 a[],定义完毕之后,在cmakelist里添加`rosidl_generate_interfaces(${PROJECT_NAME} "srv/FaceDetector.srv" DEPENDENCIES sensor_msgs)`,然后修改xml,添加`<member_of_group>rosidl_interface_packages</member_of_group>`    
请记住 接口全名等于功能包名/srv(或者msg)/接口名,例如本节中,chapt4_interfaces是功能包名,srv是存放服务接口的文件夹,是个固定名字,类似msg一样必须这样写,然后FaceDetector.srv是服务接口文件,FaceDetector是接口名.官方写的是`sensor_msgs/Image`是因为是消息的话可以省略这个msg,是简写,但是其他的诸如服务,动作等就不能够省略    
填写完毕之后 在ws下进行`colcon build` 然后查看install是否有py的文件 cpp文件hpp文件,路径要知道是怎么分配的,然后source一下添加环境变量,再`ros2 interface show chapt4_interfaces/srv/FaceDetector`可以查看该接口详细内容,可以发现show的后续结构就是 功能包名,srv,接口名.谨记srv文件的命名得是驼峰命名,不然会报错.     
另外,source之后之所以能找到这个消息接口类型,是在install下的share/srv找到的.这是ros的规定,消息接口ros会自动在个相对路径里找,外部路径则是通过source加入的   
## 4.2.2 Python人脸检测 
`pip3 install face_recognition -i https://pypi.tuna.tsinghua.edu.cn/simple` 安装人脸识别的库   
人脸识别的图片放在resource中,但colconbuild的时候不会拷贝图片过去,需要在setup.py里自己添加.在data_files=` ('share/' + package_name + '/resource', ['resource/default.jpg']),    `里面自己添加,左边是目标地址,右边是文件的相对路径.以/开头的都是绝对路径,从名字开始的是相对路径   
另外,在python中 self.a = 和 a = 在类中是不一样的,self是整个类都可以用,另一个只是临时变量,用完就会销毁.   
```python
import face_recognition
import cv2
from ament_index_python.packages import get_package_share_directory #获取功能包的share目录绝对路径

def main():
    #获取图片的真实路径
    default_image_path = get_package_share_directory('demo_python_service') + '/resource/default.jpg'
    # 加载图片并进行人脸识别
    print(f'图片真实路径: {default_image_path}')
    # 使用cv2来加载图片
    image = cv2.imread(default_image_path)
    # 使用face_recognition库来检测人脸位置
    face_locations = face_recognition.face_locations(image,1, 'hog') #返回人脸上下左右
    # 绘制人脸边框
    for (top, right, bottom, left) in face_locations:
        cv2.rectangle(image, (left, top), (right, bottom), (0, 255, 0),4)  # 返回的是左上角的x,左上角的y,和右下角的xy,矩形只需要确定两点即可绘制,后面是颜色BGR和线条粗细
    #结果显示
    cv2.imshow('Face Detection', image)
    cv2.waitKey(0)
```
## 4.2.3 人脸检测的服务端的实现

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image
from cv_bridge import CvBridge
import cv2
from chapt4_interfaces.srv import FaceDetector
import face_recognition
import os   
from ament_index_python.packages import get_package_share_directory #获取功能包的share目录绝对路径
import time 

class FaceDetectNode(Node):
    def __init__(self):
        super().__init__('face_detect_node')
        self.srv = self.create_service(FaceDetector, 'face_detect', self.face_detect_callback)
        self.bridge = CvBridge()

    def face_detect_callback(self, request, response):
        if request.image.data:
            cv_image = self.bridge.imgmsg_to_cv2(request.image)
        else:       
            default_image_path = get_package_share_directory('demo_python_service') + '/resource/default.jpg'
            self.get_logger().info(f'传入为空,默认图像')
            cv_image = cv2.imread(default_image_path)
        start_time = time.time()  # 记录开始时间
        self.get_logger().info(f'开始识别')
        face_locations = face_recognition.face_locations(cv_image, 1, 'hog')
        response.use_time = time.time() - start_time  # 计算识别时间
        response.number = len(face_locations)  # 检测到的人脸数量
        self.get_logger().info(f'识别完成,用时: {response.use_time:.2f}秒,检测到{response.number}张人脸')
        for (top, right, bottom, left) in face_locations:
            response.top.append(top)
            response.right.append(right)
            response.bottom.append(bottom)
            response.left.append(left)
        return response 


def main():
    rclpy.init()
    node = FaceDetectNode()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()
```

## 4.2.4 人脸检测客户端的实现

```python 
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image
from cv_bridge import CvBridge
import cv2
from chapt4_interfaces.srv import FaceDetector
import face_recognition 
from ament_index_python.packages import get_package_share_directory #获取功能包的share目录绝对路径
import time 

class FaceDetectClientNode(Node):
    def __init__(self):
        super().__init__('face_detect_client_node')
        self.bridge = CvBridge()
        self.default_image_path = get_package_share_directory('demo_python_service') + '/resource/test.jpg'
        self.get_logger().info(f'客户端启动')
        self.client = self.create_client(FaceDetector, 'face_detect')
        self.image = cv2.imread(self.default_image_path)
    def send_request(self):
        #判断服务端是否在线
        while self.client.wait_for_service(timeout_sec=1.0) is False:
            self.get_logger().info('服务端未上线,等待中...')
        #创建请求对象
        request = FaceDetector.Request()
        request.image = self.bridge.cv2_to_imgmsg(self.image)
        #发送请求并等待响应
        self.future = self.client.call_async(request) #现在future是一个Future对象,它表示一个异步操作的结果,我们可以在将来某个时间点检查这个对象以获取结果
        rclpy.spin_until_future_complete(self,self.future) #这个函数会阻塞当前线程,直到future对象完成(即服务端返回响应)或者发生异常
        response = self.future.result() #获取服务端返回的响应结果
        self.get_logger().info(f'服务端响应:检测到{response.number}张人脸,用时{response.use_time:.2f}秒')
        self.show_response(response)
    def show_response(self,response):
        for i in range(response.number):
            top = response.top[i]
            right = response.right[i]
            bottom = response.bottom[i]
            left = response.left[i]
            cv2.rectangle(self.image, (left, top), (right, bottom), (0, 255, 0), 2)
        cv2.imshow('Face Detection Result', self.image)
        cv2.waitKey(0) #等待用户按键后关闭窗口,也是阻塞,会导致spin无法正常运行
    

def main():
    rclpy.init()
    node = FaceDetectClientNode()
    node.send_request() #发送请求并等待响应
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()
```