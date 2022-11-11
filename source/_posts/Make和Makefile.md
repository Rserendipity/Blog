# 前言

本文将从零开始，介绍Make、Makefile，理解为什么要使用它，它的语法是怎么样的，一步一步的引导，最终完成通用文件的编写，提高Linux下 ~~写bug的~~ 效率

---
# 一、什么是Make/Makefile
## 1.详述

1. **make**是一个**命令工具**，它用来**解释Makefile** 中的指令
2. 在**Makefile文件**中**描述了整个工程**所有文件的**编译顺序**、**编译规则**
3. Makefile 有自己的书写格式、关键字、函数，像C 语言有自己的格式、关键字和函数一样，而且在Makefile 中可以使用系统shell所提供的任何命令来完成想要的工作
4. 需要我们来编写的Makefile 文件描述了整个工程的编译、连接等规则。其中包括：工程中的哪些源文件需要编译以及如何编译、需要创建哪些库文件以及如何创建这些库文件、如何最后产生我们想要的可执行文件
5. 尽管看起来可能是很复杂的事情，但是为工程编写Makefile 的好处是能够使用一行命令来完成“自动化编译”，提供一个正确的 Makefile，编译整个工程只需要在shell 提示符下输入make命令。整个工程完全自动编译，极大提高了效率。

## 2.总结
- 在Makefile文件中**编写**项目的**组成、编译的方式**
- 在shell中**输入make**即可

---

# 二、make命令/Makefile文件
## 1.make命令
**make是一个shell命令**，默认下它会寻找当前目录下的Makefile的文件，并解释与执行其中的内容。当然，你也可以加入一些选项，指定运行别的地方的Makefile文件，或者把别的文件当作Makefile文件来解释运行

命令如下：
`make [-options] [target]`

- options: **选项，可省略**
  - -f 指定一个文件作为 Makefile 文件
  - -n 只输出将要执行的命令但不执行
  - -s 只执行但不输出命令，和Makefile中命令前加@作用一样
  - -w 显示执行前后的路径
  - -C [dir] 指定 Makefile 执行的所在目录
- target: **执行目标，可省略**
	- 省略后默认使用当前目录下的Makefile文件中的第一个目标，**不存在则会报错**，`目标` 这个概念马上就会介绍
  - 可以通过指定target来指定你需要执行的目标

更多的命令可以使用`make -h`查看

---

## 2.Makefile文件
Makefile是一个文件，文件名就叫Makefile，没有后缀，首字母建议大写（历史遗留问题，早期的make不会识别小写的makefile），Makefile这个文件中的命令将被make识别并运行


Makefile中的基本命令：`目标-依赖`
```c
[目标]: [依赖项] ...
[tab键][@][执行命令]
```

- **目标**
  - 一般是需要编译完成以后生成的文件，也可以是执行的主体
  - `make`默认执行的是Makefile中的第一个`目标-依赖`
  - 可以通过指定make命令中的target来执行具体的目标
- **依赖**
  - 执行当前目标所需要的**文件**/**目标**/**需要的库**等等
  - 依赖可以是Makefile中的其他的目标，所以**可以嵌套**
- **执行命令**
  - 该目标具体要执行的`shell中`的命令，**可以没有**，**也可以有多个**
  - @:可以关闭指令回显，不加的话会在命令行显示当前执行的命令
  - 也可以在运行make时，使用 `make -s` 选项不回显指令

> 补充：
> - 目标和依赖之间有冒号
> - 在Makefile中有注释，以 # 开头，后续的内容不会被make解释
> - 请注意，makefile 中以 **tab** 键开头的会被当成执行命令，所以写注释时不要在 # 前加上**tab**键


---

**例子**：编译test.c

```c
test: test.c
    gcc test.c -o test # 前面的空白字符一定要是 tab 即'\t'字符
```

分析：
- **test**
  - 将要生成的目标
- **test.c**
  - 生成目标所依赖的文件
- **gcc test.c -o test**
  - 执行的具体命令

写好上述Makefile后在shell中执行`make`即可编译test.c
这便是最简单的Makefile编写

