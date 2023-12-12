# C++

--8<-- "docs/cs/lang/cpp-term.md"

## 基本形态

!!! quote
    任何常用的编程语言都具备一组公共的语法特征，想要学习并掌握一种编程语言，理解其语法特征的实现细节是第一步。

=== "类型"

    ```

                                             x86_64 size   + 1 bool
                                                           |
                                                           | 1 char      +
                                                           |             |
                         + void                            | 2 short     |
         + build-in type |                 + integral type |             |  signed
         |               + arithmetic type |               | 4 int       |
    type |                                 |               |             |  unsigned
         |                                 |               | 8 long      |
         + class type                      |               |             |
                                           |               + 8 long long +
                                           |
                                           + floating+point type + 4 float
                                                                 |
                                                                 + 8 double
    ```
    首先，有了这些内置类型，自然涉及到类型之间的转换。隐式内置类型转换总是从低精度向高精度靠拢，避免精度损失，这是符合现实经验的；而显式内置类型转换，如布尔与非布尔类型之间的转换、整数和浮点之间的转换、无符号类型溢出（取模）、有符号类型溢出（UB），其规则是符合直觉的，其结果是可以自行推导的。

    其次，因为类型解释了数据所占的 bit 数和取值范围，因此又衍生出了标准化的整数类型，用于不同的机器平台之间，确保可移植性。

    !!! todo
        什么是标准化的整数类型
    


=== "变量"

    ```
                                   separate compilation +
                                                        |
                       + + extern  declaration          |
                       | |                              |
                       | + static  definition           +
               + scope |
               |       | global scope  ::
               |       |
               |       |             + inner scope      +------------------------------+
               |       + block scope |                  |            type              |
               |                     + outer scope      |       specifier   identifier |
               |                                        |                              |
               | compound type: & *                     |             int   num;       |
    variable   |                                        |                              |
               |       + top-level                      | const    double   pi = 3.14; |
               | const |                                +------------------------------+
               |       + low-level   +-->  const_cast
               |
               | constexpr
               |
               + type alias +  typedef
                            |
                            |  using
                            |
                            |  auto
                            |
                            +  decltype

    ```
    首先看变量的作用范围。开始只有一个源文件，范围分为 global 和 block。有了分离式编译后，产生了多个源文件，补充了 extern 和 static，分别将作用域限定为跨文件和本文件内。这些作用域一层层嵌套，自然会产生覆盖，覆盖规则始终是内层覆盖外层。例外情况是，global 作用域没有名字，可以使用左侧为空的域作用符（::），在内层向全局作用域发出获取请求。

    接下来是变量的类型，它在内置类型和类类型基础上，增添了 & 和 *，称为复合类型。根据现实经验，变量对应着常量，于是可以用 const 和 constexpr 声明为常量，常量在编译过程就可计算出来，并且不会改变。由于有了复合类型中的指针，const 分为 top 和 low 两个 level，分别修饰类型本身和类型指向的对象。

    变量的类型有时是如此的复杂，为了编程方便，增添了 typedef 和 using 对类型（及它的修饰）取别名，增添了 auto 和 decltype 自动推导和获取变量的类型。

    !!! tip "最佳实践"
        在计算机中，拷贝就是开销。对于 string、标准库容器、自定义的类类型，在传递参数时尽量使用引用（&），并根据需要指定为 top-level const。


=== "表达式"

    ```
                          + arithmetic          +  -  *  /  %
                          |
                          | relational          < <=  > >= == !=
                          |
                          | logical            && ||  !
                          |
                          | bit                 &  |  ~  ^ << >>
                          |
                          | assign              =
                          |
                          | compound assign    arithmetic=  bit(not ~)=
                          |
                          | increase/decrease  ++ --
                          |
               + operator | member access      ->  .
               |          |
               |          | condition          ? :
               |          | 
               |          | sizeof             sizeof()
               |          |
    expression |          + comma               ,
               |
               |
               |
               + operand

    ```
    
    表达式中如果有多个运算符，怎么判断求值顺序？去背那些优先级吗？不，不要这样，代码是写给人看的，只是顺带能在机器上运行。用括号强制让表达式的组合关系按照程序逻辑的要求。
    
    关于递增递减运算符，总是有着前置还是后置的争论。尽量前置，因为后置会临时存储一份原始值。
    

=== "语句"

    ```
               + block{ }
               |                                      return   +
    statements | expression                                    |
               |                          + if     +           |
               | null(;)         + branch |        |  break    |
               |                 |        + switch |           |
               + flow of control |                 |  continue |
                        +        |         + while |           |
                        v        + iterate |       |  goto     +
                    condition              + for   +
                  +------------+
                  true     false

    ```
    语句是顺序执行的，为了满足现实需要，自然而然出现了控制流，多组 if 衍生出 switch，while 衍生出 for、do-while。再后来，出现了对控制流本身的控制，break、continue、goto。

    c++11 中，为了更方便地使用引用，又进一步对原始 for 升级为 range-for，减轻了编程负担。

    !!! tip "最佳实践"
        ``` c++
        for (const auto &s : vecStr) {
            ...
        }
        ```

=== "函数"

    ```
                         return type +-----------------+
                         +   function name             +--------> function type
                         |   +   parameter list +------+
                         +   +   +
    function prototype   int main(int argc, char *argv[])
                         {
                             return 0;
                         }

                       + void reset(int &i) { } // passed by reference  ++ candidate function
    function overload  |                                                ||
                       + void reset(int *i) { }                         +| viable function
                                                                         |
                  inline void reset(int  i) { } // passed by value       v best match

    ```

    变量有类型，函数的类型是什么？是它的返回类型和形参列表。有了类型，就可以自然过渡到函数指针。
    
    变量有类型，数组的类型是什么？是它的元素类型和维度。有了类型，就可以自然过渡到数组指针。

    函数类型由返回类型（rt）和形参列表（pl）组成。C 语言中，函数名是区别不同函数的标准，只要定义时，函数名重复出现了，编译就会失败。在 C++ 中，这个判定范围扩大了，由函数名和形参列表共同作为区别函数的标准。于是，这个特性被叫做 “支持函数重载”。（验证方法 `nm a.out | grep " T " | c++filt`）

=== "类"

    ```
                   non-member function +
                                       | interface for user
                              + public +--
                              |           \                    + delegating
            + member function | protecte   \     + constructor |
            |                 |             -----+             + converting
      class |                 + private          + destructor
        ^   |
        |   + data member
        |
    +---+----+
    | friend |
    +--------+

    ```    

    C++ 的类被设计出来，区别于 C，怎么强制用户使用接口？设置访问修饰符，将属于 public 的成员函数作为接口，供用户访问。

    成员函数可以访问数据成员，自然而然引出了 this 指针。那么问题来了，这个 this 指针能不能被修改？或者这么说，修改 this 指针符不符合常理？修改了有什么用处？没有好处，反而会破坏程序的稳定性，因此，this 是一个常量指针，不能被修改。

    继续推导，this 指针是常量指针，但它指向的对象是可以被修改的。于是，又打了一个可选补丁，可以在成员函数的参数列表后面添加 const，表明该成员函数不会修改 this 指针的对象。

    回到接口，除了 public 部分的成员函数，还包括类外的非静态函数，它们中有部分可作为友元直接访类的数据。比如 B 指定了 A 作为友元，A 就能访问 B 的一切，反过来不可以。我觉得这不能叫友元，这名字太困惑了，应该叫上级。并且它不具备传递性，也就是我上级的上级不是我的上级。学习语言有趣的一点是除了看它支持哪些，还看它为什么停留住，而不继续不支持更多。估计编译器也不想头疼。
    
    成员函数的声明在类内，而定义可以放到类外。那么问题来了，类外的定义是怎么知道 this 指针的？有一个类的域作用符，打这开始到定义结束，都是 this 的作用范围。有一个例外，就是定义的返回类型，如果返回类型是类内声明的，需要费点神，再用一次类的域作用符。

    最后是构造函数，它没有返回值，函数名就是类名，因此只剩下形参列表和数据成员的初始化可以讨论。数据成员初始化按照声明的顺序来。形参列表又衍生出委托和转换，都是简化代码复杂度。委托就是借用某个版本的构造函数，再加点自己的理解，形成的又一个构造函数。转换就是用形参构造中间对象，中间对象可以到最终结果。

    

---

## 标准库





## 面向对象

## 模板


## 疑问

!!! question "来自 基本形态-变量-复合类型-指针：c++11 为什么要引入 nullptr？"

!!! question "来自 基本形态-表达式-操作符：为什么 sizeof(void) == 1 ?"


!!! question "来自 基本形态-类-构造函数：为什么构造函数没有返回值？"
    如果有返回值，代表用户可以随意调用构造函数，这样的操作是危险的！