**特殊目标**：clean--清理
```c
clean: # 用于清理项目的中间文件
    rm -f test *.o
```
> 补充：可以看出，`clean`也是一个`目标-依赖`，只不过依赖为空，通过`make clean`可以调用它，清理工程中间文件。这里的clean本质也是普通的依赖，但是习惯上将它作为一个特殊的目标，用于清理项目，当让也可以写其他的目标作为清理方案，不过并不建议这么做

**目标和依赖之间可以嵌套**：

- 即当前的依赖暂时还没有，但是可以通过下面的目标生成，则会先调用下面的目标，生成当前的依赖，再执行当前的命令。
- 故：**目标的生成**是一个**递归**的过程，会先生成所有的依赖以后，再执行命令
- 如果有一个依赖不能生成，则make将会报错

例子：
```c
hello: need
    echo "hello world"
need:
	echo "need"
```
执行`make`，输出结果是 need ，然后才是 hello world

通过这样的作法，make一次性可以调用多个目标，只需Makefile的第一个目标的依赖是它们即可
```python
all: need1 need2
   
need1:
	echo "need1, hello"
need2:
	echo "need2, world"
```
`make`执行以后会输出 need1, hello，然后输出 need2, world 
这里的`all`目标只用于调用它的多个依赖，所以不需要命令




---

# 三、使用Makefile编译工程

## 1.了解：源文件 -> 可执行文件的编译过程

以编译main.c为例：

1. **预处理**：gcc **-E** main.c -o main.i
    - 进行**头文件的包含**，**宏的替换**，**宏的展开**
2. **处理**(也叫编译)：gcc **-S** main.i -o main.s
    - 进行**语义、语法的分析**
    - **生成符号表**
    - 生成对应语义的汇编代码（文本文件，可查看）
    - 语法错误会在这个阶段被检测出来
3. **汇编**：gcc **-c** main.s -o main.o
    - 把**汇编代码翻译成机器码**(二进制)
    - 这个二进制文件**不能执行**
    - 生成可连接的.o(目标)文件
4. **链接**：gcc main.o -o main
    - 把生成的.o文件和动态/静态库链接，生成最终的可执行文件
    - 如果只有函数声明，没有实现，会在这个阶段被检测出来：链接错误(Link err)

## 2.为什么需要分开编译

原因如下：

- 只有**第一次**编译才需要**编译全部的文件**
- 项目**修改时**只需要**编译有改动的文件**，最后链接即可
- 因此可以大大加快文件的编译速度

## 3.多文件工程的编译

- 工程文件如下

Add.c

```c
int Add(int x, int y)
{
    return x + y;
}
```

Add.h

```c
extern int Add(int x, int y);
```

main.c

```c
#include <stdio.h>
#include "Add.h"
int main()
{
    int a = 10;
    int b = 20;
    printf("%d", Add(a, b));
    return 0;
}
```

- Makefile文件的编写

```c
# 表明生成可执行文件main需要的文件：main.o Add.o这两个依赖会在下面生成
main: main.o Add.o
  gcc main.o Add.o -o main
# 命令：链接main.o和Add.o以及对应的动态静态库，生成名为main的可执行文件

# 以下是Add.o和main.o的生成方式，和上述相似
Add.o: Add.c
  gcc -c Add.c -o Add.o
main.o: main.c
  gcc -c main.c -o main.o

# 清理工程中间文件
clean:
  rm -f *.o main
```

- 使用make命令编译工程

![在这里插入图片描述](https://img-blog.csdnimg.cn/4bf340cafee44c4da3df56b810ba3922.png)
- 执行顺序：

  1. gcc -c main.c
  2. gcc -c Add.c
  3. gcc main.c Add.o -o main

- 原因：

  1. 生成可执行文件main需要的依赖文件：main.o Add.o
  2. 这两个文件目前还没有，所以会先生成依赖文件main.o和Add.o
  3. 于是最开始调用的是gcc -c main.c，接着调用gcc -c Add.c，这两个依赖文件完成生成
  4. 调用gcc main.c Add.o -o main链接这两个文件成可执行文件

---
- 修改main.c文件

```c
#include <stdio.h>
#include "Add.h"

int main()
{
    int a = 10;
    int b = 20;
    printf("%d", Add(a, b));
    printf("%d", Add(100, 200));
    return 0;
}
```

- 再次使用make命令编译工程

![在这里插入图片描述](https://img-blog.csdnimg.cn/4767293cc99f443b8ecacf2d01da5b14.png)

- 这次**只有main.c文件被编译**了，然后直接**链接了原来编译好的Add.o**

> ==**得到结论**：只有工程中**被修改的文件才会被编译**，这也是make的最**重要的功能之一**==





---

# 四、Makefile中的变量
## 1.自定义变量

**如何定义?**

```c
[变量名]=[变量值]
```

**举例：**

- ALLFILE=main.c Add.c
- 定义了一个变量，**名字为ALLFILE**，变量的**内容是main.c Add.c**

**如何使用?**

- 以下两种方法都可以，**$加()**，括号里写变量名。把 **() 换成{}也是可以**的

```c
$(变量名) / ${变量值} 
```

举例：

- $(ALLFILE)
  - 括号方式
- ${ALLFILE}
  - 大括号方式
- **两者等价**，效果是把这个变量名字，直接替换成内容，**类似于**C/C++语言中的**宏替换**

---

**改造Makefile文件**

如何改造？

- 将生成的 **中间文件** **使用OBJ变量保存**
- 将 **目标文件** **使用TARGET变量保存**

这样做有什么好处？

- 以后**再次增加新的依赖文件**，只需要**修改OBJ变量**然后编写对应的生成方式即可

**改造后Makefile文件如下：**

```c
OBJ=main.o Add.o
TARGET=main

${TARGET}: ${OBJ}
	gcc ${OBJ} -o ${TARGET}

Add.o: Add.c
	gcc -c Add.c
main.o: main.c
	gcc -c main.c

clean:
	rm -f *.o main
```

---

## 2.系统自带变量

**如何查看**

- 使用`make -p`命令可以查看Makefile自带的变量和常量，配合输出重定向，写入到文件，便于查看
- `make -p > os_define`

该命令会在当前的目录下会生成 **os_define** 文件，里面是make定义好的常量，里面的常量可以直接在Makefile中使用

**举例：**

- **$@** **目标**的完整名称 **（常用）**
- **$^** 所有不重复的**依赖**，以空格分割 **（常用）**
- **$+** 所有的依赖文件(包含重复的)，以空格分割
- **$?** 相比目标文件，修改过的依赖文件的列表，以空格分割

**再改造Makefile文件**

如何改造？

- 使用 **$^** 代替命令中的 **依赖**
- 使用 **$@** 代替命令中的 **目标**

这样做有什么好处？

- 在多文件依赖中减少需要列举的文件
- 更加自动化

再改造后Makefile文件如下：

```c
OBJ=main.o Add.o
TARGET=main

${TARGET}: ${OBJ}
	gcc $^ -o $@

Add.o: Add.c
	gcc -c $^
main.o: main.c
	gcc -c $^

clean:
	rm -f *.o $(TARGET)
```

---

## 3.系统自带常量

**如何查看？**

- 与系统变量查看方法一致

在刚刚生成的os_define的文件中，包含make定义的变量和常量

**一些例子：**

- **PWD**：保存当前位置的绝对路径
- **CXX**：当前编译 C/C++ 文件使用的编译器
- **RM**：rm -f 删除指令
- 更多的内容可以查看刚刚生成的os_define文件

**再再改造Makefile文件**

如何改造？

- 使用系统自带的 CXX 代替命令中的gcc
- 使用 RM 代替命令clean中的rm -f

这样做有什么好处？

- **可以跨平台使用**
- 由于 **CXX** 是系统定义的，所以在不同的系统中会自动查找相应的编译器，例如gcc，g++，clang等

**再再改造后Makefile文件如下：**

```c
OBJ=main.o Add.o
TARGET=main

${TARGET}: ${OBJ}
	$(CXX) $^ -o $@

Add.o: Add.c
	$(CXX) -c $^
main.o: main.c
	$(CXX) -c $^

clean:
	$(RM) *.o $(TARGET)
```

---

# 五、Makefile中的赋值



## 1.基本赋值： =

- 最简单的方式，在等号左边就是变量名，右边就是值

注意：
  - 这种赋值是取最后一次赋值的值，在任意位置定义都可以被使用
  - 以下的Makefile文件，执行 make print 输出结果是 999 ，即最后一次的赋值结果

```c
x = 100
print:
    echo $(x)
x = 999
```

注意：
- `x = $(x)`将出错，这将出现递归式循环，所以在执行make时会报错

原因：
- 在给x赋值的时候用的是x的值，x还没有被赋值，所以要给x赋值.......如此往复，死递归就这样出现了，所以make会禁止使用这样的变量，Makefile中使用这样的变量后执行`make`时会报错，报错信息如下

> Recursive variable `x' references itself (eventually)
> 译：递归变量‘x’，对自身的引用

---

## 2.覆盖赋值： :=

- 和我们平常写代码的赋值类似，覆盖原有的值，把右边新的值赋给变量
- 注意：
  - Makefile文件在读取到这个赋值时就会把右边的值给赋值过去，如果右边的值为空(或者还没有被赋值)，则将变量赋值为空，不会存在上面基本赋值中的死递归问题

例子：y变量的值为 hello word

```c
x := hello
y := $(x) world
```

例子：y变量的值为 word，因为在给y赋值的时候，x还没有被定义，所以y被赋值成后面的 world

```c
y := $(x) world
x := hello
```

例子：x变量的值为空，在给x赋值的时候，x还没有被定义

```c
x := $(x) world
```

---

## 3.条件赋值：?=

-  很简单的赋值方式：该变量**没有被赋值**过，**就给它赋值**
-  用这个赋值可以实现**用户重写变量**方式
   -  在通用文件中写上对应的变量，使用条件赋值
   -  条件赋值给定的就是默认值，如果用户没有写，就用默认的值，如果用户自己写了，就不再给对应的变量的赋值，即实现了用户重写编译方式

例子：x还没有被赋值，所以值为123

```c
x ?= 123
```

例子：x已经被赋值，所以值为999，下面的条件赋值不起作用

```c
x = 999
x ?= 123
```

---

## 4.追加赋值：+=

-  很简单的赋值方式：给该变量后面追加值

例子，x最终的值为 abcdef

```c
x = abc
x += def
```

---



# 六、Makefile中的if-else与循环
## 1.if-else

Makefile中的if语句和C语言等语言中的if有所不同

**体现在：**

1. 有 **对值的判断** 和 **变量是否存在** 两种判断
2. if需要对应的 endif
3. 由于使用 endif 来限制if的结束范围，所以在if语句内可以执行多条语句
4. 没有 else if 这样的语法，想要使用 else if 的语法，需要嵌套使用 if 来实现 else if 的功能
5. if可以不跟else
---

**1、对值的判断：**

- `ifeq (内容,比较内容)`
- 判断成功以后会执行if内的语句
- 需要注意的是，if有对应的结束if的语句 --- endif

**一般使用的例子：**

```c
A := abc
B := 
ifeq ($(A),abc)
	B:=100
else
	B:=200
endif # 注意这里的endif，这里是要求

print:
	echo $(B)
```

> 执行`make print`的结果是 100 ，可以看到if的分支控制起到了作用，将 B 的值赋值成 100

---

**嵌套使用if的例子：**

```p
A := abc
B := 123
C := 
ifeq ($(A),abc)
	ifeq ($(B),111)
		C := if-if
	else
		C := if-if-else
	endif # 注意这里的endif，与内嵌的ifeq对齐
endif # 注意这里的endif，与第一个ifeq对齐

print:
	echo $(C)
```
执行`make`，结果为 **if-if-else** 
程序进入外层if，然后进入到内层if的else语句，所以C的值最终为 **if-if-else**




---

**2、对变量是否存在的判断：**

- `ifdef (内容,比较内容)`
- 如果定义了 **内容** 这个变量，就会执行if内的内容

```c
ifdef aaa
	aaa:=aaa is exist
else
	aaa:=aaa will be defined
endif

print:
	echo $(aaa)
```

> 执行`make print`的结果是 aaa will be defined ，可以看到if的分支控制起到了作用，由于没有定义过aaa，所以执行了else的内容，定义aaa为aaa will be defined

- `ifndef (内容)`
- 没有定义过 内容 会执行if的内容，和ifdef的性质相反
- 不过多介绍了

---

## 2.循环

**foreach：**

- 语法 `$(foreach 元素,列表,语句)`
- 将 **列表** 里面以空格分割的内容分别给 **元素** 然后执行 **语句**
- 会返回所有操作完毕的结果列表

**例子：**
```c
list := a b c d
newlist := $(foreach elem,$(list),$(elem)999)

print:
	echo $(newlist)
```

> 执行`make print`的结果是 a999 b999 c999 d999
> - 定义了一个变量 list
> - 用foreach循环，分别取出 list 的元素，交给 elem 变量，执行使得elem后面跟了999数字，然后返回给newlist

---

# 七、伪目标与模式匹配

## 1.伪目标

**先来看一个现象：**

- 在项目中创建一个名为**clean**的文件
- 再使用`make clean`清除项目

**效果如下：**

- make: 'clean' is up to date.

![在这里插入图片描述](https://img-blog.csdnimg.cn/d9fad71140c3488c983d2f360a5a2ddc.png)

可以发现根本没有被执行，clean被认为是一个已存在文件了

---

**为什么会存在伪目标？**

- 由于目标 **clean** 等**不需要依赖文件**，所以如果在项目中存在 "clean" 这个文件的时候，make便找不到 "clean" 的任何依赖文件，所以**始终认为"clean"文件是最新的**，于是**不会执行**我们想要的 clean 中的**命令**

- 为了解决这个问题，便有了伪目标这个概念
- 被伪目标声明的目标在被调用时，始终会执行其中的命令，进而解决问题

**如何声明和使用伪目标：**

- .PHONY:(内容)
  - .PHONY不可缺少，这是一个标识，冒号后的内容便是声明的伪目标
- 使用命令`make clean`，clean命令**始终会被执行**
  - 被伪目标声明的目标会始终执行，也就是说**忽略了make的优势**之一：只编译被改动过的文件
- **可以声明任意目标为伪目标**

**改造 Makefile 文件，使得clean命令始终被执行：**

```c
.PHONY: clean # 重点

clean:
 RM *.o $(TARGET)
```

## 2.模式匹配
**为什么会存在模式匹配？**

- 通过检测上下文，匹配所需要的变量，完成目标编译


**如何使用模式匹配：**

- **%**
  - "%"是一个**标识**，表明开始通配，可以**匹配上下文中所需要的依赖**
  - 和shell中使用 __*__ 进行通配类似
  

**例子：**

```c
%.o:%.c
  $(CXX) -c $^ -o $@
```

**解释：**

- **生成的目标**是“通配出来的”，**所需要的依赖**也是“通配出来的”
- 例如 main.o 的生成
  1. **检测**到需要生成main.o
  2. **匹配**到依赖是main.c
  3. 在**命令**中使用 $(CXX) 来编译 main.c 从而**生成**main.o
- 其他的 .o 文件的生成也是同理，于是我们**使用了一个语句**，**解决了所有的 .o 文件的生成**
- 我们只需要在 **TARGET** 变量中添加所需要的 .o 文件，通配就可以帮我们解决对应的编译问题，**不需要重复的书写**对应的生成 

**改造Makefile文件:**

- 仅需要在OBJ中添加目标，不需要修改其他内容，即可添加项目文件：

```c
.PHONY: clean

OBJ=main.o Add.o
TARGET=main

${TARGET}: ${OBJ}
	$(CXX) $^ -o $@


# 重点
%.o:%.c 
	$(CXX) -c $^ -o $@

clean:
	$(RM) *.o $(TARGET)
```

---

# 八.函数
**原因：单纯模式匹配具有弊端**

- **通配符在定义变量的地方会失效** （重点）
- 定义的变量本质是类似于C/C++的宏，会直接替换
- 所以在变量中使用 % 通配符，会在使用的地方直接展开，不会起到通配的作用

**解决方式：**

- 使用**函数**
- 函数使得能在定义变量的地方展开通配符所代表的文件
- 从而可以使用变量保存项目所需要编译的文件

## 1.系统函数

- **wildcard** : 展开函数
 	- $(函数名 函数执行内容)
  - 例如这个函数的调用就是 $(wildcard *.c)
  - **调用**这个函数，会**把当前Makefile文件所在目录**下的所有 **.c 文件展开**，中间加上空格作为分割

- **patsubst** ：替换函数
  - 字符串替换函数
  - 格式：$(patsubst **匹配规则**,**替换目标**,**替换文本**)
  - 如果 "替换文本" 符合 "匹配规则" ，则把 "替换文本" 替换成 "替换目标" 
  - "匹配规则" 和 "替换目标" 可以**使用通配符 %**
  - 例如 $(patsubst %.c,%.o, $(wildcard *.c)) 
  - 作用：把wildcard函数获取的 *.c 文件替换成对应的 .o 文件

- **notdir** : 路径去除函数
   - 去除路径，只保留文件名
  - $(notdir 文件列表)
  - wildcard函数获取的文件带有路径信息，使用这个函数配合，可以去除路径信息，再交由 makefile 编译

**再改造Makefile文件:**

- 使用 **SOURCE** 存储所有的.c文件
- 使用 **OBJ** 存储所有的.c文件
- 使用 **TARGET**  存储所有的.c文件
- 使用 patsubst 函数把对应的函数转换成对应的 .o 依赖文件


```c
.PHONY: clean

SOURCE  = $(wildcard *.c)
OBJ     = $(patsubst %.c,%.o, $(SOURCE))
TARGET  = main

${TARGET}: ${OBJ}
	$(CXX) $^ -o $@

%.o:%.c
	$(CXX) -c $^ -o $@

clean:
	$(RM) $(TARGET) *.o
```
> 补充：以后只需要在此 Makefile 文件所在的目录下编写代码，Makefile会自动识别它们，并分别编译它们，不再需要手动添加文件了
> ==这便是最基础的通用 Makefile 文件雏形了，下面会有更好的改造文件==

---

## 2.自定义函数
自定义函数是直接在调用的地方，展开函数内的指令，然后执行函数内的指令
- 即：自定义函数本身就是多条指令的调用

**定义：**

- 关键字`define`
- 在这个关键字后是函数名的定义
- `endef`指定函数的定义到哪里结束

**使用：**
- `$(call funcName)`
- 在使用的地方使用上述命令即可，funcName是自定义定义函数的名称


**例子：**
```c
define myfunc1
	echo "this is myfunc1"
	echo "this is myfunc1, too"
endef

print:
	$(call myfunc1)
```
执行`make`，调用print目标，print调用了myfunc1，执行了函数内的内容

---

# 九、通用Makefile

## 1.工程的相似性

**每个工程的需求都大差不差：**

- 编译 **部分** 文件
- 编译 **所有** 文件
- 编译时带有 **不同的选项**
- 编译时 **链接不同的库**

**那我们有必要每个工程都从零开始写一份Makefile文件吗？**

- ~~从偷懒的角度~~ 来说肯定是不想每个工程都从零开始写一份Makefile
- 从效率来说也没有必要每份工程从零写Makefile --- ~~说不定写Makefile的时间超过写代码的时间~~ 
- **解决方案**：
  - 把工程编译的**共同点拿出来**
  - **抽象出一份共用的Makefile**，然后包含这份共用的即可
  - 把需要改变的选项的用变量保存，用户重写这些变量就可以实现不同的选项，进而实现不同工程的不同编译方式
  
---

## 2.Makefile中文件包含

**命令：**

- `include 文件路径+文件名`

**作用：**

- 对于一些通用的变量定义、通用规则，写在一个文件中，任意目录结构中的Makefile想要使用这些通用的变量或规则时，include指定的文件就好了，而不用在每个Makefile中又重写一遍，即实现了通用Makefile文件

**例子**：包含~/路径下名为cmake的文件
```c
include ~/cmake
```

## 3.通用Makefile文件的编写

**如何编写通用文件：**

- 使用 **TARGET** 变量保存编译目标
- 使用 **COMPILER** 变量保存编译器信息
- 使用 **SOURCE** 变量保存需要编译的文件
- 使用 **OPTION** 变量保存编译选项
- **利用条件赋值**，给这些变量赋值，即用户不写这些变量时的初始值

**最终的通用文件：**

- 可以把如下文件 **放在家目录** 下，每次写Makefile文件就只需要包含家目录下的该文件即可
```python
# 伪目标声明，clean始终执行
.PHONY: clean

# 默认目标   -- main
# 默认编译器 -- 让系统指定，即 CXX 系统变量
# 默认源文件 -- 当前目录下全部.c文件
# 默认选项   -- 空
TARGET   	?= main
COMPILER	?= $(CXX)
SOURCE   	?= $(wildcard *.c)
OPTION   	?= 

# 覆盖赋值，不允许用户重写
OBJ:=$(patsubst %.c,%.o,$(notdir $(SOURCE)))

$(TARGET):$(OBJ)
	$(COMPILER) $(OPTION) $(OBJ) -o $(TARGET)

%.o:%.c
	$(COMPILER) -c $^ -o $@

clean:
	$(RM) $(TARGET) *.o
```

> 我将上述代码放在了家目录下，并命名为**cmake**，于是我在编写其他程序时，只需要创建一个Makefile文件，然后在Makefile文件中写`inlcude ~/cmake`就完成了Makefile的编写，只需要在shell中输入`make`即可完成项目的编译，生成最终的目标，需要增加编译选项时，只需要在当前的Makefile文件中给OPTION变量赋值即可，需要指定文件编译时，只需要指定SOURCE变量的值即可 

---

## 4.个人版花里胡哨的Makefile
添加了一些输出语句和自动执行，和上面的通用文件没有本质区别，仅供参考
```sh
# 伪目标声明，clean,all,show始终执行
.PHONY: clean
.PHONY: all

# 默认目标       -- main
# 默认编译器     -- 让系统指定
# 默认源文件     -- 是当前目录下的全部.c
# 中间文件路径   -- build
# 默认选项      -- 空
target   	?= main
compiler	?= $(CXX)
source   	?= $(wildcard *.c)
path        ?= build
options   	?= 

obj=$(patsubst %.c,%.o, $(notdir $(source)))
link=$(patsubst %.c,./$(path)/%.o, $(source))

all: show $(target) run

show:
	@echo ----------------开始编译----------------
	@if [ ! -e $(path) ];then mkdir $(path);fi;
	@echo "目标     ：$(target)"
	@echo "编译器   : $(compiler)"
	@echo "源文件   : $(source)"
	@echo "编译选项 : $(options)"

# 主要的编译
$(target):$(link)
	@$(compiler) $(options) $^ -o $(target)

./$(path)/%.o:./%.c
	@$(compiler) -c $^ -o ./$@

run:
	@echo ----------------编译完成----------------
	@echo ----------------开始运行----------------
	@./$(target)
	@echo ----------------运行完成----------------

# 清理
clean:
	@echo ----------------开始清理----------------
	@$(RM) $(target) ./$(path)/*.o
	@echo "删除obj文件..."
	@echo "删除target..."
	@echo ----------------清理完成----------------
```

**下面是运行截图：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/008502916530426d9f8bf0aef1187ffd.png)

---

# 总结
有什么不清楚的可以在评论区提出

都这么详细了，给个关注不过分吧~