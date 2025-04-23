<h1 id="a98ebcbf">第一章 类型推导</h1>
<h4 id="2d9ca232">🌍条款一：理解模板类型推导</h4>
⭐

关联知识点：编译，item24

<h5 id="cf565141">**前置:**<font style="color:#c21c13;">值类型</font></h5>
C++中，左值(lvalue)是一种表达式，它可以标识一个内存中的位置，并且通常可以用来修改该位置的内容。左值表达式可以出现在赋值运算符的左侧，可以通过左值来改变其所指向的数据。

**左值表达式的种类**

变量、函数、模板形参。

什么是模板形参？

**左值的性质**

左值表达式可以通过取地址运算符 & 获取其地址也，可以用来初始化左值引用，这会将给变量标识一个新名字。

C++11为了支持移动语义，值类别有两种独立性质：拥有身份和可被移动。

（1）拥有身份：

如果一个表达式可以确定是否与另一个表达式比较地址，那么这个表达式就拥有身份。

1. 可被移动：如果一个表达式可以被移动构造函数、移动赋值运算符所绑定，那么这个表达式就是可被移动的。

**<font style="color:#c21c13;">左值 (lvalue)：拥有身份且不可被移动的表达式。</font>**

例子：变量、函数、返回左值引用的函数调用、内建的前置自增/减表达式、内建的间接寻址表达式、内建的下标表达式（当操作数之一是数组左值时）、对象成员表达式（除了特定情况）、内建的指针成员表达式（除了特定情况）、内建的逗号表达式（当右侧表达式是左值时）、三元条件表达式（当两侧表达式类型相同时）、字符串字面量、转换到左值引用类型的表达式等。

**<font style="color:#c21c13;">亡值：拥有身份且可被移动的表达式。</font>**

对象成员表达式（当对象是右值时）、返回对象的右值引用类型的函数调用、类型转换到对象的右值引用类型的表达式等。亡值通常由`std::move`或显式的右值引用(`T&&`)产生。

+ 亡值表示“即将过期”的对象，表明**资源可以安全地移动(**这里的移动就是所有权转移)。
+ 如果直接对左值进行移动操作，编译器无法判断你是否仍需要这个对象，所以不会自动应用移动语义。所以std::move的作用就是把左值变为亡值（亡值也是右值），如果调用std::move就相当于告诉编译器什么呢

```cpp
std::string str="hello";
std::string&& r=std::move(str);//std::move(str)是亡值

std::string&& func(){
    std::string str="world";
    return std::move(str);//std::move(str)是亡值
}
```

**<font style="color:#c21c13;">纯右值 </font>**

不拥有身份且可被移动的表达式。

例子：字面量（非字符串）、**<u>非引用返回类型的函数调用</u>**、后自增/减表达式、算术/逻辑/比较表达式、转型到非引用类型的表达式等。

```cpp
std::string func(){
    static std::string str = "example";
    return str;
}
std::string s = func();
```

`str` 是一个左值，存储在静态存储区域。函数返回的是 `str` 的副本，拷贝生成了一个临时对象，临时对象是 **纯右值。**`func()` 的返回值是一个纯右值，直接用来初始化变量 `s`。

右值 ：可被移动的表达式。

纯右值 和亡值 都是右值。

<h5 id="419ce6d6">**前置：右值引用**</h5>
`&&`是右值引用，用于绑定到右值。当右值引用(`T&&`)绑定到右值时,它允许直接操作即将被销毁的临时对象（右值）。右值的作用域延续到引用结束。

编译器如何匹配`&&`类型？

当一个函数的参数是`&&`时，编译器会在匹配时优先匹配右值对象。如果传入的是左值，通过`std::move`将左值显式转换为右值,也能匹配`&&`。 

<h5 id="3290476b">**<font style="color:#c21c13;">前置：通用引用（万能引用）</font>**</h5>
<u>通用引用通常表示为 T&&，其中 T 是一个模板参数类型（模板中的&&表示通用引用）。&& 虽然通常表示一个右值引用，但当T由模板参数推导而来时，其既可以绑定左值，又可以绑定右值。当 T 是一个模板参数类型，并且T是推导出来的类型，T&&才是通用引用</u>。

<u>通用引用 既可以接收左值，又可以接收右值。实参是左值，通用引用就是左值引用（引用折叠）实参是右值，通用引用就是右值引用。</u>

（引用折叠 ：当其传入参数为左值时，&&会折叠成&；当传入参数为右值时，&&不折叠照常接收）

```cpp
void Fun(int& x){cout<<"左值引用"<< endl; }
void Fun(const int& x){cout<<"const左值引用"<< endl; }
void Fun(int&& x) {cout<<"右值引用"<<endl; }
void Fun(const int&& x) {cout<<"const右值引用"<< endl; }
// 万能引用:既可以接收左值，又可以接收右值
// 实参左值,他就是左值引用（引用折叠）
// 实参右值,他就是右值引用
template<typename T>
void PerfectForward(T&& t){
    Fun(t);
}
int main(){
    PerfectForward(10); // 右值
    int a;
    PerfectForward(a);     
    return 0;
}
```

**<font style="color:#c21c13;">模板类型推导</font>**

理解`auto`的模板类型推导

```cpp
template<typename T>
voidf(ParamType param);
```

它的调用看起来像这样

```cpp
f(expr);                        //使用表达式调用f
```

编译期间,编译器使用`expr`进行两个类型推导:一个是针对`T`的,另一个是针对`ParamType`。这两个类型通常不同，因为`ParamType`包含一些修饰，如`const`和引用修饰符。

对于模板函数：

```cpp
template<typename T>
void f(ParamType param);
```

编译器需要完成两步类型推导：

**（**_**1）推导 T 的类型 **_

T是模板参数，当调用 f 函数并传递一个参数时，编译器会尝试根据该参数来确定 T 的具体类型。

如果调用 f(10)，编译器会推断 T 是 int。

_**（2）推导 ParamType 的类型**_

ParamType 是函数参数的类型，它可以是 T 本身，也可以是对 T 进行了一些修饰后的类型，比如加上 const、&（引用）、*（指针）等。例如，假设定义如下函数模板

```cpp
template<typename T>
void f(const T& param);
```

ParamType 是 const T&，T 的常量引用。如果调用 f(10)，编译器会推断 T 是 int，ParamType 是 const int&。

```cpp
template<typename T>
void f(const T& param) {
    // 函数体
}
```

_**<u>调用示例1</u>**_

```cpp
int x = 10;
f(x);
```

编译器推导 T 为 int，ParamType 为 const int&。

_**<u>调用示例 2</u>**_

```cpp
double y = 3.14;
f(y);
```

编译器推导 T 为 double，ParamType 为 const double&。

_**<u>调用示例 3</u>**_

```cpp
std::string s = "hello";
f(s);
```

编译器推导 T 为 std::string，ParamType 为 const std::string&。

T 是模板参数，ParamType 是函数参数的实际类型，它可能是 T 本身，也可能是对 T 进行了一些修饰后的类型（如 const T&、T* 等）。编译器会根据你传递给函数的实际参数来推导 T 和 ParamType 的具体类型。

有三种情况：

+ `ParamType`是一个指针或引用，但不是通用引用
+ `ParamType`是一个通用引用
+ `ParamType`既不是指针也不是引用

分情况讨论三种情况，每个情景基于之前给出的模板：

```cpp
template<typename T>
voidf(ParamType param);
    f(expr);                        //从expr中推导T和ParamType
```

**情景一：ParamType是一个指针或引用，但不是通用引用**

这个地方不背了

在这种情况下，类型推导会这样进行：

1. 如果`expr`的类型是一个引用，忽略引用部分。
2. 然后`expr`的类型与`ParamType`进行模式匹配来决定`T`。

```cpp
template<typename T>
void  f(T& param);               //param是一个引用
```

我们声明这些变量，

```cpp
int x=27;//x是int
const int cx=x;//cx是const int
const int& rx=x;//rx是指向作为const int的x的引用
```

在不同的调用中，对`param`和`T`推导的类型会是这样：

```cpp
f(x);     //T是int，param的类型是int&
f(cx);    //T是const int，param的类型是const int&
f(rx);    //T是const int，param的类型是const int&
```

在第二个和第三个调用中，因为`cx`和`rx`被指定为`const`值，所以`T`被推导为`const int`，从而产生了`const int&`的形参类型。这对于调用者来说很重要。当他们传递一个`const`对象给一个引用类型的形参时，他们期望对象保持不可改变性，也就是说，形参是reference-to-`const`的。

这也是为什么将一个`const`对象传递给以`T&`类型为形参的模板安全的：对象的`const`会被保留为`T`的一部分。

在第三个例子中，即使`rx`的类型是一个引用，`T`也会被推导为一个非引用 ，这是因为`rx`的引用性在类型推导中会被忽略。

如果我们将`f`的形参类型`T&`改为`const T&`，情况有所变化。`cx`和`rx`的`const`依然被遵守，但是因为现在假设`param`是reference-to-`const`，`const`不再被推导为`T`的一部分：

```cpp
template<typename T>
voidf(const T& param);//param现在是reference-to-constint x = 27;                     //如之前一样
    const int cx = x;//如之前一样constint& rx = x; 
    //如之前一样
    f(x);    //T是int，param的类型是const int&
    f(cx);   //T是int，param的类型是const int&
    f(rx);   //T是int，param的类型是const int&
```

同之前一样，`rx`的reference-ness在类型推导中被忽略了。如果`param`是一个指针（或者指向`const`的指针）而不是引用，情况本质上也一样：

```cpp
template<typename T>@
voidf(T* param); //param现在是指针
    int x = 27;//同之前一样
    const int *px = &x;//px是指向作为const int的x的指针
    f(&x);//T是int，param的类型是int*
    f(px);//T是const int，param的类型是const int*
```

**情景二：ParamType是一个通用引用**

模板使用通用引用形参的话，那事情就不那么明显了。这样的形参被声明为像右值引用一样（也就是，在函数模板中假设有一个类型形参`T`，那么通用引用声明形式就是`T&&`)，它们的行为在传入左值实参时大不相同。

+ 如果`expr`是左值，`T`和`ParamType`都会被推导为左值引用。第一，这是模板类型推导中唯一一种`T`被推导为引用的情况。第二，虽然`ParamType`被声明为右值引用类型，但是最后推导的结果是左值引用。
+ 如果`expr`是右值，就使用正常的推导规则

```cpp
template<typename T>
voidf(T&& param); //param现在是一个通用引用类型
    int x=27; //如之前一样
    const int cx = x; //如之前一样
    const int & rx =cx;//如之前一样
```

```cpp
f(x);  //x是左值，所以T是int&，//param类型也是int&
f(cx); //cx是左值，所以T是const int&，//param类型也是const int&
f(rx); //rx是左值，所以T是const int&，//param类型也是const int&
f(27);   //27是右值，所以T是int，param类型就是int&&
```

**情景三：ParamType既不是指针也不是引用**

当`ParamType`既不是指针也不是引用时，通过传值的方式处理。

```cpp
template<typename T>
voidf(T param);                
//以传值的方式处理param
```

无论传递什么，`param`都会成为它的一份拷贝一个完整的新对象。`param`成为一个新对象这一行为会影响`T`如何从`expr`中推导出结果。如果`expr`的类型是一个引用，忽略这个引用部分。如果忽略`expr`的引用性之后，`expr`是一个`const`，那就再忽略`const`。如果它是`volatile`，也忽略`volatile`

```cpp
int x=27;                       //如之前一样
const int cx=x;                 //如之前一样
const int & rx=cx;              //如之前一样
f(x);                           //T和param的类型都是int
f(cx);                          //T和param的类型都是int
f(rx);                          //T和param的类型都是int
```

注意即使`cx`和`rx`表示`const`值，`param`也不是`const`。`param`是一个完全独立于`cx`和`rx`的对象——是`cx`或`rx`的一个拷贝。具有常量性的`cx`和`rx`不可修改并不代表`param`也是一样。这就是为什么`expr`的常量性在推导`param`类型时会被忽略：因为`expr`不可修改并不意味着它的拷贝也不能被修改。

只有在传值给形参时才会忽略`const（volatile`），对于reference-to-`const`和pointer-to-`const`形参来说，`expr`的`const`在推导时会被保留。

如果`expr`是一个`const`指针，指向`const`对象，`expr`通过传值传递给`param`：

```cpp
template<typename T>
voidf(T param);                //仍然以传值的方式处理param
    const char* const ptr = ".."        //ptr是一个常量指针
    f(ptr);                         //传递const char * const类型的实参
```

在这里，解引用符号（*）的右边的`const`表示`ptr`本身是一个`const`：`ptr`不能被修改为指向其它地址，也不能被设置为null（解引用符号左边的`const`表示`ptr`指向一个字符串，这个字符串是`const`，因此字符串不能被修改）。

当`ptr`作为实参传给`f`，组成这个指针的每一比特都被拷贝进`param`。

`ptr`**自身的值会被传给形参**，根据类型推导的第三条规则，`ptr`自身的常量性`const`将会被省略，所以`param`是`const char*`，一个可变指针指向`const`字符串。在类型推导中，这个指针指向的数据的常量性`const`将会被保留，但是当拷贝`ptr`来创造一个新指针`param`时，`ptr`自身的常量性`const`将会被忽略。

**数组实参**

比如数组类型不同于指针类型，虽然它们两个有时候是可互换的。关于这个错觉最常见的例子是，在很多上下文中数组会退化为指向它的第一个元素的指针。

```cpp
constchar name[] = "J. P. Briggs";     //name的类型是const char[13]
constchar * ptrToName = name;          //数组退化为指针
```

在这里`const char*`指针`ptrToName`会由`name`初始化，而`name`的类型为`const char[13]`，这两种类型（`const char*`和`const char[13]`）是不一样的，但是由于数组退化为指针的规则，编译器允许这样的代码。

但要是一个数组传值给一个模板会怎样？会发生什么？

```cpp
template<typename T>
void f(T param); //传值形参的模板
f(name);  //T和param会推导成什么类型?
```

有一个函数的形参是数组，是的，这样的语法是合法的，

```cpp
void myFunc(int param[]);
```

但是数组声明会被视作指针声明，这意味着`myFunc`的声明和下面声明是等价的：

```cpp
void myFunc(int* param);                //与上面相同的函数
```

数组与指针形参这样的等价是C语言的产物，C++又是建立在C语言的基础上，它让人产生了一种数组和指针是等价的的错觉。因为数组形参会视作指针形参，所以传值给模板的一个数组类型会被推导为一个指针类。在模板函数`f`的调用中，它的类型形参`T`会被推导为`const char*`：

```cpp
f(name);                        //name是一个数组，但是T被推导为const char*
```

虽然函数不能声明形参为真正的数组，但是可以接受指向数组的引用**，**修改`f`为传引用：

```cpp
template<typename T>
voidf(T& param);                       //传引用形参的模板
```

我们这样进行调用

```cpp
f(name);                                //传数组给f
```

`T`被推导为了真正的数组！这个类型包括了数组的大小，在这个例子中`T`被推导为`const char[13]`，`f`的形参（该数组的引用）的类型则为`const char (&)[13]`。这种语法看起来又臭又长，但是知道它将会让你在关心这些问题的人的提问中获得大神的称号。

有趣的是，可声明指向数组的引用的能力，使得我们可以创建一个模板函数来推导出数组的大小。

```cpp
//在编译期间返回一个数组大小的常量值
constexpr std::size_t arraySize(T (&)[N]) noexcept    
return N;                                 
}
```

在[Item15](https://cntransgroup.github.io/EffectiveModernCppChinese/3.MovingToModernCpp/item15.html)提到将一个函数声明为`constexpr`使得结果在编译期间可用。这使得我们可以用一个花括号声明一个数组，然后第二个数组可以使用第一个数组的大小作为它的大小，就像这样：

```cpp
int keyVals[] = { 1, 3, 7, 9, 11, 22, 35 };             //keyVals有七个元素int maint Vals=[arraySize(keyVals)];                     //mappedVals也有七个
```

```cpp
std::array<int, arraySize(keyVals)> mappedVals;         //mappedVals的大小为7
```

至于`arraySize`被声明为`noexcept`，会使得编译器生成更好的代码，具体的细节请参见[Item14](https://cntransgroup.github.io/EffectiveModernCppChinese/3.MovingToModernCpp/item14.html)。

**函数实参**

在C++中不只是数组会退化为指针，函数类型也会退化为一个函数指针，我们对于数组类型推导的全部讨论都可以应用到函数类型推导和退化为函数指针上来。结果是：

```cpp
void someFunc(int, double);         //someFunc是一个函数，//类型是void(int, double)
template<typename T>
void f1(T param);                   //传值给f1template<typename T>
void f2(T & param);                 //传引用给f2
f1(someFunc);//param被推导为指向函数的指针，//类型是void(*)(int, double)
f2(someFunc); //param被推导为指向函数的引用，//类型是void(&)(int, double)
```

如果数组可以退化为指针，函数也会退化为指针。

`auto`依赖于模板类型推导。然而，数组和函数退化为指针把这团水搅得更浑浊。有时你只需要编译器告诉你推导出的类型是什么。这种情况下，翻到item4,它会告诉你如何让编译器这么做。

我的思考：

如果模板编程规则这么多，为什么还要有模板编程。

<h4 id="cb056456">条款二：理解auto类型推导</h4>
auto类型推导在编译器看来和模板类型推导是一样的。

item1模板类型推导包含`auto`类型推导的大部分内容，为什么不是全部是因为这里有一个`auto`不同于模板类型推导的例外。模板类型推导包括模板，函数，形参，但`auto`不处理这些东西啊。

`auto`类型推导和模板类型推导有一个直接的映射关系。在item1中，模板类型推导使用下面这个函数模板

```cpp
template<typename T>
voidf(ParmaType param);
```

```cpp
f(expr);                        //使用一些表达式调用f
```

在`f`的调用中，编译器使用`expr`推导`T`和`ParamType`的类型。

当一个变量使用`auto`进行声明时，`auto`扮演了模板中`T`的角色，变量的类型说明符扮演了`ParamType`的角色。

```cpp
auto x=27;
```

`x`的类型说明符是`auto`自己。

```cpp
const auto cx = x;
```

类型说明符是`const auto`。

```cpp
const auto& rx = x;
```

类型说明符是`const auto&`。在这里例子中要推导`x`，`cx`和`rx`的类型，编译器的行为看起来就像是认为这里每个声明都有一个模板，然后使用合适的初始化表达式进行调用：

```cpp
template<typename T>//概念化的模板用来推导x的类型
void func_for_x(T param);
func_for_x(27);//概念化调用：
//param的推导类型是x的类型
template<typename T>//概念化的模板用来推导cx的类型
void func_for_cx(const T param);
func_for_cx(x);//概念化调用：
//param的推导类型是cx的类型
template<typename T>//概念化的模板用来推导rx的类型
voidfunc_for_rx(const T & param);
    func_for_rx(x);
    //概念化调用：
    //param的推导类型是rx的类型
```

`auto`类型推导除了一个例外,其他情况都和模板类型推导一样。

item1基于`ParamType`,在函数模板中`param`的类型说明符的不同特征，把模板类型推导分成三个部分来讨论。

在使用`auto`作为类型说明符的变量声明中，类型说明符代替了`ParamType`。因此Item1描述的三个情景稍作修改就能适用于auto：

+ 情景一：类型说明符是一个指针或引用但不是通用引用。
+ 情景二：类型说明符一个通用引用。
+ 情景三：类型说明符既不是指针也不是引用。

情景一和情景三例子在上面。

```cpp
auto x = 27;//情景三(x既不是指针也不是引用)
const auto cx = x;//情景三(cx也一样)
const auto &rx = cx;//情景一(rx是非通用引用)
```

情景二像你期待的一样运作：

```cpp
auto&& uref1 = x;               //x是int左值，
//所以uref1类型为int&

auto&& uref2 = cx;              //cx是const int左值，
//所以uref2类型为const int&

auto&& uref3 = 27;              //27是int右值，
//所以uref3类型为int&&
```

item1 总结对于non-reference类型说明符，数组和函数名退化为指针。这同样适用于`auto`类型推导。

```cpp
const char name[] =             //name的类型是const char[13]
"R. N. Briggs";

auto arr1 = name;               //arr1的类型是const char*
auto& arr2 = name;              //arr2的类型是const char (&)[13]

void someFunc(int, double);     //someFunc是一个函数，
//类型为void(int, double)

auto func1 = someFunc;          //func1的类型是void (*)(int, double)
auto& func2 = someFunc;         //func2的类型是void (&)(int, double)
```

`auto`类型推导和模板类型推导几乎一样的工作。

**<font style="color:#c21c13;">下面讨论auto和模板类型推导不同的地方：</font>**花括号的处理是`auto`类型推导和模板类型推导唯一不同的地方。

如果想声明一个带有初始值27的`int`，C++98提供两种语法选择：

```cpp
int x1 = 27;
int x2(27);
```

C++11由于也添加了用于支持统一初始化的语法：

```cpp
int x3={27};
int x4{27};
```

四种不同的语法结果相同：变量类型为`int`值为27

但是[Item5](https://cntransgroup.github.io/EffectiveModernCppChinese/2.Auto/item5.html)解释了使用`auto`说明符代替指定类型说明符的好处，所以我们应该很乐意把上面声明中的`int`替换为`auto`，我们会得到这样的代码：

```cpp
auto x1 = 27;
auto x2(27);
auto x3 = {27};
auto x4{27};
```

这些声明都能通过编译，但是他们不像替换之前那样有相同的意义。

前面两个语句确实声明了一个类型为`int`值为27的变量，但是后面两个声明了一个存储一个元素27的 `std::initializer_list<int>`类型的变量。

```cpp
auto x1 = 27;
auto x2(27);
auto x3 = { 27 };
auto x4{ 27 };
```

`auto`类型推导不同于模板类型推导的特殊情况。当用`auto`声明的变量使用花括号进行初始化，`auto`类型推导推出的类型则为`std::initializer_list`。

如果这样的一个类型不能被成功推导（比如花括号里面包含的是不同类型的变量），编译器会拒绝这样的代码：

```cpp
auto x5 = { 1, 2, 3.0 };        //错误！无法推导std::initializer_list<T>中的T
```

`auto`推荐：

`x5`的类型不能被推导，因为`x5`使用花括号的方式进行初始化，`x5`必须被推导为`std::initializer_list`。但是`std::initializer_list`是一个模板。`std::initializer_list<T>`会被某种类型`T`实例化，所以这意味着`T`也会被推导。 推导落入了这里发生的第二种类型推导——模板类型推导的范围。花括号中的值并不是同一种类

对于花括号的处理是`auto`类型推导和模板类型推导唯一不同的地方。当使用`auto`声明的变量使用花括号的语法进行初始化的时候，会推导出`std::initializer_list<T>`的实例化，但是对于模板类型推导这样就行不通：

```cpp
auto x = { 11, 23, 9 };         //x的类型是std::initializer_list<int>

template<typename T>            //带有与x的声明等价的
void f(T param);                //形参声明的模板
f({ 11, 23, 9 });               //错误！不能推导出T
```

如果在模板中指定`T`是`std::initializer_list<T>`而留下未知`T`,模板类型推导就能正常工作：

```cpp
template<typename T>
void f(std::initializer_list<T> initList);
f({ 11, 23, 9 });               //T被推导为int，initList的类型为
//std::initializer_list<int>
```

`auto`类型推导和模板类型推导区别：

`auto`类型推导假定花括号表示`std::initializer_list`而模板类型推导不会这样。

上面是C++11的auto。

C++14允许`auto`用于函数返回值并会被推导(item3)，而且C++14的_lambda_函数也允许在形参声明中使用`auto`。这些情况下`auto`实际上使用**模板类型推导**的那一套规则在工作，而不是`auto`类型推导，所以说下面这样的代码不会通过编译：

```cpp
auto createInitList(){
    return { 1, 2, 3 };         //错误！不能推导{ 1, 2, 3 }的类型
}
```

同样在C++14的lambda函数中这样使用auto也不能通过编译：

```cpp
std::vector<int> v;
...
    auto resetV = [&v](const auto& newValue){ v = newValue; };//C++14
resetV({1,2,3});//错误,不能推导{1,2,3}的类型
```

**总结：**

+ `auto`类型推导通常和模板类型推导相同，但是`auto`类型推导假定花括号初始化代表`std::initializer_list`，而模板类型推导不这样做
+ 在C++14中`auto`允许出现在函数返回值或者_lambda_函数形参中，但是它的工作机制是模板类型推导那一套方案，而不是`auto`类型推导。

<h4 id="48276308">条款三：理解decltype</h4>
前置：**<font style="color:#c21c13;">decltype</font>**

decltype是C++11新增关键字，和auto的功能一样，用来在编译时期进行自动类型推导。引入decltype是因为auto并不适用于所有的自动类型推导场景，在某些特殊情况下auto用起来很不方便，甚至压根无法使用。

```cpp
auto varName=value;
decltype(exp) varName=value;
```

auto根据=右边的初始值推导出变量的类型，decltype根据exp表达式推导出变量的类型，跟=右边的value没有关系。auto要求变量必须初始化，这是因为auto根据变量的初始值来推导变量类型的，如果不初始化，变量的类型也就无法推导。decltype不要求，因此可以写成如下形式

```cpp
decltype(exp) varName;
```

exp只是一个普通的表达式，它可以是任意复杂的形式，但必须保证exp的结果是有类型的，不能是void；如exp为一个返回值为void的函数时，exp的结果也是void类型，此时会导致编译错误。

```cpp
int x = 0;
decltype(x) y = 1;  // y->int
decltype(x + y) z = 0;// z->int
const int& i = x;
decltype(i) j = y;           // j->const int &
const decltype(z) * p = &z;  //*p->const int,p->const int *
decltype(z) * pi = &z;  // *pi->int, pi->int*
decltype(pi)* pp = &pi; // *pp->int*,pp->int**
```

（1）如果exp是一个不被括号()包围的表达式，或者是一个类成员访问表达式，或者是一个单独的变量，decltype(exp)的类型和exp一致

（2）如果exp是函数调用，则decltype(exp)的类型就和函数返回值的类型一致

（3）如果exp是一个左值，或被括号()包围，decltype(exp)的类型就是exp的引用，假设exp的类型为T，则decltype(exp)的类型为T&。

对于规则三的说明：

```cpp
class A{
public：
int x;
}
int main(){
    const A obj;
    decltype(obj.x) a=0;//a的类型为int
    decltype((obj.x)) b=a;//b的类型为int&
    int n=0,m=0;
    decltype(m+n) c=0;//n+m得到一个右值，c的类型为int
    decltype(n=n+m) d=c;//n=n+m得到一个左值，d的类型为int &
    return 0;
}
```

**<font style="color:#c21c13;">前置：尾置返回类型</font>**

函数的返回类型通常需要在函数声明时明确指定。当涉及到模板函数时，有时返回类型可能依赖于模板参数的类型，这很难直接指定返回类型。C++11引入了尾置类型推导，允许在函数参数列表之后使用auto关键字和decltype来指定返回类型。

```cpp
template<typename T, typename U>  
auto add(T t, U u) -> decltype(t + u) {  
    return t + u;  
}
```

---

**item3结论：**

+ `decltype`总是不加修改的产生变量或者表达式的类型。
+ 对于`T`类型的不是单纯的变量名的左值表达式，`decltype`总是产出`T`的引用即`T&`。
+ C++14支持`decltype(auto)`，就像`auto`一样，推导出类型，但是它使用`decltype`的规则进行推导。

相比模板类型推导和`auto`类型推导(参见item1和item2)，`decltype`只是简单的返回名字或者表达式的类型：

```cpp
const int i = 0; //decltype(i)是const int
bool f(const Widget& w); //decltype(w)是const Widget&
//decltype(f)是bool(const Widget&)
struct Point{
int x,y;            //decltype(Point::x)是int
};                      //decltype(Point::y)是int
Widget w;               //decltype(w)是Widget
if (f(w))…             //decltype(f(w))是bool
    template<typename T>            //std::vector的简化版本
class vector{
public:
T& operator[](std::size_t index);
};
vector<int> v;         //decltype(v)是vector<int>
if (v[0]==0)        //decltype(v[0])是int&
```

C++11 `decltype`最主要的用途就是用于声明函数模板，

函数返回类型依赖于形参类型。

假定写一个函数，一个形参为容器，一个形参为索引值，这个函数支持使用方括号的方式访问容器中指定索引值的数据，然后在返回索引操作的结果。函数的返回类型应该和索引操作返回的类型相同。

对一个`T`类型的容器使用`operator[]` 通常会返回一个`T&`对象，比如`std::deque`就是这样。但是`std::vector`有一个例外，对于`std::vector<bool>`，`operator[]`不会返回`bool&`，它会返回一个全新的对象（译注：MSVC的STL实现中返回的是`std::_Vb_reference<std::_Wrap_alloc<std::allocator<unsigned int>>>`对象）（item6）。对一个容器进行`operator[]`操作返回的类型取决于容器本身。

使用`decltype`使得我们很容易去实现它，使用`decltype`计算返回类型，这个模板需要改良.

```cpp
template<typename Container, typename Index>
auto authAndAccess(Container& c, Index i)->decltype(c[i]){
    authenticateUser();
    return c[i];
}
```

函数前面`auto`不做任何的类型推导工作。相反的，auto只暗示使用C++11的**尾置返回类型**语法，即在函数形参列表后面使用一个`->`符号指出函数的返回类型，尾置返回类型的好处是可以在函数返回类型中使用函数形参相关的信息。

在`authAndAccess`函数中,使用`c`和`i`指定返回类型。如果按照传统语法把函数返回类型放在函数名称之前，`c`和`i`就未被声明所以不能使用。

C++11允许自动推导单一语句的_lambda_表达式的返回类型， C++14扩展到允许自动推导所有的_lambda_表达式和函数，甚至它们内含多条语句。

对于`authAndAccess`来说在C++14标准下我们可以忽略尾置返回类型，只留下一个`auto`。使用这种声明形式，auto标示这里会发生类型推导。更准确的说，编译器将会从函数实现中推导出函数的返回类型。

```cpp
template<typename Container, typename Index>    //C++14版本，
auto authAndAccess(Container& c, Index i)       //不那么正确
{
    authenticateUser();
    return c[i];                                //从c[i]中推导返回类型
}
```

[Item2](https://cntransgroup.github.io/EffectiveModernCppChinese/1.DeducingTypes/item2.html)解释了函数返回类型中使用`auto`，编译器实际上是使用的模板类型推导的那套规则。

如果那样的话这里就会有一些问题。正如我们之前讨论的，`operator[]`对于大多数`T`类型的容器会返回一个`T&`，

但是[Item1](https://cntransgroup.github.io/EffectiveModernCppChinese/1.DeducingTypes/item1.html)解释了在模板类型推导期间，表达式的引用性（reference-ness）会被忽略。

```cpp
std::deque<int> d;
…
authAndAccess(d, 5) = 10;     //然后把10赋值
```

`d[5]`本该返回一个`int&`，但是模板类型推导会剥去引用的部分，因此产生了`int`返回类型。函数返回的那个`int`是一个右值，上面的代码尝试把10赋值给右值`int`，C++11禁止这样做，所以代码无法编译。

要想让`authAndAccess`像我们期待的那样工作，需要使用`decltype`类型推导来推导它的返回值。 

C++期望在某些情况下当类型被暗示时需要使用`decltype`类型推导的规则，C++14通过使用`decltype(auto)`说明符使得这成为可能。`decltype(auto)`可能觉得非常的矛盾（到底是`decltype`还是`auto`？），实际上我们可以这样解释它的意义：`auto`说明符表示这个类型将会被推导，`decltype`说明`decltype`的规则将会被用到这个推导过程中。

```cpp
template<typename Container, typename Index>    //C++14版本，
decltype(auto)                                  //可以工作，
authAndAccess(Container& c, Index i)            //但是还需要
{                                               //改良
    authenticateUser();
    return c[i];
}
```

`authAndAccess`将会真正的返回`c[i]`的类型。一般情况下`c[i]`返回`T&`，`authAndAccess`也会返回`T&`，特殊情况下`c[i]`返回一个对象，`authAndAccess`也会返回一个对象。`decltype(auto)`的使用不仅仅局限于函数返回类型，当想对初始化表达式使用`decltype`推导的规则。

```cpp
Widget w;
const Widget& cw = w;
auto myWidget1 = cw;                    //auto类型推导
//myWidget1的类型为Widget
decltype(auto) myWidget2 = cw;          //decltype类型推导
//myWidget2的类型是const Widget&
```

再看看C++14版本的`authAndAccess`声明：

```cpp
template<typename Container, typename Index>
decltype(auto) authAndAccess(Container& c, Index i);
```

容器通过传引用的方式传递非常量左值引用，因为返回一个引用允许用户可以修改容器。但是这意味着不能给这个函数传递右值容器，右值不能被绑定到左值引用上（除非这个左值引用是一个const（，但是这里明显不是）。

公认的向`authAndAccess`传递一个右值是一个小概率事件。一个右值容器，是一个临时对象，通常会在`authAndAccess`调用结束被销毁，这意味着`authAndAccess`返回的引用将会成为一个悬置的引用。但是使用向`authAndAccess`传递一个临时变量也并不是没有意义，有时候用户可能只是想简单的获得临时容器中的一个元素的拷贝，比如这样：

```cpp
std::deque<std::string> makeStringDeque();      //工厂函数
//从makeStringDeque中获得第五个元素的拷贝并返回
auto s = authAndAccess(makeStringDeque(), 5);
```

要想支持这样使用`authAndAccess`就得修改一下当前的声明使得它支持左值和右值。重载是一个不错的选择（一个函数重载声明为左值引用，另一个声明为右值引用），但是我们就不得不维护两个重载函数。另一个方法是使`authAndAccess`的引用可以绑定左值和右值，[Item24](https://cntransgroup.github.io/EffectiveModernCppChinese/5.RRefMovSemPerfForw/item24.html)解释了那正是通用引用能做的，所以我们这里可以使用通用引用进行声明：

```cpp
template<typename Containter, typename Index>   //现在c是通用引用
decltype(auto) authAndAccess(Container&& c, Index i);
```

在这个模板中，我们不知道我们操纵的容器的类型是什么，那意味着我们同样不知道它使用的索引对象（index objects）的类型，对一个未知类型的对象使用传值通常会造成不必要的拷贝，对程序的性能有极大的影响，还会造成对象切片行为（参见[item41](https://cntransgroup.github.io/EffectiveModernCppChinese/8.Tweaks/item41.html))。但是容器索引来说，我们遵照标准模板库对于索引的处理是有理由的（比如`std::string`，`std::vector`和`std::deque`的`operator[]`），所以我们坚持传值调用。需要更新一下模板的实现，让它能听从[Item25](https://cntransgroup.github.io/EffectiveModernCppChinese/5.RRefMovSemPerfForw/item25.html)的告诫应用`std::forward`实现通用引用：

```cpp
template<typename Container, typename Index>    //最终的C++14版本
decltype(auto) authAndAccess(Container&& c, Index i){
    authenticateUser();
    return std::forward<Container>(c)[i];
}
```

```cpp
template<typename Container, typename Index>    //最终的C++11版本
auto authAndAccess(Container&& c, Index i)
->decltype(std::forward<Container>(c)[i]){
    authenticateUser();
    returnstd::forward<Container>(c)[i];
}
```

`decltype`通常会产生你期望的结果，但并不总是这样。在**极少数情况下会出现歧义。**

<h4 id="6a762204">条款四：学会查看类型推导结果</h4>
在编辑代码的时候获得类型推导的结果，在编译期间获得结果，在运行时获得结果。

在IDE中的代码编辑器通常可以显示程序代码中变量，函数，参数的类型，你只需要简单的把鼠标移到它们的上面。另一个获得推导结果的方法是使用编译器出错时提供的错误消息。这些错误消息无形的提到了造成我们编译错误的类型是什么。

使用`printf`的方法（并不是说我推荐你使用`printf`）类型信息要在运行时才会显示出来，但是它提供了一种格式化输出的方法。现在唯一的问题是对于你关心的变量使用一种优雅的文本表示。“这有什么难的，“你这样想，”这正是`typeid`和`std::type_info::name`的价值所在”。

```cpp
std::cout << typeid(x).name() << '\n';  
//显示x和y的类型std::cout << typeid(y).name() << '\n';
```

这种方法对一个对象如`x`或`y`调用`typeid`产生一个`std::type_info`的对象，然后`std::type_info`里面的成员函数`name()`来产生一个C风格的字符串（即一个`const char*`）表示变量的名字。`std::type_info::name`的结果并不总是可信的，就像上面一样，三个编译器对`param`的报告都是错误的。此外，它们在本质上必须是这样的结果，因为`std::type_info::name`规范批准像传值形参一样来对待这些类型。正如[Item1](https://cntransgroup.github.io/EffectiveModernCppChinese/1.DeducingTypes/item1.html)提到的，如果传递的是一个引用，那么引用部分（reference-ness）将被忽略，如果忽略后还具有`const`或者`volatile`，那么常量性`const`ness或者易变性`volatile`ness也会被忽略。那就是为什么`param`的类型`const Widget * const &`会输出为`const Widget *`，首先引用被忽略，然后这个指针自身的常量性`const`ness被忽略，剩下的就是指针指向一个常量对象。IDE编辑器显示的类型信息也不总是可靠的。

Boost TypeIndex库（通常写作**Boost.TypeIndex**）被设计成可以正常运作。这个库不是标准C++的一部分，也不是IDE或者`TD`这样的模板。Boost库（可在[boost.com](http://boost.org/)获得）是跨平台，开源，有良好的开源协议的库，这意味着使用Boost和STL一样具有高度可移植性。

这里是如何使用Boost.TypeIndex得到`f`的类型的代码

```cpp
#include<boost/type_index.hpp>template<typename T>
voidf(const T& param){
    usingstd::cout;
    using boost::typeindex::type_id_with_cvr;
    //显示Tcout << "T =     "
    << type_id_with_cvr<T>().pretty_name()
    << '\n';
    //显示param类型cout << "param = "
    << type_id_with_cvr<decltype(param)>().pretty_name()
    << '\n';
}
```

`boost::typeindex::type_id_with_cvr`获取一个类型实参（我们想获得相应信息的那个类型），它不消除实参的`const`，`volatile`和引用修饰符（因此模板名中有“`with_cvr`”）。结果是一个`boost::typeindex::type_index`对象，它的`pretty_name`成员函数输出一个`std::string`，包含我们能看懂的类型表示。 基于这个`f`的实现版本，再次考虑那个使用`typeid`时获取`param`类型信息出错的调用。这样近乎一致的结果是很不错的，但是请记住IDE，编译器错误诊断或者像Boost.TypeIndex这样的库只是用来帮助你理解编译器推导的类型是什么。它们不能替代对[Item1](https://cntransgroup.github.io/EffectiveModernCppChinese/1.DeducingTypes/item1.html)-[3](https://cntransgroup.github.io/EffectiveModernCppChinese/1.DeducingTypes/item3.html)提到的类型推导的理解。

<h1 id="da7d1d7e">**第二章 **`**auto**`</h1>
`auto`可以自动推导类型，避免手动声明类型可能出错。`auto`类型推导的一些结果是错误的，这时需要手动声明类型。

<h4 id="666fa0e9">条款五：优先考虑auto而非显式类型声明</h4>
_**<font style="color:#c21c13;">前置：闭包类型</font>**_

在C++中，闭包指由lambda表达式生成的对象。闭包对象封装了lambda表达式的函数体、捕获列表以及相关的上下文信息。每个lambda表达式都会产生一个唯一的匿名类，这个类有一个operator()成员函数，用于执行lambda表达式定义的操作。由于这些匿名类是由编译器自动生成的，所以它们的类型是独一无二且不可直接命名的——这就是所谓的“闭包类型”。

闭包类型的特性包括：

（1）唯一性：每次编写不同的lambda表达式，即使其逻辑看起来相同，编译器也会生成不同的匿名类，因此每个lambda都有自己的闭包类型。

（2）不可直接命名：因为闭包类型是由编译器自动生成的，所以程序员无法直接指定或使用这些类型的名称。

（3）可调用性：闭包对象可以通过调用操作符（operator()）来执行lambda表达式定义的行为。

（4）捕获机制：闭包可以捕获其定义环境中的变量，这使得闭包可以访问和修改这些变量。

匿名类包含捕获变量的成员变量。如果变量是按值捕获的，则这些成员变量是复制的；如果变量是按引用捕获的，则这些成员变量是指向外部变量的引用。匿名类的构造函数负责初始化捕获的变量。按值捕获的变量会被复制到成员变量中，按引用捕获的变量会被绑定到成员引用上。匿名类的operator()成员函数实现了lambda表达式的主体。在这个函数中，可以访问和修改捕获的变量。

**<font style="color:#c21c13;">前置：std::function</font>**

`std::function`是一个C++11标准模板库中的一个模板，它泛化了函数指针的概念。与函数指针只能指向函数不同，`std::function`可以指向那些像函数一样能进行调用的东西。当声明函数指针时你必须指定函数类型（即函数签名），同样当你创建`std::function`对象时你也需要提供函数签名，由于它是一个模板所以需要在它的模板参数里面提供。声明一个`std::function`对象`func`使它指向一个可调用对象，比如一个具有这样函数签名的函数。

```cpp
bool(const std::unique_ptr<Widget> &,           //C++11
    const std::unique_ptr<Widget> &)           //std::unique_ptr<Widget>
    //比较函数的签名
    std::function<bool(const std::unique_ptr<Widget> &,
    const std::unique_ptr<Widget> &)>func;
```

std::function内部通常使用一种称为"类型擦除"的技术来实现这一点(这个需要结合设计模式看)。

阅读材料：

💡

类型擦除

[https://davekilian.com/cpp-type-erasure.html](https://davekilian.com/cpp-type-erasure.html)

std::function 包含以下几个部分：

一个小的内部缓冲区：用于存储较小的可调用对象,如简单的函数指针或无捕获的 lambda 表达式。这个缓冲区的大小是固定的，通常几个字节。这个大小不足以存储一个闭包，这个时候`std::function`的构造函数将会在堆上面分配内存来存储，这就造成了使用`std::function`比`auto`声明变量会消耗更多的内存。一个指向外部存储的指针：如果可调用对象太大，无法放入内部缓冲区，std::function 会使用这个指针指向堆上分配的内存。

**auto 可以化简表达式**

`auto`变量从初始化表达式中推导出类型，必须初始化。

一个局部变量使用解引用迭代器的方式初始化：

```cpp
template<typename It>           //对从b到e的所有元素使用
void dwim(It b, It e)           //dwim（“do what I mean”）算法
{
    while (b != e) {
        typename std::iterator_traits<It>::value_type
        currValue = *b;
    }
}
```

`typename std::iterator_traits<It>::value_type`是迭代器指向的元素的值的类型。模板中如果解引用迭代器返回的类型类型，如果手写类型比较麻烦，可以使用auto。

因为使用[Item2](https://cntransgroup.github.io/EffectiveModernCppChinese/1.DeducingTypes/item2.html)所述的`auto`类型推导技术，它甚至能表示一些只有编译器才知道的类型：

```cpp
auto derefUPLess = 
[](const std::unique_ptr<Widget> &p1,  //用于std::unique_ptr
const std::unique_ptr<Widget> &p2)  //指向的Widget类型的
{ return *p1 < *p2; };                 //比较函数
```

如果使用C++14，_lambda_表达式中的形参也可以使用`auto`。

```cpp
auto derefLess =                                //C++14版本
[](const auto& p1,                          //被任何像指针一样的东西
const auto& p2)                          //指向的值的比较函数
{ return *p1 < *p2; };
```

使用`auto`除了可以避免未初始化的无效变量，省略冗长的声明类型，还可以避免type shortcuts的问题

```cpp
std::vector<int> v;
unsigned sz = v.size();
```

`v.size()`的标准返回类型是`std::vector<int>::size_type，std::vector<int>::size_type`实际上被指定为无符号整型，在**Windows 32bit**上`std::vector<int>::size_type`和`unsigned`是一样的大小，但是在**Windows 64bit**上`std::vector<int>::size_type`是64位，`unsigned`是32位。这意味着这段代码在Windows 32-bit上正常工作，但是当把应用程序移植到Windows 64-bit上时就可能会出现一些问题，使用`auto`可以避免这个问题。

```cpp
auto sz =v.size();//sz的类型是std::vector<int>::size_type
```

例2：

```cpp
std::unordered_map<std::string, int> m;
for(conststd::pair<std::string, int>& p : m){
}
```

`std::unordered_map`的_key_是`const`的，所以_hash table_（`std::unordered_map`本质上的东西）中的`std::pair`的类型不是`std::pair<std::string, int>`，而是`std::pair<const std::string, int>`。但那不是在循环中的变量`p`声明的类型。编译器会努力的找到一种方法把`std::pair<const std::string, int>`（即_hash table_中的东西）转换为`std::pair<std::string, int>(p`的声明类型)。通过拷贝`m`中的对象创建一个临时对象，这个临时对象的类型是`p`想绑定到的对象的类型，即`m`中元素的类型，然后把`p`的引用绑定到这个临时对象上。在每个循环迭代结束时，临时对象将会销毁。

使用`auto`可以避免这些不匹配的错误：

```cpp
for(const auto& p : m)
```

如果获取`p`的地址，确实会得到一个指向`m`中元素的指针。在没有`auto`的版本中`p`会指向一个临时变量，这个临时变量在每次迭代完成时会被销毁。

前面这两个例子，应当写`std::vector<int>::size_type`时写了`unsigned`，应当写`std::pair<const std::string, int>`时写了`std::pair<std::string, int>`说明了显式的指定类型可能会导致不想看到的类型转换。

**auto 存储闭包 ****VS function存储闭包**

可以把闭包存放到`std::function`对象中。

```cpp
std::function<bool(const std::unique_ptr<Widget>&,const std::unique_ptr<Widget>&)>derefUPLess=
[](const std::unique_ptr<Widget> &p1,
const std::unique_ptr<Widget> &p2)
{ return *p1 < *p2; };
```

使用std::function存储闭包 语法冗长，需要重复写很多形参类型，而且耗时，并且占用空间，使用`std::function`不如使用`auto`。

(可以使用`std::bind`来生成一个闭包,[Item34](https://cntransgroup.github.io/EffectiveModernCppChinese/6.LambdaExpressions/item34.html):使用_lambda_表达式代替`std::bind`)

muduo使用的bind传递函数指针，但是可以用lambda。

用`auto`声明的变量保存一个闭包类型，因此需要的存储空间与闭包大小相同。实例化`std::function`并声明一个对象这个对象将会有固定的大小。每次创建这样的std::function对象时，都会涉及一次额外的内存分配操作，增加内存使用量，导致内存碎片化。由于涉及到额外的内存分配和间接调用(通过指针)，使用std::function可能会比直接使用auto来存储lambda表达式慢。

当使用auto关键字来声明一个变量以存储lambda表达式时，编译器可以直接为这个lambda表达式创建一个合适的类型，并且这个类型是专门为这个特定的lambda表达式定制的。。lambda表达式的存储不需要额外的堆内存分配，因为它会被内联到调用点或者作为一个局部对象存在。由于没有间接调用的开销，直接使用auto声明的lambda表达式通常可以提供更好的性能。

你优先考虑`auto`而非显式类型声明。然而`auto`也不是完美的。

每个`auto`变量都从初始化表达式中推导类型，有一些表达式的类型和我们期望的大相径庭。

关于在哪些情况下会发生这些问题，以及你可以怎么解决这些问题我们在[Item2](https://cntransgroup.github.io/EffectiveModernCppChinese/1.DeducingTypes/item2.html)和[6](https://cntransgroup.github.io/EffectiveModernCppChinese/2.Auto/item6.html)讨论，所以这里我不再赘述。

**使用auto代替传统类型声明对源码可读性的影响**

如果使用显式类型声明比`auto`更清晰更易维护，那就不必再坚持使用`auto`。一些开发者也担心使用`auto`就不能瞥一眼源代码便知道对象的类型，然而，IDE扛起了部分担子（也考虑到了[Item4](https://cntransgroup.github.io/EffectiveModernCppChinese/1.DeducingTypes/item4.html)中提到的IDE类型显示问题），在很多情况下，少量显示一个对象的类型对于知道对象的确切类型是有帮助的，这通常已经足够了。举个例子，要想知道一个对象是容器还是计数器还是智能指针，不需要知道它的确切类型。一个适当的变量名称就能告诉我们大量的抽象类型信息。

+ `auto`变量必须初始化，通常它可以避免一些移植性和效率性的问题，也使得重构更方便，还能让你少打几个字。而且优先考虑使用auto存储lambda表达式产生的闭包。
+ 正如[Item2](https://cntransgroup.github.io/EffectiveModernCppChinese/1.DeducingTypes/item2.html)和[6](https://cntransgroup.github.io/EffectiveModernCppChinese/2.Auto/item6.html)讨论的，`auto`类型的变量可能会踩到一些陷阱。

<h4 id="28836f90">条款六：auto推导若非己愿，使用显式类型初始化惯用法</h4>
**<font style="color:#c21c13;">前置：vector<bool></font>**

首先vector<bool>并不是一个通常意义上的vector容器。早在C++98的时候，就有vector< bool>，但是因为当时为了考虑到节省空间，所以vector<bool>里面不是一个Byte储存的，它是一个bit一个bit储存的。

因为没有直接去给一个bit来操作，所以用operator[]的时候，正常容器返回的应该是一个对应元素的引用，但是对于vector< bool>实际上访问的是一个"proxy reference"而不是一个"true reference",返回的是std::vector< bool>:reference类型的对象。

一般情况:

```cpp
vector c {false, true, false, true, false};
bool b=c[0];
auto d=c[0];
```

b的初始化暗含一个隐式的类型转换。对于d，类型并不是bool，而是一个vector<bool>中的一个内部类。此时如果修改d的值，c中的值也会跟着修改。如果c被销毁，d就会变成一个悬垂指针，再对d操作就属于未定义行为。所以对于容器一些基本的操作它并不能满足，诸如取地址给指针初始化操作，没有办法给单一bit来取地址或者引用。

```cpp
vector c{ false, true, false, true, false };
bool &tmp = c[0]; //错误,不能编译,对于引用来说,因为c[0]不是一个左值。
bool *p = &c[0]; //错误,不能编译,因为无法将一个临时量地址给绑定到指针。
```

std::vector<bool>的operator[] 实际上返回的是一个 std::vector<bool>::reference 类型的对象，这是一个代理类，它模仿了 bool& 的行为。这个代理类提供了对单个位的操作能力，包括读取和设置位的值。

std::vector<bool>::reference 是 std::vector<bool> 特化中使用的一个代理类。由于 std::vector<bool> 以位的形式存储布尔值，导致了与普通 std::vector<T> 行为上的差异。

std::vector<bool> 不能像其他 std::vector<T> 那样直接返回 bool& 类型的引用，因为 C++ 语言不允许对单独的位进行引用。为了解决这个问题，std::vector<bool> 使用了一个代理类 std::vector<bool>::reference 来模拟 bool& 的行为。

std::vector<bool>::reference 包含一个指向内部位数组的指针：这个指针指向存储布尔值的整数。

一个表示位位置的索引：这个索引表示具体的位在整数中的位置。

```cpp
#include <iostream>
#include <vector>
int main() {
    std::vector<bool> v = {true, false, true, false};
    // 获取第 2 个元素的引用
    std::vector<bool>::reference ref = v[2];
    // 打印第 2 个元素的值
    std::cout << "v[2] = " << ref << std::endl;  // 输出: v[2] = 1
    // 修改第 2 个元素的值
    ref = false;
    // 再次打印第 2 个元素的值
    std::cout << "v[2] = " << v[2] << std::endl;  // 输出: v[2] = 0
    return 0;
}
```

**<font style="color:#c21c13;">前置：代理类</font>**

待整理

**使用 auto 时的问题**

当使用 auto 来声明变量并初始化时，编译器会根据初始化表达式的类型来推断变量的类型。对于 std::vector<bool> 的 operator[]，auto 会推断出 std::vector<bool>::reference 而不是 bool。

这可能导致问题，特别是当 std::vector<bool> 是一个临时对象时，std::vector<bool>::reference 中的指针会变成悬空指针，导致未定义行为。

调用features将返回一个std::vector<bool>临时对象，这个对象没有名字，为了方便我们的讨论，我这里叫他temp。operator[]在temp上调用，它返回的std::vector<bool>::reference包含一个指向存着这些bits的一个数据结构中的一个word的指针(temp管理这些bits)，还有相应于第5个bit的偏移。highPriority是这个std::vector<bool>::reference的拷贝，所以highPriority也包含一个指针，指向temp中的这个word，加上相应于第5个bit的偏移。在这个语句结束的时候temp将会被销毁，因为它是一个临时变量。因此highPriority包含一个悬置的指针，如果用于processWidget调用中将会造成未定义行为。

```cpp
processWidget(w, highPriority); //未定义行为.highPriority包含一个悬置指针.
```

下面这个没看懂。

```cpp
// 模拟 std::vector<bool>::reference 类
class vector_bool_reference {
friend class vector_bool;
//私有成员
unsigned char* ptr;//指向存储位的整数的指针
unsigned char index;//位的位置
//私有构造函数
vector_bool_reference(unsigned char* p, unsigned char i) noexcept:ptr(p),index(i) {}
public:
//析构函数
~vector_bool_reference() = default;
//转换为 bool
operator bool() const noexcept {
    return (*ptr & (1 << index)) != 0;
}
//从 bool 赋值
vector_bool_reference& operator=(const bool x) noexcept {
    if (x) {
        *ptr |= (1 << index);
    } else {
        *ptr &= ~(1 << index);
    }
    return *this;
}
//从另一个 reference 赋值
vector_bool_reference& operator=(const vector_bool_reference& x) noexcept {
    *ptr = ((*ptr & ~(1 << index)) | (static_cast<bool>(x) << index));
    return *this;
}
// 翻转位的值
void flip() noexcept {
    *ptr ^= (1 << index);
}
};
class vector_bool {
unsigned char* data;  // 指向存储位的数组
size_t size_;         // 位的数量
public:
vector_bool(size_t n) : data(new unsigned char[(n + 7) / 8]), size_(n) {
    for (size_t i = 0; i < (n + 7) / 8; ++i) {
        data[i] = 0;
    }
}
~vector_bool() {
    delete[] data;
}
//获取位的引用
vector_bool_reference operator[](size_t pos) {
    return vector_bool_reference(&data[pos / 8], pos % 8);
}
```

为了避免这些问题，建议显式地将结果转换为 bool：

```cpp
bool highPriority = features(w)[5];
```

或者使用 static_cast 显式转换：

```cpp
bool highPriority = static_cast<bool>(features(w)[5]);
```

这样可以确保 highPriority 是一个真正的 bool 变量，而不是一个代理类实例。一些代理类被设计于用以对客户可见。比如std::shared_ptr和std::unique_ptr。其他的代理类则或多或少不可见，比如std::vector<bool>::reference就是不可见代理类的一个例子，还有它在std::bitset的胞弟std::bitset::reference。在不可见代理类里一些C++库也是用了表达式模板(expression templates)。

这些库通常被用于提高数值运算的效率。给出一个矩阵类Matrix和矩阵对象m1，m2，m3，m4，举个例子，这个表达式

```cpp
Matrix sum = m1 + m2 + m3 + m4;
```

可以使计算更加高效，只需要使让operator+返回一个代理类代理结果而不是返回结果本身。也就是说，对两个Matrix对象使用operator+将会返回如Sum<Matrix, Matrix>这样的代理类作为结果而不是直接返回一个Matrix对象。在std::vector<bool>::reference和bool中存在一个隐式转换，同样对于Matrix来说也可以存在一个隐式转换允许Matrix的代理类转换为Matrix，这让表达式等号“=”右边能产生代理对象来初始化sum。（这个对象应当编码整个初始化表达式，即类似于Sum<Sum<Sum<Matrix, Matrix>, Matrix>, Matrix>的东西。客户应该避免看到这个实际的类型。）

作为一个通则，不可见的代理类通常不适用于auto。这样类型的对象的生命期通常不会设计为能活过一条语句，所以创建那样的对象你基本上就走向了违反程序库设计基本假设的道路。std::vector<bool>::reference就是这种情况，违反这个基本假设将导致未定义行为。

因此想避开这种形式的代码：

```cpp
auto someVar = expression of "invisible" proxy class type;
```

显式类型初始器惯用法使用auto声明一个变量，然后对表达式强制类型转换（cast）得出你期望的推导结果。例：

```cpp
auto highPriority = static_cast<bool>(features(w)[5]);
```

features(w)[5]还是返回一个std::vector<bool>::reference对象，就像之前那样，但是这个转型使得表达式类型为bool，然后auto才被用于推导highPriority。在运行时，对std::vector<bool>::operator[]返回的std::vector<bool>::reference执行它支持的向bool的转型，在这个过程中指向std::vector<bool>的指针已经被解引用。这就避开了我们之前的未定义行为。

对于Matrix来说，显式类型初始器惯用法是这样的：

```cpp
auto sum = static_cast<Matrix>(m1 + m2 + m3 + m4);
```

<h1 id="a512c308">第三章 移步现代C++</h1>
<h4 id="933fbf30">条款七：区别使用()和{}创建对象</h4>
**<font style="color:#c21c13;">前置：最令人烦恼的解析(Most Vexing Parse，MVP)</font>**

最令人烦恼的解析(Most Vexing Parse，MVP)C++编程语言语法歧义的解析。C++的语法规则无法区分创建对象参数和指定函数类型这两种操作。在这种情况下，编译器被要求将相关行解释为函数类型的声明。

**C风格的转换**

一个简单的例子出现在当意图使用函数式转换来初始化变量时：

```cpp
void f(double my_dbl) {
    int i(int(my_dbl));
}
```

上述第2行代码是模糊的。一个可能的解释是声明一个变量i，其初始值是通过将my_dbl转换为int类型得到的。C允许在函数参数声明周围有多余的括号；在这种情况下，i的声明实际上是一个函数声明，等价于下面的声明。

```cpp
int i(int my_dbl);//声明一个名为i的函数,函数接受一个整数并返回一个整数.
```

另一个例子：

```cpp
struct Timer{};
struct TimeKeeper {
explicit TimeKeeper(Timer t);
int get_time();
};
int main() {
    TimeKeeper time_keeper(Timer());
    return time_keeper.get_time();
}
```

这行代码

TimeKeeper time_keeper(Timer())；

是模糊的

（1）定义一个time_keeper变量，类型为TimeKeeper，使用匿名的Timer实例进行初始化。

或者

（2）声明一个名为time_keeper的函数，该函数返回一个TimeKeeper类型的对象，并且有一个单一的（未命名的）参数，该参数的类型是一个（指向）不接受任何输入并返回Timer对象的函数。

**解决方案**

在变量声明的例子中，自C++11以来，首选的方法是使用统一（花括号）初始化。这还允许有限地省略类型名称：

```cpp
// 以下任一方法均有效:
TimeKeeper time_keeper(Timer{})；
TimeKeeper time_keeper{Timer()}；
TimeKeeper time_keeper{Timer{}}；
TimeKeeper time_keeper(  {})；
TimeKeeper time_keeper{  {}}；
```

C++11对象初始化的语法有多种，初始化值要用圆括号()或者花括号{}括起来，或者放到等号"="的右边。

```cpp
int x(0);//使用圆括号初始化
int y = 0;//使用"="初始化
int z {0}; //使用花括号初始化
```

可以使用"="和花括号的组合,C++通常把它视作和只有花括号一样。

```cpp
int z = { 0 };          //使用"="和花括号
```

对于用户定义的类型而言，赋值运算符和初始化涉及不同的函数调用：

```cpp
Widget w1;//调用默认构造函数
Widget w2 = w1;//不是赋值运算,调用拷贝构造函数
w1 = w2;//是赋值运算,调用拷贝赋值运算符（copy operator=）
```

C++11使用统一初始化来整合这些混乱且不适于所有情景的初始化语法，统一初始化是指在任何涉及初始化的地方都使用单一的初始化语法。

它基于花括号，出于这个原因更喜欢称之为"括号初始化"。统一初始化是一个概念上的东西，而括号初始化是一个具体语法结构。使用花括号，创建并指定一个容器的初始元素变得很容易：

```cpp
std::vector<int> v{ 1, 3, 5 };  //v初始内容为1,3,5
```

括号初始化也能被用于为非静态数据成员指定默认初始值。C++11允许"="初始化不加花括号也拥有这种能力。另一方面，不可拷贝的对象（例如`std::atomic` item40）可以使用花括号初始化或者圆括号初始化，但是不能使用"="初始化：

```cpp
std::atomic<int> ai1{ 0 }; //没问题
std::atomic<int> ai2(0); //没问题std::atomic<int> ai3 = 0;       //错误！
```

括号表达式还有一个少见的特性，它不允许内置类型间隐式的变窄转换（_narrowing conversion_）。如果一个使用了括号初始化的表达式的值，不能保证由被初始化的对象的类型来表示，代码不会通过编译：

```cpp
double x, y, z;
int sum1{x+y+z};//错误.double的和可能不能表示为int
```

使用圆括号和"="的初始化不检查是否转换为变窄转换，由于历史遗留问题它们必须要兼容老旧代码：

```cpp
int sum2(x + y +z); //可以(表达式的值被截为int)
int sum3 = x + y + z; //同上
```

总结上面内容：

能用于各种不同的上下文，防止了隐式的变窄转换，防止C++解析问题。

括号初始化的缺点

**（1）auto**

这些行为使得括号初始化、`std::initializer_list`和构造函数参与重载决议时本来就不清不楚的暧昧关系进一步混乱。把它们放到一起会让看起来应该左转的代码右转。

[Item2](https://cntransgroup.github.io/EffectiveModernCppChinese/1.DeducingTypes/item2.html)解释了当`auto`声明的变量使用花括号初始化，变量类型会被推导为`std::initializer_list`，但是使用相同内容的其他初始化方式会产生更符合直觉的结果。

**（2）构造函数调用**

在构造函数调用中，只要不包含`std::initializer_list`形参，那么花括号初始化和圆括号初始化都会产生一样的结果：

```cpp
classWidget {public:  
    Widget(int i, bool b);  //构造函数未声明
Widget(int i, double d); //std::initializer_list这个形参 
    …
};
Widget w1(10, true); //调用第一个构造函数
Widget w2{10, true}; //也调用第一个构造函数Widget w3(10, 5.0);        
//调用第二个构造函数
Widget w4{10, 5.0}; //也调用第二个构造函数
```

然而，如果有一个或者多个构造函数的声明包含一个`std::initializer_list`形参，那么使用括号初始化语法的调用更倾向于选择带`std::initializer_list`的那个构造函数。如果编译器遇到一个括号初始化并且有一个带std::initializer_list的构造函数，那么它一定会选择该构造函数。如果上面的`Widget`类有一个`std::initializer_list<long double>`作为参数的构造函数，就像这样：

```cpp
class Widget { 
public:
Widget(int i, bool b);//同上
Widget(int i, double d);//同上
Widget(std::initializer_list<long double> il);//新添加的
};
```

`w2`和`w4`将会使用新添加的构造函数，即使另一个非`std::initializer_list`构造函数和实参更匹配：

```cpp
Widget w1(10, true);    //使用圆括号初始化，同之前一样
//调用第一个构造函数

Widget w2{10, true};    //使用花括号初始化，但是现在
//调用带std::initializer_list的构造函数
//(10 和 true 转化为long double)

Widget w3(10, 5.0);     //使用圆括号初始化，同之前一样
//调用第二个构造函数 

Widget w4{10, 5.0};     //使用花括号初始化，但是现在
//调用带std::initializer_list的构造函数
//(10 和 5.0 转化为long double)
```

甚至普通构造函数和移动构造函数都会被带`std::initializer_list`的构造函数劫持：

```cpp
class Widget { 
public:  
Widget(int i, bool b);                              //同之前一样
Widget(int i, double d);                            //同之前一样
Widget(std::initializer_list<long double> il);      //同之前一样
operator float() const;                             //转换为float
…
};
Widget w5(w4);//使用圆括号，调用拷贝构造函数
Widget w6{w4};//使用花括号，调用std::initializer_list构造
//函数(w4转换为float，float转换为double)

Widget w7(std::move(w4));//使用圆括号，调用移动构造函数
Widget w8{std::move(w4)};//使用花括号，调用std::initializer_list构造
//函数(与w6相同原因)
```

`std::vector`作为受众之一会直接受到影响。

`std::vector`有一个非`std::initializer_list`构造函数允许你去指定容器的初始大小，以及使用一个值填满你的容器。但它也有一个`std::initializer_list`构造函数允许你使用花括号里面的值初始化容器。如果你创建一个数值类型的`std::vector`（比如`std::vector<int>`），然后你传递两个实参，把这两个实参放到圆括号和放到花括号不同。

```cpp
std::vector<int> v1(10, 20);    //使用非std::initializer_list构造函数
//创建一个包含10个元素的std::vector，
//所有的元素的值都是20

std::vector<int> v2{10, 20};    //使用std::initializer_list构造函数
//创建包含两个元素的std::vector，
//元素的值为10和20
```

**第一，**作为一个类库作者，需要意识到如果一堆重载的构造函数中有一个或者多个含有`initializer_list`形参，用户代码如果使用了括号初始化，可能只会看到你`initializer_list`版本的重载的构造函数。因此，最好把构造函数设计为不管用户是使用圆括号还是使用花括号进行初始化都不会有什么影响。

**第二，**作为一个类库使用者，须认真在花括号和圆括号之间选择一个来创建对象。大多数开发者都使用其中一种作为默认情况，只有当他们不能使用这种的时候才会考虑另一种。默认使用花括号初始化的开发者主要被适用面广、禁止变窄转换、免疫C++最令人头疼的解析这些优点所吸引。这些开发者知道在一些情况下（比如给定一个容器大小和一个初始值创建`std::vector`）要使用圆括号。默认使用圆括号初始化的开发者主要被C++98语法一致性、避免`std::initializer_list`自动类型推导、避免不会不经意间调用`std::initializer_list`构造函数这些优点所吸引。**建议是选择一种并坚持使用它。**

**总结**

+ 花括号初始化是最广泛使用的初始化语法，它防止变窄转换，并且对于C++最令人头疼的解析有天生的免疫性
+ 在构造函数重载决议中，编译器会尽最大努力将括号初始化与`std::initializer_list`参数匹配，即便其他构造函数看起来是更好的选择。
+ 对于数值类型的`std::vector`来说使用花括号初始化和圆括号初始化会不同。
+ 在模板类选择使用圆括号初始化或使用花括号初始化创建对象是一个挑战。

<h4 id="65f0e318">条款八：优先考虑nullptr而非0和NULL</h4>
**<font style="color:#c21c13;">0 和 NULL </font>**

0 的类型：在 C++ 中，字面值 0 默认是一个 int 类型，而不是指针类型。只有在上下文明确需要指针时，编译器才会不情愿地将其解释为指针。

NULL 的实现：NULL 通常被定义为 0 或 0L（long 类型的 0）。尽管 NULL 可以被定义为其他整型类型，但它的主要问题是它本质上仍然是一个整型，而不是指针类型。

在 C++98 中，如果同时重载了指针和整型的函数，传递 0 或 NULL 会调用整型版本的函数，而不是指针版本的函数。这是因为 0 和 NULL 被解析为整型，而不是指针。

**<font style="color:#c21c13;">nullptr 的优势</font>**

nullptr 的类型：nullptr 是 C++11 引入的一个关键字，其类型是 std::nullptr_t。std::nullptr_t 可以隐式转换为任何指针类型，但它不是整型。使用 nullptr 调用函数时，编译器会正确地选择指针版本的重载函数，而不是整型版本的函数。使用 nullptr 可以使代码更加清晰，尤其是当与 auto 声明的变量一起使用时。例如，if (result == nullptr) 明确表示 result 是一个指针类型。在模板中，传递 0 或 NULL 会导致类型推导出整型，而不是指针类型。这会导致类型错误，因为模板实例化时会尝试将整型传递给期望指针的函数。使用 nullptr 可以避免这些问题，因为 std::nullptr_t 可以隐式转换为任何指针类型，而不会导致类型错误。在 C++11 及以上版本中,优先使用nullptr 而不是 0 或 NULL。尽量避免同时重载指针和整型的函数，以减少潜在的解析问题和代码复杂性。

```cpp
void f(int);
void f(bool);
void f(void*);
f(0);//调用f(int)
f(NULL);//通常调用f(int)，可能有二义性
f(nullptr);//调用 f(void*)
```

模板示例

```cpp
#include <memory>
#include <mutex>
template<typename FuncType, typename MuxType, typename PtrType>
auto lockAndCall
(FuncType func, MuxType& mutex, PtrType ptr) -> decltype(func(ptr)) {
    std::lock_guard<MuxType> g(mutex);
    return func(ptr);
}
template<typename FuncType, typename MuxType, typename PtrType>
decltype(auto) lockAndCall(FuncType func, MuxType& mutex, PtrType ptr) {
    std::lock_guard<MuxType> g(mutex);
    return func(ptr);
}
int f1(std::shared_ptr<int>);
double f2(std::unique_ptr<int>);
bool f3(int*);
std::mutex m1, m2, m3;
int main() {
    auto result1 = lockAndCall(f1, m1, 0);         // 错误！
    auto result2 = lockAndCall(f2, m2, NULL);      // 错误！
    auto result3 = lockAndCall(f3, m3, nullptr);   // 没问题
}
```

结论

使用 nullptr 可以避免许多与 0 和 NULL 相关的类型解析问题，使代码更加清晰和健壮。在现代 C++ 编程中，推荐优先使用 nullptr。

看到这__-----_x

<h4 id="e9187f0e">条款九：优先考虑别名声明而非typedef</h4>
**<font style="color:#c21c13;">前置：Type Traits</font>**

type_traits 是 STL的一个头文件，定义了一系列模板类，这些模板类在编译期获取某一参数、某一变量、某一个类等的类型信息，用于静态检查。通过使用 type_traits，可以在编译时就获得关于类型的详细信息，从而可以在不运行程序的情况下进行类型相关的优化和检查。

分类：

**辅助基类**

如std::integral_constant 以及其特化 true_type 和 false_type，这些类用于创建编译器常量，同时也是类型萃取类模板的基类。

**类型萃取类模板**

用于在编译期以常量的形式获取类型特征。例如，std::is_integral 用于检查一个类型是否为整数类型，std::is_floating_point用于检查一个类型是否为浮点类型，std::is_base_of用于检查一个类型是否是另一个类型的基类等。

**类型转换类模板**

通过执行特定操作从已有类型获得新类型。例如，std::add_const 用于给类型添加 const 限定符，std::remove_volatile 用于移除类型的 volatile 限定符等。

C++11 给出 type_traits 之前，对于类型处理有很多非常不方便的地方。C++是一种静态类型语言，在编译时，每个变量和表达式都必须有明确的类型。C++ 并没有提供一个直接的机制来获取和操作这些类型信息。在没有 type_traits 的情况下，程序员通常需要手动跟踪和处理类型信息。C++的模板元编程是一种在编译期进行计算和类型操作的技术。然而，由于缺乏 type_traits 这样的工具，模板元编程变得异常复杂和难以维护。程序员需要手动编写大量的模板特化和递归代码，以处理各种类型情况。这不仅增加了开发难度，也降低了代码的可读性和可维护性。在泛型编程中，通常需要编写能够处理多种类型的代码。然而，在没有 type_traits 的情况下，很难在编译期对类型进行严格的检查和约束。这可能导致类型不安全的代码，例如错误将一个浮点数传递给期望整数的函数。此外由于缺乏类型萃取和转换的工具，泛型编程的灵活性也会受到限制。在某些情况下，需要根据类型信息来优化代码的性能。例如，对于某些特定的类型，希望使用特定的算法或数据结构。在没有 type_traits 的情况下，这种优化变得非常困难。我们需要编写复杂的条件编译代码，或者使用运行时类型信息(RTTI)来动态地选择实现。这些方法要么增加了编译的复杂性，要么引入了额外的运行时开销。type_traits 提供一套丰富的工具，使得程序员可以在编译期获取和操作类型信息，从而简化了模板元编程，提高了类型安全性，并使得性能优化变得更加容易。

type_traits为 STL 中的算法和容器提供了类型相关的支持，例如，STL 中的算法通过迭代器存取容器内容，而 type_traits 可以协助算法根据迭代器类型或容器类型选择不同的策略。

**<font style="color:#c21c13;">type_traits 核心类型特性</font>**

**std::is_same**

std::is_same 是一个模板类，用于检查两个类型是否相同。其定义如下：

```cpp
template< class T, class U >  
struct is_same;
```

当T和U指名同一类型（考虑 const 和 volatile 限定）时，is_same<T, U>::value 为 true，否则为 false。

```cpp
#include <type_traits>   
std::cout<<"int and int are"<<std::is_same<int,int>::value <<'\n';//输出 true  
std::cout<<"int and float are"<<std::is_same<int,float>::value <<'\n';//输出 false
```

**std::is_integral**

std::is_integral 用于检查一个类型是否为整数类型。

```cpp
template<class T>  
struct is_integral;
```

当 T 为整数类型时，is_integral<T>::value 为 true，否则为 false。

```cpp
#include <type_traits>   
std::cout << "int is " << std::is_integral<int>::value << '\n';
// 输出 true
std::cout << "double is " << std::is_integral<double>::value << '\n';
//输出 false
```

**std::is_floating_point**

std::is_floating_point 用于检查一个类型是否为浮点类型。

```cpp
template<class T>  
struct is_floating_point;
```

当 T 为 float、double 或 long double（包括任何 cv 限定变体）时，is_floating_point<T>::value 为 true，否则为 false。

```cpp
#include <type_traits>   
std::cout << "float is " << std::is_floating_point<float>::value << '\n'; // 输出 true  
std::cout << "int is " << std::is_floating_point<int>::value << '\n'; // 输出 false
```

**std::is_array**

std::is_array 用于检查一个类型是否为数组类型。

```cpp
template< class T >  struct is_array;
```

当 T 为数组类型时，is_array<T>::value 为 true，否则为 false。

```cpp
#include <type_traits>   
std::cout << "int[5] is " << std::is_array<int[5]>::value << '\n';// 输出 true  
std::cout << "int is " << std::is_array<int>::value << '\n';//输出 false
```

**std::is_pointer**

std::is_pointer 用于检查一个类型是否为指针类型。

```cpp
template< class T >  
struct is_pointer;
```

当 T 为指针类型时，is_pointer<T>::value 为 true，否则为 false。

样例:

```cpp
#include <type_traits>  
std::cout << "int* is"<<std::is_pointer<int*>::value<<'\n';// 输出 true  
std::cout << "int is"<<std::is_pointer<int>::value<<'\n';// 输出 false
```

**std::is_reference**

std::is_reference用于检查一个类型是否为引用类型。

```cpp
template<class T>struct is_reference;
```

当 T 为引用类型时，is_reference<T>::value 为 true，否则为 false。

```cpp
#include <type_traits>   
std::cout << "int& is " << std::is_reference<int&>::value << '\n'; // 输出 true  
std::cout << "int is " << std::is_reference<int>::value << '\n'; // 输出 false
```

**std::is_void**

std::is_void 用于检查一个类型是否为 void 类型。其定义如下：

```cpp
template< class T >  
struct is_void;
```

当 T 为 void 类型时，is_void<T>::value 为 true，否则为 false。

```cpp
#include <type_traits>  
std::cout << "void is " << std::is_void<void>::value << '\n'; // 输出 true  
std::cout << "int is " << std::is_void<int>::value << '\n'; // 输出 false
```

**std::is_const**

std::is_const 是一个模板类，用于检查一个类型是否为常量类型。

```cpp
template<class T>  
struct is_const;
```

当 T 为常量类型时，is_const<T>::value 为 true，否则为 false。

```cpp
#include <type_traits>   
std::cout << "const int is " << std::is_const<const int>::value << '\n'// 输出 true  
    std::cout << "int is " << std::is_const<int>::value << '\n'; // 输出 false
```

**std::is_volatile**

std::is_volatile 用于检查一个类型是否为volatile类型。

```cpp
template< class T >  
struct is_volatile;
```

当 T 为volatile类型时，is_volatile<T>::value为 true,否则为 false。

```cpp
#include <type_traits>
std::cout << "volatile int is "<< std::is_volatile<volatile int>::value << '\n'; // 输出 true  
std::cout << "int is"<< std::is_volatile<int>::value << '\n'; // 输出 false
```

除了上面介绍的类型，type_traits 还有如 std::is_signed （检查类型是否有符号）、std::is_unsigned （检查类型是否无符号）、std::is_arithmetic （ 检查类型是否为算术类型（整数或浮点数））等基础核心类型。

**使用核心类型特性实现泛型编程中的条件编译**

在 C++11 中，可以使用模板特化或者 std::enable_if 来实现类似条件编译的效果。使用 std::is_same 和 std::is_integral 以及 std::enable_if 来实现泛型编程中条件编译的实例：

```cpp
#include<type_traits>  
// 泛型函数模板,用于非整数类型  
template<typename T, typename = void>
void print_type_info
(T value, typename std::enable_if<!std::is_integral<T>::value>::type* = nullptr) {
    std::cout << "Value is of a non-integral type." << std::endl;
}

//对 int 类型特化的版本  
template<typename T>
void print_type_info(T value, typename std::enable_if<std::is_same<T, int>::value>::type* = nullptr) {
    std::cout << "Value is of type int with value: " << value << std::endl;
}

//对整数类型特化的版本，但排除 int
template<typename T>
void print_type_info(T value, typename std::enable_if<std::is_integral<T>::value && !std::is_same<T, int>::value>::type* = nullptr) {
    std::cout << "Value is of an integral type (non-int) with value: " << value << std::endl;
}

int main() { 
    //调用 print_type_info 模板函数，传入不同类型的值   
    print_type_info(12); // 调用 int 特化的版本  
    print_type_info(3.14f); // 调用非整数类型特化版本  
    print_type_info(static_cast<long long>(1234567890123456789)); // 调用整数类型特化的版本（排除 int）  
    return 0;
}
```

非特化的版本，它接受非整数类型的参数 T，并打印一个通用消息。对int 类型特化的版本，它只接受 int 类型的参数，并打印 int 类型的特定消息。对整数类型(除了int)特化的版本，它接受任何整数类型（除了 int）的参数，并打印整数类型的特定消息。

std::enable_if 用于在编译时启用或禁用模板函数的不同特化版本。std::enable_if 的第二个模板参数是一个类型，当该类型存在时，模板会被启用；当该类型为 void 时，模板会被禁用。std::is_same<T, int>::value 和 std::is_integral<T>::value 用于在编译时检查类型。

---

**问题背景**

C++编程中，使用复杂的类型(如 STL 容器和智能指针)时,经常需要定义类型别名以简化代码。C++98 提供了 typedef，而 C++11 引入了别名声明(using)。

typedef 是 C++98 中定义类型别名的方式。

```cpp
typedef complex_type alias_name;
```

_简单类型别名_

```cpp
typedef std::vector<int> IntVector;
```

_复杂类型别名_

```cpp
typedef std::unique_ptr<std::unordered_map<std::string, std::string>> UPtrMapSS;
```

限制：typedef 不能直接用于模板化类型别名。

_使用 typedef 定义模板化类型别名_

```cpp
template<typename T>
struct MyAllocList {
typedef std::list<T, MyAlloc<T>> type;
};
MyAllocList<Widget>::type  lw;
```

_**在模板内使用类型别名**_

```cpp
template<typename T>
class Widget {
private:
typename MyAllocList<T>::type list;  // 需要 typename
};
```

在模板内使用 typedef 定义的类型别名时，需要在前面加上 typename，因为编译器不能确定 ::type 是否是一个类型。

_**函数指针的别名**_

```cpp
typedef void (*FP)(int, const std::string&);
```

C++11 引入的 using 关键字可以定义类型别名。

```cpp
using alias_name = complex_type;
```

支持模板化类型别名（别名模板）。在模板中使用时不需要 typename 和 ::type 后缀。

定义类型别名

简单类型别名

```cpp
using IntVector = std::vector<int>;
```

复杂类型别名

```cpp
using UPtrMapSS = std::unique_ptr<std::unordered_map<std::string, std::string>>;
```

使用 using 定义模板化类型别名

```cpp
template<typename T>
using MyAllocList = std::list<T, MyAlloc<T>>;
MyAllocList<Widget> lw;
```

在模板内使用类型别名

```cpp
template<typename T>
class Widget {
private:
MyAllocList<T> list;  // 不需要 typename
};
Widget<int> w;
```

别名声明版本：在模板内使用 using 定义的类型别名时，不需要 typename，因为编译器知道MyAllocList<T> 是一个类型别名。

函数指针的别名：

```cpp
using FP = void (*)(int, const std::string&)；
```

Type Traits

使用 typedef 实现,需要 typename 和 ::type 后缀。

```cpp
//去除常量性
typedef std::remove_const<const int>::type IntType;
//去除引用
typedef std::remove_reference<int&>::type IntRefRemovedType;
//添加左值引用
typedef std::add_lvalue_reference<int>::type IntLValueRefType;
```

**using**提供了别名声明版本，更加简洁。

```cpp
// 去除常量性
using IntType = std::remove_const_t<int>;
// 去除引用
using IntRefRemovedType = std::remove_reference_t<int&>;
// 添加左值引用
using IntLValueRefType = std::add_lvalue_reference_t<int>;
```

优先使用别名声明（using）：支持模板化类型别名。在模板内使用时更加简洁，不需要 typename 和 ::type 后缀。

<h4 id="4a93e8c3">条款十：优先考虑限域enum而非未限域enum</h4>
**<font style="color:#c21c13;">前置：std::tuple</font>**

C++11引入了一个模板类型std::tuple(元组)。元组是一个固定大小的不同类型异质值的集合，也即它可以同时存放不同类型的数据。类似于python中用小括号表示的元组类型。C++已有的std::pair类型类似于一个二元组，可看作是std::tuple的一个特例，std::tuple也可看作是std::pair的泛化。std::pair的长度限制为2，而std::tuple的元素个数为0~任意个。

**元组的使用**

如果希望将一些数据组合成单一对象，但又不想麻烦地定义一个新数据结构来表示这些数据时，可以使用std::tuple。

可以将std::tuple看作一个“快速而随意”的数据结构，把它当作一个通用的结构体使用，但又不需要创建和获取结构体的特征，使得程序更加简洁。

创建一个std::tuple对象时，可以使用tuple的默认构造函数，它会对每个成员进行值初始化；也可以为每个成员提供一个初始值，此时的构造函数是explicit的，因此必须使用直接初始化方法。

使用get获取元组成员。为了使用get，必须指定一个显式模板实参，指出想要访问第几个成员。

传递给get一个tuple对象，返回指定成员的引用。get尖括号中的值必须是一个整型常量表达式。从0开始计数，get<0>是第一个成员。

为了使用tuple_size或tuple_element，需要知道一个元组对象的类型。确定一个对象的类型的最简单方法就是使用decltype。

std::tuple的一个常见用途是从一个函数返回多个值std::tuple中元素被紧密地存储的(位于连续的内存区域)，而不是链式结构。std::tuple实现了多元组，这是一个编译期就确定大小的容器，可以容纳不同类型的元素。多元组类型在当前标准库中被定义为可以用任意数量参数初始化的类模板。每一模板参数确定多元组中一元素的类型。所以，多元组是一个多类型、大小固定的值的集合。

**创建和初始化**

```cpp
std::tuple<int, double, std::string> first;
//创建一个空的元组，需要指定元组元素的数据类型，调用各个成员的默认构造函数进行初始化。
std::tuple<int, double, std::string> second(first);  
//拷贝构造
std::tuple<int, char> third(10, 'a');        
//创建并初始化,使用小括号初始化
std::tuple<int, std::string, double> fourth{42, "Test", -3.14};
// 创建并初始化,使用新的大括号初始化列表方式初始化
std::tuple<int, char> fifth(std::make_tuple(20, 'b'));
//移动构造,使用模板库的make_tuple
first = std::make_tuple(1, 3.14, "tuple"); //移动赋值
int i_third = 3;
std::tuple<int&> sixth(std::ref(i_third));//创建一个元组,元组的元素可以被引用
```

**元组的访问和修改：std::get<N>(）**

```cpp
int n=1;
auto t=std::make_tuple(10, "Test", 3.14, std::ref(n), n);
//get尖括号中的值必须是一个整型常量表达式。从0开始计数，意味着get<0>是第一个成员。
std::cout<<"The value of t is "<< "(" << 
    std::get<0>(t) << ", " << std::get<1>(t) << ", "<< 
    std::get<2>(t) << ", " << std::get<3>(t) << ", "<<
    std::get<4>(t) << ")\n";
    std::get<3>(t) = 9;
    std::cout << n << std::endl;
```

**元组的元素个数：使用std::tuple_size<>()**

```cpp
std::tuple<char,int,long,std::string> first('A',2,3,"4");
int i_count = std::tuple_size<decltype(first)>::value;
// 使用std::tuple_size计算元组个数
std::cout<<"the number of elements of a tuple:"<<i_count<<"\n";
```

**元组的解包**

std::tie() 元组包含一个或者多个元素，使用std::tie解包。首先需要定义对应元素的变量，再使用tie。

```cpp
{ // std::tie: function template, Constructs a tuple object whose elements are references
    // to the arguments in args, in the same order
    //std::ignore: object, This object ignores any value assigned to it. It is designed to be used as an
    // argument for tie to indicate that a specific element in a tuple should be ignored.
    int myint;
    char mychar;
    std::tuple<int, float, char> mytuple;
    mytuple = std::make_tuple(10, 2.6, 'a');          // packing values into tuple
    std::tie(myint, std::ignore, mychar) = mytuple;   // unpacking tuple into variables 
    std::cout << "myint contains: " << myint << '\n';
    std::cout << "mychar contains: " << mychar << '\n';
}
```

**元组的元素类型获取**

获取元组中某个元素的数据类型，需要用到 std::tuple_element。

语法：std::tuple_element<index, tuple>

```cpp
std::tuple<int, std::string> third(9, std::string("ABC"));
// 得到元组第1个元素的类型，用元组第一个元素的类型声明一个变量
std::tuple_element<1, decltype(third)>::type val_1;
// 获取元组的第一个元素的值
val_1 = std::get<1>(third);
std::cout << "val_1 = " << val_1.c_str() << "\n";
```

**元组的拼接**

使用 std::tuple_cat 执行拼接

```cpp
std::tuple<char, int, double> first('A', 1, 2.2f);
//组合到一起,使用auto,自动推导
auto second = std::tuple_cat(first, std::make_tuple('B', std::string("-=+")));
//组合到一起，可以知道每一个元素的数据类型时什么 与 auto推导效果一样
std::tuple<char, int, double, char, std::string>third = 
std::tuple_cat(first, std::make_tuple('B', std::string("-=+")));
```

**元组的遍历**

元组没用提供operator []重载，遍历起来较为麻烦，需要为其单独提供遍历模板函数

```cpp
template<class T>
void print_single(T const& v){
    if constexpr (std::is_same_v<T, std::decay_t<std::string>>)
        std::cout << std::quoted(v);
    else if constexpr (std::is_same_v<std::decay_t<T>, char>)
        std::cout << "'" << v << "'";
    else
        std::cout << v;
}
//helper function to print a tuple of any size
template<class Tuple, std::size_t N>
struct TuplePrinter {
static void print(const Tuple& t)
{
    TuplePrinter<Tuple, N-1>::print(t);
    std::cout << ", ";
    print_single(std::get<N-1>(t));
}
};
template<class Tuple>
struct TuplePrinter<Tuple, 1>{
static void print(const Tuple& t){
    print_single(std::get<0>(t));
}
};
template<class... Args>
void print(const std::tuple<Args...>& t){
    std::cout << "(";
    TuplePrinter<decltype(t), sizeof...(Args)>::print(t);
    std::cout << ")\n";
}
// end helper function
```

优先使用枚举类而不是枚举类型：

**（1）减少命名空间污染**

枚举类（enum class）的枚举名仅在其枚举类型内可见，避免了命名冲突，减少了命名空间污染。枚举（enum）类型的枚举名会泄漏到其所在的作用域，可能导致命名冲突。

```cpp
enum Color { black, white, red };   // black, white, red 在 Color 所在的作用域
auto white = false;                 // 错误! white 已经在这个作用域中声明
std::vector<std::size_t> primeFactors(std::size_t x); // func 返回 x 的质因子
Color c = red;
if (c < 14.5) {                     // Color 与 double 比较 (!)
    auto factors = primeFactors(c); // 计算一个 Color 的质因子 (!)
}

enum class Color { black, white, red }; // black, white, red 限制在 Color 域内
auto white = false;                 // 没问题，域内没有其他 “white”
Color c = Color::red;               //没问题
if (c < 14.5) {                     //错误.不能比较Color和double
}
if (static_cast<double>(c) < 14.5) { // 奇怪的代码，但是有效
    auto factors = primeFactors(static_cast<std::size_t>(c)); // 有问题，但是能通过编译
}
```

**（2）强类型检查**

限域枚举是强类型的，不允许隐式转换为整型或其他类型，从而避免了潜在的错误。例如，比较一个限域枚举和一个 double 会导致编译错误，而未限域枚举则会隐式转换为整型，可能导致意外的行为。

**（3）前置声明**

限域枚举可以被前置声明，有助于减少编译依赖。未限域枚举在 C++98 中不能被前置声明，但在 C++11 中可以通过指定底层类型来实现前置声明。

前置声明

```cpp
enum class Status;    //前置声明
void continueProcessing(Status s); //使用前置声明 enum
// 枚举定义
enum class Status: std::uint32_t { good = 0,
failed = 1,incomplete = 100,
corrupt = 200,audited = 500,
indeterminate = 0xFFFFFFFF
};
```

在某些情况下，使用未限域枚举可以更方便地与 std::tuple 配合使用，因为枚举名可以隐式转换为 std::size_t。然而，使用限域枚举需要额外的类型转换函数 toUType，虽然代码量增加，但可以避免命名空间污染和隐式类型转换带来的风险。

使用非限域枚举与 std::tuple

```cpp
using UserInfo = std::tuple<std::string, std::string, std::size_t>;
enum UserInfoFields { uiName, uiEmail, uiReputation };
UserInfo uInfo;
auto val = std::get<uiEmail>(uInfo); // 获取用户 email 字段的值
```

使用限域枚举与 std::tuple

```cpp
enum class UserInfoFields { uiName, uiEmail, uiReputation };
UserInfo uInfo;
template<typename E>
constexpr std::underlying_type_t<E> toUType(E enumerator) noexcept {
    return static_cast<std::underlying_type_t<E>>(enumerator);
}
auto val = std::get<toUType(UserInfoFields::uiEmail)>(uInfo);
```

作者建议在大多数情况下优先使用限域枚举，因为它们提供了更好的类型安全性和更清晰的命名空间管理。只有在特定场景下，如与 std::tuple 配合使用时，未限域枚举可能更加方便。

<h4 id="7b803fda">条款十一：优先考虑使用deleted函数而非使用未定义的私有声明</h4>
<font style="color:#c21c13;">C++98 方法：private</font>

C++98 将特殊成员函数（如拷贝构造函数和拷贝赋值运算符）声明为私有且不定义。这种方法可以防止客户端调用这些函数，但如果在成员函数或友元函数中调用这些函数，会在链接时引发错误。C++11 使用 = delete 将这些函数标记为删除的函数。删除的函数不能以任何方式被调用，即使在成员函数或友元函数中调用也会在编译时失败。这比 C++98 的方法更安全，因为错误在编译阶段就能捕获。删除的函数通常声明为 public 而不是 private。这是因为当客户端代码试图调用成员函数时，编译器会先检查访问性，再检查删除状态。如果函数是 private 的，编译器可能会只报告访问性错误，而不会提到函数已被删除。

```cpp
template <class charT, class traits = char_traits<charT> >
class basic_ios : public ios_base {
public:
…
private:
basic_ios(const basic_ios& );           // not defined
basic_ios& operator=(const basic_ios&); // not defined
}；
```

**<font style="color:#c21c13;">C++11 方法</font>**

使用=delete可以禁止特定类型的函数调用,可以禁止特定类型的模板实例化,类内的模板函数特化，如果类内有一个模板函数，使用 = delete 可以在类外删除特定的模板实例。这是因为在类内不能给特化的成员模板函数指定不同的访问级别，而在类外删除这些函数不会有问题。

```cpp
template <class charT, class traits = char_traits<charT> >
class basic_ios : public ios_base {
public:
basic_ios(const basic_ios& ) = delete;
basic_ios& operator=(const basic_ios&) = delete;
};
```

禁止特定类型的函数调用

```cpp
bool isLucky(int number);       // 原始版本
bool isLucky(char) = delete;    // 拒绝 char
bool isLucky(bool) = delete;    // 拒绝 bool
bool isLucky(double) = delete;  // 拒绝 float 和 double
```

禁止特定类型的模板实例化

```cpp
template<typename T>
void processPointer(T* ptr);
template<>
void processPointer<void>(void*) = delete;
template<>
void processPointer<char>(char*) = delete;
template<>
void processPointer<const void>(const void*) = delete;
template<>
void processPointer<const char>(const char*) = delete;
template<>
void processPointer<const volatile void>(const volatile void*) = delete;
template<>
void processPointer<const volatile char>(const volatile char*) = delete;
template<>
void processPointer<wchar_t>(wchar_t*) = delete;
template<>
void processPointer<char16_t>(char16_t*) = delete;
template<>
void processPointer<char32_t>(char32_t*) = delete;
```

类内的模板函数特化

```cpp
class Widget {
public:
template<typename T>
void processPointer(T* ptr){}
};
template<>
void Widget::processPointer<void>(void*) = delete;
```

<h4 id="01e04227">条款十二：使用override声明重写函数</h4>
重写函数的要求：基类函数必须是virtual。基类和派生类函数名必须完全一致（除非是析构函数）。基类和派生类函数的形参类型必须完全一致。基类和派生类函数的常量性（const性）必须完全一致。基类和派生类函数的返回值和异常说明必须兼容。如果基类的虚函数有引用限定符，派生类的重写也必须具有相同的引用限定符。

基类和派生类的定义

```cpp
class Base {
public:
virtual void mf1() const;
virtual void mf2(int x);
virtual void mf3() &;
virtual void mf4() const;
};
class Derived: public Base {
public:
virtual void mf1() const override;
virtual void mf2(int x) override;
virtual void mf3() & override;
void mf4() const override; // 可以添加virtual，但不是必要
}；
```

成员函数引用限定示例

```cpp
class Widget {
public:
using DataType = std::vector<double>;
DataType& data() & { return values; } // 对于左值Widgets, 返回左值
DataType data() && { return std::move(values); } // 对于右值Widgets, 返回右值
private:
DataType values;
};

成员函数引用限定：
引用限定符（&和&&）可以用于成员函数，以区分左值对象和右值对象。
左值引用限定符（&）表示该成员函数只能被左值对象调用。
右值引用限定符（&&）表示该成员函数只能被右值对象调用。
```

**重写函数的重要性：**

正确声明派生类的重写函数至关重要，但容易出错。为了确保重写函数按预期工作，应使用override关键字明确声明重写函数。这样，编译器会在重写不匹配时给出错误提示。

**override关键字的作用：**

override关键字用于显式声明一个派生类函数是基类版本的重写。这有助于捕获重写错误，确保编译器在重写不匹配时给出错误提示。

<h4 id="6116cb8f">条款十三：优先考虑const_iterator而非iterator</h4>
**<font style="color:#c21c13;">前置：std::find</font>**

std::find 是一个泛型算法，它可以接受多种类型的迭代器，包括 iterator 和 const_iterator。

template< class InputIt, class T > InputIt find( InputIt first, InputIt last, const T& value );

这里，InputIt 是一个输入迭代器类型，可以是 iterator 或 const_iterator。std::find 本身并不关心迭代器的具体类型，只要这些迭代器符合输入迭代器的要求即可。

C++98使用iterator

```cpp
std::vector<int> values;
// 假设 values 已经被填充
// 使用 iterator 查找并插入
std::vector<int>::iterator it = std::find(values.begin(), values.end(), 1983);
values.insert(it, 1998);
```

C++98 尝试使用 const_iterator

```cpp
typedef std::vector<int>::iterator IterT;
typedef std::vector<int>::const_iterator ConstIterT;
std::vector<int> values;
// 假设 values 已经被填充
// 尝试使用 const_iterator 查找并插入
ConstIterT ci = std::find(static_cast<ConstIterT>(values.begin()),
static_cast<ConstIterT>(values.end()),
1983);
// 将 const_iterator 转换为 iterator
values.insert(static_cast<IterT>(ci), 1998);  // 可能无法通过编译
```

static_cast<ConstIterT>(values.begin()) 和 static_cast<ConstIterT>(values.end())：将 values 的 iterator 转换为 const_iterator。这里使用 static_cast 是因为 values 是一个非 const 容器，直接调用 values.begin() 和 values.end() 会返回 iterator 而不是 const_iterator。通过 static_cast 将 iterator 转换为 const_iterator，以便 std::find 可以使用 const_iterator 进行查找。

插入操作：

```cpp
values.insert(static_cast<IterT>(ci), 1998);  // 可能无法通过编译
```

static_cast<IterT>(ci)将 const_iterator 转换为 iterator。

const_iterator 不能直接转换为 iterator，因为 const_iterator 是只读的，而 iterator 是可读写的。

这种转换在 C++98 中是不安全的，编译器可能无法通过编译，因为没有标准的方法将 const_iterator 转换为 iterator。

在C++11及以后的版本中，cbegin 和 cend 既可以是成员函数，也可以是非成员函数。

```cpp
std::vector<int> values;
// 假设 values 已经被填充
// 使用 const_iterator 查找并插入
auto it =
std::find(values.cbegin(), values.cend(), 1983);
values.insert(it, 1998);
```

C++11及以后的版本中，std::vector 的 insert 成员函数可以接受 const_iterator。std::vector 的 insert 成员函数的几种重载形式：

```cpp
iterator insert(const_iterator pos, const T& value);
iterator insert(const_iterator pos, T&& value);
iterator insert(const_iterator pos, size_type count, const T& value);
```

std::list 和 std::deque 的 insert 成员函数也可以接受 const_iterator。

最大程度通用的库代码（C++14）

```cpp
template<typename C, typename V>
void findAndInsert(C& container, const V& targetVal, const V& insertVal)
{
    using std::cbegin;
    using std::cend;
    auto it = std::find(cbegin(container), cend(container), targetVal);
    container.insert(it, insertVal);
}
```

自定义非成员函数 cbegin（C++11）

C++11如果处理某些没有 cbegin 和 cend 成员函数的容器，可以自己实现这些非成员函数。

```cpp
template <class C>
auto cbegin(const C& container) -> decltype(std::begin(container)){
    return std::begin(container);
}
template <class C>
auto cend(const C& container) -> decltype(std::end(container)){
    return std::end(container);
}
int main() {
    std::vector<int> values = {1, 2, 3, 4, 5};
    std::array<int, 5> arr = {6, 7, 8, 9, 10};
    // 使用自定义的非成员函数 cbegin 和 cend
    auto cit = cbegin(values);
    auto cend = cend(values);
    for (; cit != cend; ++cit) {
        std::cout << *cit << " ";
    }
    std::cout << std::endl;
    // 处理原生数组
    int rawArray[] = {11, 12, 13, 14, 15};
    auto rawCit = cbegin(rawArray);
    auto rawCend = cend(rawArray);
    for (; rawCit != rawCend; ++rawCit) {
        std::cout << *rawCit << " ";
    }
    std::cout << std::endl;
    return 0;
}
```

```cpp
#include <vector>
#include <array>
#include <algorithm>
template <class C>
auto cbegin(const C& container) -> decltype(std::begin(container))
{
    return std::begin(container);
}
// 示例使用
std::vector<int> values = {1, 2, 3, 4, 5};
auto it = std::find(cbegin(values), cend(values), 3);
values.insert(it, 1983);
```

std::begin 是一个泛型函数，可以根据传递的容器类型返回相应的迭代器。如果 container 是 const 的，std::begin(container) 会返回 const_iterator。如果container是非 const 的,std::begin(container) 会返回 iterator。通过将 container 声明为 const C&，确保传递给 std::begin 的容器总是 const 的，std::begin(container)总是返回 const_iterator。

**作者观点总结**

优先考虑 const_iterator而非 iterator:const_iterator 等价于指向常量的指针，不允许修改其指向的值。标准实践是能加上 const 就加上,防止意外修改数据。

<h4 id="3b9c8d33">条款十四：如果函数不抛出异常请使用noexcept</h4>
<h5 id="714e3b18">**<font style="color:#c21c13;">前置：std::move</font>**</h5>
`std::move` 就是一个 `static_cast` 的转换，就是把左值转换为亡值，这样告诉编译器可以对这个右值进行移动语义，转移这个右值对象的资源所有权（见item1前置）

```cpp
template <typename T>
constexpr typename remove_reference<T>::type&& move(T&& arg) noexcept {
    return static_cast<typename remove_reference<T>::type&&>(arg);
}
```

<h5 id="d581e449"><font style="color:#c21c13;">前置：移动语义/移动构造函数</font></h5>
**什么是语义？**

语义就是一个函数的行为。

**移动语义**

移动语义C++11 引入的新特性，移动语义规定了移动构造函数和移动拷贝函数的行为，就是让资源从一个对象移动到另一个对象，而不是进行昂贵的复制操作。核心思想是“转移资源所有权"，而不是复制资源。而拷贝语义就是规定了拷贝构造函数和复制函数的行为，核心就是复制对象，可以是深拷贝也可以是浅拷贝。

_移动语义= 转移资源所有权= (操作)浅拷贝+置空原指针。_

**对于资源的理解：**

一个对象的资源就是对象的成员变量，可以是动态申请的内存，文件描述符/网络套接字，数据库对象，或者 锁。也可以是非指针的成员变量，如内置类型(内置或者平凡类型的移动拷贝就是普通的复制操作，而且还可能会因为调用std::move变得更慢)；容器(string,vector等)，这些类型已经定义了移动构造函数和移动赋值运算符，可以使用移动语义来转移资源的所有权。

用户定义的类型如果定义了移动构造函数和移动赋值运算符，传入右值会调用移动构造函数。如果没有定义这些函数，编译器会使用默认的拷贝构造函数和拷贝赋值运算符。

_为什么一定要在移动构造函数中实现移动语义，在拷贝构造函数中实现语义可以吗?_

+ 拷贝构造函数语义是**拷贝**，需要创建原始对象的副本，不修改原始对象；移动语义通过转移资源避免拷贝的开销，同时使原对象进入安全状态(通常是清空资源)。
+ 将移动语义放入拷贝构造会破坏语义一致性，导致程序的行为不可预测。

_移动语义是否移动数据的物理地址？_

移动语义不转移对象的物理地址，而是转移对象资源的所有权。

移动构造函数是一种特殊的构造函数，用于将一个对象的所有权从一个对象转移到另一个对象。移动构造函数不会复制数据，而是直接转移资源的所有权。

所有权的转移取决于自己的实现。`<font style="background-color:rgb(248, 248, 250);">std::move</font>`只是用来为了匹配到移动构造(移动赋值)。所有权与移动语义密不可分，可以说移动语义的诞生就是方便区分这个到底是转移所有权转移，还是进行复制。

💡

阅读材料：[https://zhuanlan.zhihu.com/p/658035687](https://zhuanlan.zhihu.com/p/658035687)

```cpp
//定义结构体X，用于展示不同构造函数和析构函数的调用情况
struct X {
X() { puts("X()"); } 
X(const X&) { puts("X(const X&)"); }
X(X&&) noexcept { puts("X(X&&)"); }
~X() { puts("~X()"); }
};
//定义类Test，包含一个指向X的指针成员
class Test {
private:
X* m_p; // 指向X的指针
public:
// 默认构造函数
Test() : m_p{ nullptr } {}
// 构造函数，初始化m_p为给定的X指针
Test(X* x) : m_p{ x } {}
// 拷贝构造函数，深拷贝X对象
Test(const Test& t) {
    m_p = new X(*t.m_p);
}
// 移动构造函数，转移所有权
Test(Test&& t) noexcept {
    m_p = t.m_p;
    t.m_p = nullptr;
}
// 析构函数，释放m_p指向的资源
~Test() {
    if (m_p != nullptr) {
        delete m_p;
    }
}
// 判断是否为空
constexpr bool empty() const noexcept {
    return m_p == nullptr;
}
};
}
int main() { 
    {
        Test t{ new X };
        std::cout << t.empty() << '\n'; // 应输出0，表示非空
        Test t2{ std::move(t) };         // 调用移动构造函数
        std::cout << t.empty() << '\n';  // 应输出1，表示t现在为空
    }
    /*
X()
0
1
~X()
*/
    {
        Test t{ new X };
        std::cout << t.empty() << '\n'; // 应输出0，表示非空
        Test t2{ t };                   // 调用拷贝构造函数
        std::cout << t.empty() << '\n'; // 应输出0，表示t仍然非空
    }
    /*
X()
0
X(const X&)
0
~X()
~X()
*/

}
```

`<font style="background-color:rgb(248, 248, 250);">Test t{new X };</font>`首先`<font style="background-color:rgb(248, 248, 250);">new X</font>`，调用X的构造函数，返回了一个指针，调用了`<font style="background-color:rgb(248, 248, 250);">Test</font>`的有参构造函数，用来初始化了它的数据成员`<font style="background-color:rgb(248, 248, 250);">m_p</font>`，t拥有x的所有权。`<font style="background-color:rgb(248, 248, 250);">t.empty()</font>`不为空。`<font style="background-color:rgb(248, 248, 250);">Test t2{ std::move(t)}; std::move(t)</font>`是一个亡值表达式，调用`<font style="background-color:rgb(248, 248, 250);">t2</font>`的移动构造。将`<font style="background-color:rgb(248, 248, 250);">t</font>`的`<font style="background-color:rgb(248, 248, 250);">m_p</font>`指针赋值给`<font style="background-color:rgb(248, 248, 250);">t2</font>`，并给`<font style="background-color:rgb(248, 248, 250);">t</font>`的`<font style="background-color:rgb(248, 248, 250);">m_p</font>`赋空。完成了所有权的转移，t1不再拥有X的所有权，t2拥有了X的所有权。此时`<font style="background-color:rgb(248, 248, 250);">t</font>`的`<font style="background-color:rgb(248, 248, 250);">m_p</font>`已经为空了，`<font style="background-color:rgb(248, 248, 250);">m_p ==nullptr</font>`。

<h5 id="b8316a24"><font style="color:#c21c13;">前置：vector的emplace_back</font></h5>
emplace_back 是 std::vector 的成员函数，用于在vector的末尾直接构造一个对象，与 push_back 不同，emplace_back 不需要先创建一个临时对象，再将其复制或移动到向量中。相反，它直接在向量的末尾位置构造对象，从而避免了额外的复制或移动操作。

假设我们有以下代码：

```cpp
std::vector<Test> v;
v.emplace_back(Test{});
```

分配内存：std::vector 会检查当前是否有足够的空间来容纳新元素。如果没有足够的空间，它会重新分配更大的内存块，并将现有元素移动到新的内存块中。

构造对象：emplace_back 调用 Test 的构造函数，在vector的末尾位置直接构造一个 Test 对象。

**<font style="color:#c21c13;">前置：调用移动构造函数构造vector</font>**

```cpp
std::vector<Test> v2{ std::move(v) };
```

将 v 中的所有元素转移到 v2 中。由于 v2 是通过移动构造函数创建的，v2 会接管 v 的内部缓冲区，而 v 变成一个vector。

C++11 noexcept关键字用于指定函数不会抛出异常，有助于提高程序的异常安全性，还能够使编译器生成更加高效的代码。

1. noexcept 是函数接口的一部分

函数是否声明为 noexcept 是接口设计的一部分，客户端代码可能会依赖这一点。如果一个函数被声明为 noexcept，客户端代码可能会假设这个函数在任何情况下都不会抛出异常，从而在调用这个函数时不会进行异常处理。更改函数的 noexcept 状态会影响到依赖这个函数的客户端代码。客户端代码可能需要进行相应的调整，例如增加异常处理逻辑，以应对新的异常抛出情况。

```cpp
void safeFunction() noexcept;  // 客户端代码可能依赖此函数不抛出异常
```

2. noexcept 函数更容易优化

编译器可以利用 noexcept 信息进行更多的优化，例如不需要为异常处理保留栈信息，从而减少运行时开销。当一个异常被抛出时，程序需要找到一个合适的异常处理程序来处理这个异常。为了做到这一点，程序需要“展开”调用栈，即逐层回溯调用栈，直到找到一个匹配的异常处理程序。

栈展开的开销：保存当前函数的上下文(寄存器状态、局部变量等)。回溯调用栈，检查每个函数的异常处理信息；析构已构造的对象，释放资源；找到并跳转到合适的异常处理程序。这些步骤需要额外的运行时支持，包括额外的内存和计算资源，从而增加了程序的运行时开销。

1. **<font style="color:#c21c13;">省略栈展开信息 </font>**

如果一个函数被声明为 noexcept，编译器知道这个函数不会抛出异常，因此可以省略与栈展开相关的元数据和代码。编译器不需要为 noexcept 函数生成异常表(exception table)，这些表通常用于记录每个函数的异常处理信息。编译器可以在 noexcept 函数中省略与栈展开相关的代码。

**<font style="color:#c21c13;">（2）减少运行时检查</font>**

编译器可以减少或消除对 noexcept 函数的动态异常检查。

**<font style="color:#c21c13;">（3）内联优化</font>**

编译器倾向于将 noexcept 函数内联到调用点，内联可以减少函数调用的开销，提高性能。

```cpp
int nonOptimizedFunction() {
    try {
        // 可能抛出异常的代码
    } catch (...) {
        // 异常处理
    }
    return 0;
}
int optimizedFunction() noexcept {
    // 不会抛出异常的代码
    return 0;
}
```

3. noexcept 对于移动语义、swap、内存释放函数和析构函数非常有用

对于移动操作和 swap，声明为 noexcept 可以提高性能，因为编译器可以更安全地优化这些操作。内存释放函数和析构函数默认是 noexcept 的，因为它们不应该抛出异常。

```cpp
class Widget {
public:
Widget(Widget&& other) noexcept;// 移动构造函数
void swap(Widget& other) noexcept;// 交换函数
};
template <class T, size_t N>
void swap(T (&a)[N], T (&b)[N]) noexcept(noexcept(swap(*a, *b)));
template <class T1, class T2>
struct pair {
void swap(pair& p) noexcept(noexcept(swap(first, p.first)) && noexcept(swap(second, p.second)));
};
```

**（1）移动语义**

声明为 noexcept 可以告诉编译器移动构造函数不会抛出异常，从而允许编译器进行优化。移动操作通常比复制操作更快，因为它们只需转移资源的所有权，而不是复制资源。如果移动操作是 noexcept 的，编译器可以更安全地使用移动操作来优化代码。

```cpp
class Widget {
public:
Widget(Widget&& other) noexcept;  // 移动构造函数
};
```

Widget 类的移动构造函数被声明为 noexcept，表示这个操作不会抛出异常。编译器可以利用这一点进行优化，例如在 std::vector 的 push_back 操作中优先使用移动操作。

**（2）swap 函数**

swap 函数用于交换两个对象的内容。声明为 noexcept 可以提高性能，因为编译器可以更安全地优化 swap 操作。swap 操作通常涉及多个赋值和移动操作。如果这些操作是 noexcept 的，编译器可以更自由地进行优化，例如内联和减少异常处理的开销。

```cpp
class Widget {
public:
void swap(Widget& other) noexcept;  // 交换函数
};
```

Widget 类的 swap 函数被声明为 noexcept，表示这个操作不会抛出异常。编译器可以利用这一点进行优化，例如在 std::sort 等算法中优先使用 swap 操作。

1. **noexcept 对于内存释放函数和析构函数**

内存释放函数（如 operator delete 和 operator delete[]）用于释放动态分配的内存。这些函数默认是 noexcept 的，因为它们不应该抛出异常。如果内存释放函数抛出异常，程序将处于不确定的状态，可能导致资源泄露或其他严重问题。因此，默认情况下，这些函数是 noexcept 的，以确保程序的稳定性。析构函数用于释放对象占用的资源。这些函数默认是 noexcept 的，因为它们不应该抛出异常。如果析构函数抛出异常，可能会导致未定义行为，特别是在标准库容器和算法中。例如，如果 std::vector 中的元素的析构函数抛出异常，可能会导致 std::vector 的析构函数无法正常完成，从而导致资源泄露。

```cpp
class Widget {
public:
~Widget() noexcept;  // 析构函数
};
```

Widget 类的析构函数被声明为 noexcept，表示这个操作不会抛出异常。编译器可以利用这一点进行优化，如在 std::vector 的析构过程中更安全地释放资源。

4. 异常中立的函数不加no except

大多数函数是异常中立的，即它们自己不抛异常，但可能调用其他会抛异常的函数。这些函数不应该声明为 noexcept，因为它们可能传递异常。

```cpp
void exceptionNeutralFunction() {
    someOtherFunction();  // 可能抛出异常
}
```

虽然 noexcept 可以带来性能提升，但不应该为了 noexcept 而扭曲函数的实现。如果一个函数的实现可能会抛出异常，强行捕获异常并转换为状态码或特殊返回值会使代码变得复杂且难以维护。

5. 区分宽泛契约和严格契约的函数

宽泛契约的函数没有前置条件，可以随时调用，不应表现出未定义行为,无论程序状态如何，都可以安全地调用这些函数。严格契约的函数有前置条件，违反前置条件会导致未定义行为。对于严格契约的函数，即使它自然不会抛出异常，也应该谨慎声明 noexcept，因为前置条件的检查可能需要抛出异常。

```cpp
void wideContractFunction() noexcept {
    // 没有前置条件，自然不会抛出异常
}
```

严格契约函数

严格契约的函数有前置条件，调用者必须确保这些前置条件满足。如果违反前置条件，函数的行为是未定义的。函数有特定的输入要求，调用者必须确保这些要求满足。如果前置条件不满足，函数的行为是未定义的，可能会导致程序崩溃或其他不可预测的行为。谨慎声明 noexcept，即使这些函数自然不会抛出异常，也应该谨慎声明为 noexcept，因为前置条件的检查可能需要抛出异常。

```cpp
void narrowContractFunction(const std::string& s) noexcept{
    assert(s.length() <= 32);
}
```

<h4 id="a7906d61">条款十五：尽可能的使用constexpr</h4>
<h5 id="bdf513fe">**constexpr表达式/对象/函数**</h5>
constexpr表达式值不会改变并且在编译过程就能得到计算结果的表达式。

声明为constexpr的变量一定是一个const变量，而且必须用常量表达式初始化：

```cpp
constexpr int mf = 20;  //20是常量表达式
constexpr int limit = mf + 1;// mf + 1是常量表达式
constexpr int sz = size(); //之后当size是一个constexpr函数时才是一条正确的声明语句
```

constexpr声明中如果定义了一个指针，限定符conxtexpr仅对指针有效，与指针所指的对象无关。

```cpp
const int*p=nullptr；
constexpr int* q = nullptr；
```

p是一个指向常量的指针，q是一个常量指针，其中的关键在于constexpr把它所定义的对象置为了顶层const。

**constexpr 函数**

constexpr还能定义一个常量表达式函数，即constexpr函数，常量表达式函数的返回值可以在编译

阶段就计算出来。当constexpr应用于函数时，表示函数可以在编译期执行，因为参数也是编译期常量。即使某些参数在运行时才确定，constexpr函数也能正常工作，这时它就相当于普通的函数，在运行时计算结。

C++11对constexpr函数的实现有严格限制，仅允许单行return语句，但可以通过递归和三元运算符来增加表达力。到了C++14，限制被大幅放宽，允许更复杂的逻辑，包括循环和局部变量（如下）。适合于那些需要在编译期确定值的情况，例如计算数组大小或作为模板参数等。

约束规则[C++14]：

函数体允许声明变量，但是不允许static和thread_local变量。允许if/switch，不能使用goto语句。允许循环语句,包括for/while/do-while、函数可以修改生命周期和常量表达式相同的对象。 函数的返回值可以声明为void。constexpr声明的成员函数不再具有const属性。

规则约束[c++20前]

a. 必须非虚;   
b. 函数体不能是函数 try 块; [c++20前]  
c. 不能是协程; [c++20起]  
d. 对于构造函数与析构函数 [C++20 起]，该类必须无虚基类  
e. 它返回类型（如果存在）和每个参数都必须是字面类型 (LiteralType)  
f. 至少存在一组实参值，使得函数的一个调用为核心常量表达式的被求值的子表达式（对于构造函 数为足以用于常量初始化器） (C++14 起)。不要求诊断是否违反这点。

```cpp
constexpr int pow(int base, int exp) noexcept {
    auto result = 1;
    for (int i = 0; i < exp; ++i) result *= base;
    return result;
}
constexpr auto numConds = 5; // 实验条件数量
std::array<int, pow(3, numConds)> results; // 结果数组，大小为3^5
```

定义constexpr函数pow，用于计算幂。numConds是一个constexpr变量，表示实验条件的数量。 results是一个std::array，pow(3, numConds)在编译期计算得出，用于存储实验的可能结果。

```cpp
#include<iostream>
#include<array>
// 计算绝对值的 constexpr 函数
constexpr int abs_(int x) {
    return x > 0 ? x : -x;
}
// 计算从1到x的累加和的 constexpr 函数
constexpr int sum(int x) {
    int result = 0;
    while (x > 0) {
        result += x--;
    }
    return result;
}
//返回x+1的constexpr 函数
constexpr int next(int x){
    return ++x; // 注意，这里的 ++x 是前缀自增操作符
}
//主函数
int main() {
    //使用 constexpr 函数初始化数组大小
    char buffer1[sum(5)] = {0};//编译期计算
    char buffer2[abs_(-5)] = {0};//编译期计算
    char buffer3[next(5)] = {0};//编译期计算
    //使用常量表达式作为模板参数
    std::array<int, size(10)> arr;// 编译期计算
    std::cout<<"Array size:"<<arr.size()<< std::endl;
    //尝试使用非常量表达式作为 constexpr 函数的参数
    int i = 10;
    // 下面这行会导致编译错误，因为 'i' 不是常量表达式
    // constexpr int s = size(i); // 编译错误
    return 0;
}
```

编译错误：

```cpp
constexpr int s = size(i);// 编译错误
```

导致编译错误，因为 i 是一个运行时变量，不是常量表达式。因此，size(i) 不能在编译时确定其值，不能用于初始化 constexpr 变量 s。运行时计算：

```cpp
int s = size(i); // 正常工作，运行时计算
```

s不是 constexpr 变量，size(i) 的结果可以在运行时计算并赋值给 s。

常量表达式参数

```cpp
constexpr int s_constexpr = size(10); // 正常工作，编译时计算
```

正常工作，因为 10 是一个常量表达式，size(10) 可以在编译时计算其结果，并用于初始化 constexpr 变量 s_constexpr。

**constexpr对象**

constexpr还能够修饰对象。constexpr对象本质上是const对象的加强版，它们不仅在运行时不可变，而且其值必须在编译期确定。放置在只读存储区域，适用于需要整型常量表达式（如数组大小、模板参数等）的场景。虽然所有constexpr对象都是const，但并非所有const对象都能被视为constexpr，因为后者要求其值必须在编译期可得。

```cpp
#include <iostream>
struct X {
int value;
};
int main(){
    constexpr X x = { 1 };
    char buffer[x.value] = { 0 };
}
```

以上代码自定义了一个结构体X，并且使用constexpr声明和初始化了变量x。到目前为止一切顺利，不过有时候我们并不希望成员变量被暴露出来，于是修改了X的结构：

```cpp
#include <iostream>
class X {
public:
X() : value(5) {}
int get() const{ return value;}
private:
int value;
};
int main(void){
    constexpr X x; //error: constexpr variable cannot have non-literal type 'const X
    char buffer[x.get()] = { 0 };//无法在编译期计算
}
```

解决上述问题只需要用constexpr声明X类的构造函数，即声明一个常量表达式构造函数，当然这个构造函数也有一些规则需要遵循。

1构造函数必须用constexpr声明。2构造函数初始化列表中必须是常量表达式。3构造函数的函数体必须为空（这一点基于构造函数没有返回值，所以不存在return expr。

根据这个constexpr构造函数规则修改如下

```cpp
#include <iostream>
class X {
public:
constexpr X():value(5){}
constexpr X(int i):value{i} {}
constexpr int get() const{return value;}
private:
int value;
};
int main(void){
    constexpr X x; 
    // error: constexpr variable cannot have non-literal type const X.
    char buffer[x.get()] = { 0 };
}
```

上面这段代码给构造函数和get函数添加constexpr说明符可以编译成功，它们本身都符合常量表达式构造函数和常量表达式函数的要求，称这样的类为字面量类类型.

在C++11中,constexpr会自动给函数带上const属性。从C++14起constexpr返回类型的类成员函数不在是const函数了。

常量表达式构造函数拥有和常量表达式函数相同的退化特性，当它的实参不是常量表达式的时候，构造函数可以退化为普通构造函数，当然，这么做的前提是类型的声明对象不能为常量表达式值。

```cpp
int i=8;
constexpr X x(i); //编译失败,不能使用constexpr声明.
X y(i); //编译成功.
```

由于i不是一个常量，因此X的常量表达式构造函数退化为普通构造函数，这时对象x不能用constexpr声明，否则编译失败。 

**constexpr lambda**

C++17开始,lambda表达式在条件允许的情况下(常量表达式函数的规则)都会隐式声明为constexpr。

```cpp
#include <iostream>
#include <array>
constexpr int foo(){
    return [](){return 58;}();
}
auto get_size =[](int i) {return i * 2;};
int main(void){
    std::array<int,foo()> arr1= { 0 };
    std::array<int, get_size(5)> arr2= { 0 };
}
```

lambda表达式却可以用在常量表达式函数和数组长度中，可见该lambda表达式的结果在编译阶段已经计算出来了。实际上这里的[](int i) { return i * 2; }相当于:

```cpp
class GetSize {
public:
constexpr int operator() (int i) const 
{
    return i * 2;
}
};
```

当lambda表达式不满足constexpr的条件时，lambda表达式也不会出现编译错误，它会作为运行时lambda表达式存在。

```cpp
// 情况1
int i = 5;
auto get_size = [](int i) {return i * 2;};
char buffer1[get_size(i)] = {0}; //编译失败，get_size需要运行时调用
int a1 = get_size(i);
// 情况2
auto get_count = []() {
    static int x = 5;
    return x;
};
int a2 = get_count();
```

情况1和常量表达式函数相同，get_size可能会退化为运行时lambda表达式对象。当这种情况发生的时候，get_size的返回值不再具有作为数组长度的能力，但是运行时调用get_size对象还是没有问题的。

+ `get_size` 是一个普通的 lambda 表达式，它依赖于传入的参数 `i`，并在运行时返回 `i * 2`。当尝试将 `get_size(i)` 用作数组的大小时，编译器需要在编译时确定数组的大小（这是 C++ 标准要求的）。然而，由于 `get_size` 不是 `constexpr`，它不能在编译时进行求值，所以 `get_size(i)` 不能用于数组的大小。这会导致编译错误，因为`get_size(i)`需要在编译时计算，它只是一个运行时 lambda，编译器无法在编译时计算出它的结果。
+ 可以强制要求lambda表达式是一个常量表达式，用constexpr声明。做好处是可以检查lambda表达式是否有可能是一个常量表达式，如果不能则会编译报错。

```cpp
auto get_size = [](int i) constexpr -> int { return i*2; };
char buffer2[get_size(5)] = { 0 };
auto get_count = []() constexpr -> int 
{
    static int x = 5; // 编译失败，x是一个static变量
    return x;
};
int a2 = get_count();
```

`get_count` 是一个 lambda 表达式，它包含一个静态局部变量 `x`。`static` 变量在第一次调用时初始化，并在之后的每次调用中保持其值。所以 `get_count` 返回的值是 `x`，而 `x` 是静态的，因此 `get_count()` 会返回一个固定值。`static`变量并不符合`constexpr`的要求，因为`constexpr`需要在编译时就能求值，而`static`变量的生命周期是运行时管理的。如果尝试将这个 lambda 声明为`constexpr`，会导致编译失败。

<h4 id="982c9180">条款十六：确保const成员函数线程安全</h4>
C++ const成员函数表示该函数不会修改对象的状态。如果这些函数内部使用了可变（mutable）成员变量来缓存计算结果，那么它们在多线程环境中可能不是线程安全的。

例子中的Polynomial类有一个roots()方法，它是一个const成员函数，但会更新mutable成员rootsAreValid和rootVals来缓存根值。在没有同步的情况下，这些代码会有不同的线程读写相同的内存，会产生数据竞争。`root`成员函数虽然被声明为`const`，但不是线程安全的。

```cpp
class Polynomial {
public:
using RootsType = std::vector<double>;
RootsType roots() const
{
    if (!rootsAreValid) {               //如果缓存不可用
        …                               //计算根
        //用rootVals存储它们
        rootsAreValid = true;
    }
    return rootVals;
}
private:
mutable bool rootsAreValid{ false };    //初始化器（initializer）的
mutable RootsType rootVals{};           //更多信息请查看条款7
};
```

const成员函数应当是线程安全的，以避免数据竞争。

**解决方案**

_<font style="color:#c21c13;">使用互斥锁(mutex)进行同步</font>_

通过在roots()方法中使用std::lock_guard<std::mutex>,可以确保在访问或更新缓存时只有一个线程能够执行相关代码段。使用std::mutex可以保护共享资源，防止多个线程同时修改相同的内存位置。

```cpp
class Polynomial {
public:
using RootsType = std::vector<double>;    
RootsType roots() const{
    std::lock_guard<std::mutex> g(m); //锁定互斥量
    if (!rootsAreValid) {//如果缓存无效
        //计算/存储根值
        rootsAreValid = true;
    }
    return rootVals;//返回缓存的根值
}
private:
mutable std::mutex m;  // 互斥量
mutable bool rootsAreValid { false };
mutable RootsType rootVals {};
};
```

_<font style="color:#c21c13;">使用原子变量（Atomic）</font>_

对于仅涉及单个变量或内存位置的操作，可以使用std::atomic来替代互斥量，以减少同步开销。

std::atomic类型提供了一种更轻量级的方式来保证某些操作的原子性，适用于单个变量或内存位置的操作。但是，当需要对多个变量作为单元操作时，仍需使用互斥量，因为std::atomic无法保证跨多个变量的操作的原子性。

**结论**

const成员函数应当设计为线程安全，在预期会被多个线程并发调用的情况下。选择合适的同步机制（如std::mutex或std::atomic)，取决于具体需求和性能考虑。当需要保证多个变量或内存位置的一致性时，应该使用互斥量而不是原子变量。

<h4 id="9cbf3b0f">条款十七：理解特殊成员函数的生成</h4>
特殊成员函数是指C++自己生成的函数。C++98有四个：默认构造函数，析构函数，拷贝构造函数，拷贝赋值运算符。当然在这里有些细则要注意。默认构造函数仅在类完全没有构造函数的时候才生成。（防止编译器为某个类生成构造函数，但是你希望那个构造函数有参数）生成的特殊成员函数是隐式public且`inline`，它们是非虚的，除非相关函数是在派生类中的析构函数，派生类继承了有虚析构函数的基类。在这种情况下，编译器为派生类生成的析构函数是虚的。C++11特殊成员函数新增移动构造函数和移动赋值运算符。

```cpp
class Widget {
public:
…
Widget(Widget&& rhs);               //移动构造函数
Widget& operator=(Widget&& rhs);    //移动赋值运算符
…
};
```

移动操作仅在需要的时候生成，如果生成了，就会对类的non-static数据成员执行逐成员的移动。

移动构造函数根据`rhs`参数里面对应的成员移动构造出新non-static部分，移动赋值运算符根据参数里面对应的non-static成员移动赋值。移动构造函数也移动构造基类部分（如果有的话），移动赋值运算符也是移动赋值基类部分。当对一个数据成员或者基类使用移动构造或者移动赋值时，没有任何保证移动一定会真的发生。逐成员移动，实际上，更像是逐成员移动**请求**，因为对**不可移动类型**（即对移动操作没有特殊支持的类型，比如大部分C++98传统类）使用“移动”操作实际上执行的是拷贝操作。

逐成员移动的核心是对对象使用`std::move`，然后函数决议时会选择执行移动还是拷贝操作。

**关于拷贝和移动的一些规则**

（1）拷贝操作（拷贝构造函数与拷贝赋值运算符）

如果声明了一个拷贝构造函数，但没有声明拷贝赋值运算符，编译器会自动生成拷贝赋值运算符；反之亦然。两个拷贝操作是独立的，一个的存在不会阻止另一个的自动生成。如果类没有任何用户定义的拷贝操作，编译器将为类生成默认的拷贝构造函数和拷贝赋值运算符。默认情况下，这两个函数都会执行逐成员的浅拷贝。

（2）移动操作（移动构造函数与移动赋值运算符）

移动构造函数和移动赋值运算符不是独立的。一旦显式声明了其中一个，编译器就不会再生成另一个。这是因为如果需要定制移动构造函数的行为，很可能移动赋值运算符也需要类似的处理。如果类没有任何用户定义的拷贝或移动操作，并且没有用户定义的析构函数，那么编译器可以自动生成移动构造函数和移动赋值运算符。默认情况下，这两个函数都会执行逐成员的移动操作，即调用每个成员的移动构造函数或移动赋值运算符。

（3）拷贝操作与移动操作的关系

拷贝抑制移动：如果一个类显式声明了拷贝构造函数或拷贝赋值运算符中的任何一个，那么编译器将不会自动生成移动构造函数和移动赋值运算符。如果用户定义了拷贝操作，逐成员拷贝不合适，那么逐成员移动也可能不合适。

（4）移动抑制拷贝

如果一个类显式声明了移动构造函数或移动赋值运算符中的任何一个，那么编译器将不会自动生成拷贝构造函数和拷贝赋值运算符。相反，它会将这些拷贝操作标记为delete，以防止通过拷贝进行对象的复制。这确保了如果逐成员移动不适合这个类，那么逐成员拷贝同样也不适合。

（5）对C++98代码的影响

C++11引入了移动语义，但是为了保持向后兼容性，C++98的代码不会因为新规则而被破坏。C++98标准中没有移动操作的概念，因此老代码不会受到影响。如果要让旧的C++98代码支持移动语义，需要使用C++11标准，并在类中添加相应的移动构造函数和移动赋值运算符。这样做之后，类就必须遵循C++11的特殊成员函数生成规则。

示例1: 使用 = default 显式声明特殊成员函数

```cpp
class Widget {
public:
~Widget(); // 用户声明的析构函数
Widget(const Widget&) = default; // 默认拷贝构造函数
Widget& operator=(const Widget&) = default; // 默认拷贝赋值运算符
};
```

通过使用= default，明确表示希望使用编译器提供的默认实现。防止因添加新的功能（如日志记录）而无意间影响到特殊成员函数的自动生成，从而避免性能问题或其他意外行为。

示例2: 多态基类的特殊成员函数

```cpp
class Base {
public:
virtual ~Base() = default; // 虚析构函数
Base(Base&&) = default; // 移动构造函数
Base& operator=(Base&&) = default; // 移动赋值运算符
Base(const Base&) = default; // 拷贝构造函数
Base& operator=(const Base&) = default; // 拷贝赋值运算符
};
```

确保多态基类支持拷贝和移动语义。提供了完整的拷贝和移动语义支持，同时保持虚析构函数以正确处理派生类对象的删除。

示例3: 日志记录导致的性能问题

```cpp
class StringTable {
public:
StringTable() { makeLogEntry("Creating StringTable object"); }
~StringTable() { makeLogEntry("Destroying StringTable object"); }
private:
std::map<int, std::string> values;
};
```

添加了用户定义的析构函数后，编译器不再生成移动构造函数和移动赋值运算符。移动操作退化为拷贝操作，导致性能下降，因为std::map的拷贝比移动慢得多。

解决方案：显式声明并使用= default来恢复移动操作。

```cpp
class StringTable {
public:
StringTable() { makeLogEntry("Creating StringTable object"); }
~StringTable() { makeLogEntry("Destroying StringTable object"); }
StringTable(StringTable&&) = default; // 移动构造函数
StringTable& operator=(StringTable&&) = default; // 移动赋值运算符
private:
std::map<int, std::string> values;
};
```

总结：C++11对于特殊成员函数处理的规则如下：

+ **默认构造函数**：和C++98规则相同。仅当类不存在用户声明的构造函数时才自动生成。
+ **析构函数**：基本上和C++98相同，稍微不同的是现在析构默认`noexcept`（参见[Item14](https://cntransgroup.github.io/EffectiveModernCppChinese/3.MovingToModernCpp/item14.html)）。和C++98一样，仅当基类析构为虚函数时该类析构才为虚函数。
+ **拷贝构造函数**：和C++98运行时行为一样：逐成员拷贝non-static数据。仅当类没有用户定义的拷贝构造时才生成。如果类声明了移动操作它就是_delete_的。当用户声明了拷贝赋值或者析构，该函数自动生成已被废弃。
+ **拷贝赋值运算符**：和C++98运行时行为一样：逐成员拷贝赋值non-static数据。仅当类没有用户定义的拷贝赋值时才生成。如果类声明了移动操作它就是_delete_的。当用户声明了拷贝构造或者析构，该函数自动生成已被废弃。
+ **移动构造函数**和**移动赋值运算符**：都对非static数据执行逐成员移动。仅当类没有用户定义的拷贝操作，移动操作或析构时才自动生成。

**CHAPTER 4 Smart Pointers**

诗人和歌曲作家喜欢爱。有时候喜欢计数。很少情况下两者兼有。受伊丽莎白·巴雷特·勃朗宁（Elizabeth Barrett Browning）对爱和数的不同看法的启发（“我怎么爱你？让我数一数。”）和保罗·西蒙（Paul Simon）（“离开你的爱人必须有50种方法。”），我们可以试着枚举一些为什么原始指针很难被爱的原因：

1. 它的声明不能指示所指到底是单个对象还是数组。
2. 它的声明没有告诉你用完后是否应该销毁它，即指针是否拥有所指之物。
3. 如果你决定你应该销毁指针所指对象，没人告诉你该用`delete`还是其他析构机制（比如将指针传给专门的销毁函数）。
4. 如果你发现该用`delete`。 原因1说了可能不知道该用单个对象形式（“`delete`”）还是数组形式（“`delete[]`”）。如果用错了结果是未定义的。
5. 假设你确定了指针所指，知道销毁机制，也很难确定你在所有执行路径上都执行了**恰为一次**销毁操作（包括异常产生后的路径）。少一条路径就会产生资源泄漏，销毁多次还会导致未定义行为。
6. 一般来说没有办法告诉你指针是否变成了悬空指针（dangling pointers），即内存中不再存在指针所指之物。在对象销毁后指针仍指向它们就会产生悬空指针。

原始指针是强大的工具，当然，另一方面几十年的经验证明，只要注意力稍有疏忽，这个强大的工具就会攻击它的主人。

**智能指针**（_smart pointers_）是解决这些问题的一种办法。智能指针包裹原始指针，它们的行为看起来像被包裹的原始指针，但避免了原始指针的很多陷阱。你应该更倾向于智能指针而不是原始指针。几乎原始指针能做的所有事情智能指针都能做，而且出错的机会更少。

在C++11中存在四种智能指针：`std::auto_ptr`，`std::unique_ptr`，`std::shared_ptr`，` std::weak_ptr`。都是被设计用来帮助管理动态对象的生命周期，在适当的时间通过适当的方式来销毁对象，以避免出现资源泄露或者异常行为。

`std::auto_ptr`是来自C++98的已废弃遗留物，它是一次标准化的尝试，后来变成了C++11的`std::unique_ptr`。要正确的模拟原生指针需要移动语义，但是C++98没有这个东西。取而代之，`std::auto_ptr`拉拢拷贝操作来达到自己的移动意图。这导致了令人奇怪的代码（拷贝一个`std::auto_ptr`会将它本身设置为null！）和令人沮丧的使用限制（比如不能将`std::auto_ptr`放入容器）。

`std::unique_ptr`能做`std::auto_ptr`可以做的所有事情以及更多。它能高效完成任务，而且不会扭曲自己的原本含义而变成拷贝对象。在所有方面它都比`std::auto_ptr`好。现在`std::auto_ptr`唯一合法的使用场景就是代码使用C++98编译器编译。除非你有上述限制，否则你就该把`std::auto_ptr`替换为`std::unique_ptr`而且绝不回头。

各种智能指针的API有极大的不同。唯一功能性相似的可能就是默认构造函数。因为有很多关于这些API的详细手册，所以我将只关注那些API概览没有提及的内容，比如值得注意的使用场景，运行时性能分析等，掌握这些信息可以更高效的使用智能指针。

<h1 id="9a75bb70">第四章 智能指针</h1>
<h4 id="bea4e0a3">条款十八：对于独占资源使用std::unique_ptr</h4>
当需要一个智能指针时，`std::unique_ptr`通常是最合适的。默认情况下，`std::unique_ptr`大小等同于原始指针，而且对于大多数操作（包括取消引用），他们执行的指令完全相同。甚至可以在内存和时间都比较紧张的情况下使用它。如果原始指针够小够快，那么`std::unique_ptr`一样可以。

`std::unique_ptr`体现了专有所有权（_exclusive ownership_）语义。一个non-null `std::unique_ptr`始终拥有其指向的内容。移动一个`std::unique_ptr`将所有权从源指针转移到目的指针。（源指针被设为null。）拷贝一个`std::unique_ptr`是不允许的，因为如果你能拷贝一个`std::unique_ptr`，会得到指向相同内容的两个`std::unique_ptr`，每个都认为自己拥有（并且应当最后销毁）资源，销毁时就会出现重复销毁。因此，`std::unique_ptr`是一种只可移动类型（_move-only type)。_当析构时，一个non-null `std::unique_ptr`销毁它指向的资源。默认情况下，资源析构通过对`std::unique_ptr`里原始指针调用`delete`来实现。

`std::unique_ptr`的常见用法是作为继承层次结构中对象的工厂函数返回类型。假设有一个投资类型（的继承结构，使用基类`Investment`。

```cpp
class Investment{ … };
class Stock:public Investment { … };
class Bond:public Investment { … };
class RealEstate:public Investment { … };
```

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742811188596-cf53a92a-6986-4bc7-9f78-d3fa11e15286.png)

使用std::unique_ptr智能指针来管理工厂函数返回的对象，并且在需要时提供自定义的删除器。以下是整理和解释：

**工厂函数**

工厂函数用于在堆上分配一个对象并返回指向该对象的std::unique_ptr。调用者对工厂返回的资源负责，即拥有专有所有权。std::unique_ptr确保当其自身被销毁时，所指向的对象也会被自动销毁。

```cpp
template<typename... Ts>
std::unique_ptr<Investment> makeInvestment(Ts&&... params);
```

模板参数包：Ts&&... params允许传递任意数量和类型的参数。

返回类型：std::unique_ptr<Investment>，表示返回一个指向Investment基类的智能指针。

调用工厂函数

```cpp
{
    auto pInvestment = makeInvestment(arguments);
    // 使用 pInvestment
} // pInvestment 在作用域结束时自动销毁
```

pInvestment在其作用域结束时会自动销毁，并释放其所拥有的资源。

自定义删除器

```cpp
auto delInvmt = [](Investment* pInvestment) {
    makeLogEntry(pInvestment); // 记录日志
    delete pInvestment;        // 删除对象
};
```

使用lambda表达式，作为自定义删除器。在删除对象之前记录日志，然后通过delete释放资源。

修改后的工厂函数

```cpp
template<typename... Ts>
std::unique_ptr<Investment, decltype(delInvmt)> makeInvestment(Ts&&... params)
{
    std::unique_ptr<Investment, decltype(delInvmt)> pInv(nullptr, delInvmt);
    if (/* 创建Stock对象 */) {
        pInv.reset(new Stock(std::forward<Ts>(params)...));
    } else if (/* 创建Bond对象 */) {
        pInv.reset(new Bond(std::forward<Ts>(params)...));
    } else if (/* 创建RealEstate对象 */) {
        pInv.reset(new RealEstate(std::forward<Ts>(params)...));
    }
    return pInv;
}
```

返回类型：std::unique_ptr<Investment, decltype(delInvmt)>，使用自定义删除器delInvmt。pInv初始化为空指针，并关联自定义删除器delInvmt。

创建对象：根据条件创建不同的派生类对象（如Stock、Bond或RealEstate），并通过reset方法将所有权转移给pInv。

完美转发：使用std::forward将参数完美转发给构造函数，保留参数的值类别。

```cpp
class Investment {
public:
virtual ~Investment(); // 关键设计部分！
// 其他成员函数
};
```

虚析构函数：virtual ~Investment()确保通过基类指针删除派生类对象时，派生类的析构函数会被正确调用。

C++14中函数返回类型推导的引入使得makeInvestment工厂函数可以以更简洁和封装的方式实现。使用C++14的返回类型推导

```cpp
template<typename... Ts>
auto makeInvestment(Ts&&... params) // C++14
{
    auto delInvmt = [](Investment* pInvestment) { // 自定义删除器
        makeLogEntry(pInvestment);
        delete pInvestment;
    };
    std::unique_ptr<Investment, decltype(delInvmt)> pInv(nullptr, delInvmt);
    if (/* 创建Stock对象 */) {
        pInv.reset(new Stock(std::forward<Ts>(params)...));
    } else if (/* 创建Bond对象 */) {
        pInv.reset(new Bond(std::forward<Ts>(params)...));
    } else if (/* 创建RealEstate对象 */) {
        pInv.reset(new RealEstate(std::forward<Ts>(params)...));
    }
    return pInv; // 返回带有自定义删除器的std::unique_ptr
}
```

返回类型推导：auto关键字用于让编译器自动推导makeInvestment的返回类型。

自定义删除器：

delInvmt是一个lambda表达式，作为自定义删除器。在删除Investment对象之前，它会记录日志并调用delete来释放资源。

std::unique_ptr的构造：

pInv初始化为空指针，并关联自定义删除器delInvmt。

decltype(delInvmt)用于获取delInvmt的类型，确保std::unique_ptr使用正确的删除器类型。

创建对象：

根据条件创建不同的派生类对象（如Stock、Bond或RealEstate）。使用reset方法将所有权转移给pInv，并通过std::forward完美转发参数。

返回智能指针：函数返回带有自定义删除器的std::unique_ptr。

**关于std::unique_ptr的大小**

默认删除器：当使用默认删除器（即delete）时，std::unique_ptr的大小通常与原始指针相同。

自定义删除器：函数指针形式的删除器通常会使std::unique_ptr的大小从一个字（word）增加到两个字。

如果自定义删除器是无状态的lambda表达式（不捕获任何变量），则对std::unique_ptr的大小没有影响。如果有状态的删除器（例如捕获了变量的lambda），则std::unique_ptr的大小会增加，具体取决于捕获的状态量。

示例：无状态lambda vs 函数指针

```cpp
// 无状态lambda
auto delInvmt1 = [](Investment* pInvestment) {
    makeLogEntry(pInvestment);
    delete pInvestment;
};
// 函数指针
void delInvmt2(Investment* pInvestment) {
    makeLogEntry(pInvestment);
    delete pInvestment;
}
```

无状态lambda：delInvmt1不会增加std::unique_ptr的大小。

函数指针：delInvmt2会增加std::unique_ptr的大小，至少增加一个函数指针的大小。

**其他用途**

**Pimpl Idiom：**std::unique_ptr常用于实现Pimpl Idiom，隐藏实际实现以减少编译依赖性。

数组管理：std::unique_ptr<T[]>用于管理动态分配的数组，但通常建议使用std::array、std::vector或std::string等容器代替原始数组。

1. std::unique_ptr<T[]> 是一个智能指针，专门用于管理动态分配的数组。它确保在智能指针生命周期结束时自动释放数组内存，提供对数组的独占所有权。使用 operator[] 来访问数组元素。调用delete[]而不是delete来释放内存。

```cpp
int main() {
    std::unique_ptr<int[]> arr(new int[5]{1, 2, 3, 4, 5});
    for (int i = 0; i < 5; ++i) {
        std::cout << arr[i] << " ";
    }
}
```

（2）std::array 是一个固定大小的数组容器，它封装了 C 风格的数组，并提供了额外的安全性和便利性。大小在编译时确定且不可更改。不会发生越界访问（通过 at 方法）。提供了标准容器接口，如 begin、end、size 等。

```cpp
int main() {
    std::array<int, 5> arr = {1, 2, 3, 4, 5};
    for (const auto& elem : arr) {
        std::cout << elem << " ";
    }
}
```

1. std::vector 是动态数组，见STL。

推荐使用 std::array、std::vector 或 std::string。std::array 和 std::vector 提供了边界检查，避免了越界访问。std::string 提供了各种字符串操作方法，减少了手动处理字符串的错误。这些容器提供了丰富的操作方法，简化了代码编写。

std::unique_ptr可以轻松转换为std::shared_ptr，这使得工厂函数返回std::unique_ptr非常灵活。

如果调用者需要共享所有权，可以直接将std::unique_ptr转换为std::shared_ptr。

```cpp
std::shared_ptr<Investment> sp = makeInvestment(arguments);
```

这种灵活性使得工厂函数能够提供最高效的智能指针，同时不妨碍调用者根据需要选择更适合的智能指针类型。 工厂函数无法知道调用者是否要对它们返回的对象使用专有所有权语义，或者共享所有权（即`std::shared_ptr`）是否更合适。 通过返回`std::unique_ptr`，工厂为调用者提供了最有效的智能指针，但它们并不妨碍调用者用其更灵活的兄弟替换它。

**请记住**

+ `std::unique_ptr`是轻量级、快速的、只可移动（_move-only_）的管理专有所有权语义资源的智能指针。
+ 默认情况资源销毁通过`delete`实现，但是支持自定义删除器。有状态的删除器和函数指针会增加`std::unique_ptr`对象的大小。
+ 将`std::unique_ptr`转化为`std::shared_ptr`非常简单。

<h4 id="998d52aa">条款十九：对于共享资源使用std::shared_ptr</h4>
自动管理资源的语言（Java，python）通常内置垃圾回收机制，能够自动识别不再使用的对象并释放它们占用的资源。垃圾回收器负责清理未被引用的对象，所以使用这类语言的程序员不需要手动管理每个对象的生命周期，减少了出错的可能性。虽然方便，但这意味资源的具体回收时间点是不确定的，这可能会影响到性能敏感的应用。

C++更倾向于拥有对资源生命周期的精细控制,这样可以更好地优化性能和资源利用。通过显式地定义构造函数、析构函数以及使用RAII技术，C++程序可以在特定的时间点准确地释放资源。但是这种级别的控制也增加了代码的复杂度，并且需要程序员更加小心以防止资源泄露等问题。

为了解决上述两种观点之间的矛盾，C++11引入了std::shared_ptr智能指针。std::shared_ptr结合了自动化管理和可预测销毁的优点：

（1）共享所有权：多个std::shared_ptr可以指向同一个对象，每个std::shared_ptr都持有该对象的所有权。

（2）自动销毁：当最后一个指向某个对象的std::shared_ptr被销毁或重新分配给另一个对象时，该对象就会被自动销毁。

**std::shared_ptr的基本机制**

1. **引用计数：**std::shared_ptr通过维护一个引用计数来跟踪有多少个std::shared_ptr实例指向同一个资源。每当一个新的std::shared_ptr被创建或复制时，该计数递增；当一个std::shared_ptr被销毁或重置为指向其他对象时，该计数递减。一旦引用计数降至0，没有活跃的std::shared_ptr再指向该资源，则会自动调用资源的析构函数并释放内存。
2. **性能考量**
3. _**大小增加**_

与普通指针相比，std::shared_ptr通常需要额外的空间来存储引用计数的信息，因此其占用的内存通常是原始指针的两倍左右。std::shared_ptr内部不仅包含了一个指向实际资源的指针，还包含了一个指向引用计数结构的指针。

1. _**动态内存分配**_

引用计数本身是存储在独立于所管理对象的内存块中的。这意味着即使是很小的对象也可能导致一次额外的内存分配操作，以存放这个计数器。这增加内存使用的复杂性和可能的碎片化问题。不过，使用std::make_shared可以优化这一点，因为它允许在一个单独的内存分配中同时创建std::shared_ptr和它所管理的对象，从而减少内存分配次数。

1. _**线程安全的引用计数更新**_

在多线程程序中，多个线程可能同时尝试修改同一个std::shared_ptr对象的引用计数。例如，一个线程可能正在销毁一个std::shared_ptr实例（这会导致引用计数递减），而另一个线程可能正在创建一个新的std::shared_ptr实例指向同一个资源（这会导致引用计数递增）。如果这些操作不是原子性的，那么就可能发生数据竞争，即两个或多个线程同时读取和写入相同的内存位置，导致不确定的行为。为了保证引用计数的一致性和正确性，每次递增或递减操作都必须是原子的，这意味着在任何给定时间点，只有一个线程能够执行这些操作。

**<font style="color:#c21c13;">shared_ptr移动构造/复制</font>**

如果使用移动构造函数或移动赋值运算符从另一个std::shared_ptr中移动资源（而不是复制），则不会增加引用计数。相反，源std::shared_ptr将被设置为nullptr，而新的std::shared_ptr将接管对资源的所有权。移动构造函数和移动赋值运算符使得std::shared_ptr能够更高效地转移所有权，因为它们避免了引用计数的增减操作。这允许std::shared_ptr在某些情况下比使用拷贝构造函数或拷贝赋值运算符更快，尤其是在频繁进行所有权转移的情景下。

**<font style="color:#c21c13;">shared_ptr 自定义删除器</font>**

std::shared_ptr支持自定义删除器，允许用户指定如何释放所管理的对象。与std::unique_ptr不同的是，对于std::shared_ptr来说，删除器类型不是智能指针类型的一部分。这意味着即使两个std::shared_ptr实例拥有不同的删除器，它们仍然可以共享相同的类型，并且可以相互赋值或者一起存储在容器中。

代码展示使用lambda表达式作为删除器：

```cpp
auto loggingDel = [](Widget *pw) { makeLogEntry(pw); delete pw; };
std::shared_ptr<Widget> spw(new Widget, loggingDel);

auto customDeleter1 = [](Widget *pw){…};//自定义删除器，
auto customDeleter2 = [](Widget *pw){…};//每种类型不同
std::shared_ptr<Widget> pw1(new Widget,customDeleter1);
std::shared_ptr<Widget> pw2(new Widget,customDeleter2);
std::vector<std::shared_ptr<Widget>> vpw{ pw1, pw2 };
```

**<font style="color:#c21c13;">shared_ptr 控制块</font>**

另一个不同于`std::unique_ptr`的地方是，指定自定义删除器不会改变`std::shared_ptr`对象的大小。不管删除器是什么，一个`std::shared_ptr`对象都是两个指针大小。自定义删除器可以是函数对象，函数对象可以包含任意多的数据。`std::shared_ptr`怎么能引用一个任意大的删除器而不使用更多的内存?它不能。它必须使用更多的内存。然而那部分内存不是`std::shared_ptr`对象的一部分。那部分在堆上面,`std::shared_ptr`创建者利用`std::shared_ptr`对自定义分配器的支持能力，那部分内存随便在哪都行。前面提到了`std::shared_ptr`对象包含了所指对象的引用计数的指针有点误导人。因为引用计数是另一个更大的数据结构的一部分，那个数据结构通常叫做**控制块**（_control block_）。每个`std::shared_ptr`管理的对象都有个相应的控制块。控制块除了包含引用计数值外还有一个自定义删除器的拷贝，当然前提是存在自定义删除器。如果用户还指定了自定义分配器，控制块也会包含一个分配器的拷贝。

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742811188638-36f32f4b-6258-42f7-a937-06d30428f9cc.png)

**控制块的创建规则**

+ 使用std::make_shared：总是创建一个新的控制块。这是因为std::make_shared不仅分配内存给对象本身，还为控制块分配内存，并且初始化引用计数。
+ 从独占指针（如std::unique_ptr）构造：创建新的控制块。因为独占指针没有共享的控制块，所以转换成std::shared_ptr时需要创建一个。
+ 从原始指针构造：同样会创建新的控制块。如果有一个已经存在的对象，并希望创建指向它的std::shared_ptr，则应该确保只有一个控制块被创建。如果尝试从同一个原始指针创建多个std::shared_ptr，将会导致每个std::shared_ptr都有自己的控制块，从而导致对象被多次销毁。

```cpp
auto pw = new Widget;  //创建原始指针
...
    std::shared_ptr<Widget> spw1(pw, loggingDel);  // 为*pw创建第一个控制块
...
    std::shared_ptr<Widget> spw2(pw, loggingDel); // 为*pw创建第二个控制块
```

这段代码的问题在于它创建了两个独立的std::shared_ptr，每个都拥有自己的控制块。这将导致当两个std::shared_ptr都离开作用域或被重置时，它们都会尝试删除同一个对象，造成双重释放。

**正确的方法**

使用std::make_shared：

```cpp
auto spw1 = std::make_shared<Widget>();  // 使用std::make_shared创建
```

如果必须使用原始指针和自定义删除器，应直接使用new表达式的结果，而不是存储后的指针变量：

```cpp
std::shared_ptr<Widget> spw1(new Widget, loggingDel);  // 直接使用new结果
```

当你需要另一个std::shared_ptr来共享所有权时，通过拷贝现有的std::shared_ptr来避免创建新的控制块：

```cpp
std::shared_ptr<Widget> spw2(spw1);  //使用spw1的控制块.
```

在C++中，this指针是一个指向当前对象的指针。如果你尝试将this指针直接传递给std::shared_ptr的构造函数，可能会导致多重所有权问题，尤其是在Widget对象已经被std::shared_ptr管理的情况下。为了避免这种情况，可以使用std::weak_ptr或者确保通过std::shared_ptr来引用对象。

```cpp
class Widget {
public:
void process() {
    // ...
    processedWidgets.emplace_back(this);  // 错误地使用this指针
}
};

std::vector<std::shared_ptr<Widget>> processedWidgets;
```

问题：this是一个原始指针，而processedWidgets中的元素是std::shared_ptr<Widget>。如果Widget对象已经被std::shared_ptr管理，那么将this传递给emplace_back会创建一个新的std::shared_ptr，这可能导致多重控制块问题（即多个std::shared_ptr管理同一个对象）。

正确的用法：

**<font style="color:#c21c13;">（1）</font>****<font style="color:#c21c13;">使用std::weak_ptr</font>**

```cpp
#include <memory>
#include <vector>
class Widget {
public:
void process(){
    processedWidgets.emplace_back(std::weak_ptr<Widget>(std::static_pointer_cast<Widget>(self)));
}
private:
std::weak_ptr<Widget> self;  // 保存一个弱引用
};
std::vector<std::weak_ptr<Widget>> processedWidgets;
```

self是一个std::weak_ptr<Widget>，它不会增加引用计数。std::weak_ptr用于记录Widget对象，但不会影响对象的生命周期。std::static_pointer_cast用于将self转换为std::shared_ptr<Widget>，以便将其存储在processedWidgets中。

**<font style="color:#c21c13;">（2）通过std::shared_ptr引用对象</font>**

```cpp
class Widget {
public:
void process(std::shared_ptr<Widget> self) {
    //...
    processedWidgets.push_back(self);
}
};
std::vector<std::shared_ptr<Widget>> processedWidgets;
int main() {
    auto widget = std::make_shared<Widget>();
    widget->process(widget);  // 传递self
}
```

process方法接受一个std::shared_ptr<Widget>参数self。self是对Widget对象的std::shared_ptr引用，这样可以确保processedWidgets中的std::shared_ptr不会引起多重控制块问题。在调用process方法时，传递widget这个std::shared_ptr。

1. **<font style="color:#c21c13;">使用enable_shared_from_this</font>**

直接使用 this 指针创建新的 std::shared_ptr 可能会导致多重控制块问题，因为每个新创建的 std::shared_ptr 都会尝试创建自己的控制块。通过继承 std::enable_shared_from_this，类可以调用 shared_from_this() 成员函数来获取当前对象的 std::shared_ptr，而不会创建新的控制块。在类的成员函数中，可以通过调用 shared_from_this() 来获取指向当前对象的 std::shared_ptr。shared_from_this() 会查找当前对象已有的控制块，并返回一个新的 std::shared_ptr，该指针与现有控制块关联。调用 shared_from_this() 之前，必须已经有一个 std::shared_ptr 管理着当前对象。如果没有这样的 std::shared_ptr 存在，调用 shared_from_this() 将抛出异常。

**私有构造函数和工厂函数：**

为了确保对象总是通过 std::shared_ptr 创建，通常将类的构造函数设为私有，并提供一个静态工厂函数来创建并返回 std::shared_ptr。这样可以防止客户端直接创建对象，从而保证对象总是被 std::shared_ptr 管理。

```cpp
#include <memory>
#include <vector>
class Widget : public std::enable_shared_from_this<Widget> {
public：
// 工厂函数，用于创建并返回一个 std::shared_ptr<Widget>
template<typename... Ts>
static std::shared_ptr<Widget> create(Ts&&... params) {
    auto widget = std::shared_ptr<Widget>(new Widget(std::forward<Ts>(params)...));
    return widget;
}
void process() {
    // 处理 Widget
    // 使用 shared_from_this() 获取指向当前对象的 std::shared_ptr
    processedWidgets.emplace_back(shared_from_this());
}
private:
// 私有构造函数
Widget(/* 构造参数 */) {
    // 初始化
}
};
// 存储已处理的 Widget 对象
std::vector<std::shared_ptr<Widget>>processedWidgets;
int main() {
    // 通过工厂函数创建 Widget 对象
    auto widget = Widget::create(/* 参数 */);
    // 调用 process 方法
    widget->process();
    // 现在 processedWidgets 中包含了一个指向 widget 的 std::shared_ptr
    return 0;
```

**使用建议**

（1）默认配置：使用默认删除器和默认分配器，并通过 std::make_shared 创建 std::shared_ptr 时，控制块的大小和开销都较小。

（2）独占资源：如果不需要共享所有权，使用 std::unique_ptr 更合适，因为它具有接近原始指针的性能，并且可以从 std::unique_ptr 转换为 std::shared_ptr，反之则不行。

（3）std::shared_ptr 通过引用计数自动管理对象的生命周期，提供了强大的共享所有权功能。尽管有控制块、虚函数和原子操作带来的开销，但在大多数情况下，这些开销是可以接受的。对于不需要共享所有权的情况，应优先考虑使用 std::unique_ptr。

（4）使用 std::shared_ptr 管理数组时，应使用 std::shared_ptr<T[]>，而不是尝试用std::shared_ptr<T> 加自定义删除器来管理数组。

<h4 id="8b3b34c0">条款二十：当std::shared_ptr可能悬空时使用std::weak_ptr</h4>
使用shared_ptr可能出现的问题：

（1）如果从std::shared_ptr获取原始指针（通过.get()方法），然后继续使用这个原始指针，即使所有std::shared_ptr都已释放资源，原始指针仍然存在，但它指向的对象已经被销毁。原始指针就变成了悬空指针。

```cpp
std::shared_ptr<int> sp(new int(42));
int* rawPtr = sp.get();  // 获取原始指针
sp.reset();  // 释放sp持有的资源
// 此时rawPtr是悬空指针
```

_**<u>多个线程操作</u>**_

在多线程环境中，如果一个线程释放了std::shared_ptr，而另一个线程还在使用从之前共享的std::shared_ptr得到的原始指针，那么后者的指针也可能变成悬空指针。

_**<u>循环引用</u>**_

如果两个或多个std::shared_ptr之间存在循环引用，并且没有其他方式打破这个循环，这些std::shared_ptr将永远不会释放它们所指向的对象，从而可能导致内存泄漏。虽然这不是悬空指针的问题，但是它显示了std::shared_ptr需要谨慎使用的场景之一。在这种情况下，可以考虑使用std::weak_ptr来避免循环引用。

_**<u>异常</u>**_

在函数内部创建std::shared_ptr并在抛出异常前返回它时，如果异常处理逻辑不正确，可能会导致局部作用域内的std::shared_ptr被销毁，而外部代码可能还持有相关的原始指针，从而导致悬空指针。

为了解决这个问题，C++提出了std::weak_ptr。std::weak_ptr是一个与std::shared_ptr相关的类，它不会增加所指向对象的引用计数。即使没有std::shared_ptr实例持有对象，只要存在std::weak_ptr，对象也不会因为引用计数归零而被销毁。但是，一旦所有std::shared_ptr都释放了该对象，std::weak_ptr就会变成过期状态（expired），此时尝试访问对象是不安全的。

（1）创建std::weak_ptr

```cpp
auto spw = std::make_shared<Widget>(); // 创建一个 shared_ptr，引用计数为 1
std::weak_ptr<Widget> wpw(spw);        // 创建一个 weak_ptr，指向相同的 Widget 对象，但不增加引用计数
```

（2）检查是否过期

```cpp
if (wpw.expired()) {
    // wpw 已经过期，表示没有有效的 shared_ptr 指向该对象
}
```

（3）安全地转换为 std::shared_ptr:

使用 lock() 方法，如果 std::weak_ptr 过期则返回空的 std::shared_ptr。

```cpp
std::shared_ptr<Widget> spw1 = wpw.lock();  // 如果 wpw 过期，spw1 将为空
```

或者直接构造 std::shared_ptr，如果 std::weak_ptr 过期则抛出 std::bad_weak_ptr 异常。

```cpp
try {
    std::shared_ptr<Widget> spw3(wpw);  // 如果 wpw 过期，这里会抛出异常
} catch (const std::bad_weak_ptr& e) {
    // 处理异常
}
```

避免竞态条件:

由于从检查是否过期到实际使用对象之间可能存在竞态条件，因此推荐使用原子操作，如 lock() 来确保安全地获取共享所有权。

引用场景：缓存和观察者设计模式。

**<font style="color:#c21c13;">缓存场景</font>**

有一个工厂函数loadWidget(WidgetID id)，它根据唯一ID加载一个只读对象，并返回一个std::unique_ptr<const Widget>。

加载操作可能非常昂贵（例如涉及文件或数据库I/O）。

为了优化性能,希望实现一个缓存机制来存储已经加载的对象，以避免重复加载相同的对象。

_**<u>（1）为什么使用std::shared_ptr而不是std::unique_ptr？</u>**_

std::unique_ptr是独占所有权的智能指针，意味着同一时间只能有一个std::unique_ptr指向同一个对象。这不适合缓存场景，因为缓存需要多个引用同时访问同一个对象。

std::shared_ptr允许多个智能指针共享同一个对象的所有权，通过引用计数来管理对象的生命周期。这使得缓存可以安全地存储std::shared_ptr，并且当没有其他std::shared_ptr引用该对象时，对象会被自动销毁。

_**<u>（2）为什么使用std::weak_ptr作为缓存条目？</u>**_

如果缓存直接使用std::shared_ptr来存储对象，那么即使所有客户端都释放了它们的std::shared_ptr，缓存中的std::shared_ptr仍然会保持对象的存活，导致内存泄漏。std::weak_ptr不会增加对象的引用计数，可以用来检测对象是否已经被销毁。当所有std::shared_ptr都释放了对象后，std::weak_ptr将过期，此时我们可以从缓存中移除这些过期的条目。

```cpp
std::shared_ptr<const Widget> fastLoadWidget(WidgetID id) {
    static std::unordered_map<WidgetID,std::weak_ptr<const Widget>>cache;
    auto objPtr = cache[id].lock();  // 尝试获取缓存中的对象
    if (!objPtr) {  // 如果对象不在缓存中或者已经过期
        objPtr = loadWidget(id);  // 从工厂函数加载新的对象
        cache[id] = objPtr;  // 将新对象添加到缓存
    }
    return objPtr;
}
```

**<font style="color:#c21c13;">观察者设计模式</font>**

在观察者模式中，有两类对象：被观察者（subject）和观察者（observer）。当被观察者的状态发生变化时，它需要通知所有的观察者。被观察者不应该控制观察者的生命周期，但应该确保在观察者被销毁后不再尝试访问它。

为什么使用std::weak_ptr？

如果被观察者持有观察者的std::shared_ptr，则会导致观察者的生命周期被不必要地延长，即使没有其他地方还在使用这个观察者。使用std::weak_ptr存储观察者，可以在每次通知之前检查std::weak_ptr是否已经过期，从而确保不会访问已经被销毁的观察者。

```cpp
class Subject {
public:
void addObserver(const std::shared_ptr<Observer>& observer) {
    observers.push_back(observer);
}
void notifyObservers() {
    for (auto it = observers.begin(); it != observers.end();) {
        if (auto sharedObserver = it->lock()) {
            sharedObserver->update(this);  // 通知观察者
            ++it;
        } else {
            it = observers.erase(it);  // 移除过期的观察者
        }
    }
}
private:
std::vector<std::weak_ptr<Observer>> observers;  // 存储观察者的弱指针
};
```

Subject类维护了一个std::vector<std::weak_ptr<Observer>>，用于存储其观察者的弱指针。在notifyObservers方法中，它遍历这个向量，并且对于每个std::weak_ptr，都会尝试将其转换为std::shared_ptr。如果转换成功，则调用相应的update方法；如果转换失败（即std::weak_ptr已经过期），则从向量中移除该条目。这样可以确保只有那些仍然有效的观察者才会收到通知。

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742811188974-a7e42702-c509-4e50-8f59-9fd34b29a2ea.png)

假定从B指向A的指针也很有用。应该使用哪种指针？

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742811189007-4ca15395-7ad3-48fd-a3cb-483459dc7c3e.png)

在考虑持有三个对象A、B、C的数据结构时，其中A和C共享B的所有权，并且从B指向A的指针也很有用的情况下，我们有三种选择来实现这个指针：原始指针、std::shared_ptr 和 std::weak_ptr。每种选

原始指针:

优点: 简单直接。

缺点: 如果A被销毁而C继续存在，B中的指向A的指针会变成悬空指针。这可能导致未定义行为，因为B不知道A已经被销毁，可能会尝试访问已经无效的内存。

std::shared_ptr:

优点: 可以确保只要有一个std::shared_ptr引用A，A就不会被销毁。

缺点: 如果A和B互相持有对方的std::shared_ptr，会导致循环引用问题。即使没有其他部分代码需要A或B了（例如C不再指向B），A和B的引用计数仍然为1，导致它们无法被销毁，从而引起内存泄漏。

std::weak_ptr:

优点: 避免了原始指针和std::shared_ptr的问题。如果A被销毁，B可以通过std::weak_ptr检测到A是否已经被销毁。这样，B可以安全地检查std::weak_ptr的状态，避免访问已被销毁的对象。此外，std::weak_ptr不会增加A的引用计数，因此不会阻止A的正常销毁。

缺点: 使用起来稍微复杂一些，因为它需要额外的步骤来锁定std::shared_ptr。

最佳实践: 在这种情况下，使用std::weak_ptr是从B指向A的最佳选择。它既解决了原始指针带来的悬空指针问题，也避免了std::shared_ptr可能引起的循环引用和内存泄漏问题。

**<font style="color:#c21c13;">控制块</font>**

std::shared_ptr和std::weak_ptr都使用控制块的数据结构来管理对象的生命周期。

_<u>强引用计数:</u>_

记录当前有多少个std::shared_ptr指向这个对象。每次创建一个新的std::shared_ptr指向同一个对象时，强引用计数增加1；每当一个std::shared_ptr被销毁或重置时，强引用计数减少1。当强引用计数降到0时，对象会被删除，并且控制块中的析构函数会被调用。

_<u>弱引用计数:</u>_

记录当前有多少个std::weak_ptr指向这个对象。每次创建一个新的std::weak_ptr指向同一个对象时，弱引用计数增加1；每当一个std::weak_ptr被销毁或重置时，弱引用计数减少1。即使所有std::shared_ptr都被销毁了，只要还有std::weak_ptr存在，控制块就会保持不被删除，直到所有的std::weak_ptr也都被销毁。

_<u>析构函数(Deleter)</u>_

当最后一个std::shared_ptr被销毁时，用来释放资源的对象。

**weak_ptr的操作对计数的影响：**

构造：当从std::shared_ptr创建std::weak_ptr时，不会影响强引用计数，但会增加弱引用计数。

析构：当std::weak_ptr被销毁时，弱引用计数减少1。如果此时弱引用计数变为0，并且强引用计数也为0，那么控制块会被删除。

赋值：将一个std::weak_ptr赋值给另一个std::weak_ptr时，源std::weak_ptr的弱引用计数减少1，目标std::weak_ptr的弱引用计数增加1。

锁定（lock）：std::weak_ptr的lock方法尝试获取一个std::shared_ptr。如果成功，返回一个有效的std::shared_ptr；如果失败（即对象已经被销毁），则返回一个空的std::shared_ptr。

效率比较:std::shared_ptr和std::weak_ptr在大多数实现中具有相同的大小，因为它们都存储了一个指向控制块的指针。由于两者都涉及引用计数的更新，这些操作通常是原子的，以确保线程安全。因此，在多线程环境中，它们的性能开销是相似的。

<h4 id="ee78a6f1">条款二十一：优先考虑使用std::make_unique和std::make_shared，而非直接使用new</h4>
`std::make_shared`是C++11标准,`std::make_unique`是C++14标准。一个基础版本的`std::make_unique`很容易自己写出的

```cpp
template<typename T, typename... Ts>
std::unique_ptr<T> make_unique(Ts&&... params){
    return std::unique_ptr<T>(new T(std::forward<Ts>(params)...));
}
```

`make_unique`只是将它的参数完美转发到所要创建的对象的构造函数，从`new`产生的原始指针里面构造出`std::unique_ptr`，并返回这个`std::unique_ptr`。这种形式的函数不支持数组和自定义析构，但它给出了一个示范：只需一点努力就能写出你想要的`make_unique`函数。

`std::make_unique`和`std::make_shared`是三个**make函数** 中的两个：接收任意的多参数集合，完美转发到构造函数去动态分配一个对象，然后返回这个指向这个对象的指针。

第三个`make`函数是`std::allocate_shared`。它行为和`std::make_shared`一样，只不过第一个参数是用来动态分配内存的_allocator_对象。

使用std::make_unique和std::make_shared函数来创建智能指针

_**<u>1. 避免重复代码</u>**_

使用make函数可以避免在代码中重复写类型名称，这符合DRY（Don't Repeat Yourself）原则。

```cpp
// 使用 make 函数
auto upw1 = std::make_unique<Widget>();
auto spw1 = std::make_shared<Widget>();
// 不使用 make 函数
std::unique_ptr<Widget> upw2(new Widget);
std::shared_ptr<Widget> spw2(new Widget);
```

在上面的代码中，upw1 和 spw1 的定义只提到了一次Widget类型，而upw2 和 spw2 的定义则需要两次。

_**<u>2. 异常安全性</u>**_

直接使用new可能会导致资源泄漏，尤其是在构造函数抛出异常的情况下。make函数可以确保即使在异常情况下也能正确管理资源。

```cpp
void processWidget ( std::shared_ptr<Widget> spw，int priority);
int computePriority();
// 潜在的资源泄漏
processWidget(std::shared_ptr<Widget>(new Widget), computePriority());
// 使用 make 函数，保证异常安全
processWidget(std::make_shared<Widget>(), computePriority());
```

如果computePriority()抛出异常，在第一种情况下，new Widget分配的内存不会被释放。

假设computePriority()抛出了异常，那么执行流程如下：new Widget被执行，一个Widget对象在堆上被创建。然后尝试调用computePriority()来计算优先级。如果computePriority()抛出异常，在这个异常被捕获之前，std::shared_ptr<Widget>(new Widget)还没有机会完成其构造函数，因此它还不能管理刚刚创建的Widget对象。异常处理机制会跳过剩余的代码（包括std::shared_ptr的构造），直接进入异常处理代码。结果是new Widget分配的内存没有被任何std::shared_ptr管理，因此无法自动释放，造成内存泄漏。

_**使用 std::make_shared 的情况**_

在这种情况下，std::make_shared<Widget>()是一个单一操作，它同时负责创建Widget对象和初始化std::shared_ptr。这意味着即使在computePriority()抛出异常的情况下，也不会发生内存泄漏。

std::make_shared<Widget>()开始执行，它会在堆上分配一块内存用于Widget对象及其控制块。std::make_shared已经完成了对Widget对象的构造，并且std::shared_ptr也已经被正确初始化，可以管理这块内存。接着调用computePriority()来计算优先级。如果computePriority()抛出异常，由于std::make_shared已经成功创建了std::shared_ptr，并且这个智能指针已经开始管理Widget对象，所以在异常处理过程中，std::shared_ptr的析构函数会被调用，从而释放Widget对象占用的内存。因此，即使有异常抛出，Widget对象的内存也会被正确释放，避免了内存泄漏。

_**<font style="color:#c21c13;">3. 性能提升</font>**_

std::make_shared可以通过一次性分配内存来减少内存分配次数，从而提高效率。控制块和对象本身都存储在同一块内存区域中，这减少了内存碎片，并且提高了内存访问速度。

```cpp
// 可能进行两次内存分配
std::shared_ptr<Widget>spw1(new Widget);
// 只进行一次内存分配
auto spw2 = std::make_shared<Widget>();
```

_**<font style="color:#c21c13;">4. make函数的限制</font>**_

自定义删除器：make函数不支持传递自定义删除器。如果需要指定自定义删除器，则必须直接使用new。

```cpp
auto widgetDeleter = [](Widget* pw) { /* ... */ };
std::unique_ptr<Widget, decltype(widgetDeleter)> upw(new Widget, widgetDeleter);
```

花括号初始化：make函数不能直接处理花括号初始化。如果要使用花括号初始化，你需要先创建一个std::initializer_list，然后将其传递给make函数。

```cpp
// 花括号初始化
auto initList = { 10, 20 };
auto spv = std::make_shared<std::vector<int>>(initList); // 通过 initializer_list 创建
```

当类重载了operator new和operator delete时，这些自定义的内存管理函数通常只处理特定大小的内存块（通常是对象本身的大小）。这与std::shared_ptr通过std::allocate_shared进行的内存分配不同，后者需要额外的空间来存放控制块。因此，在这种情况下使用std::make_shared可能会导致问题，因为std::make_shared尝试一次性分配足够的内存来容纳对象本身及其控制块。

重载 operator new 和 operator delete 的类

假设有一个类Widget重载了operator new和operator delete，并且这些操作符只处理sizeof(Widget)大小的内存块。

```cpp
class Widget {
public:
void* operator new(size_t size) {
    // 自定义内存分配逻辑
    return malloc(sizeof(Widget));
}
void operator delete(void* p) noexcept {
    // 自定义内存释放逻辑
    free(p);
}
};
```

在这种情况下，如果使用std::make_shared<Widget>()，它会尝试分配比sizeof(Widget)更大的内存块，这可能与Widget的自定义内存分配逻辑不兼容。

_使用 new 创建对象并传递给 std::shared_ptr_

为了确保异常安全，并且在使用自定义删除器的情况下，可以将对象的创建和智能指针的构造分开，以避免潜在的内存泄漏。例如：

```cpp
void processWidget(std::shared_ptr<Widget> spw, int priority);
int computePriority();
void cusDel(Widget *ptr);  // 自定义删除器
// 非异常安全调用
processWidget(std::shared_ptr<Widget>(new Widget, cusDel), computePriority());
// 异常安全调用
std::shared_ptr<Widget> spw(new Widget, cusDel);
processWidget(spw, computePriority());
```

在这个例子中，spw的构造函数在单独的语句中执行，确保即使computePriority()抛出异常，Widget对象也会被正确地销毁。

_**<u>性能优化：使用 std::move</u>**_

为了提高性能，可以使用std::move将spw转换为右值，从而允许processWidget函数内部进行移动而非拷贝。

```cpp
processWidget(std::move(spw), computePriority());  // 高效且异常安全
```

这里性能提高了多少？

_**<u>大型对象的内存延迟释放</u>**_

对于大型对象，如果使用std::make_shared，则对象的内存会在最后一个std::shared_ptr和最后一个std::weak_ptr都被销毁后才释放。这意味着在对象被销毁和内存实际释放之间可能会有一段延迟。

```cpp
class ReallyBigType { /* ... */ };
auto pBigObj = std::make_shared<ReallyBigType>();  // 通过std::make_shared创建大对象
// ... 使用pBigObj
// 最后一个std::shared_ptr在这里销毁，但std::weak_ptrs还在
// 在这个阶段，原来分配给大对象的内存还分配着
// 最后一个std::weak_ptr在这里销毁；控制块和对象的内存被释放
```

相比之下，直接使用new创建对象时，一旦最后一个std::shared_ptr被销毁，对象的内存就会立即释放，而只有控制块的内存保持分配状态直到最后一个std::weak_ptr也被销毁。

```cpp
class ReallyBigType { /* ... */ };
std::shared_ptr<ReallyBigType> pBigObj(new ReallyBigType);  // 通过new创建大对象
// ... 使用pBigObj
// 最后一个std::shared_ptr在这里销毁，但std::weak_ptrs还在；对象的内存被释放
// 在这个阶段，只有控制块的内存仍然保持分配
// 最后一个std::weak_ptr在这里销毁；控制块内存被释放
```

**总结：**

+ 和直接使用`new`相比，`make`函数消除了代码重复，提高了异常安全性。对于`std::make_shared`和`std::allocate_shared`，生成的代码更小更快。
+ 不适合使用`make`函数的情况包括需要指定自定义删除器和希望用花括号初始化。
+ 对于`std::shared_ptr`s，其他不建议使用`make`函数的情况包括(1)有自定义内存管理的类；(2)特别关注内存的系统，非常大的对象，以及`std::weak_ptr`s比对应的`std::shared_ptr`s活得更久。

<h4 id="df5d80a3">条款二十二：当使用Pimpl惯用法，请在实现文件中定义特殊成员函数</h4>
Pimpl惯用法是一种用于减少编译依赖并隐藏实现细节的技术。它通过将类的数据成员移动到一个私有结构体中，并且在主类中仅保留指向该结构体的指针，从而达到封装的目的。这样做的好处是使用者不需要包含那些定义了数据成员类型的头文件，减少了编译时间和依赖性。通过将Impl的定义移到.cpp文件中，减少了用户代码对<string>、<vector>和gadget.h等头文件的直接依赖，从而减少了编译时间。

传统方式

在C++98中，你可以使用原始指针来实现Pimpl惯用法。

```cpp
// widget.h
#ifndef WIDGET_H
#define WIDGET_H

#include <string>
#include <vector>
#include "gadget.h"  // 假设Gadget类型定义在此头文件中

class Widget {
public:
Widget();
~Widget();

private:
struct Impl; // 声明但未定义
Impl* pImpl; // 指向实现结构体的指针
};

#endif // WIDGET_H

// widget.cpp
#include "widget.h"
struct Widget::Impl {  // 实现结构体的定义
std::string name;
std::vector<double> data;
Gadget g1, g2, g3;
};
Widget::Widget() : pImpl(new Impl) {}  // 构造函数

Widget::~Widget() {  // 析构函数
    delete pImpl;  // 释放内存
}
```

封装性：通过将数据成员移到私有结构体Impl中，可以隐藏这些数据成员的具体类型和实现细节。这样即使Impl的内部发生变化，只要对外接口不变，用户代码就不需要重新编译。

减少编译依赖：由于Impl是在.cpp文件中定义的，所以用户不需要包含那些定义了std::string、std::vector或Gadget的头文件，从而减少了编译时间和依赖性。

使用原始指针时，需要手动管理内存，如在构造函数中分配，在析构函数中释放。这虽然有效，但不如智能指针安全。如果想要提高安全性，可以考虑使用std::unique_ptr替代原始指针。

但是直接替换原始指针会导致编译错误，因为std::unique_ptr的默认删除器需要完整类型才能工作。因此，需要确保在销毁std::unique_ptr之前，Impl已经是一个完整类型。

```cpp
// widget.h
class Widget {
public:
Widget();
~Widget();//只声明析构函数
Widget(Widget&&) = default;//移动构造
Widget& operator=(Widget&&) = default;//移动赋值
private:
struct Impl;//声明但未定义
std::unique_ptr<Impl> pImpl;//使用智能指针
};
// widget.cpp
#include "widget.h"
#include "gadget.h"
#include <string>
#include <vector>
struct Widget::Impl { // 定义实现结构体
std::string name;
std::vector<double> data;
Gadget g1, g2, g3;
};

Widget::Widget() : pImpl(std::make_unique<Impl>()) {}
Widget::~Widget() = default; // 定义析构函数

// 如果需要支持复制操作，还需添加.
Widget::Widget(const Widget& rhs):
pImpl(std::make_unique<Impl>(*rhs.pImpl)){}

Widget& Widget::operator=(const Widget& rhs) {
    *pImpl=*rhs.pImpl;
    return *this;
}
```

Pimpl惯用法：通过将数据成员移至一个私有的、不完整的结构体中，并在主类中保持一个指向该结构体的指针，可以减少编译依赖。为了更安全地管理资源，可以使用std::unique_ptr替代原始指针。

在Pimpl惯用法中，Impl结构体通常只在类的私有部分进行声明，而在头文件中并没有给出它的定义。这样做的目的是为了隐藏实现细节，并减少编译依赖。但是，这会导致一个问题：如果std::unique_ptr<Impl>试图销毁一个Impl对象，而此时Impl仍然是一个不完整类型，那么编译器就会报错，因为不知道如何正确地销毁这个对象。

为了解决这个问题，需要确保在std::unique_ptr<Impl>尝试销毁Impl对象之前，Impl已经被定义为完整类型。

.cpp文件中定义Impl：

.cpp文件中任何可能使用std::unique_ptr<Impl>的地方之前，先定义Impl结构体。当std::unique_ptr<Impl>尝试销毁Impl对象时，编译器已经有了完整的类型信息。

在头文件中声明析构函数，但不要在头文件中定义它。在.cpp文件中定义析构函数，这样可以确保在析构函数被执行时，Impl已经是一个完整类型。如果使用std::unique_ptr，需要显式地声明析构函数并在实现文件中定义它，以确保在销毁std::unique_ptr前Impl已被定义。对于移动操作，也需采取类似措施。

使用了Pimpl惯用法的类自然适合支持移动操作，因为编译器自动生成的移动操作正合我们所意：对其中的`std::unique_ptr`进行移动。 正如[Item17](https://cntransgroup.github.io/EffectiveModernCppChinese/3.MovingToModernCpp/item17.html)所解释的那样，声明一个类`Widget`的析构函数会阻止编译器生成移动操作，所以如果你想要支持移动操作，必须自己声明相关的函数。考虑到编译器自动生成的版本会正常运行，可能会按如下方式实现它们：

```cpp
#include"widget.h"
#include"gadget.h"
#include<string>
#include<vector>

struct Widget::Impl {
//跟之前一样，定义Widget::Impl
std::string name;
std::vector<double> data;
Gadget g1,g2,g3;
}

Widget::Widget()//跟之前一样
: pImpl(std::make_unique<Impl>())
{}

Widget::~Widget()//析构函数的定义
{}
```

这样的做法会导致同样的错误，和之前的声明一个不带析构函数的类的错误一样，并且是因为同样的原因。 编译器生成的移动赋值操作符，在重新赋值之前，需要先销毁指针`pImpl`指向的对象。然而在`Widget`的头文件里，`pImpl`指针指向的是一个不完整类型。移动构造函数的情况有所不同。 移动构造函数的问题是编译器自动生成的代码里，包含有抛出异常的事件，在这个事件里会生成销毁`pImpl`的代码。然而，销毁`pImpl`需要`Impl`是一个完整类型。

因为这个问题同上面一致，解决方案：移动操作的定义移动到实现文件里：

```cpp
class Widget {                          //仍然在“widget.h”中
public:
Widget();
~Widget();
Widget(Widget&& rhs);               //只有声明
Widget& operator=(Widget&& rhs);
private:                                //跟之前一样
struct Impl;
std::unique_ptr<Impl> pImpl;
};

#include <string>                   //跟之前一样，仍然在“widget.cpp”中
…

struct Widget::Impl { … };          //跟之前一样

Widget::Widget()                    //跟之前一样
: pImpl(std::make_unique<Impl>())
{}

Widget::~Widget() = default;        //跟之前一样

Widget::Widget(Widget&& rhs) = default;             //这里定义
Widget& Widget::operator=(Widget&& rhs) = default;
```

使用std::shared_ptr作为替代方案时的一些不同之处

**<font style="color:#c21c13;">头文件 (widget.h)</font>**

```cpp
#ifndef WIDGET_H
#define WIDGET_H
#include <memory>
#include <string>
#include <vector>
#include "gadget.h"// 假设Gadget类定义在此头文件中
class Widget {
public:
Widget();
~Widget();  // 只声明析构函数
// 拷贝构造函数和赋值操作符
Widget(const Widget& rhs);
Widget& operator=(const Widget& rhs);
private:
struct Impl;  // 声明但未定义
std::unique_ptr<Impl> pImpl;  // 使用智能指针
};
#endif // WIDGET_H
```

Widget类声明了构造函数、析构函数以及拷贝构造函数和赋值操作符。Impl结构体仅被声明而没有定义，以隐藏实现细节。pImpl是一个指向Impl的unique_ptr用于独占所有权。

**实现文件 (widget.cpp)**

```cpp
#include "widget.h"
#include <memory>  // 包含std::make_unique
#include <string>
#include <vector>
#include "gadget.h"  // 包含Gadget类定义
// 定义Impl结构体
struct Widget::Impl {
std::string name;
std::vector<double> data;
Gadget g1,g2,g3;
};
// 构造函数
Widget::Widget() : pImpl(std::make_unique<Impl>()) {}
// 析构函数
Widget::~Widget() = default;
// 拷贝构造函数
Widget::Widget(const Widget& rhs) : pImpl(std::make_unique<Impl>(*rhs.pImpl)) {}
// 赋值操作符
Widget& Widget::operator=(const Widget& rhs) {
    if (this != &rhs) {  // 防止自赋值
        *pImpl = *rhs.pImpl;  // 利用编译器生成的Impl的复制操作
    }
    return *this;
}
```

拷贝构造函数和赋值操作符：由于std::unique_ptr是移动语义的(即只可移动),所以需要手动实现拷贝构造函数和赋值操作符。在拷贝构造函数中，使用std::make_unique创建一个新的Impl对象，并将其初始化为rhs.pImpl所指向的对象的副本。在赋值操作符中，检查自赋值情况，并使用*pImpl = *rhs.pImpl进行深拷贝。

析构函数：析构函数在头文件中声明，在实现文件中定义为默认析构函数~Widget() = default;。这确保了在销毁Widget对象时，pImpl所指向的内存会被正确释放。

使用std::shared_ptr的差异：如果使用std::shared_ptr而不是std::unique_ptr，则不需要在头文件中声明析构函数，因为std::shared_ptr的删除器类型不是智能指针的一部分，因此它可以在不完整类型上工作。std::shared_ptr会导致更大的运行时开销，并且通常用于共享所有权的情况，而Widget和Impl之间的关系更适合独占所有权。

std::unique_ptr 与 std::shared_ptr 的区别

_**<u><font style="color:#c21c13;">所有权模型</font></u>**_

unique_ptr提供独占所有权，一个对象只能由一个std::unique_ptr管理。当unique_ptr被销毁或重置时，它所管理的对象也会被销毁。shared_ptr提供共享所有权，允许多个std::shared_ptr实例共同管理同一个对象。只有当所有shared_ptr实例都被销毁时，它们所管理的对象才会被销毁。

_**<u><font style="color:#c21c13;">删除器类型</font></u>**_

unique_ptr默认的删除器是delete操作符，但用户可以自定义删除器。删除器类型是unique_ptr的一部分，因此编译器需要知道完整的类型信息来生成正确的删除代码。

shared_ptr默认的删除器也是delete操作符，但删除器类型不是std::shared_ptr的一部分。即使类型不完整，std::shared_ptr也可以工作，因为它可以在运行时动态地确定删除器。

unique_ptr由于独占所有权，不需要维护引用计数，因此具有较小的运行时开销。shared_ptr由于需要维护引用计数，因此具有较大的运行时开销。每个std::shared_ptr实例都需要额外的内存来存储引用计数，并且每次创建或销毁std::shared_ptr时都需要更新引用计数。

_**<u>使用std::shared_ptr 时不需要声明析构函数的原因</u>**_

由于std::unique_ptr的删除器类型是其一部分，编译器需要知道完整的类型信息来生成正确的删除代码。因此，在头文件中声明析构函数并在实现文件中定义它是非常重要的。这样可以确保在析构函数执行时，Impl已经是一个完整类型。

由于std::shared_ptr的删除器类型不是其一部分，编译器不需要完整的类型信息来生成删除代码。因此，即使Impl仍然是不完整类型，std::shared_ptr也可以正常工作。这意味着你不需要在头文件中声明析构函数，因为编译器生成的默认析构函数可以正确地处理std::shared_ptr的销毁。

_**<font style="color:#c21c13;">适用场景</font>**_

虽然std::shared_ptr可以用于Widget和Impl之间的关系，但由于其较大的运行时开销和不必要的共享所有权特性，通常不推荐在这种情况下使用。

<h1 id="18bbb752">第五章 右值引用/移动语义/完美转发</h1>
<h4 id="5bea880c">条款二十三：理解std::move和std::forward</h4>
可以从它们不做什么理解。`<u>std::move</u>`<u>不移动任何东西，</u>`<u>std::forward</u>`<u>也不转发任何东西。</u>

在运行时，它们不做任何事情。它们不产生任何可执行代码，一字节也没有。`<u>std::move</u>`<u>和</u>`<u>std::forward</u>`<u>仅仅是执行转换(cast)的函数(事实上是函数模板)。</u>

`<u>std::move</u>`<u>无条件的将它的实参转换为右值，而</u>`<u>std::forward</u>`<u>只在特定情况满足时下进行转换。从根本上而言,这就是全部内容。</u>

C++11的`std::move`的示例实现。它并不完全满足标准细则，但是它已经非常接近了。

```cpp
template<typename T>//在std命名空间
typename remove_reference<T>::type&& move(T&& param){
    using ReturnType = typename remove_reference<T>::type&&;//别名声明，见条款9
    return static_cast<ReturnType>(param);
}
```

`std::move`接受一个对象的通用引用(见item24)，返回一个指向同对象的引用。该函数返回类型的`&&`部分表明`std::move`函数返回的是一个右值引用，但是，正如[Item28](https://cntransgroup.github.io/EffectiveModernCppChinese/5.RRefMovSemPerfForw/item28.html)所解释的那样，如果类型`T`恰好是一个左值引用，<u>那么</u>`<u>T&&</u>`<u>将会成为一个左值引用。为了避免如此，</u>_<u>type trait </u>_<u>(见item 9)</u>`<u>std::remove_reference</u>`<u>应用到了类型</u>`<u>T</u>`<u>上，因此确保了</u>`<u>&&</u>`<u>被正确的应用到了一个不是引用的类型上。这保证了</u>`<u>std::move</u>`<u>返回的真的是右值引用，因为函数返回的右值引用是右值。因此，</u>`<u>std::move</u>`<u>将它的实参转换为一个右值</u>。

`std::move`在C++14中可以被更简单地实现，使用函数返回值类型推导（见[Item3](https://cntransgroup.github.io/EffectiveModernCppChinese/1.DeducingTypes/item3.html)）和标准库的模板别名`std::remove_reference_t`（见[Item9](https://cntransgroup.github.io/EffectiveModernCppChinese/3.MovingToModernCpp/item9.html)），`std::move`可以这样写：

```cpp
template<typename T>
decltype(auto) move(T&& param)          //C++14，仍然在std命名空间
{
    using ReturnType = remove_referece_t<T>&&;
    return static_cast<ReturnType>(param);
}
```

`std::move`它只进行转换，不移动任何东西。右值是移动操作的候选者，所以对一个对象使用`std::move`就是告诉编译器，这个对象很适合被移动。`std::move`告诉编译器指定可以被移动的对象。

**返回值能直接使用auto吗?**

将返回类型改为 auto：

```cpp
template<typename T>
auto move(T&& param)
{
    using ReturnType = remove_reference_t<T>&&;
    return static_cast<ReturnType>(param);
}
```

auto 会忽略表达式的引用和 const/volatile 限定符，只保留其基础类型。static_cast<ReturnType>(param) 的结果是一个右值引用（T&&），但 auto 会将其退化为非引用类型 T。返回类型不再是右值引用（T&&），而是普通的值类型（T）。这会导致对象被拷贝而不是被移动，违背了 std::move 的初衷。

在C++中，std::move 是一个用于将对象转换为右值引用的工具，它通常用来提示编译器可以对对象进行移动语义操作。

然而，当涉及到 const 对象时，事情会变得有些复杂。

```cpp
class Annotation {
public:
//原始版本:使用值传递
explicit Annotation(std::string text):value(text){}
//修改后的版本:使用 const 引用传递
explicit Annotation(const std::string& text):value(std::move(text)) { }
private:
std::string value;
}
```

Annotation(std::string text) 接受一个 std::string 的副本。这里 text 是一个临时变量，它被复制到 value 中。如果 text 是一个临时变量（例如，通过 new Annotation("Some Text") 创建），那么这个复制可以通过移动语义优化。

修改后的构造函数：

Annotation(const std::string& text) 接受一个 const std::string&，这是一个常量引用。使用 std::move(text) 尝试将 text 转换为右值引用。然而，因为 text 是 const，所以即使使用了 std::move，它仍然是一个 const 右值引用。std::string 的移动构造函数接受非 const 的右值引用 (std::string&&)，因此不能被调用。编译器选择调用拷贝构造函数，因为它可以接受 const 右值引用。

结论

（1）<u><font style="background-color:#fbf5b3;">不要对 const 对象使用 std::move：如果希望利用移动语义，不应该声明参数为 const。否则，std::move 会静默地退化为拷贝操作。</font></u>

（2）s<u><font style="background-color:#fbf5b3;">td::move 不保证移动：std::move 只是将对象转换为右值引用，但并不保证实际的移动会发生。如果对象类型不支持或不允许移动（如 const 对象），则可能会发生拷贝而非移动。</font></u>

为了确保移动语义，如果确实需要移动对象，应该避免将其声明为 const。如果不需要修改传入的对象，同时又希望保留移动语义的可能性，可以考虑使用非常量引用：

```cpp
explicit Annotation(std::string&& text) : value(std::move(text)) { }
```

_**<u><font style="color:#c21c13;">std::move 与 std::forward 的区别</font></u>**_

<u>std::move 是一个无条件的转换，它总是将它的参数转换为右值引用（rvalue reference）。这个操作本身并不移动任何东西；它只是允许对象被移动构造或移动赋值，通常用于当你确定不再需要一个对象，并且希望将其资源转移给另一个对象时。</u>

<u><font style="background-color:#fbf5b3;">std::forward 是一个有条件的转换，只有当传递给它的参数是右值时，它才会将参数转换为右值引用。如果参数是左值，则保持其左值属性。std::forward 主要用于模板函数中，以确保参数在转发到其他函数时保留其原始的值类别（左值还是右值）。</font></u>

std::forward 的典型用法

考虑一个模板函数 logAndProcess，它接收一个通用引用参数 param 并将其传递给 process 函数。<u>process 函数有</u>**<u><font style="color:#c21c13;">两个重载版本</font></u>**<u>：</u>**<u><font style="color:#c21c13;">一个处理左值引用，另一个处理右值引用</font></u>****<u>。</u>****<u><font style="color:#c21c13;">为了正确地调用相应的 process 版本，我们需要使用 std::forward 来保持 param 的原始值类别</font></u>**。

```cpp
template<typename T>
void logAndProcess(T&& param) {
    // 记录日志
    makeLogEntry("Calling 'process'", std::chrono::system_clock::now());
    // 转发参数到 process
    process(std::forward<T>(param));
}
```

当 logAndProcess 用左值调用时，param 应该作为左值传递给 process；当用右值调用时，param 应该作为右值传递。std::forward 通过检查 T 是否包含引用类型来决定是否进行转换。

使用 std::move 当你需要明确地进行移动操作。使用 std::forward 在模板中转发参数时，保持参数的原始值类别。std::move 和 std::forward 都是在编译期起作用的，不会在运行时产生额外开销。

<u><font style="background-color:#fbf5b3;">在模板函数 logAndProcess 中，如果不使用 std::forward<T>(param) 而直接使用 param，那么无论传递给 logAndProcess 的是左值还是右值，param 都会被视为左值。这是因为函数参数总是左值，即使它是一个通用引用（universal reference）。</font></u>

<h6 id="c829e17d">**🔅****思考1：如果没有右值和左值的重载版本，是否需要使用 **`**std::forward**`**？**</h6>
    - 如果目标函数（如 `process`）的参数是按值传递或需要移动语义优化，即使没有重载版本，使用 `std::forward` 仍然有意义。例如：

void process(Widget w);// 按值传递

当 `logAndProcess` 接收右值时，`std::forward` 会将参数转换为右值，触发移动构造；若直接传递 `param`（左值），则会调用拷贝构造。此时 `std::forward` 可优化性能。

    - 如果目标函数仅接受左值引用（如 `void process(Widget&);`），则无需 `std::forward`，因为无论原始参数是左值还是右值，`param` 始终被视为左值。
    - **通用建议**：即使没有重载，使用 `std::forward` 是良好实践，因为它保留值类别，避免不必要的拷贝，确保移动语义正确应用。

不传右值，也保留这个转发的功能。

<h6 id="cf8aac6a">🔅思考2：参数为通用引用时，不是既可以绑定左值也可以绑定右值吗？为什么函数参数又总是左值了？</h6>
即使通用引用（`T&& param`）可以绑定到左值或右值，`param` 本身作为函数内的局部变量，始终是一个左值表达式（有名字，可取地址）。

需要恢复原始值类别：直接传递 `param` 会丢失原始参数的值类别(始终视为左值)。`std::forward<T>(param)` 的作用是根据 `T` 的类型推导结果：

    - 若原始参数是左值（`T` 推导为左值引用），`std::forward` 返回左值引用。
    - 若原始参数是右值（`T` 推导为非引用类型），`std::forward` 返回右值引用。  
从而确保参数转发时保留其原始值类别。

```cpp
template<typename T>
void logAndProcess(T&& param) {
    // 记录日志
    makeLogEntry("Calling 'process'", std::chrono::system_clock::now());
    // 直接使用 param 而不是 std::forward<T>(param)
    process(param);
}
```

当调用 logAndProcess 时

传递左值：

```cpp
Widget w;
logAndProcess(w);  // w 是左值
```

T 将被推导为 Widget&（即 Widget 的左值引用），param 是一个左值引用。因此，process(param) 实际上会调用process(const Widget& lvalArg)。

传递右值

```cpp
logAndProcess(Widget());  // 临时对象是右值
```

在这种情况下，T 将被推导为 Widget（即 Widget 的非引用类型），param 是一个右值引用。但是，由于 param 作为函数参数，它仍然是一个左值。因此，process(param) 实际上也会调用 process(const Widget& lvalArg)，而不是 process(Widget&& rvalArg)。

**结论**

如果不使用 std::forward，process 函数的重载版本将总是选择处理左值引用的那个版本，即使原始参数是一个右值。如果 process 的右值引用版本可以更高效地处理右值（例如，通过移动语义），那么将失去这种优化。如果 process 的两个重载版本有不同的行为，那么不使用 std::forward 可能会导致错误的行为，因为总是会调用左值引用版本。

<h4 id="07c785c9">条款二十四：区分通用引用与右值引用</h4>
T&& 的两种含义

_**<font style="color:#c21c13;">右值引用</font>**_

<u>当 T&& 用在一个没有类型推导的地方时，它表示一个右值引用。</u>

<u>右值引用只能绑定到右值上，并且主要用于支持移动语义。</u>

例如：

```cpp
void f(Widget&& param);  // param 是一个右值引用
Widget&& var1 = Widget();  // var1 是一个右值引用
```

_**<font style="color:#c21c13;">通用引用</font>**_

当 <u>T&& 用在一个有类型推导的地方时，它被称为通用引用或转发引用。</u>

通用引用可以绑定到左值或右值，具体取决于传递给它的参数。

通用引用主要出现在函数模板参数和 auto 声明符中。

```cpp
template<typename T>
void f(T&& param);  // param 是一个通用引用
auto&& var2 = var1;  // var2 是一个通用引用
```

_**<u>通用引用的行为</u>**_

<u>类型推导：通用引用的存在依赖于类型推导。在模板函数参数或 auto 声明符中，编译器会根据传递的实参来推导 T 的类型。</u>

初始值决定引用类型：通用引用的初始值决定了它是左值引用还是右值引用。如果初始值是左值，那么通用引用将是一个左值引用。如果初始值是右值，那么通用引用将是一个右值引用。

```cpp
template<typename T>
void f(T&& param);//param 是一个通用引用
Widget w;
f(w);//传递给函数 f 一个左值；param 的类型将会是 Widget&，即左值引用
f(std::move(w));//传递给 f 一个右值；param 的类型将会是 Widget&&，即右值引用
```

限制条件

必须恰好为 T&&：为了使引用成为通用引用，其声明形式必须恰好为 T&&。任何其他修饰符（如 const 或 volatile）都会使其失去通用引用的资格。

通用引用必须出现在有类型推导的上下文中。如果没有类型推导，T&& 将被视为普通的右值引用。

非通用引用：

```cpp
template <typename T>
void f(std::vector<T>&& param);  // param 是一个右值引用
std::vector<int> v;
f(v);  // 错误！不能将左值绑定到右值引用
```

std::vector 的 emplace_back

```cpp
template<class T, class Allocator = allocator<T>>
class vector {
public:
template <class... Args>
void emplace_back(Args&&... args);
// ...
};
```

Args 是独立于 vector 的类型参数 T 的，因此每次调用 emplace_back 时，Args 都会被推导。Args 实际上是一个参数包（parameter pack），但为了方便讨论，我们可以将其视为一个类型参数。Args&&... args 是一个通用引用，因为形式是 type&&，并且 Args 的类型会在每次调用时被推导。

思考：这里有个面试题。

模板函数中的通用引用

```cpp
template<typename MyTemplateType>
void  someFunc(MyTemplateType&& param)；// param 是通用引用
```

MyTemplateType 可以是任何类型，param 是一个通用引用，因为其形式是 type&&，并且 MyTemplateType 的类型会在调用时被推导。

auto&& 作为通用引用

```cpp
auto&& var2 = var1;  // var2 是一个通用引用
```

auto&& 声明的变量是通用引用，因为会发生类型推导，并且形式是 type&&。

Lambda 表达式

```cpp
auto timeFuncInvocation = [](auto&& func, auto&&... params) {
    start timer;
    std::forward<decltype(func)>(func)(std::forward<decltype(params)>(params)...);
    stop timer and record elapsed time;
};
```

func 是一个通用引用，可以绑定到任何可调用对象，无论是左值还是右值。params 是一个通用引用参数包，可以绑定到任意数量和类型的对象。std::forward 用于完美转发参数，保持它们的原始值类别。

<h4 id="69b01453"><font style="background-color:#fbf5b3;">条款二十五：对右值引用使用std::move，对通用引用使用std::forward</font></h4>
右值引用仅绑定可以移动的对象

```cpp
class Widget {
Widget(Widget&& rhs);       //rhs定义上引用一个有资格移动的对象
…
};
```

希望传递这样的对象给其他函数，允许那些函数利用对象的右值性（rvalueness）。`std::move`将绑定到此类对象的形参转换为右值。

```cpp
class Widget {
public:
Widget(Widget&& rhs):name(std::move(rhs.name)),
p(std::move(rhs.p)){ }
// rhs是右值引用
private:
std::string name;
std::shared_ptr<SomeDataStructure> p;
};
```

通用引用**可能**绑定到有资格移动的对象上。通用引用使用右值初始化时，才将其强制转换为右值。

```cpp
class Widget {public:
template<typename T>
void setName(T&& newName)  //newName是通用引用
{ name = std::forward<T>(newName); }
};
```

<u><font style="background-color:#fbf5b3;">当把右值引用转发给其他函数时，右值引用应该被</font></u>**<u><font style="background-color:#fbf5b3;">无条件转换</font></u>**<u><font style="background-color:#fbf5b3;">为右值（通过</font></u>`<u><font style="background-color:#fbf5b3;">std::move</font></u>`<u><font style="background-color:#fbf5b3;">）</font></u>，<u><font style="background-color:#fbf5b3;">因为它们</font></u>**<u><font style="background-color:#fbf5b3;">总是</font></u>**<u><font style="background-color:#fbf5b3;">绑定到右值；当转发通用引用时，通用引用应该</font></u>**<u><font style="background-color:#fbf5b3;">有条件地转换</font></u>**<u><font style="background-color:#fbf5b3;">为右值（通过</font></u>`<u><font style="background-color:#fbf5b3;">std::forward</font></u>`<u><font style="background-color:#fbf5b3;">），因为它们只是</font></u>**<u><font style="background-color:#fbf5b3;">有时</font></u>**<u><font style="background-color:#fbf5b3;">绑定到右值。</font></u>

避免在右值引用上使用`std::forward`。更糟的是在通用引用上使用`std::move`，这可能会意外改变左值（比如局部变量）：

```cpp
class Widget {
public:
template<typename T>
voidsetName(T&& newName) //通用引用可以编译，
{ name = std::move(newName); }  //但是代码太太太差了！
private:
std::string name;
std::shared_ptr<SomeDataStructure> p;
};
std::stringgetWidgetName(); //工厂函数
Widget w;
auto n = getWidgetName(); //n是局部变量
w.setName(n);             //把n移动进w！
```

局部变量`n`被传递给`w.setName`，调用方可能认为这是对`n`的只读操作。但是因为`setName`内部使用`std::move`无条件将传递的引用形参转换为右值，`n`的值被移动进`w.name`，调用`setName`返回时`n`最终变为未定.

可能争辩说`setName`不应该将其形参声明为通用引用。此类引用不能使用`const`，但是`setName`肯定不应该修改其形参。你可能会指出，如果为`const`左值和为右值分别重载`setName`可以避免整个问题，比如这样：

```cpp
class Widget {
public:
void setName(const std::string& newName) //用const左值设置
{ name = newName;}
void setName(std::string&& newName) //用右值设置
{ name = std::move(newName); }
};
```

但是有缺点。首先编写和维护的代码更多（两个函数而不是单个模板）;其次，效率下降。

```cpp
w.setName("Adela Novak");
```

使用通用引用的版本的`setName`，字面字符串“`Adela Novak`”可以被传递给`setName`，再传给`w`内部`std::string`的赋值运算符。`w`的`name`的数据成员通过字面字符串直接赋值，没有临时`std::string`对象被创建。但是，`setName`重载版本，会有一个临时`std::string`对象被创建，`setName`形参绑定到这个对象，然后这个临时`std::string`移动到`w`的数据成员中。一次`setName`的调用会包括`std::string`构造函数调用（创建中间对象），`std::string`赋值运算符调用（移动`newName`到`w.name`），`std::string`析构函数调用（析构中间对象）。这比调用接受`const char*`指针的`std::string`赋值运算符开销昂贵许多。增加的开销根据实现不同而不同。

将通用引用模板替换成对左值引用和右值引用的一对函数重载在某些情况下会导致运行时的开销。如果把例子泛化，`Widget`数据成员是任意类型（而不是知道是个`std::string`），性能差距可能会变得更大，因为不是所有类型的移动操作都像`std::string`开销较小。

但是，关于对左值和右值的重载函数最重要的问题是设计的可扩展性差。`Widget::setName`有一个形参，因此需要两种重载实现，但是对于有更多形参的函数，每个都可能是左值或右值，重载函数的数量几何式增长：n个参数的话，就要实现2<sup>n</sup>种重载。

有的函数模板接受**无限制**个数的参数，每个参数都可以是左值或者右值。此类函数的典型代表是`<u><font style="background-color:#fbf5b3;">std::make_shared</font></u>`，还有对于C++14的`std::make_unique`（见[Item21](https://cntransgroup.github.io/EffectiveModernCppChinese/4.SmartPointers/item21.html)）

```cpp
template<classT, class... Args>                //来自C++11标准
shared_ptr<T> make_shared(Args&&... args);

template<classT, class... Args>                //来自C++14标准
unique_ptr<T> make_unique(Args&&... args);
```

<u><font style="color:#c21c13;background-color:#fbf5b3;">对于左值和右值分别重载就不能考虑了：通用引用是仅有的实现方案。对这种函数，肯定使用</font></u>`<u><font style="color:#c21c13;background-color:#fbf5b3;">std::forward</font></u>`<u><font style="color:#c21c13;background-color:#fbf5b3;">传递通用引用形参给其他函数。</font></u>

补充:项目里面用到了这个。

仅在最后一次使用时转换为右值

当函数参数是通用引用时，如果需要确保在完成其他操作前对象不会被移动，可以这样做：

```cpp
template<typename T>
void setSignText(T&& text) // text 是通用引用
{
    sign.setText(text);                       // 使用 text 但不改变它
    auto now = std::chrono::system_clock::now(); // 获取当前时间
    signHistory.add(now, std::forward<T>(text)); // 最后一次使用时有条件地转换为右值
}
```

这里，setText函数只是读取text而不改变它，所以在调用signHistory.add时，我们使用std::forward来保持text的原始值类别。

对于右值引用，同但在某些情况下，应该使用std::move_if_noexcept而不是std::move，以确保在移动构造函数不可用或抛出异常时仍能安全地进行拷贝。

```cpp
class Matrix {
public:
// 移动构造函数，可能抛出异常
Matrix(Matrix&& other) noexcept(false) {
    // 可能抛出异常的操作
}
//拷贝构造函数
Matrix(const Matrix& other) {
    // 安全的拷贝操作
}
};
Matrix operator+(Matrix&& lhs, const Matrix& rhs) {
    lhs += rhs;
    // 使用 std::move_if_noexcept 而不是 std::move
    return std::move_if_noexcept(lhs);
}
```

在按值返回的函数中使用std::move和std::forward。

对于右值引用

当你有一个按值返回的函数，其中返回值绑定到一个右值引用上时，应该使用std::move来确保执行移动操作，从而提高效率。

```cpp
class Matrix {
public:
Matrix& operator+=(const Matrix& rhs);
// 其他成员...
};
Matrix operator+(Matrix&& lhs, const Matrix& rhs){
    lhs += rhs;
    return std::move(lhs); // 移动lhs到返回值中
}
```

如果不使用std::move，编译器将被迫拷贝lhs，这可能会导致性能损失。

_**<u>返回值优化（RVO）与std::move的误用</u>**_

有些开发者认为可以在返回局部对象时使用std::move来避免复制，但实际上这是不必要的，因为现代编译器通常会自动应用返回值优化（RVO)：

```cpp
Widget makeWidget() // 正确版本
{
    Widget w; // 局部对象
    // 配置 w
    return w; // 编译器可能应用 RVO，直接在返回位置构造 w
}
// 错误版本：试图手动应用移动
Widget makeWidget( ) {
    Widget w;
    // 配置 w
    return std::move(w); // 不要这样做！
}
```

在这种情况下，即使没有显式使用std::move，编译器也可能通过RVO直接在返回值的位置构造w，从而避免了不必要的复制。因此，手动对局部对象使用std::move是多余的，并且可能会干扰编译器的优化。

**请记住：**

+ 最后一次使用时，在右值引用上使用`std::move`，在通用引用上使用`std::forward`。
+ 对按值返回的函数要返回的右值引用和通用引用，执行相同的操作。
+ 如果局部对象可以被返回值优化消除，就绝不使用`std::move`或者`std::forward`。

看到这。

<h4 id="dd4dc692">条款二十六：避免在通用引用上重载</h4>
该条款介绍如何使用标签分派来解决使用普通重载，可能绑定到通用引用重载的问题。

**<font style="color:#c21c13;">问题引入：</font>**假定需要写一个函数，使用name作为形参，打印当前日期和时间到日志中，然后将名字加入到一个全局数据结构中。

```cpp
std::multiset<std::string> names;//全局数据结构
void logAndAdd(const std::string& name){
    auto now=std::chrono::system_clock::now();//获取当前时间
    log(now, "logAndAdd");                
    names.emplace(name);  //把name加到全局数据结构中；
}
std::string petName("Darla");
logAndAdd(petName); //传递左值std::string
logAndAdd(std::string("Persephone")); //传递右值std::string
logAndAdd("Patty Dog");  //传递字符串字面值
```

在第一个调用中，`logAndAdd`的形参`name`绑定到变量`petName`。在`logAndAdd`中`name`最终传给`names.emplace`。因为`name`是左值，会拷贝到`names`中。没有方法避免拷贝，因为是左值（`petName`）传递给`logAndAdd`的。

<u>在第二个调用中，形参</u>`<u>name</u>`<u>绑定到右值（显式从“</u>`<u>Persephone</u>`<u>”创建的临时</u>`<u>std::string</u>`<u>）。</u>`<u>name</u>`<u>本身是个左值，所以它被拷贝到</u>`<u>names</u>`<u>中，但原则上，它的值可以被移动到</u>`<u>names</u>`<u>中。本次调用中，有个拷贝代价，但是应该能用移动。</u>

在第三个调用中，形参`name`也绑定一个右值，但是这次是通过“`Patty Dog`”隐式创建的临时`std::string`变量。就像第二个调用中，`name`被拷贝到`names`，但是这里，传递给`logAndAdd`的实参是一个字符串字面量。如果直接将字符串字面量传递给`emplace`，就不会创建`std::string`的临时变量，而是直接在`std::multiset`中通过字面量构建`std::string`。

可以通过使用通用引用(item24)重写`logAndAdd`来使第二个和第三个调用效率提升，按照item25的说法，`std::forward`转发这个引用到`emplace`。

```cpp
template<typename T>
void logAndAdd(T&& name){
    auto now = std::chrono::system_clock::now();
    log(now, "logAndAdd");
    names.emplace(std::forward<T>(name));
}

std::string petName("Darla");           //跟之前一样
logAndAdd(petName);                     //跟之前一样，拷贝左值到multiset
logAndAdd(std::string("Persephone"));	//移动右值而不是拷贝它
logAndAdd("Patty Dog");                 //在multiset直接创建std::string
//而不是拷贝一个临时std::string
```

效率优化了。

**问题的引申：**

客户不总是有直接访问`logAndAdd`要求的名字的权限。有些客户只有索引，`logAndAdd`拿着索引在表中查找相应的名字。为了支持这些客户，`logAndAdd`需要重载为：

```cpp
std::string nameFromIdx(int idx);   //返回idx对应的名字
void logAndAdd(int idx)             //新的重载
{
    auto now = std::chrono::system_clock::now();
    log(now, "logAndAdd");
    names.emplace(nameFromIdx(idx));
}
```

之后两个调用按照预期工作：

```cpp
std::string petName("Darla");          //跟之前一样
logAndAdd(petName);                    //跟之前一样，
logAndAdd(std::string("Persephone")); //这些调用都去调用
logAndAdd("Patty Dog");//T&&重载版本
logAndAdd(22);         //调用int重载版本
```

只能基本按照预期工作，假定一个客户将`short`类型索引传递给`logAndAdd`

```cpp
short nameIdx;
…                                       //给nameIdx一个值
logAndAdd(nameIdx);                     //错误！
```

_**<u>出现的问题：</u>**_

原始的logAndAdd函数使用模板和通用引用T&& name来支持左值和右值的不同处理方式。

通过std::forward<T>(name)将参数完美地转发给names.emplace，这样可以避免不必要的拷贝。

新增的logAndAdd(int idx)重载：该版本用于处理那些只能提供索引而不是直接名字的情况。

通过nameFromIdx(idx)获取对应的名字，并将其添加到names集合中。

当调用logAndAdd并传入short类型的索引时，编译器会优先选择T&&版本的logAndAdd，因为short可以直接绑定到T&&上，而不需要转换。由于short不是std::string构造函数可接受的类型之一，因此尝试将short转发给emplace时会导致错误。

解决方法：

一种解决方案是显式地将short转换为int，然后再调用logAndAdd。但这增加了客户端代码的复杂性。

更好的做法可能是重新设计接口，确保不会出现这样的歧义。

**问题2：**

有一个Person类，有两个构造函数。

（1）完美转发构造函数，使用模板来接受任意类型参数，并将该参数传递给std::string成员变量。

（2）整数索引构造函数，接受一个int类型的索引，并通过某个函数获取对应的std::string。

```cpp
class Person {
public:
template<typename T>
explicit Person(T&& n) : name(std::forward<T>(n)) {}
explicit Person(int idx) : name(nameFromIdx(idx)) {}
private:
std::string name;
};
```

1. 整数类型的传递问题

<u><font style="background-color:#fbf5b3;">如果用户尝试用非int类型的整数（例如short, long, std::size_t等）来创建Person对象，编译器会优先选择模板化的完美转发构造函数，而不是预期的int构造函数</font></u>。这是因为模板构造函数可以精确匹配任何类型的右值或左值，而int构造函数则需要进行整型提升。因此，传递这些整数类型会导致调用模板构造函数，进而可能导致编译错误，因为std::string没有接受这些整数类型的构造函数。

2. 拷贝构造函数的问题

即使Person类没有显式定义拷贝构造函数，编译器也会生成默认的拷贝构造函数。但是，由于存在模板构造函数，这可能会导致意外行为。例如，当我们尝试拷贝一个Person对象时，编译器可能会选择模板构造函数而不是拷贝构造函数，从而导致编译错误。

```cpp
Person p("Nancy");
auto cloneOfP(p);  // 尝试拷贝p，但编译失败
```

编译器认为模板构造函数可以被实例化为接受Person&类型，这比拷贝构造函数更匹配。因此，编译器选择了模板构造函数，但这会导致试图将Person对象直接赋值给std::string成员变量，从而引发编译错误。

使用标签分派：

通过引入额外的参数或使用标签分派技术来区分不同的构造函数逻辑。

<u><font style="color:#c21c13;">标签分派是一种编程技术，它通过传递一个额外的类型或值来区分不同的函数实现。</font></u>

在C++中，这种技术通常用于解决<u>重载解析问题</u>，特别是在<u><font style="color:#c21c13;">模板构造函数和普通构造函数共存的情况下。标签分派可以帮助编译器选择正确的构造函数实现。</font></u>

**标签分派的基本思想**

标签分派的核心思想是通过引入一个“标签”参数，这个参数可以是一个类型或一个值，来指示应该调用哪个具体的函数实现。标签通常不会影响函数的实际逻辑，但会被编译器用来决定调用哪个重载版本。

解决模板构造函数与普通构造函数共存的重载问题

假设有一个Person类，它有两个构造函数：一个是完美转发构造函数，另一个是从索引创建对象的构造函数。我们可以使用标签分派来确保编译器正确选择合适的构造函数。首先，定义两个标签类型，分别用于标识不同的构造函数：

```cpp
struct NameTag {};
struct IndexTag {};
```

接下来，修改构造函数，使其接受一个标签参数，并根据标签的不同执行不同的逻辑：

```cpp
class Person {
public:
// 完美转发构造函数
template<typename T>
explicit Person(NameTag, T&& n)
: name(std::forward<T>(n)) {}
// 从索引创建Person对象
explicit Person(IndexTag, int idx)
: name(nameFromIdx(idx)) {}
private:
std::string name;
// 假设这个函数已经定义好了
static std::string nameFromIdx(int idx) {
    // 实现细节
    return "Some Name";
}
};
```

标签分派详细介绍：

标签分派通过传递一个额外的类型（称为标签）来区分不同的函数或构造函数实现，从而在编译期静态选择正确的代码路径。根据类型特性或条件选择不同的实现方式。

定义标签类型

标签通常是一个空结构体（或枚举类型），用于标识不同的类型特征或分支。

```cpp
struct FastPathTag {};
struct SlowPathTag {};
```

重载函数/构造函数

为不同标签类型编写重载的函数或构造函数，每个重载版本对应特定的逻辑：

```cpp
// 普通构造函数（无标签）
MyClass(int value) { /* 普通初始化逻辑 */ }
// 模板构造函数（带标签）
template <typename T>
MyClass(T value, FastPathTag) { /* 快速路径逻辑 */ }

template <typename T>
MyClass(T value, SlowPathTag) { /* 慢速路径逻辑 */ }
```

选择标签并调用

在主函数或构造函数中，根据类型特征选择合适的标签，并传递给对应的重载版本：

```cpp
template <typename T>
MyClass(T value) {
    using Tag = typename Trait<T>::tag; // 根据类型选择标签
    MyClass::MyClass(value, Tag{});     // 调用带标签的重载构造函数
}
```

当模板构造函数和普通构造函数共存时，可能因类型匹配优先级导致歧义或错误选择。标签分派可通过以下方式解决：

假设有一个类 MyClass，需要支持：

普通构造函数：直接初始化 int 类型。

模板构造函数：根据类型特性选择不同的初始化路径（如 double 使用慢速路径，int 使用快速路径）。

若直接定义：

```cpp
class MyClass {
public:
template <typename T>
MyClass(T value) { /* 模板逻辑 */ }
MyClass(int value) { /* 普通逻辑 */ }
};
```

当用户用 int 实例化时，模板构造函数可能因“更通用”而被优先选择，导致普通构造函数被忽略。

解决方案：使用标签分派

通过标签分派明确区分不同路径:

```cpp
class MyClass {
private:
// 定义标签类型
struct FastPathTag {};
struct SlowPathTag {};
public:
// 普通构造函数（无标签）
MyClass(int value) { /* 直接初始化 */ }
// 模板构造函数（带标签）
template <typename T>
MyClass(T value, FastPathTag) { /* 快速路径逻辑 */ }
template <typename T>
MyClass(T value, SlowPathTag) { /* 慢速路径逻辑 */ }
public:
// 主模板构造函数，根据类型选择标签
template <typename T>
MyClass(T value) {
    using Tag = typename TypeTrait<T>::tag;
    MyClass::MyClass(value, Tag{}); // 调用带标签的重载
}
private:
// 类型特征：根据类型选择标签
template <typename T>
struct TypeTrait {
using tag = SlowPathTag; // 默认使用慢速路径
};
template <>
struct TypeTrait<int> { // 特化为快速路径
using tag = FastPathTag;
};
};
```

标签选择：通过 TypeTrait<T> 根据类型特性选择标签。

重载调用：主模板构造函数通过标签分派调用对应的重载版本。

普通构造函数：保持独立，避免与模板构造函数冲突。

类型特化

根据类型特性（如迭代器类型、POD类型）选择不同实现：

```cpp
// 示例：根据迭代器类型选择处理逻辑
template <typename Iter>
void process(Iter it, std::random_access_iterator_tag) { /* 随机访问迭代器优化 */ }
template <typename Iter>
void process(Iter it, std::bidirectional_iterator_tag) { /* 双向迭代器通用逻辑 */ }
```

编译期条件分支

替代 if constexpr 或 std::enable_if，通过标签选择不同代码路径：

```cpp
template <typename T>
void algorithm(T value, std::true_type) { /* 条件为真时的逻辑 */ }
template <typename T>
void algorithm(T value, std::false_type) { /* 条件为假时的逻辑 */ }
标准库函数（如 std::advance, std::copy）使用标签分派选择不同算法：
```

STL

```cpp
// 示例：std::copy 的简化实现
template <typename T>
struct CopyTrait {
using tag = NonPODTag; // 非POD类型使用拷贝构造
};
template <>
struct CopyTrait<int> {
using tag = PODTag; // POD类型使用内存拷贝
};
template <typename Iter>
void copy_helper(Iter first, Iter last, Iter out, PODTag) {
    std::memmove(out, first, sizeof(T) * (last - first));
}
template <typename Iter>
void copy_helper(Iter first, Iter last, Iter out, NonPODTag) {
    for (; first != last; ++first, ++out) *out = *first;
}
```

<h6 id="a289358c">🍀思考：使用通用引用和string_view的拷贝有什么区别？为什么不用string_view，而使用移动操作？</h6>
函数`logAndAdd`使用`const std::string&`接收参数，无论传入左值还是右值，内部都将`name`视为左值，导致`multiset`中始终调用拷贝构造函数。

右值无法移动：即使传入右值（如`std::string("Persephone")`），因形参为`const`左值引用，`emplace`只能拷贝，无法利用移动语义。

字符串字面值处理：传入字面值（如`"Patty Dog"`）时，会隐式构造临时`std::string`，再拷贝到容器中，存在额外开销。

优化思路：完美转发

通过通用引用和 `std::forward`保留值类别（左值/右值），直接将参数高效传递给容器。

```cpp
#include <string>
#include <set>
#include <chrono>
#include <utility>
std::multiset<std::string> names;
template<typename T>
void logAndAdd(T&& name) {
    auto now = std::chrono::system_clock::now();
    log(now, "logAndAdd");
    names.emplace(std::forward<T>(name)); // 保留值类别
}
```

左值(如`petName`)：调用拷贝构造函数。

右值(如`std::string("Persephone")`):调用移动构造函数，避免拷贝。

字面值(如`"Patty Dog"`):直接在容器中构造`std::string`，无需创建临时对象。

<h6 id="19d126fd">🍀思考：一个参数可能传入左值，也可能传入临时对象或者右值，那么最好将这个参数变为通用引用，即使对传入右值之后并没有什么优化。这个理解是否准确？</h6>
上面说法太绝对，这样操作会把代码复杂化。

原则：仅在需要优化时使用通用引用

**移动语义的价值：**<u>对拥有动态资源的类型（如 </u>`<u>std::string</u>`<u>、</u>`<u>std::vector</u>`<u>），移动操作将指针所有权转移，时间复杂度为 O(1)；而拷贝需要复制所有数据，时间复杂度为 O(n)。</u>**<u>移动在资源转移上一定比拷贝高效</u>**<u>。</u>

**通用引用的适用场景**

场景一：参数需要存储或转发

当函数需将参数传递给内部容器或构造函数时(`names.emplace`)，使用通用引用。

    - 对左值调用拷贝（如 `petName`）。
    - 对右值调用移动（如 `std::string("Persephone")`）。
    - 对字面值直接构造（如 `"Patty Dog"`，避免临时对象）。

```cpp
template<typenameT>voidlogAndAdd(T&& name){// ...
    names.emplace(std::forward<T>(name));// 最优传递}
```

场景二：避免不必要临时对象

对字面值 `"Patty Dog"`，<font style="background-color:#fbf5b3;">通用引用允许直接在容器中构造 </font>`<font style="background-color:#fbf5b3;">std::string</font>`，而 `const std::string&` 会强制先创建临时对象再拷贝。

<u>不适用的场景：</u>

<font style="background-color:#fbf5b3;">若函数逻辑强制拷贝（如后续仍需使用原对象），或参数类型无移动语义（如 </font>`<font style="background-color:#fbf5b3;">int</font>`<font style="background-color:#fbf5b3;">、</font>`<font style="background-color:#fbf5b3;">std::array</font>`<font style="background-color:#fbf5b3;">）</font>，则无需通用引用。

<h6 id="d8c2593c">🍀思考:什么时候用string_view name作为参数，什么时候用T &&name作为参数？我不理解，我看到有说string_view可以避免拷贝，又有说T&&name可以避免拷贝？</h6>
`std::string_view` 是 C++17 引入的轻量级只读字符串视图。

**<font style="background-color:#fbf5b3;">避免拷贝</font>**<font style="background-color:#fbf5b3;">：</font>当函数只需要读取字符串内容时，使用 `std::string_view` 可以避免拷贝，无论传入的是 `std::string`、`const char*`、还是其他字符串类型。

**<font style="background-color:#fbf5b3;">统一接口</font>**<font style="background-color:#fbf5b3;">：</font>可以接受多种字符串类型（如 `std::string`、`std::string_view`、`const char*`、字面量等），无需额外重载。

**<font style="background-color:#fbf5b3;">只读操作</font>**<font style="background-color:#fbf5b3;">：</font>函数不会修改字符串内容，仅用于读取或计算。

```cpp
#include <string_view>
void print(const std::string_view& name) {
    std::cout << name << std::endl;
}
// 调用方式：
std::string s = "Hello";
print(s);         // 无需拷贝
print("World");   // 无需拷贝（字面量）
print(std::string_view("C++")); // 无需拷贝
```

2. **通用引用（**`**T&&**`**）的适用场景**

通用引用（通过 `T&&` 形式）主要和完美转发一起使用：

**完美转发**：<font style="background-color:#fbf5b3;">保留参数的值类别（左值或右值）和 </font>`<font style="background-color:#fbf5b3;">const</font>`<font style="background-color:#fbf5b3;"> 属性，以便后续传递给其他函数。</font>当需要移动右值（如临时对象）时，避免拷贝，直接转移资源。<font style="background-color:#fbf5b3;">如果函数不需要传递参数，通用引用可能多余。</font>

```cpp
template <typename T>
void process(T&& name) {
    // 完美转发到其他函数
    some_function(std::forward<T>(name));
}
//调用方式:
std::string s = "Hello";
process(s); //左值,传递为lvalue引用
process("World");//右值,传递为rvalue引用(可移动)
```

**如何选择？**

**选择 **`**std::string_view**`** 的情况**：

函数只需要读取字符串内容，不需要修改。需要兼容多种字符串类型（如 `std::string`、`const char*`、字面量等）。

**选择 **`**T&&**`** 的情况**：需要将参数**完美转发**到其他函数（如构造函数、工厂函数）。

需要利用移动语义（如接收临时对象并转移资源）。函数需要处理多种类型参数，而不仅仅是字符串。

**误区 1**：认为 `std::string_view` 和 `T&&` 都能避免拷贝，所以可以随意替换。

**纠正**：`std::string_view` 是针对字符串的只读视图，而 `T&&` 是通用转发机制，两者解决的问题不同。

**误区 2**：认为 `T&&` 总是比 `std::string_view` 更灵活。

**纠正**：如果只需要读取字符串且不需要转发，`std::string_view` 更简洁且无需模板。

**误区 3**：认为 `std::string_view` 无法处理右值。

    - **纠正**：`std::string_view` 可以接受右值（如临时字符串），但不会延长其生命周期。

适用于日志、解析、计算等只读操作,**优先使用 **`**std::string_view**`：

```cpp
void log(const std::string_view& message) { ... }
```

<h4 id="283fff68">条款二十七：熟悉通用引用重载的替代方法</h4>
item26中说明对使用通用引用形参的函数，无论是独立函数还是成员函数，进行重载都会导致一系列问题。但是也提供了一些示例，如果能够按照我们期望的方式运行，重载可能也是有用的。<u>这个条款探讨了几种通过避免在通用引用上重载的设计，或者通过限制通用引用可以匹配的参数类型，来实现所期望行为的方法。</u>

_**<u>放弃重载</u>**_

为了解决这个问题，一个方法是完全避免重载。例如，如果你有两个不同行为的logAndAdd函数，你可以给它们不同的名字，如logAndAddName和logAndAddNameIdx。这种方法简单明了，但并不总是可行，特别是对于构造函数这样的情况，因为构造函数的名字是由类名决定的，不能随意更改。

_**<u>传递const T&</u>**_

另一种方法是使用const T&来代替通用引用。这种方式虽然可能没有通用引用那么高效，因为它不允许移动语义，但它可以确保更可预测的行为。例如，你可以将接受std::string的构造函数改为接受const std::string&，这样就不会与整数类型混淆了。

_**<u>传值</u>**_

还有一种方法是按值传递参数，这通常看起来像是降低了效率，但实际上可以通过移动语义优化性能。当你知道你将要拷贝对象时，直接传递值可以让编译器利用RVO（返回值优化）或NRVO（命名返回值优化），甚至是在某些情况下应用移动语义。

```cpp
class Person {
public:
// 使用std::string按值传递
explicit Person(std::string n)
: name(std::move(n)) {}
// 整数索引构造函数保持不变
explicit Person(int idx)
: name(nameFromIdx(idx)) {}
private:
static std::string nameFromIdx(int idx) {
    // 实现从索引获取名字的逻辑
    return "Name" + std::to_string(idx);
}
std::string name;
};
```

第一个构造函数接受std::string类型的参数，并使用std::move来转移所有权，允许编译器进行潜在的优化。第二个构造函数保持不变，接受整数作为参数并调用nameFromIdx函数来生成名字。由于std::string构造函数不会接受整型参数，所以这两个构造函数之间不会有冲突。如果用户尝试用整数初始化Person对象，那么将会调用正确的构造函数。对于std::string或者能够隐式转换为std::string的类型，将使用第一个构造函数。这样做既保证了代码的行为符合预期，又保持了良好的性能。

_**<u>Tag Dispatch</u>**_

Tag dispatch通过向函数传递一个额外的类型参数来帮助编译器选择正确的重载版本。这个类型参数通常是一个std::true_type或std::false_type，是标准库提供的类型，用于表示布尔值。这些类型在编译时已知，因此可以帮助编译器做出正确的决策。

logAndAdd 函数

有一个logAndAdd函数，能够处理两种情况：

当传入的是字符串或者可以转换为字符串的类型时，将该名字添加到全局数据结构。当传入的是整数时，使用这个整数作为索引来查找对应的名字，然后调用logAndAdd。

首先，需要定义两个实现函数logAndAddImpl，一个处理非整型，另一个处理整型。

```cpp
//非整型实参：添加到全局数据结构中
template<typename T>
void logAndAddImpl(T&& name, std::false_type) {
    auto now = std::chrono::system_clock::now();
    log(now, "logAndAdd");
    names.emplace(std::forward<T>(name));
}
//整型实参：查找名字并用它调用logAndAdd
void logAndAddImpl(int idx, std::true_type) {
    logAndAdd(nameFromIdx(idx));
}
//辅助函数，从索引获取名字
std::string nameFromIdx(int idx) {
    //实现从索引获取名字的逻辑
    return "Name" + std::to_string(idx);
}
```

接下来,编写主函数logAndAdd，根据传入的类型选择正确的logAndAddImpl版本：

```cpp
template<typename T>
void logAndAdd(T&& name) {
    using UnrefType = typename std::remove_reference<T>::type;
    logAndAddImpl(
        std::forward<T>(name),
        std::is_integral<UnrefType>()
        );
}
```

这里的关键点在于std::remove_reference<T>::type。当T是左值引用时（如int&），std::is_integral<int&>()会返回false，因为引用不是整型。因此我们需要使用std::remove_reference来移除引用，从而正确地判断T是否为整型。

```cpp
//全局数据结构
std::multiset<std::string> names;
//日志记录函数
void log(const std::chrono::system_clock::time_point&, const char*) {
    //实现日志记录逻辑
}
//根据索引获取名字
std::string nameFromIdx(int idx) {
    //实现从索引获取名字的逻辑
    return "Name" + std::to_string(idx);
}
//非整型实参：添加到全局数据结构中
template<typename T>
void logAndAddImpl(T&& name, std::false_type) {
    auto now = std::chrono::system_clock::now();
    log(now, "logAndAdd");
    names.emplace(std::forward<T>(name));
}
//整型实参：查找名字并用它调用logAndAdd
void logAndAddImpl(int idx, std::true_type) {
    logAndAdd(nameFromIdx(idx));
}
//主函数，根据传入类型选择正确的logAndAddImpl版本
template<typename T>
void logAndAdd(T&& name) {
    using UnrefType = typename std::remove_reference<T>::type;
    logAndAddImpl(
        std::forward<T>(name),
        std::is_integral<UnrefType>()
        );
}
int main() {
    logAndAdd("Alice");   // 添加字符串
    logAndAdd(42);        // 通过索引查找名字
    return 0;
}
```

通过这种方式，可以避免在通用引用上重载带来的问题，同时还能保持代码的简洁性和可读性。在C++中，当使用通用引用（T&&）时，尤其是与重载结合时，可能会导致一些意外的行为。为了解决这些问题，可以采用tag dispatch和std::enable_if来控制模板的启用条件。

std::enable_if 的基本形式

```cpp
template<bool B, class T = void>
struct enable_if;
// 当 B 为 true 时，enable_if<B, T>::type 为 T
// 当 B 为 false 时，enable_if<B, T> 没有 type 成员
```

```cpp
// 辅助函数，从索引获取名字
std::string nameFromIdx(int idx) {
    // 实现从索引获取名字的逻辑
    return "Name" + std::to_string(idx);
}
class Person {
public:
// 完美转发构造函数，仅当T不是Person或其派生类且不是整型时启用
template<typename T,typename = std::enable_if_t<!std::is_base_of<Person, std::decay_t<T>>::value&&!std::is_integral<std::remove_reference_t<T>>::value>>
explicit Person(T&& n):name(std::forward<T>(n)) {}
// 整型实参的构造函数
explicit Person(int idx)
: name(nameFromIdx(idx)) {}
// 拷贝构造函数
Person(const Person& other)
: name(other.name) {}
// 移动构造函数
Person(Person&& other) noexcept
: name(std::move(other.name)) {}
private:
std::string name;
};
// 派生类示例
class SpecialPerson : public Person {
public:
//拷贝构造函数
SpecialPerson(const SpecialPerson& rhs)
: Person(rhs) {}
//移动构造函数
SpecialPerson(SpecialPerson&& rhs) noexcept
: Person(std::move(rhs)) {}
};
```

完美转发构造函数：

使用std::enable_if来限制模板的启用条件。

!std::is_base_of<Person, std::decay_t<T>>::value 确保T不是Person或其派生类。!std::is_integral<std::remove_reference_t<T>>::value 确保T不是整型。std::decay_t<T> 用于移除T的引用和cv限定符。

std::remove_reference_t<T> 用于移除T的引用。

整型实参的构造函数：直接处理整型参数，调用nameFromIdx函数来获取名字。

拷贝和移动构造函数：显式定义了拷贝和移动构造函数，确保它们不会被完美转发构造函数覆盖。

派生类：SpecialPerson 类继承自 Person，并显式定义了拷贝和移动构造函数，确保它们调用基类的相应构造函数。

std::enable_if 的基本形式如下：

```cpp
template<bool B, class T = void>
struct enable_if;
// 当 B 为 true 时，enable_if<B, T>::type 为 T
// 当 B 为 false 时，enable_if<B, T> 没有 type 成员
```

在模板声明中，std::enable_if 通常用于 SFINAE（Substitution Failure Is Not An Error）规则，即如果模板实例化失败，则该模板不会被考虑为候选函数。

假设有一个 Person 类，有一个接受通用引用的构造函数，并且希望这个构造函数只对非 Person 类型及其派生类和非整型参数启用。使用 std::enable_if 来确保只有当传入的类型不是 Person 或其派生类，并且不是整型时，才启用该构造函数。

定义处理整型参数的构造函数：提供一个专门处理整型参数的构造函数。

使用 std::is_base_of 和 std::is_integral，std::is_base_of 用于检查类型是否是 Person 或其派生类。

std::is_integral 用于检查类型是否是整型。

使用std::decay 用于移除引用和 cv 限定符，确保类型比较时忽略这些修饰。

**请记住：**

+ 通用引用和重载的组合替代方案包括使用不同的函数名，通过lvalue-reference-to-`const`传递形参，按值传递形参，使用_tag dispatch_。
+ 通过`std::enable_if`约束模板，允许组合通用引用和重载使用，但它也控制了编译器在哪种条件下才使用通用引用重载。
+ 通用引用参数通常具有高效率的优势，但是可用性就值得斟酌。

<h4 id="6e0df42c">条款二十八：理解引用折叠</h4>
[Item23](https://cntransgroup.github.io/EffectiveModernCppChinese/5.RRefMovSemPerfForw/item23.html)中指出，当实参传递给模板函数时，被推导的模板形参`T`根据实参是左值还是右值来编码。但是那条款并没有提到只有当实参被用来实例化通用引用形参时，上述推导才会发生，但是有充分的理由忽略这一点：因为通用引用是[Item24](https://cntransgroup.github.io/EffectiveModernCppChinese/5.RRefMovSemPerfForw/item24.html)中才提到。回过头来看，对通用引用和左值/右值编码的观察意味着对于这个模板，

```cpp
template<typename T>
void func(T&& param);
```

不管传给param的实参是左值还是右值，模板形参`T`都会编码。

编码机制是简单的。当左值实参被传入时，`T`被推导为左值引用。当右值被传入时，`T`被推导为非引用。（请注意不对称性：左值被编码为左值引用，右值被编码为**非引用**。）因此：

```cpp
Widget widgetFactory();   //返回右值的函数
Widget w;                 //一个变量（左值）
func(w);                  //用左值调用func；T被推导为Widget&
func(widgetFactory());    //用右值调用func；T被推导为Widget
```

上面的两种`func`调用中，`Widget`被传入，因为一个是左值，一个是右值，模板形参`T`被推导为不同的类型，这决定了通用引用成为左值还是右值，也是`std::forward`的工作基础。

明确在C++中引用的引用是非法的。下面的写法，编译器会报错：

```cpp
int x;
auto& & rx = x;             //错误！不能声明引用的引用
```

考虑下，如果一个左值传给接受通用引用的模板函数会发生什么：

```cpp
template<typename T>
void func(T&& param);       //同之前一样
func(w);                    //用左值调用func；T被推导为Widget&
```

如果用`T`推导出来的类型(即`Widget&`)初始化模板,会得到：

```cpp
void func(Widget& && param);
```

引用的引用!

当用一个左值调用 func(w)时，w 是一个已经存在的对象(左值)，编译器会推导出 T 为 Widget&。因此原本的 T&& 实际上变成了 Widget& &&。

当用一个右值调用 func(widgetFactory())时，widgetFactory()返回的是一个临时对象(右值)，编译器会推导出 T 为 Widget。因此，原本的 T&& 变成了 Widget&&。但是编译器没有报错。从item24中了解到因为通用引用`param`被传入一个左值，所以`param`的类型应该为左值引用，但是编译器如何把`T`推导的类型带入模板变成如下的结果，也就是最终的函数签名？

```cpp
void func(Widget& param);
```

答案是**引用折叠**。引用折叠规则有两条核心原则：

（1）左值引用优先：如果参与引用组合中的任一引用为左值引用（&），那么最终结果总是左值引用。

（2）右值引用次之：如果两个引用都是右值引用（&&），那么最终结果是右值引用。

禁止声明引用的引用，但是**编译器**会在特定的上下文中产生这些,当编译器生成引用的引用时，引用折叠指导下一步发生什么。存在两种类型的引用（左值和右值），所以有四种可能的引用组合（左值的左值，左值的右值，右值的右值，右值的左值）。如果一个上下文中允许引用的引用存在（比如，模板的实例化），引用根据规则**折叠**为单个引用：

如果任一引用为左值引用，则结果为左值引用。否则（即，如果引用都是右值引用），结果为右值引用。

(1)T&& 折叠成 T& (左值引用)

模板函数

```cpp
template<typename T>
void func(T& param) {
    // 函数体
}
```

当调用 func 并传递一个左值引用时：

```cpp
int x = 10;
int &rx = x;
func(rx);  // 所以 T&& 是 int&
```

(2) T& && 折叠成 T& (左值引用)

考虑通用引用的模板函数：

```cpp
template<typename T>
void func(T&& param) {
    // 函数体
}
```

当用一个左值调用 func 时：

```cpp
int x = 20;
func(x);  // 这里 T 被推导为 int&，所以 T&& 是 int& &&
```

在这个例子中，T 被推导为 int&，因此 T&& 实际上是 int& &&。根据引用折叠规则，int& && 折叠成 int&。param 的类型最终是 int&。

(3) T&& & 折叠成 T& (左值引用)

在实际代码中不太常见，因为通常不会直接声明 T&& &。但是，在某些情况下，模板参数推导可能会导致这样的情况出现。

(4) T&& && 折叠成 T&& (右值引用)

再次考虑通用引用的模板函数：

```cpp
template<typename T>
void func(T&& param) {
    // 函数体
}
```

当用一个右值调用 func 时：

```cpp
func(40);  // 这里 T 被推导为 int，所以 T&& 是 int&&
```

T 被推导为 int，因此 T&& 直接就是 int&&。因为这里没有引用的引用，所以不需要进行引用折叠。这意味着 param 的类型最终是 int&&。

std::forward 的实现

以下是简化版的 std::forward 实现：

```cpp
template<typename T>
T&& forward(typename std::remove_reference<T>::type& param) {
    return static_cast<T&&>(param);
}
```

T 是一个模板参数，std::remove_reference<T>::type 用来去除 T 中可能存在的引用类型，从而得到原始类型。然后使用 static_cast<T&&>(param) 将 param 转换为 T&& 类型。

当传入左值时，假设有一个通用引用的模板函数 f：

```cpp
template<typename T>
void f(T&& fParam) {
    // 做些工作
    someFunc(std::forward<T>(fParam));  // 转发 fParam 到 someFunc
}
```

将一个左值传递给 f 时，例如：

```cpp
Widget w;
f(w); //T被推导为Widget&
```

在这个情况下,T 被推导为 Widget&。

std::forward<T> 实例化为 std::forward<Widget&>。

代入到 std::forward 的实现中：

```cpp
Widget& && forward(typename std::remove_reference<Widget&>::type& param) {
    return static_cast<Widget& &&>(param);
}
```

std::remove_reference<Widget&>::type 会移除引用，结果是 Widget。std::forward 变为:

```cpp
Widget& && forward(Widget& param) {
    return static_cast<Widget& &&>(param);
}
```

根据引用折叠规则，Widget& && 折叠成 Widget&。因此，最终的 std::forward 实现为：

```cpp
Widget& forward(Widget& param) {
    return static_cast<Widget&>(param);
}
```

这里，static_cast<Widget&>(param) 实际上不做任何事情，因为 param 已经是 Widget& 类型。所以，左值被传递给 std::forward 时，返回的是左值引用，保持了左值的特性。

当传入右值时

假设将一个右值传递给 f

```cpp
f(Widget());  // 这里 T 被推导为 Widget
```

T被推导为 Widget。std::forward<T> 实例化为 std::forward<Widget>。

代入到 std::forward 的实现中：

```cpp
Widget&& forward(typename std::remove_reference<Widget>::type& param) {
    return static_cast<Widget&&>(param);
}
```

这里 std::remove_reference<Widget>::type 仍然是 Widget，因为 Widget 本身不是引用类型。因此，std::forward 变为：

```cpp
Widget&& forward(Widget& param) {
    return static_cast<Widget&&>(param);
}
```

没有引用折叠的问题，因为 T 是非引用类型。static_cast<Widget&&>(param) 将 param 转换为右值引用，即使 param在std::forward 内部是一个左值引用，但通过 static_cast<Widget&&>(param)，它被转换为右值引用。

从函数返回的右值引用被定义为右值，`std::forward`会将`f`的形参`fParam`（左值）转换为右值。最终结果是，传递给`f`的右值参数将作为右值转发给`someFunc`，正是想要的结果。

在C++14中，`std::remove_reference_t`的存在使得实现变得更简洁：

```cpp
template<typename T>                        //C++14；仍然在std命名空间
T&& forward(remove_reference_t<T>& param)
{
    return static_cast<T&&>(param);
}
```

引用折叠发生在四种情况下。

第一,最常见的就是模板实例化。

第二,是`auto`变量的类型生成，具体细节类似于模板，因为`auto`变量的类型推导基本与模板类型推导雷同(item2)

```cpp
Widget widgetFactory();//返回右值的函数
Widget w;              //一个变量（左值）
func(w);               //用左值调用func；T被推导为Widget&
func(widgetFactory()); //用右值调用func；T被推导为Widget
```

在auto的写法中，规则是类似的。

```cpp
auto&& w1 = w;
```

用一个左值初始化`w1`，因此为`auto`推导出类型`Widget&`。把`Widget&`代回`w1`声明中的`auto`里，产生了引用的引用，

```cpp
Widget& && w1 = w;
```

应用引用折叠规则，就是

```cpp
Widget& w1 = w
```

结果就是`w1`是一个左值引用。另一方面，这个声明，

```cpp
auto&& w2 = widgetFactory();
```

使用右值初始化`w2`，为`auto`推导出非引用类型`Widget`。把`Widget`代入`auto`得到：

```cpp
Widget&& w2 = widgetFactory()
```

没有引用的引用，这就是最终结果，`w2`是个右值引用。

通用引用不是一种新的引用，它实际上是满足以下两个条件下的右值引用：

+ **类型推导区分左值和右值**。`T`类型的左值被推导为`T&`类型，`T`类型的右值被推导为`T`。
+ **发生引用折叠**。

通用引用的概念是有用的，因为它使你不必一定意识到引用折叠的存在，从直觉上推导左值和右值的不同类型，在凭直觉把推导的类型代入到它们出现的上下文中之后应用引用折叠规则。

第三种情况是`typedef`和别名声明的产生和使用中（参见item9）。

如果，在创建或者评估`typedef`过程中出现了引用的引用，则引用折叠就会起作用。

```cpp
template<typename T>
class Widget {
public:
typedef T&& RvalueRefToT;
};
```

假设使用左值引用实例化`Widget`

```cpp
Widget<int&> w;
```

`Widget`模板中把`T`替换为`int&`得到

```cpp
typedef int& && RvalueRefToT;
```

引用折叠就会发挥作用

```cpp
typedef int& RvalueRefToT;
```

表明为`typedef`选择的名字可能不是我们希望的那样：当使用左值引用类型实例化`Widget`时，`RvalueRefToT`是**左值引用**的`typedef`。

最后一种引用折叠发生的情况是，`decltype`使用的情况。如果在分析`decltype`期间，出现了引用的引用，引用折叠规则就会起作用（item3）

**请记住：**

+ 引用折叠发生在四种情况下：模板实例化，`auto`类型推导，`typedef`与别名声明的创建和使用，`decltype`。
+ 当编译器在引用折叠环境中生成了引用的引用时，结果就是单个引用。有左值引用折叠结果就是左值引用，否则就是右值引用。
+ 通用引用就是在特定上下文的右值引用，上下文是通过类型推导区分左值还是右值，并且发生引用折叠的那些地方。

<h4 id="e74e2c28">条款二十九：假定移动操作不存在，成本高，未被使用</h4>
移动语义可以说是C++11最主要的特性。"移动容器和拷贝指针一样开销小"，"拷贝临时对象现在如此高效，“写代码避免这种情况简直就是过早优化"。很多开发者认为通过采用移动语义可以极大地优化代码效率，甚至有时会听到“移动容器与拷贝指针一样开销小”这样的说法，以及建议不必过分担心临时对象的复制问题，因为移动语义已经让这个过程变得非常高效。然而，虽然移动语义确实是一项强大的功能，并且能够显著改善特定场景下的性能表现，但该段落也提醒读者要保持理性看待这项技术。

移动语义允许通过移动操作来提高程序性能。然而，并非所有类型都支持移动操作，且即使支持，其带来的性能提升也未必如预期般显著。本文将讨论C++11中移动语义的局限性及其在不同类型中的表现。

1. 容器差异

_**<u>案例1：std::array</u>**_

并非所有容器都能从移动语义中获得相同程度的好处。例如std::vector等基于堆分配内存的容器可以通过简单地转移指针实现高效移动。std::array直接存储元素而非指向动态分配内存的指针，因此其移动操作涉及每个元素的单独处理，开销为线性时间复杂度。

考虑一下`std::array`(C++11中的新容器)>。`std::array`本质上是具有STL接口的内置数组。这与其他标准容器将内容存储在堆内存不同。存储具体数据在堆内存的容器，本身只保存了指向堆内存中容器内容的指针(实现更复杂,基本逻辑是这样)。这个指针的存在使得在常数时间移动整个容器成为可能，只需要从源容器拷贝保存指向容器内容的指针到目标容器，然后将源指针置为空指针就可以了：

```cpp
std::vector<Widget> vw1;
//把数据存进vw1.
//把vw1移动到vw2.
//以常数时间运行.只有vw1和vw2中的指针被改变
auto vw2 = std::move(vw1);
```

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742811188976-312f7b86-5663-492d-ab7a-201c1cc4589a.png)

`std::array`没有这种指针实现，数据就保存在`std::array`对象中：

```cpp
std::array<Widget, 10000> aw1;
//把数据存进aw1
//把aw1移动到aw2。以线性时间运行
//aw1中所有元素被移动到aw2
auto aw2 = std::move(aw1);
```

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742811189133-7efc3105-10ef-4279-a90f-7b7e63692518.png)

`aw1`中的元素被**移动**到了`aw2`中。假定`Widget`类的移动操作比复制操作快，移动`Widget`的`std::array`就比复制要快。所以`std::array`确实支持移动操作。但是使用`std::array`的移动操作还是复制操作都将花费线性时间的开销，因为每个容器中的元素终归需要拷贝或移动一次，这与“移动一个容器就像操作几个指针一样方便”的含义相去甚远。

**案例2：std::string**

尽管std::string提供了常数时间复杂度的移动操作与线性时间复杂度的复制操作，但小字符串优化（SSO）使得对于短字符串来说，移动并不比复制更高效。SSO允许短字符串直接存储在std::string对象缓冲区，避免了额外的堆内存分配。在这种情况下，移动这样的字符串不会比复制更快。S大量证据表明，短字符串是大量应用使用的习惯。使用内存缓冲区存储而不分配堆内存空间，是为了更好的效率。

_**<u>2. 异常安全与移动操作</u>**_

异常安全性：C++标准库中的某些容器为了提供强大的异常安全保障，要求移动操作必须是noexcept。如果一个类提供的移动操作没有声明为noexcept，即使它实际上更高效，编译器也可能选择使用复制操作以确保代码的异常安全性。这限制了移动语义在这些情况下的应用。

_**<u>情况分析</u>**_

（1）没有移动操作:如果要移动的对象不支持移动操作，则移动表达式会退化为复制操作。

（2）移动不比复制快:即使存在移动操作，如果移动的成本不低于复制，那么移动可能并不会带来性能上的提升。

（3）移动不可用：在需要保证移动操作不会抛出异常的情况下，若移动操作未声明为noexcept，则编译器将不得不使用复制操作。

（4）源对象是左值：通常只有右值可以作为移动操作的来源；左值作为来源时，除非特别设计，否则一般采用复制而非移动。

_**<u>通用编程中的考虑</u>**_

编写泛型代码或模板时，由于无法预知具体类型是否支持高效的移动操作，应谨慎地依赖于复制操作。这类不稳定的代码经常变更，导致类型特性变化，也应采取保守策略。当对所使用的类型有充分了解，并且该类型确实支持快速移动操作时，可以在合适的上下文中利用这一点来替换复制操作，从而提高性能。

<h4 id="f5f5377f">条款三十：熟悉完美转发失败的情况</h4>
C++11的完美转发是非常好用，但是只有当你愿意忽略一些误差情况(完美转发失败的情况)，这个条款就是使你熟悉这些情形。

**完美转发(**_perfect forwarding)_意味着我们不仅转发对象，我们还转发显著的特征：它们的类型，是左值还是右值，是`const`还是`volatile`。结合到我们会处理引用形参，意味着将使用通用引用，因为通用引用形参被传入实参时才确定是左值还是右值。

假定有一些函数`f`，编写一个转发给它的函数（事实上是一个函数模板）。我们需要的核心看起来像是这样：

```cpp
template<typename T>
void fwd(T&& param)//接受任何实参{
f(std::forward<T>(param));  //转发给f
}
```

转发函数是通用的。例如`fwd`模板，接受任何类型的实参，并转发得到的任何东西。这种通用性的逻辑扩展是，转发函数不仅是模板，而且是可变模板，因此可以接受任何数量的实参。`fwd`的可变形式如下：

```cpp
template<typename... Ts>
void fwd(Ts&&... params)//接受任何实参{
f(std::forward<Ts>(params)...); //转发给f
}
```

这种形式你会在容器emplace functions中（item42）和 智能指针的工厂函数`std::make_unique`和`std::make_shared`中（item21）看到，当然还有其他一些地方。

给定我们的目标函数`f`和转发函数`fwd`，如果`f`使用某特定实参会执行某个操作，但是`fwd`使用相同的实参会执行不同的操作，完美转发就会失败。导致这种失败的实参种类有很多。知道它们是什么以及如何解决它们很重要，因此让我们来看看无法做到完美转发的实参类型。

_**<u>花括号初始化器</u>**_

假定`f`这样声明

```cpp
void f(conststd::vector<int>& v);
```

用花括号初始化调用`f`通过编译

```cpp
f({ 1, 2, 3 });  //可以,“{1, 2, 3}”隐式转换为std::vector<int>
```

但是传递相同的列表初始化给fwd不能编译

```cpp
fwd({ 1, 2, 3 }); //错误!不能编译
```

这是完美转发失效的一种情况。当通过调用函数模板`fwd`间接调用`f`时，编译器不再把调用地传入给`fwd`的实参和`f`的声明中形参类型进行比较。而是**推导**传入给`fwd`的实参类型，然后比较推导后的实参类型和`f`的形参声明类型。当下面情况任何一个发生时，完美转发就会失败：

**编译器不能推导出**`**fwd**`**的一个或者多个形参类型。** 这种情况下代码无法编译。

**编译器推导“错”了**`**fwd**`**的一个或者多个形参类型。**

"错误"可能意味着`fwd`的实例将无法使用推导出的类型进行编译，但是也可能意味着使用`fwd`的推导类型调用`f`，与用传给`fwd`的实参直接调用`f`表现出不一致的行为。

这种不同行为的原因可能是因为`f`是个重载函数的名字，并且由于是“不正确的”类型推导，在`fwd`内部调用的`f`重载和直接调用的`f`重载不一样。

在上面的`fwd({ 1, 2, 3 })`例子中，将花括号初始化传递给未声明为`std::initializer_list`的函数模板形参。着编译器不准在对`fwd`的调用中推导表达式`{ 1, 2, 3 }`的类型，因为`fwd`的形参没有声明为`std::initializer_list`。对于`fwd`形参的推导类型被阻止，编译器只能拒绝该调用。

item2说明了使用花括号初始化的`auto`的变量的类型推导是成功的。这种变量被视为`std::initializer_list`对象，在转发函数应推导出类型为`std::initializer_list`的情况，这提供了一种简单的解决方法：使用`auto`声明一个局部变量，然后将局部变量传进转发函数：

```cpp
auto il = { 1, 2, 3 };  //il的类型被推导为std::initializer_list<int>
fwd(il);                //可以，完美转发il给f
```

(这个地方没看懂)

_**<u>0或者NULL作为空指针</u>**_

item8说明当试图传递`0`或者`NULL`作为空指针给模板时，类型推导会出错，会把传来的实参推导为一个整型类型（典型情况为`int`）而不是指针类型。

结果就是不管是`0`还是`NULL`都不能作为空指针被完美转发。解决方法非常简单，传一个`nullptr`而不是`0`或者`NULL`(参考item8)。

_**<u>仅有声明的整型static const数据成员</u>**_

通常无需在类中定义整型`static const`数据成员；声明就可以了。这是因为编译器会对此类成员实行常量传播（_const propagation_），因此消除了保留内存的需要。

**声明与定义的区别**

告诉编译器某个实体(变量,函数,类等)的名称和类型,不为其分配存储空间。不仅告诉编译器实体的名称和类型，还为其分配存储空间，并可能提供初始化值或实现。

static const 整型数据成员的特殊情况。

对于static const整型数据成员，可以在类内部进行初始化，但这实际上是声明的一部分。编译器会将这个常量的值传播到所有使用它的地方，从而不需要为它分配实际的存储空间。例如：

```cpp
class Widget {
public:
static const std::size_t MinVals = 28;  // 声明并初始化
};
```

MinVals被声明为static const整型数据成员，并且在类内部进行了初始化。编译器会在所有使用Widget::MinVals的地方直接替换为28，因此不需要为MinVals分配内存。

**为什么这仍然是声明？**

编译器优化：编译器会对static const整型数据成员进行常量传播，这意味着它可以直接将值28插入到所有使用MinVals的地方，而不需要实际的存储空间。由于不需要实际的存储空间，MinVals不会在对象文件中生成符号，因此不会产生外部链接。

如果需要通过引用或指针传递MinVals，或者需要获取其地址，那么必须提供一个定义。

```cpp
// 在Widget的.cpp文件中
const std::size_t Widget::MinVals;  // 定义
```

引用和指针需要实际的内存地址来指向，而不仅仅是值。如果没有定义，编译器将无法找到实际的内存地址，导致链接错误。

```cpp
class Widget{
public:
static const std::size_t MinVals = 28; //声明并初始化
};
//使用
void printValue(std::size_t val) {
    std::cout << "Value: " << val << std::endl;
}

template<typename T>
void fwd(T&& arg) {
    printValue(std::forward<T>(arg));
}

int main() {
    std::vector<int> widgetData;
    widgetData.reserve(Widget::MinVals);//使用MinVals
    printValue(Widget::MinVals);//可以，视为“printValue(28)”
    //下面的调用会导致链接错误，因为fwd需要实际的内存地址
    //fwd(Widget::MinVals);  // 错误！不应该链接
    return 0;
}
```

为了修复链接错误，需要在.cpp文件中提供定义：

```cpp
// 在Widget的.cpp文件中
const std::size_t Widget::MinVals;  // 定义
```

尽管代码中没有使用`MinVals`的地址，但是`fwd`的形参是通用引用,引用在编译器生成的代码中，通常被视作指针。在程序的二进制底层代码中指针和引用是一样的。在这个水平上，引用只是可以自动解引用的指针。通过引用传递`MinVals`实际上与通过指针传递`MinVals`是一样的，因此，必须有内存使得指针可以指向。通过引用传递的整型`static const`数据成员，通常需要定义它们，这个要求可能会造成在不使用完美转发的代码成功的地方，使用等效的完美转发失败。

根据标准，通过引用传递`MinVals`要求有定义。但不是所有的实现都强制要求这一点。所以，取决于你的编译器和链接器，你可能发现你可以在未定义的情况使用完美转发，恭喜你，但是这不是那样做的理由。为了具有可移植性，只要给整型`static const`提供一个定义：

```cpp
const std::size_t Widget::MinVals;  //在Widget的.cpp文件
```

注意定义中不要重复初始化（这个例子中就是赋值28）。但是不要忽略这个细节。如果你忘了，并且在两个地方都提供了初始化，编译器就会报错，提醒你只能初始化一次。

**重载函数与模板名称在完美转发中的问题**

假设有一个函数f，接受一个函数指针作为参数，并且这个函数指针的类型是int (*)(int)。可以通过传递一个符合条件的函数来定制f的行为。

```cpp
void f(int (*pf)(int));  // pf = "process function"
// 或者使用更简单的非指针语法
void f(int pf(int));
```

重载函数的问题

如果有一个重载函数processVal

```cpp
int processVal(int value);
int processVal(int value, int priority = 1);
```

可以直接将processVal传递给f，因为编译器可以根据f的参数类型推断出需要哪个版本的processVal：

```cpp
f(processVal);  //可以,编译器选择正确的processVal版本
```

当使用一个完美转发函数fwd时，问题出现了。fwd是一个模板函数，没有具体的类型信息，因此编译器无法确定应该传递哪个版本的processVal。

```cpp
template<typename T>
void fwd(T&& arg) {
    f(std::forward<T>(arg));
}
fwd(processVal);  // 错误！哪个processVal？
```

函数模板的问题

同样的问题也出现在函数模板上。假设有一个函数模板workOnVal：

```cpp
template<typename T>
T workOnVal(T param) {
    // 处理值的模板
}
```

尝试将workOnVal传递给fwd也会失败，因为编译器不知道应该实例化哪个版本的workOnVal：

```cpp
fwd(workOnVal);  // 错误！哪个workOnVal实例？
```

解决方法

为了使完美转发能够处理重载函数或函数模板，需要显式地指定要传递的具体函数或函数模板实例。这可以通过创建一个具体类型的函数指针来实现。（这个没看懂）

定义类型别名：

定义一个类型别名，表示所需的函数指针类型。

```cpp
using ProcessFuncType = int (*)(int);  // 定义类型别名
```

创建函数指针：

使用该类型别名创建一个函数指针，并将其初始化为所需的重载函数或函数模板实例。

```cpp
ProcessFuncType processValPtr = processVal;  // 指定所需的processVal签名
```

传递函数指针：

将创建的函数指针传递给fwd。

```cpp
fwd(processValPtr);  // 可以
```

处理函数模板：

对于函数模板，可以使用static_cast来显式地实例化并传递。

```cpp
fwd(static_cast<ProcessFuncType>(workOnVal));  // 也可以
```

```cpp
#include <iostream>
// 目标函数
void f(int (*pf)(int)) {
    std::cout << "f called with: " << pf(42) << std::endl;
}
// 重载函数
int processVal(int value) {
    return value * 2;
}
int processVal(int value, int priority) {
    return value + priority;
}
// 函数模板
template<typename T>
T workOnVal(T param) {
    return param * 3;
}
// 完美转发函数
template<typename T>
void fwd(T&& arg) {
    f(std::forward<T>(arg));
}
int main() {
    // 定义类型别名
    using ProcessFuncType = int (*)(int);
    // 创建函数指针
    ProcessFuncType processValPtr = processVal;
    // 传递函数指针
    fwd(processValPtr);  // 正确
    // 传递函数模板实例
    fwd(static_cast<ProcessFuncType>(workOnVal<int>));  // 正确
    return 0;
}
```

_**<u>位域与完美转发的问题</u>**_

位域（bit-fields）是一种特殊的成员变量，用于节省内存空间。它们通常用来表示结构体或类中的小整数。然而，当涉及到完美转发时，位域会带来一些特殊的问题。位域可能只占用一个机器字的部分位，例如32位整型中的某些位。这些位无法直接寻址，因此不能通过指针或引用直接访问。<u>C++标准明确禁止非const引用绑定到位域上</u>，因为位域可能不是对齐的，且不支持直接寻址。

假设有一个IPv4头部结构体定义如下：

```cpp
struct IPv4Header {
std::uint32_t version:4,IHL:4,DSCP:6,ECN:2,totalLength:16;
//其他字段...
};
```

如果有一个函数f接收一个std::size_t类型的参数，并且希望使用IPv4Header对象的totalLength字段调用它，那么直接调用是可行的：

```cpp
void f(std::size_t sz);  //要调用的函数
IPv4Header h;
// 填充h.totalLength
f(h.totalLength);  //可以正常工作
```

但是，如果你希望通过一个转发函数fwd来调用f，并且fwd的参数是引用类型，那么就会出现问题。

```cpp
template<typename T>
void fwd(T&& arg) {
    f(std::forward<T>(arg));
}
IPv4Header h;
// 填充h.totalLength
fwd(h.totalLength);  // 错误！non-const引用不能绑定到位域
```

解决方法

为了使完美转发能够处理位域，可以采取以下几种方法：

（1）按值传递

将位域的值复制到一个新的变量中，然后将这个新变量传递给目标函数。

```cpp
auto length = static_cast<std::uint16_t>(h.totalLength);
fwd(length);  // 正确，传递的是副本
```

（2）传const引用

使用const引用传递位域的值。根据C++标准，const引用实际上绑定到一个包含位域值的临时整型对象。

```cpp
void f(const std::uint16_t& sz); //修改f的参数为const引用
template<typename T>
void fwd(T&& arg){
    f(std::forward<T>(arg));
}
IPv4Header h;
//填充h.totalLength
fwd(h.totalLength);  //正确,const引用绑定到临时对象
```

（3）显式创建临时对象

在调用转发函数之前，显式地创建一个临时对象，然后传递这个临时对象。

```cpp
auto length = static_cast<std::uint16_t>(h.totalLength);
fwd(length);  // 正确，传递的是副本
```

**请记住：**

+ 当模板类型推导失败或者推导出错误类型，完美转发会失败。
+ 导致 完美转发失败的实参种类有 花括号初始化，作为 空指针的`0`或者 `NULL`，仅有声明的整型`static const`数据成员，模板和重载函数的名字，位域。

<h1 id="baf4262e">第6章 lambda表达式</h1>
<h5 id="555f4d10">前置：lambda表达式</h5>
lambda 表达式的特点

（1）即时定义:在使用的位置直接定义函数逻辑，使得代码更加直观和紧凑。

（2）减少错误:在需要的地方定义函数，避免因函数分离而导致上下文理解困难或参数传递错误。

**捕获列表**

捕获列表用于指定lambda表达式能够访问的外部变量。它位于[]中，并且支持以下几种方式：

_<u>（1）按值捕获</u>_

[=]:以值传递的方式捕捉所有父作用域中的变量，包括this指针。这意味着被捕捉的变量的值会被复制到lambda表达式的环境中。如果变量在lambda表达式创建后发生了变化，这些变化不会反映在lambda内部。

_<u>（2）按引用捕获</u>_

[&]:以引用传递的方式捕捉所有父作用域中的变量，包括this指针。这种方式使得lambda表达式可以直接操作原始变量，因此任何对这些变量的修改都会影响到原始变量。

悬空引用风险：当闭包对象的生命周期超过其所引用的局部变量时，可能会导致未定义行为。这是因为局部变量在其作用域结束时即被销毁，而闭包可能仍然存在并尝试访问这些已不存在的变量。

**闭包和闭包类**

+ **闭包**是_lambda_创建的运行时对象。依赖捕获模式，闭包持有被捕获数据的副本或者引用。在上面的`std::find_if`调用中，闭包是作为第三个实参在运行时传递给`std::find_if`的对象。
+ **闭包类**（_closure class_）是从中实例化闭包的类。每个_lambda_都会使编译器生成唯一的闭包类。_lambda_中的语句成为其闭包类的成员函数中的可执行指令。

闭包可以拷贝，并且可以作为函数的参数。

```cpp
{
    int x;//x是局部对象
    auto c1 = [x](int y){return x*y >55;};//c1是lambda产生的闭包的副本
    auto c2 = c1; //c2是c1的拷贝 
    auto c3=c2;                           //c3是c2的拷贝
}
```

`c1`，`c2`，`c3`都是_lambda_产生的闭包的副本。

模糊_lambda_，闭包和闭包类之间的界限是可以接受的。在随后的Item中，区分什么存在于编译期（_lambdas_ 和闭包类），什么存在于运行时（闭包）以及它们之间的相互关系是重要的。

**应用**

_lambda_表达式没有给语言带来新的表达能力。_lambda_可以做的所有事情都可以通过其他方式完成。但是_lambda_是创建函数对象相当便捷的一种方法。没有_lambda_时，STL中的“`_if`”算法（比如，`std::find_if`，`std::remove_if`，`std::count_if`等）通常需要繁琐的谓词，但是当有_lambda_可用时，这些算法使用起来就变得相当方便。用比较函数（比如，`std::sort`，`std::nth_element`，`std::lower_bound`等）来自定义算法也是同样方便的。

在STL外，_lambda_可以快速创建`std::unique_ptr`和`std::shared_ptr`的自定义删除器（见item18和item19)，并且使线程API中条件变量的谓词指定变得同样简单(参见item39)。_lambda_有利于即时的回调函数，接口适配函数和特定上下文中的一次性函数。

<h4 id="4584bed2">条款三十一：避免使用默认捕获模式</h4>
C++11两种默认的捕获模式：按引用捕获和按值捕获。默认按引用捕获模式可能会带来悬空引用的问题，而默认按值捕获模式也没有解决这个问题，还会让你以为你的闭包是独立的（事实上也不是独立的）。

按引用捕获会导致闭包中包含了对某个局部变量或者形参的引用，变量或形参只在定义_lambda_的作用域中可用。如果该_lambda_创建的闭包生命周期超过了局部变量或者形参的生命周期，那么闭包中的引用将会变成悬空引用。

举个例子，假如过滤函数(filtering function)的一个容器，该函数接受一个`int`，并返回一个`bool`，该`bool`的结果表示传入的值是否满足过滤条件：

```cpp
using FilterContainer = std::vector<std::function<bool(int)>>;
//“using”参见条款9， //std::function参见条款2
FilterContainer filters;  //过滤函数
```

我们可以添加一个过滤器，用来过滤掉5的倍数：

```cpp
filters.emplace_back(                       //emplace_back的信息见条款42
    [](int value) { return value % 5 == 0; }
    );
```

可能需要的是能够在运行期计算除数(divisor)，不能将5硬编码到_lambda_中。

因此添加的过滤器逻辑将会是如下这样：

```cpp
void addDivisorFilter(){
    auto calc1 = computeSomeValue1();
    auto calc2 = computeSomeValue2();
    auto divisor = computeDivisor(calc1, calc2);
    filters.emplace_back(
        // 危险.对divisor的引用
        // 将会悬空.
        [&](int value) { return value % divisor == 0; } 
        );
}
```

_lambda_对局部变量`divisor`进行了引用，但该变量的生命周期会在`addDivisorFilter`返回时结束，刚好就是在语句`filters.emplace_back`返回之后。因此添加到`filters`的函数添加完，该函数就死亡了。使用这个过滤器（那个添加进`filters`的函数）会导致未定义行为，这是由它被创建那一刻起就决定了的。

_**<u>立即使用的闭包按引用捕获是安全的，但是这种安全是不确定的。</u>**_

当一个lambda表达式被立即使用（例如作为STL算法的参数）且不会被拷贝或存储时，默认按引用捕获模式（[&]）是安全的。这是因为此时闭包的生命周期与父函数局部变量的生命周期一致，不存在悬空引用的风险。

当谈论“立即使用的闭包”时，指的是lambda表达式在其创建后马上被使用，并且不会在函数之外保留或存储。这种情况下，默认按引用捕获模式（[&]）是安全的，因为此时lambda表达式的生命周期与父函数中局部变量的生命周期是一致的。在lambda表达式执行期间，所依赖的所有局部变量都仍然有效。

具体的例子

假设有一个容器，想要检查容器中的所有元素是否都是某个特定除数的倍数。可以使用std::all_of算法和一个lambda表达式来完成这个任务。在这个场景中，lambda表达式将作为std::all_of的一个参数立即被调用，并且不会被保存下来以供以后使用。

```cpp
// 假设这些函数已经定义好了
int computeSomeValue1(){return 2;}
int computeSomeValue2(){return 3;}
int computeDivisor(int a, int b){return a + b;}
template<typename C>
void checkAllElementsAreMultiples(const C& container) {
    auto calc1 = computeSomeValue1();
    auto calc2 = computeSomeValue2();
    auto divisor = computeDivisor(calc1, calc2);
    //立即使用的闭包
    if(std::all_of(container.begin(), container.end(),
        [&](const typename C::value_type& value)->bool
    {
        return value % divisor == 0;
    })){
        std::cout << "All elements are multiples of the divisor.\n";
    } 
    else
    {
        std::cout << "Not all elements are multiples of the divisor.\n";
    }
}
int main() {
    std::vector<int> numbers = {5,10,15,20};
    checkAllElementsAreMultiples(numbers);
}
```

calc1、calc2 和 divisor 是在 checkAllElementsAreMultiples 函数内部定义的局部变量。lambda表达式：使用默认按引用捕获模式[&]的lambda表达式，它可以访问并修改checkAllElementsAreMultiples 函数内的局部变量。该lambda表达式作为 std::all_of 算法的第三个参数传递，std::all_of 会立即遍历容器并对每个元素应用lambda表达式。一旦 std::all_of 完成，lambda表达式就不再存在。由于lambda表达式是在 checkAllElementsAreMultiples 函数内创建并立即使用，它的生命周期完全包含在该函数的作用域内。因此，lambda表达式访问的局部变量在整个操作期间都是有效的，不存在悬空引用的风险。

C++14支持了在_lambda_中使用`auto`来声明变量，上面的代码在C++14中可以进一步简化，`ContElemT`的别名可以去掉，`if`条件可以修改为：

```cpp
if (std::all_of(begin(container), end(container),
    [&](const auto& value)               // C++14
{ return value % divisor == 0; }))
```

这种做法的安全性是不确定的。如果后续开发中有人将这个lambda表达式复制到其他上下文中使用(比如添加到filters容器中)，而此时divisor等局部变量已经超出作用域，就会重新引入悬空引用的问题。

**<u>显式捕获的重要性</u>**

显式列出lambda依赖的所有局部变量和形参（如使用[=]或明确指定变量名）是一种更加符合软件工程规范的做法。

（1）显式捕获的重要性在于有助于提高代码可读性。

通过明确指出lambda表达式依赖的外部变量，可以更清晰地传达开发者意图，并且帮助其他阅读代码的人更好地理解代码的行为和潜在的风险。当使用显式捕获时，可以清楚地看到哪些变量被lambda表达式所依赖,使得代码更加直观，读者无需查看整个函数或作用域来确定lambda可能访问哪些变量。如果出现问题，例如某个变量在lambda中未定义或行为不符合预期，显式捕获可以帮助快速定位问题所在。你只需检查lambda的捕获列表即可知道它依赖哪些变量。

（2）预防悬空引用

通过显式列出需要捕获的变量，可以提醒开发者考虑这些变量的生命周期。比如，如果一个变量是在lambda创建后很快就会销毁的局部变量，那么按值捕获可能是更好的选择，以确保lambda内部持有该变量的一个独立副本。

默认按引用捕获模式 [&] 会捕获所有父作用域中的变量，可能导致一些隐藏的依赖关系，增加了代码复杂度和出错几率。显式捕获则强制开发者思考并声明每个依赖项，减少不必要的依赖。对于团队协作开发而言，显式捕获能确保每个成员对代码的理解一致，降低因不同理解而导致的错误风险。

一个解决问题的方法是，`divisor`默认按值捕获进去，可以按照以下方式来添加_lambda_到`filters`。

```cpp
filters.emplace_back(
    [=](int value) { return value % divisor == 0; }
    );
//现在divisor不会悬空了
```

在通常情况下，按值捕获并不能完全解决悬空引用的问题。如果按值捕获的是一个指针，将该指针拷贝到_lambda_对应的闭包里，但这样并不能避免_lambda_外`delete`这个指针的行为，从而导致副本指针变成悬空指针。

当按值捕获一个指针变量时，实际上是复制了这个指针的值(内存地址)，不是它所指向的对象。因此，如果原始对象在lambda表达式之外被销毁或删除，那么即使你有一个指针的副本，它也指向了一个已经无效的内存位置。这种情况下的指针被称为悬空指针，使用它会导致未定义行为。

```cpp
void addPointerFilter() {
    int* ptr = new int(10);//动态分配的整数
    //按值捕获指针
    filters.emplace_back(
        [ptr](int value) { return value == *ptr; }
        );
    delete ptr; //销毁指针指向的对象
}
```

ptr指向的内存是在addPointerFilter函数内部动态分配的。按值捕获了ptr，意味着闭包内有一个指向相同内存位置的指针副本。然而，在delete ptr;之后，该内存位置不再有效，所以闭包内的指针变成了悬空指针。

**使用智能指针解决按值捕获指针的问题**

使用如std::shared_ptr或std::unique_ptr等智能指针来管理对象的生命周期。智能指针可以通过引用计数等方式自动处理对象的释放，确保只要还有引用存在，对象就不会被销毁。

```cpp
void addSmartPointerFilter() {
    auto ptr = std::make_shared<int>(10);// 使用智能指针
    //捕获智能指针
    filters.emplace_back(
        [ptr](int value) { return value == *ptr; }
        );
    //不需要手动delete，智能指针会自动管理内存
}
```

假设在一个`Widget`类，可以实现向过滤器的容器添加条目：

```cpp
class Widget {
public:
// 构造函数
void addFilter() const; // 向filters添加条目private:
int divisor; // 在Widget的过滤器使用
};
```

这是`Widget::addFilter`的定义：

```cpp
void Widget::addFilter()const{
    filters.emplace_back(
        [=](int value) { return value % divisor == 0; }
        );
}
```

这个做法看起来是安全的代码。_lambda_依赖于`divisor`，但默认的按值捕获确保`divisor`被拷贝进了_lambda_对应的所有闭包中。

但是捕获只能应用于_lambda_被创建时所在作用域里的non-`static`局部变量（包括形参）。在`Widget::addFilter`的视线里，`divisor`并不是一个局部变量，而是`Widget`类的一个成员变量。它不能被捕获。而如果默认捕获模式被删除，代码就不能编译了：

```cpp
voidWidget::addFilter()const{
    filters.emplace_back(                               //错误！
        [](int value) { return value % divisor == 0; }  //divisor不可用
        ); 
}
```

另外，如果尝试去显式地捕获`divisor`变量（或者按引用或者按值——这不重要），也一样会编译失败，因为`divisor`不是一个局部变量或者形参。

```cpp
voidWidget::addFilter() const{
    filters.emplace_back(
        [divisor](int value)  //错误.没有名为divisor局部变量可捕获
        { return value % divisor == 0; }
        );
}
```

如果默认按值捕获不能捕获`divisor`，而不用默认按值捕获代码就不能编译。解释就是这里隐式使用了一个原始指针:`this`。每一个non-`static`成员函数都有一个`this`指针，每次使用一个类内的数据成员时都会使用到这个指针。例如，在任何`Widget`成员函数中，编译器会在内部将`divisor`替换成`this->divisor`。在默认按值捕获的`Widget::addFilter`版本中，

```cpp
voidWidget::addFilter()const{
    filters.emplace_back(
        [=](int value) { return value % divisor == 0; }
        );
}
```

真正被捕获的是`Widget`的`this`指针，而不是`divisor`。编译器会将上面的代码看成以下的写法：

```cpp
void Widget::addFilter()const{
    auto currentObjectPtr = this;
    filters.emplace_back(
        [currentObjectPtr](int value)
        { return value % currentObjectPtr->divisor == 0; }
        );
}
```

明白了这个就相当于明白了_lambda_闭包的生命周期与`Widget`对象的关系，闭包内含有`Widget`的`this`指针的拷贝。

特别是考虑以下的代码，参考[第4章](https://cntransgroup.github.io/EffectiveModernCppChinese/4.SmartPointers/item18.html)的内容，只使用智能指针：

```cpp
using FilterContainer = std::vector<std::function<bool(int)>>; 
//跟之前一样
FilterContainer filters;  //跟之前一样
void doSomeWork(){
    auto pw = std::make_unique<Widget>();                     
    //创建Widget；
    pw->addFilter();             //添加使用Widget::divisor的过滤器
}                            
//销毁Widget；filters现在持有悬空指针！
```

当调用`doSomeWork`时，就会创建一个过滤器，其生命周期依赖于由`std::make_unique`产生的`Widget`对象，即一个含有指向`Widget`的指针——`Widget`的`this`指针——的过滤器。这个过滤器被添加到`filters`中，但当`doSomeWork`结束时，`Widget`会由管理它的`std::unique_ptr`来销毁。这时`filter`会含有一个存着悬空指针的条目。

这个特定的问题可以通过给想捕获的数据成员做一个局部副本,然后捕获这个副本去解决：

```cpp
void Widget::addFilter()const{
    auto divisorCopy = divisor;                 //拷贝数据成员
    filters.emplace_back(
        [divisorCopy](int value)                //捕获副本
        { return value % divisorCopy == 0; }	//使用副本
        );
}
```

事实上如果采用这种方法，默认的按值捕获也是可行的。

```cpp
void Widget::addFilter() const{
    auto divisorCopy = divisor;            //拷贝数据成员
    filters.emplace_back(
        [=](int value)                      //捕获副本
        { return value % divisorCopy == 0; }//使用副本
        );
}
```

C++14中,一个更好的捕获成员变量的方式时使用通用的_lambda_捕获：

```cpp
void Widget::addFilter() const{
    filters.emplace_back(                //C++14：
        [divisor = divisor](int value)   //拷贝divisor到闭包
        { return value % divisor == 0; } //使用这个副本
        );
}
```

这种通用的_lambda_捕获并没有默认的捕获模式，因此在C++14中，本条款的建议——避免使用默认捕获模式——仍然是成立的。

_**<u>使用C++14通用lambda捕获</u>**_

```cpp
void Widget::addFilter() const {
    filters.emplace_back(
        [divisor = divisor](int value) { // C++14通用捕获，直接拷贝成员变量
            return value % divisor == 0;
        }
        );
}
```

C++14引入通用初始化捕获，使得lambda表达式的捕获列表更加灵活。通过通用初始化捕获，你可以在捕获列表中直接初始化变量，并且这些变量可以是任意类型，包括临时对象、成员变量的副本等。这一特性允许开发者在lambda表达式内部创建并使用局部变量，而无需依赖外部作用域中的变量。

通用初始化捕获的基本形式

```cpp
[ capture-list ] ( parameters ) mutable -> return-type {
    // lambda body
    }
```

其中，capture-list 可以包含形如 [identifier = expression] 的元素，用于初始化捕获的变量。这里 identifier 是一个新定义的名字，expression 是用来初始化它的表达式。

**静态变量按值捕获可能存在的问题**

代码中的关键点是使用了[=]作为捕获列表，这表示lambda会以值的形式捕获外部作用域的所有变量。然而，divisor、calc1 和 calc2 都是静态变量，它们具有静态存储持续时间，意味着它们的生命周期与程序相同，并且不受lambda捕获规则的影响。尽管lambda看起来像是按值捕获了所有东西，实际上它并没有捕获这些静态变量；相反，它直接引用了这些静态变量。

由于divisor是一个静态变量，它在每次调用addDivisorFilter时都会递增。这意味着每一个新创建的lambda都将使用更新后的divisor值，而不是创建时的值。因为读者可能误以为每个lambda都是独立的，并且保存了创建时的环境状态。

为了消除这种风险，可以明确地指定要捕获的变量，而不是依赖默认的捕获模式。如果确实需要使用静态变量，应该清楚地表明这一点，并确保理解其行为。

```cpp
void addDivisorFilter() {
    static auto calc1 = computeSomeValue1(); // 静态计算值1
    static auto calc2 = computeSomeValue2(); // 静态计算值2
    static auto divisor = computeDivisor(calc1, calc2); // 静态除数
    // 明确指出我们不捕获任何局部变量
    filters.emplace_back(
        [] (int value) -> bool { 
            return value % divisor == 0; // 直接使用静态变量divisor
        }
        );
    ++divisor; // 调整静态除数
}
```

如果希望每个lambda都拥有自己的一份divisor副本，应该在divisor更新之前就对其进行捕获。

```cpp
void addDivisorFilter() {
    static auto calc1 = computeSomeValue1();
    static auto calc2 = computeSomeValue2();
    static auto divisor = computeDivisor(calc1, calc2);
    //在divisor更新前进行捕获
    auto current_divisor = divisor;
    filters.emplace_back(
        [current_divisor](int value) -> bool { 
            return value % current_divisor == 0; // 使用当前的divisor值
        }
        );
    ++divisor;
}
```

这样可以确保每个lambda都有自己的divisor值副本，并且不会受到后续对静态变量divisor修改的影响。

<h4 id="9c400246">条款三十二：使用初始化捕获来移动对象到闭包中</h4>
C++11闭包捕获机制仅支持按值或按引用捕获变量，这使得对于只能移动的对象（如std::unique_ptr或std::future）的处理变得困难。此外，对于复制开销高的对象，C++11无法直接实现更高效的移动语义来代替复制。

C++14引入了初始化捕获（也称为通用lambda捕获），使得开发者可以将表达式的结果直接作为闭包的数据成员进行初始化，并且允许使用std::move将不可复制的对象移动到闭包中。这种新形式的捕获为开发者提供了更大的灵活性和效率。使用初始化捕获将std::unique_ptr<Widget>移动到闭包中：

```cpp
auto pw = std::make_unique<Widget>();
// 设置 *pw
auto func = [pw = std::move(pw)] { 
    return pw->isValidated() && pw->isArchived(); 
};
```

对于不支持C++14的编译器，或者当开发者希望模拟C++14中的移动捕获时，有几种方法可以在C++11中实现类似的行为：

手动创建一个类来模拟lambda表达式的行为，包括使用构造函数接收要移动的对象。使用std::bind结合lambda表达式来间接实现移动捕获的效果。

```cpp
class IsValAndArch {
public:
using DataType = std::unique_ptr<Widget>;
explicit IsValAndArch(DataType&& ptr) : pw(std::move(ptr)) {}
bool operator()() const { 
    return pw->isValidated() && pw->isArchived(); 
}
private:
DataType pw;
};
auto func = IsValAndArch(std::make_unique<Widget>())();
```

虽然C++14带来了更方便的初始化捕获功能，但即使是在C++11中，也可以通过额外的手动编码来实现等效的功能。尽管这样做可能会增加一些代码量，但它并不影响最终能够达到的目标——即有效地将对象移动到闭包中，避免不必要的复制开销。

**使用std::bind模拟C++14的移动捕获**

在C++11中，通过结合std::bind和lambda表达式，可以间接地将对象移动到闭包中。下面是如何在C++11中模拟上述C++14代码的例子：

```cpp
std::vector<double> data;
// 填充data
auto func = std::bind(
[](const std::vector<double>& data) // lambda接收data作为参数
{ 
    /* 使用data */
},
std::move(data)                     // 将data作为右值传递给bind
);
```

std::bind的第一个参数是一个可调用对象（在这个例子中是一个lambda表达式），后续参数是要传递给该可调用对象的实参。由于std::move(data)产生的是一个右值，因此std::bind会尝试移动构造这个实参，而不是复制它。当调用func时，std::bind会调用其内部存储的lambda，并将移动构造的数据副本作为参数传递给lambda。对于需要修改数据的情况，可以通过将lambda声明为mutable，并且接受非const引用参数来允许修改：

```cpp
auto func = std::bind(
[](std::vector<double>& data) mutable{  /*使用data*/
},
std::move(data)
);
```

如果要移动初始化的对象直接在std::bind中创建(如std::unique_ptr<Widget>)，则可以这样做：

```cpp
auto func = std::bind(
[](const std::unique_ptr<Widget>& pw){ 
    return pw->isValidated() && pw->isArchived(); 
},
std::make_unique<Widget>() // 直接在bind中创建
);
```

使用std::bind创建一个可调用对象，将其赋值给变量func。std::bind允许我们绑定函数或函数对象及其参数，使得可以稍后调用这个绑定了参数的函数对象。

第一个参数：lambda表达式

```cpp
[](const std::unique_ptr<Widget>& pw) { 
    return pw->isValidated() && pw->isArchived(); 
}
```

它接收一个常量引用参数pw，类型为std::unique_ptr<Widget>。这是因为std::unique_ptr是不可复制的，但是可以被移动；同时，不打算修改pw所指向的对象，因此使用了const修饰符。Lambda主体内的逻辑是检查pw所指向的Widget对象是否经过验证且已归档（即调用isValidated()和isArchived()方法)。

第二个参数std::make_unique<Widget>()，std::make_unique是一个辅助函数，用于创建并返回一个std::unique_ptr，该指针指向一个新分配的Widget对象。

std::make_unique<Widget>()直接在std::bind中调用，意味着它创建的对象将作为右值传递给std::bind，进而尝试通过移动语义将这个std::unique_ptr<Widget>移动到由std::bind创建的函数对象内部。

std::bind会创建一个函数对象func，当调用func()时，它实际上是在调用上述lambda表达式,并将std::make_unique<Widget>()创建的std::unique_ptr<Widget>作为实参传递给lambda。

因为std::make_unique<Widget>()创建的是一个临时对象（右值），所以std::bind会尝试移动构造这个临时对象，而不是复制它，从而有效地避免了不必要的资源复制。当lambda执行时，它操作的是由std::bind移动构造的std::unique_ptr<Widget>副本，这副本是作为左值存储在std::bind生成的函数对象中的。最终，func()的返回值是pw->isValidated() && pw->isArchived()的结果，即判断Widget对象是否既经过验证又已被归档。

<h4 id="e4edc89c">条款三十三：对auto&&形参使用decltype以std::forward它们</h4>
**泛型**_**lambda**_是C++14中特性之一,因为在_lambda_的形参中可以使用`auto`关键字。

特性的实现:在闭包类中的`operator()`函数是一个函数模版。

例如存在这么一个_lambda_

```cpp
auto f=[](auto x){ return func(normalize(x)); };
```

对应的闭包类中的函数调用操作符看来就变成这样：

```cpp
class SomeCompilerGeneratedClassName {
public:
template<typename T> //auto返回类型见条款3
auto operator()(T x)const{ return func(normalize(x)); }
//其他闭包类功能
};
```

_lambda_对变量`x`做的唯一一件事就是把它转发给函数`normalize`。如果函数`normalize`对待左值右值的方式不一样，这个_lambda_的实现方式就不大合适了，因为即使传递到_lambda_的实参是一个右值，_lambda_传递进`normalize`的总是一个左值（形参`x`）。

作者的观点是关于如何在C++14中实现泛型lambda表达式的完美转发。当使用auto关键字定义lambda的参数时，它创建了一个通用引用（universal reference），这允许参数既可以绑定到左值也可以绑定到右值。然而，简单地将参数传递给另一个函数并不会保留原始值类别的信息（即是否为左值或右值）。为了保持这种价值类别，并正确地进行转发，需要使用std::forward结合decltype来确定原始参数的价值类别。

下面是作者提出的解决方案代码整理：

```cpp
// 单个参数的完美转发
auto f = [](auto&& param)
{
    return func(normalize(std::forward<decltype(param)>(param)));
};
// 支持可变数量参数的完美转发
auto f = [](auto&&... params)
{
    return func(normalize(std::forward<decltype(params)>(params)...));
};
```

auto&& param 定义了一个可以绑定到任何类型（无论是左值还是右值）的参数。

decltype(param) 用来获取参数的实际类型，包括它的引用资格。

std::forward<decltype(param)>(param) 用来根据原始实参的价值类别（左值或右值）正确地转发参数。通过这种方式，无论输入是什么类型的值（左值或右值），它们都会被适当地转发给内部调用的func(normalize(...))，确保了转发的“完美”，即不丢失任何有关原始值类别的信息。

**请记住：**对`auto&&`形参使用`decltype`以`std::forward`它们。

<h4 id="483cfada">条款三十四：考虑lambda而非std::bind</h4>
C++11中的`std::bind`是C++98的`std::bind1st`和`std::bind2nd`的后续，C++11 _lambda_几乎总是比`std::bind`更好的选择。 从C++14开始，_lambda_的作用不仅强大，而且是完全值得使用的。

与item32中一样，我们将从`std::bind`返回的函数对象称为**bind对象(**_bind objects)_。优先_lambda_而不是`std::bind`的最重要原因是_lambda_更易读。 

Lambda 表达式版本

```cpp
#include <chrono>
#include <functional>
// 假设 setAlarm 已定义
void setAlarm(std::chrono::steady_clock::time_point t,enum class Sound s,
std::chrono::seconds d);
enum class Sound {Beep,Siren,Whistle};
//使用lambda创建一个函数对象,设置一小时后响30秒的警报器
auto setSoundL = [](Sound s) {
    using namespace std::chrono;
    using namespace std::literals; // 对于 C++14 后缀
    setAlarm(steady_clock::now() + 1h, s, 30s);
};
```

创建一个函数对象(即lambda)，接受一个Sound类型的参数，并用这个声音参数调用setAlarm，同时自动计算一小时后的时间点以及30秒的持续时间。使用C++14的时间字面量（如1h和30s）让代码更简洁。延迟求值：在lambda内部，steady_clock::now() + 1h是在调用lambda时才被计算，这意味着警报会在实际调用时的一小时后触发。

std::bind 版本 (原始尝试)

```cpp
#include <chrono>
#include <functional>
using namespace std::chrono;
using namespace std::placeholders;
// 错误的 std::bind 调用
auto setSoundB = std::bind(setAlarm, steady_clock::now() + 1h, _1, 30s); // 不正确！
std::bind 版本 (修正后的)
// 修正后的 std::bind 调用，使用嵌套 bind 推迟计算
auto setSoundB = std::bind(
setAlarm,
std::bind(std::plus<>(), std::bind(steady_clock::now), 1h),
_1,
30s
);
```

std::bind调用不正确的原因在于steady_clock::now() + 1h的求值时机。当使用steady_clock::now() + 1h作为std::bind的一个参数时，这个表达式是在std::bind被调用的那一刻就被计算（即立即求值），而不是在最终调用setAlarm时计算。

<u>第一层 std::bind </u>

setAlarm是最终要调用的目标函数。**<u>时间点:</u>**第二层std::bind的结果。**<u>声音类型：</u>**_1占位符，最终调用setSoundB时，传递一个Sound类型的值给它，这个值会被当作setAlarm的第二个参数。持续时间：30s，是一个固定的持续时间，代表警报响铃的时长。

<u>第二层 std::bind</u>

std::bind(std::plus<>(), std::bind(&std::chrono::steady_clock::now), 1h)

目标操作：std::plus<>()：加法操作。**(**使用当前时间(左侧操作数)加上1h(右侧操作数)**)**

左侧操作数：通过第三层std::bind获取，即当前时间steady_clock::now()。右侧操作数：1h，表示一小时的时间段。

<u>第三层 std::bind</u>

std::bind(&std::chrono::steady_clock::now)

&std::chrono::steady_clock::now取的是steady_clock::now函数的地址，为了推迟求值。当外部的setSoundB被调用时，这层std::bind会调用steady_clock::now()来获取当前时间。创建一个函数对象，该对象封装了对 std::chrono::steady_clock::now() 函数的调用。&std::chrono::steady_clock::now：取 steady_clock::now 函数的地址。steady_clock::now 是一个静态成员函数，它返回一个代表当前时间点的对象。通过在前面加上&，获取函数的指针。

std::bind：模板函数，用于创建一个函数对象，该对象可以存储、复制和调用一个可调用对象（如函数、lambda表达式或其它函数对象）以及与之关联的一组参数。

当执行 std::bind(&std::chrono::steady_clock::now)时，实际上在创建一个函数对象，不带任何参数，并且当这个函数对象被调用时，会调用 steady_clock::now() 来获取当前的时间点。

std::bind 版本 (C++11)

```cpp
#include <chrono>
#include <functional>
using namespace std::chrono;
using namespace std::placeholders;
// C++11 中等价于 lambda 的 std::bind 实现
auto setSoundB = std::bind(
setAlarm,
std::bind(std::plus<steady_clock::time_point>(), std::bind(&std::chrono::steady_clock::now), hours(1)),
_1,
seconds(30)
);
```

Lambda 表达式更易读、更直观，且能自然地处理延迟求值。std::bind虽然功能强大，但语法复杂，特别是在需要推迟计算时，会导致代码难以理解和维护。

创建一个检查值是否在一个范围内的函数对象

```cpp
// C++11版本使用std::bind
auto betweenB = std::bind(std::logical_and<bool>(),
std::bind(std::less_equal<int>(),lowVal,_1),
std::bind(std::less_equal<int>(),_1,highVal));
// C++11版本使用Lambda表达式
auto betweenL = [lowVal,highVal](int val)
{ return lowVal<=val&&val<=highVal; };
```

在C++11中，std::bind可用于模拟移动捕获和创建多态函数对象。C++14随着lambda对初始化捕获的支持以及auto形参的引入，这些特殊情况也不再是std::bind的优势所在。

与_lambda_相比，使用`std::bind`进行编码的代码可读性较低，表达能力较低，并且效率可能较低。 在C++14中，没有`std::bind`的合理用例。 但是，在C++11中，可以在两个受约束的情况下证明使用`std::bind`是合理的：

+ **移动捕获**。C++11的_lambda_不提供移动捕获，但是可以通过结合_lambda_和`std::bind`来模拟。 有关详细信息，请参阅item32，该条款还解释了在C++14中，_lambda_对初始化捕获的支持消除了这个模拟的需求。
+ **多态函数对象**。因为bind对象上的函数调用运算符使用完美转发，所以它可以接受任何类型的实参（以item30中描述的完美转发的限制为界限）。当要绑定带有模板化函数调用运算符的对象时，此功能很有用。 

```cpp
class PolyWidget {
public:
template<typename T>
void operator()(const T& param);
};
```

`std::bind`可以如下绑定一个`PolyWidget`对象：

```cpp
PolyWidget pw;
auto boundPW = std::bind(pw, _1);
```

`boundPW`可以接受任意类型的对象了：

```cpp
boundPW(1930);              //传int给PolyWidget::operator()
boundPW(nullptr);           //传nullptr给PolyWidget::operator()
boundPW("Rosebud"); 		//传字面值给PolyWidget::operator()
```

这一点无法使用C++11的_lambda_做到。 但是，在C++14中，可以通过带有`auto`形参的_lambda_轻松实现：

```cpp
auto boundPW = [pw](constauto& param)  //C++14 
{ pw(param); };
```

在C++11中增加了_lambda_支持，这使得`std::bind`几乎已经过时了，从C++14开始，更是没有很好的用例了。

**请记住：**

+ 与使用`std::bind`相比，_lambda_更易读，更具表达力并且可能更高效。
+ 只有在C++11中，`std::bind`可能对实现移动捕获或绑定带有模板化函数调用运算符的对象时会很有用。

<h1 id="263efd46">第7章 并发API</h1>
C++11将并发编程的概念整合到了标准库中。C++11引入了:

+ 任务 (tasks): 代表异步操作。
+ 期望 (futures) 和 共享期望 (shared_futures): 用于获取异步操作的结果。
+ 线程 (threads): 管理操作系统级别的线程。
+ 互斥 (mutexes): 保护共享资源，防止数据竞争。
+ 条件变量 (condition variables): 实现线程间的通信与同步。
+ 原子对象 (atomic objects): 提供无锁编程的基本构造块。

标准库提供了两种future模板：std::future和std::shared_future。虽然它们之间存在差异，但在许多应用场景下，这些区别并不显著(统称futures)。

<h4 id="9b943237">条款三十五：优先考虑基于任务的编程而非基于线程的编程</h4>
什么是任务？为什么要优先考虑基于任务的编程而不是基于线程的编程。

C++中开发者可以通过两种主要方式异步执行一个函数，如doAsyncWork()。这两种方法分别是基于线程(thread-based)和基于任务(task-based)的方式。

**基于线程的方式**

使用std::thread创建一个新的线程来执行doAsyncWork()函数，直接且直观，但也有其局限性。

```cpp
int doAsyncWork();
std::thread t(doAsyncWork);
```

（1）资源管理：开发者需要负责线程的生命周期管理，确保线程正确地启动、运行和结束。

（2）返回值处理：如果doAsyncWork有返回值或需要处理结果，基于线程的方法并不直接支持这一点。必须通过额外的机制（例如共享变量、管道等）来传递结果，增加复杂性。

（3）异常处理：如果doAsyncWork抛出异常，线程会调用std::terminate终止整个程序，除非在线程内部捕获了异常。

**基于任务的方式**

使用std::async将doAsyncWork作为任务提交，自动处理任务的调度和执行，并返回一个std::future对象用于获取任务的结果。

auto fut = std::async(doAsyncWork); // "fut"表示"future"

（1）简化代码：相比基于线程的方法，std::async的语法更简洁，减少了代码量。

（2）方便获取返回值：std::future提供了get方法，可以直接获取异步操作的结果或处理异常，而无需额外的同步机制。

（3）内置异常处理：如果任务抛出了未捕获的异常，get方法调用时会重新抛出该异常，允许调用者以受控方式处理异常，而不是直接导致程序崩溃。

**选择哪种方式？**

基于任务的方法通常优于基于线程的方法，在需要处理返回值或可能发生的异常的情况下。

std::async简化了异步编程模型，提高代码的可读性和维护性。然而对于一些特定的应用场景，比如需要对线程进行细粒度控制时，基于线程的方法可能仍然是必要的。

在C++并发编程，基于线程和基于任务的方式有本质的区别。"Thread”的三种含义

_**（1）硬件线程(Hardware Threads)**_

物理CPU核心上实际执行计算的能力。现代计算机架构通常为每个CPU核心提供一个或多个硬件线程，以支持并行处理。

_**（2）软件线程(software threads)或系统线程(OS Threads)**_

由操作系统管理,在硬件线程上运行的任务单元。操作系统可以调度更多的软件线程，超过硬件线程的数量，以便当某些线程被阻塞时(例如等待I/O操作完成),其他线程可以继续执行,从而提高整体吞吐量。

_**（3）std::thread对象**_

C++标准库提供的类，作为软件线程的句柄。可以代表一个正在运行的软件线程，也可以是未关联任何线程的空句柄（如默认构造的std::thread）。通过std::thread，我们可以启动、暂停、等待线程结束或分离线程。

**线程资源的限制**

（1）线程限额：操作系统能够支持的线程数量是有限的。如果尝试创建超出这个限额的线程，会抛出std::system_error异常。即使函数本身声明为noexcept，也不能避免这种情况发生。

（2）资源超额：当准备运行的软件线程数超过了可用硬件线程的数量时，就会出现资源超额的情况。这会导致操作系统频繁地进行上下文切换，增加调度开销，降低效率。此外，由于不同的软件线程可能在不同的硬件线程间迁移，导致缓存不友好，影响性能。

_设计良好并发软件需要考虑以下策略：_

<u>线程池（Thread Pool）</u>：使用线程池来重用一组固定的线程，而不是为每个任务创建新的线程。这有助于控制线程数量，减少上下文切换和初始化开销。

<u>任务调度（Task Scheduling）</u>：采用更高级别的任务调度机制，<u><font style="color:#c21c13;">比如使用std::async或第三方库，它们可以根据当前系统的负载动态调整任务的分配，确保高效的资源利用</font></u>。

异步I/O和非阻塞操作：

<u><font style="color:#c21c13;">尽量使用异步I/O和非阻塞API，减少线程因等待外部资源而阻塞的可能性，从而提高并发性。</font></u>

使用std::async的优势

std::async提供了一种更高级别的抽象，将线程管理的责任交给了标准库的开发者。

（1）<u><font style="color:#c21c13;">简化线程管理：开发者不再需要直接处理线程创建、销毁等细节问题</font></u>。

（2）<u><font style="color:#c21c13;">减少资源超额的风险：std::async默认启动策略并不保证会立即创建新的线程，而是允许调度器根据系统资源情况灵活决定任务的执行方式</font></u>。例如，在资源紧张时，它可以选择在等待结果的线程上执行任务，而不是开启新线程。

（3）提高负载均衡：<u><font style="color:#c21c13;">运行时调度程序</font></u>对整个系统的执行过程有更全面的理解，能够更好地处理负载不均衡的问题。

（4）利用先进的调度技术：<u><font style="color:#c21c13;">最前沿的线程调度器采用系统级线程池和</font></u>**<u><font style="color:#c21c13;">工作窃取算法</font></u>**<u><font style="color:#c21c13;">来优化资源利用和跨核心负载均衡。</font></u>

适用场景与局限性

尽管std::async提供了诸多便利，但在某些特定情况下，直接使用std::thread仍然可能是更好的选择。

访问底层线程API：当需要使用非常基础的线程功能或操作系统级别的特性（如线程优先级、亲和性等)，std::thread提供的native_handle成员函数可以直接访问底层API，而std::future不具备这种能力。

优化线程使用：对于已知执行概况的应用程序（如服务器软件），如果部署环境固定且作为唯一关键进程，可能更适合直接管理线程以实现最优性能。

实现特殊线程技术：当需要使用C++并发API之外的技术（如未被支持的平台上的线程池）时，std::thread提供了更大的灵活性。

**工作窃取算法**

任务分割:一个大的任务被分割成若干个互不依赖的小任务，这些小任务可以独立地并行执行。

队列分配:每个工作线程拥有自己的双端队列（deque），用来存储待处理的任务。线程通常会从自己队列的头部（front）获取任务执行。

任务窃取:当某个线程完成了自己队列中的所有任务而变得空闲时，它可以尝试从其他线程队列的尾部(back)窃取任务来执行。这样可以避免直接竞争同一个队列的头部位置，从而降低锁争用的可能性。

双端队列的作用:使用双端队列的原因是为了让原线程可以从一端高效地添加和移除任务（通常是从前端），同时允许其他线程从另一端安全地窃取任务而不引起冲突。

+ `std::thread` API不能直接访问异步执行的结果，如果执行函数有异常抛出，代码会终止执行。
+ 基于线程的编程方式需要手动的线程耗尽、资源超额、负责均衡、平台适配性管理。
+ 通过带有默认启动策略的`std::async`进行基于任务的编程方式会解决大部分问题。

<h4 id="030e4230">条款三十六：如果有异步的必要请指定std::launch::async</h4>
前置：async的启动策略

（1）std::launch::async

这个策略要求传给 std::async 的函数 f 必须在另一个线程上异步执行。函数 f 的执行与当前线程是并发的，即不会阻塞调用线程。

（2）std::launch::deferred:

使用此策略时，函数 f 不会立即执行。只有当返回的 std::future 对象上调用了 get() 或 wait() 方法时，函数 f 才会被调用并同步执行。同步执行意味着调用 get() 或 wait() 的线程会被阻塞，直到 f 完成执行。

如果没有调用 get() 或 wait()，那么 f 将永远不会被执行。此外，std::async 允许你不显式地指定启动策略，在这种情况下，实现可以自由选择 std::launch::async 或 std::launch::deferred 来执行任务。某些平台可能会优先考虑性能，选择最适合的策略，这可能取决于系统的当前负载、可用资源等。

（3）使用 std::launch::async 策略

```cpp
#include<iostream>
#include<future>
#include<chrono>
void print_id(int id) {
    std::cout << "Thread ID: " << id << "\n";
}
int main() {
    // 使用 launch::async 立即在另一个线程上异步执行任务
    auto fut = std::async(std::launch::async, print_id, 42);
    // 模拟主程序继续做其他事情
    std::this_thread::sleep_for(std::chrono::seconds(1));
    // 获取结果（这里只是等待异步操作完成）
    fut.get();
    return 0;
}
```

对这里的理解

在C++中，当调用fut.get()时，无论关联的异步任务是否返回有效值，该调用都会阻塞当前线程，直到任务完成。即使任务返回类型为void，fut.get()仍会等待任务执行完毕。以下是关键点分析：

std::launch::async策略：通过指定此策略，任务会立即在新线程中启动。因此，print_id(42)会异步执行。

std::future::get()的行为：

若任务未完成，get()会阻塞调用线程（此处为主线程），直到任务结束。

对于void返回类型，get()没有实际返回值，但仍需等待任务完成，以确保同步。

示例中的场景：

主线程休眠1秒（sleep_for），此时异步任务可能已执行完毕。

即使未休眠，若任务耗时较长，fut.get()仍会阻塞主线程，直到任务完成。

<h6 id="9c259e25">🌀思考：如果fut.get()会阻塞的话，那么异步执行的好处是什么？</h6>
异步的本质：重叠执行，减少总耗时

异步的本质是将任务启动与结果获取解耦。主线程启动异步任务后，可以立即继续执行后续代码，而非等待任务完成。fut.get()仅在需要结果时阻塞，此时异步任务可能已执行完毕，阻塞时间趋近于零。

```cpp
auto fut1 = std::async(std::launch::async, download_data, "url1"); 
// 异步下载任务1
auto fut2 = std::async(std::launch::async, download_data, "url2"); 
// 异步下载任务2
// 主线程继续处理其他工作（如界面响应、计算等）
process_other_tasks();
// 仅在需要结果时阻塞
auto data1 = fut1.get(); // 若任务1已下载完，不阻塞；否则等待
auto data2 = fut2.get();
```

总耗时 ≈ max(任务1耗时, 任务2耗时) + 主线程其他任务耗时（可并行）。

异步和同步的区别在于阻塞时机的选择

同步执行：任务启动后立即阻塞，无法做任何其他事情。

```cpp
download_data("url1"); // 阻塞直到完成
download_data("url2"); // 再次阻塞
```

总耗时 = 任务1耗时 + 任务2耗时

3. 异步的典型应用场景

I/O密集型任务(如文件读写、网络请求)：主线程启动异步I/O后，可继续处理计算或UI响应。

I/O操作由操作系统在后台完成，不占用CPU时间。

延迟敏感型程序（如游戏、实时系统）：

将耗时任务（如物理模拟）交给异步线程，主线程保持高响应性。

仅在关键帧同步时调用get()，避免卡顿。

批量任务并行处理：

```cpp
std::vector<std::future<void>> futures;
for (int i = 0; i < 1000; ++i) {
    futures.push_back(std::async(std::launch::async, process_data, i));
}
// 所有任务并行执行...
for (auto& fut : futures) {
    fut.get(); // 统一等待所有任务完成
}
```

总耗时 ≈ 单个任务耗时（假设资源充足）。

异步的代价与权衡

线程创建开销：频繁创建线程可能影响性能(可通过线程池优化)。

资源竞争：需注意数据竞争和锁的使用。

适用场景：异步适合任务独立、无严格顺序要求的场景。若任务强依赖，可能更适合同步。

<font style="background-color:#fbf5b3;">异步的优势不在于完全避免阻塞，而是通过任务的并行执行和灵活的阻塞时机选择，最大化利用CPU和I/O资源，缩短总耗时，提升程序响应性。</font>

这里思考为什么大多数网络库不选择使用异步的方式。

fut.get()的阻塞是可控的，且通常发生在任务可能已经完成的时刻，这是异步编程的关键设计哲学。

**async可能导致线程数量不可控的问题：**

在 C++ 中，直接无节制地使用 `std::async(std::launch::async, ...)` 确实可能导致线程数量不可控。但这一问题可以通过合理的设计和工具来解决。

1. `**std::async**`** 的线程管理机制**

+ **默认行为**：
    - C++ 标准未强制规定 `std::async` 的底层线程管理策略，具体实现由编译器决定。
    - 部分编译器（如 MSVC）可能复用线程池，而另一些（如 GCC）可能直接创建新线程。
+ **显式指定 **`**std::launch::async**`：

```cpp
auto fut = std::async(std::launch::async, task);
```

    - 强制每次调用都在新线程中启动任务。
    - **若高频调用，线程数会持续增长**，最终导致资源耗尽（如内存、线程句柄限制）。

2. **线程失控的风险场景**

```cpp
// 错误示例：循环中频繁创建异步任务
for (int i = 0; i < 1000000; ++i) {
    auto fut = std::async(std::launch::async, [i] {
        process_data(i);
    });
    // 未等待 fut 完成，可能导致线程堆积
}
```

+ **问题**：每个循环迭代都启动新线程，但未同步等待任务完成。
+ **后果**：线程数可能爆炸式增长，导致程序崩溃或系统资源耗尽。

3. **可控的异步线程管理策略**

策略 1：**限制并发任务数量**

通过信号量或计数器限制同时运行的异步任务数，避免资源竞争。

```cpp
#include <semaphore>
std::counting_semaphore<10> sem(10); // 最多允许 10 个并发任务
for (int i = 0; i < 1000; ++i) {
    sem.acquire(); // 等待空闲槽位
    std::async(std::launch::async, [i, &sem] {
        process_data(i);
        sem.release(); // 释放槽位
    });
}
```

策略 2：**显式等待任务完成**

确保在启动新任务前，已有任务已完成或可控。

```cpp
std::vector<std::future<void>> futures;
futures.reserve(1000);
for (int i = 0; i < 1000; ++i) {
    futures.push_back(std::async(std::launch::async, [i] {
        process_data(i);
    }));
}
//等待所有任务完成后再继续
for (auto& fut : futures) {
    fut.get();
}
```

策略 4：**结合 **`**std::async**`** 与任务队列**

手动实现任务队列，控制线程创建数量。

结合线程池的替代方案

由于 std::async 无法直接绑定到自定义线程池，需改用以下方法。

<h6 id="7c7876ad">`std::packaged_task`+队列</h6>
在C++ 中，结合 `std::packaged_task` 和任务队列是一种**手动实现线程池或异步任务调度系统**的经典方式。

1. **将任务封装为 **`**std::packaged_task**`** 对象**（关联 `std::future` 用于获取结果）。
2. **将任务提交到线程安全的队列中**。
3. **工作线程从队列中取出任务并执行**。

**核心组件**

| **组件** | **作用** |
| :--- | :--- |
| `std::packaged_task` | 封装任务逻辑，将结果绑定到 `std::future`。 |
| 线程安全队列 | 存储待执行的任务（需支持多线程并发访问）。 |
| 工作线程池 | 多个线程不断从队列中取出任务并执行。 |


**步骤 1：定义任务队列**

使用 `std::queue` + 互斥锁 (`std::mutex`) + 条件变量 (`std::condition_variable`) 实现线程安全队列：

```cpp
#include <queue>
#include <mutex>
#include <condition_variable>
#include <functional>

class TaskQueue {
public:
void Push(std::function<void()> task) {
    std::lock_guard<std::mutex> lock(mutex_);
    tasks_.push(std::move(task));
    cv_.notify_one();
}

std::function<void()> Pop() {
    std::unique_lock<std::mutex> lock(mutex_);
    cv_.wait(lock, [this] { return !tasks_.empty(); });
    auto task = std::move(tasks_.front());
    tasks_.pop();
    return task;
}

private:
std::queue<std::function<void()>> tasks_;
std::mutex mutex_;
std::condition_variable cv_;
};
```

**步骤 2：封装任务为 **`**std::packaged_task**`

将任务函数和返回值绑定到 `std::packaged_task`，并生成 `std::future`：

```cpp
template <typename Func, typename... Args>
auto SubmitTask(TaskQueue& queue, Func&& func, Args&&... args) 
-> std::future<decltype(func(args...))> 
{
    // 定义返回类型
    using ReturnType = decltype(func(args...));

    // 创建 packaged_task，绑定函数和参数
    auto task = std::make_shared<std::packaged_task<ReturnType()>>(
    std::bind(std::forward<Func>(func), std::forward<Args>(args)...)
    );

    // 获取 future
    std::future<ReturnType> future = task->get_future();

    // 将任务包装为 void() 类型，以便存入队列
    queue.Push([task]() { (*task)(); });

    return future;
}
```

**步骤 3：创建工作线程池**

启动多个线程不断从队列中取出任务并执行：

```cpp
class ThreadPool {
public:
ThreadPool(size_t thread_count) {
    for (size_t i = 0; i < thread_count; ++i) {
        workers_.emplace_back([this] {
            while (true) {
                auto task = queue_.Pop();
                if (!task) break; // 收到空任务时退出
                task(); // 执行任务
            }
        });
    }
}

~ThreadPool() {
    // 发送停止信号（例如推送空任务）
    for (size_t i = 0; i < workers_.size(); ++i) {
        queue_.Push(nullptr);
    }
    for (auto& worker : workers_) {
        worker.join();
    }
}

template <typename Func, typename... Args>
auto Submit(Func&& func, Args&&... args) {
    return SubmitTask(queue_, std::forward<Func>(func), std::forward<Args>(args)...);
}

private:
TaskQueue queue_;
std::vector<std::thread> workers_;
};
```

3. **使用示例**

```cpp
#include <iostream>
#include <chrono>

int main() {
    ThreadPool pool(4); // 4 个工作线程
    // 提交任务并获取 future
    auto future1 = pool.Submit([](int a, int b) {
        std::this_thread::sleep_for(std::chrono::seconds(1));
        return a + b;
    }, 2, 3);
    auto future2 = pool.Submit([]() {
        std::this_thread::sleep_for(std::chrono::seconds(2));
        return "Hello, World!";
    });
    // 主线程继续做其他事情...
    // 获取结果（会阻塞直到任务完成）
    std::cout << "Result 1: " << future1.get() << std::endl; // 输出 5
    std::cout << "Result 2: " << future2.get() << std::endl; // 输出 Hello, World!
    return 0;
}
```

4. **关键机制**

| **机制** | **说明** |
| :--- | :--- |
| **类型擦除** | 使用 `std::function<void()>` 包装不同签名的任务，统一存入队列。 |
| **返回值传递** | 通过 `std::future` 实现异步结果的传递。 |
| **线程安全** | 队列的 `Push` 和 `Pop` 操作受互斥锁保护，条件变量实现高效等待。 |


5. **优缺点**

| **优点** | **缺点** |
| :--- | :--- |
| 完全控制线程和任务调度逻辑。 | 需要手动实现线程安全和任务管理。 |
| 可灵活处理不同返回类型的任务。 | 代码复杂度较高，需注意资源释放。 |
| 适合需要定制化线程池的场景。 | C++17/20 中无标准线程池，需自行实现。 |


6. **改进方向**

+ **支持优先级队列**：修改 `TaskQueue` 使用 `std::priority_queue`。
+ **动态调整线程数**：根据任务负载自动增减工作线程。
+ **任务取消机制**：通过 `std::future` 的 `wait_for`/`wait_until` 实现超时或中断。

`std::packaged_task + 队列` 是一种**手动构建异步任务系统的基础方案**，通过将任务封装、队列调度和线程池结合，能够实现灵活的并行执行和结果管理。尽管需要自行处理线程安全和资源管理，但其底层机制是理解现代 C++ 并发库（如线程池实现）的重要基础。

使用 std::launch::deferred 策略

```cpp
#include <iostream>
#include <future>
#include <chrono>
int delayed_sum(int a, int b) {
    std::cout << "Calculating sum...\n";
    std::this_thread::sleep_for(std::chrono::seconds(2)); // 模拟长时间运行的任务
    return a + b;
}
int main() {
    // 使用 launch::deferred 延迟执行任务
    auto fut = std::async(std::launch::deferred, delayed_sum, 3, 7);
    // 模拟主程序继续做其他事情
    std::cout << "Doing other work...\n";
    std::this_thread::sleep_for(std::chrono::seconds(1));
    // 调用 get() 来触发延迟任务的执行，并获取结果
    int result = fut.get();
    std::cout << "Sum is: " << result << "\n";
    return 0;
}
```

让编译器选择启动策略

<u>如果不指定启动策略，那么可以让编译器根据情况选择最合适的策略。</u>这通常是最简单的做法，因为它允许实现优化性能。

```cpp
#include <iostream>
#include <future>
#include <chrono>
int fast_sum(int a, int b) {
    return a + b; // 假设这是一个快速计算
}
int main() {
    // 不指定启动策略，让编译器选择
    auto fut = std::async(fast_sum, 5, 8);
    // 主线程可以继续工作
    std::this_thread::sleep_for(std::chrono::seconds(1));
    // 获取结果
    int result = fut.get();
    std::cout << "Sum is: " << result << "\n";
    return 0;
}
```

`std::async`的默认启动策略——你不显式指定一个策略时它使用的那个——不是上面中任意一个。相反，是求或在一起的。下面的两种调用含义相同

```cpp
auto fut1 = std::async(f);//使用默认启动策略运行f
auto fut2 = std::async(std::launch::async|std::launch::deferred,f);
// 使用 async 或者 deferred 运行 f
```

因此默认策略允许`f`异步或者同步执行。如同[Item35](https://cntransgroup.github.io/EffectiveModernCppChinese/7.TheConcurrencyAPI/Item35.html)中指出，这种灵活性允许`std::async`和标准库的线程管理组件承担线程创建和销毁的责任，避免资源超额，以及平衡负载。这就是使用`std::async`并发编程如此方便的原因。

<h5 id="9f50bb4f">thread_local 变量</h5>
<u>thread_local 变量是一种特殊类型的变量，它为每个线程提供独立的实例</u>。每个线程都有自己版本的 thread_local 变量<u>，并且这些变量在各自的线程中是独立初始化和销毁的</u>。这种机制对于需要在线程之间保持隔离状态的情况非常有用。

不可预测的行为：

如果有一个 thread_local 变量，并且在任务中对其进行了读写操作，那么这个任务究竟会在哪个线程上运行是不确定的。如果任务是在不同的线程上执行的，那么它可能会访问到该线程的thread_local 变量的不同实例，这可能导致意外的行为或错误。

资源泄漏：如果任务从未被执行（因为没有调用 get 或 wait），那么为 thread_local 变量分配的资源可能永远不会被释放，从而导致内存泄漏。

当涉及到 thread_local 变量时，如果任务可以在任意线程上执行，那么跟踪和调试程序行为可能会变得更加复杂，因为同样的代码可能在不同的线程上下文中表现出不同的行为。

考虑下面的代码示例：

```cpp
#include <iostream>
#include <future>
#include <thread>
thread_local int thread_specific_data = 0;
void modify_thread_local() {
    ++thread_specific_data;
    std::cout << "Thread ID: " << std::this_thread::get_id()
        << ", thread_specific_data: " << thread_specific_data << "\n";
}
int main() {
    auto fut = std::async(std::launch::deferred, modify_thread_local);   
    // 在主线程上调用 get，这将使任务在这个线程上同步执行
    fut.get();
    return 0;
}
```

modify_thread_local 函数修改了一个 thread_local 变量 thread_specific_data。由于我们使用了 std::launch::deferred 策略，modify_thread_local 将在调用 fut.get() 的线程（即主线程）上同步执行。因此，thread_specific_data 将被初始化并更新为主线程上的实例。

为了避免这些问题，如果你的任务逻辑中涉及 thread_local 变量，最好显式指定 std::launch::async 策略来确保任务始终在一个新的线程上异步执行，或者重构你的代码以避免对 thread_local 变量的依赖。

<h5 id="3af6efe3">默认启动策略的影响</h5>
由于这种默认行为，以下几点值得注意：

1. 无法预测并发性：你不能确定任务是否会与调用 std::async 的线程并行运行。

(2) 线程不确定性：你也不能确定任务是否会在与调用 get 或 wait 的线程不同的线程上执行。

(3) 执行的不确定性：如果你不在程序的所有路径上调用 get 或 wait，那么任务可能永远不会被执行。

(4) 影响 thread_local 变量：因为不知道哪个线程会执行任务，所以也就不知道哪个线程的 thread_local 变量会被访问。

（5）循环等待问题,如果任务被推迟执行，而在等待它完成时使用了超时机制（如 wait_for 或 wait_until)，则可能导致无限循环，因为任务永远不会变为就绪状态。

解决方案

为了处理上述提到的问题，防止潜在的无限循环，可以检查任务是否被延迟执行。

下面是一个简化的方法来确保不会进入无限循环：

```cpp
auto fut = std::async(f);
// 检查任务是否被延迟执行
if (fut.wait_for(std::chrono::seconds(0)) == std::future_status::deferred) {
    // 如果任务被延迟，则强制同步执行
    fut.get(); // 这里也可以选择使用 wait() 方法
} else {
    // 如果任务不是被延迟的状态，可以安全地使用带有超时的等待循环
    while (fut.wait_for(std::chrono::milliseconds(100)) != std::future_status::ready) {
        // 执行其他工作...
    }
}
```

<u>强制异步执行的工具函数</u>

<u>为了提供一个总是异步执行任务的简便方法，你可以创建一个模板函数 reallyAsync，它总是使用 std::launch::async 启动策略：</u>

```cpp
#include <future>
#include <utility>
// C++11 版本
template<typename F, typename... Ts>
inline std::future<typename std::result_of<F(Ts...)>::type>
reallyAsync(F&& f, Ts&&... params) {
    return std::async(std::launch::async,
        std::forward<F>(f),
        std::forward<Ts>(params)...);
}
// C++14 版本，使用返回类型推导简化声明
template<typename F, typename... Ts>
inline auto reallyAsync(F&& f, Ts&&... params) {
    return std::async(std::launch::async,
        std::forward<F>(f),
        std::forward<Ts>(params)...);
}
```

<u>这个 reallyAsync 函数模板接受一个可调用对象 f 和一组参数 params，然后使用完美转发将它们传递给 std::async，同时保证任务总是以异步方式启动。这样就可以避免默认启动策略带来的不确定性和复杂性。</u>

**请记住：**

+ `<u>std::async</u>`<u>默认启动策略是异步和同步执行兼有的。</u>
+ <u>这个灵活性导致访问</u>`<u>thread_local</u>`<u>的不确定性，隐含了任务可能不会被执行的意思，会影响调用基于超时的</u>`<u>wait</u>`<u>的程序逻辑。</u>
+ <u>如果异步执行任务非常关键，则指定</u>`<u>std::launch::async</u>`<u>。</u>

所以尽量不要使用async进行异步编程。

<h4 id="a3a19730">条款三十七：使std::thread在所有路径最后都不可结合</h4>
<h5 id="5d627c5a">_**前置：线程的状态( joinable 和unjoinable)**_</h5>
在C++中，std::thread 类用于表示和管理线程。有两个主要状态:可结合的(joinable)和不可结合的(unjoinable)。反映了 std::thread 对象是否与一个实际的操作系统线程关联。

_**<u>可结合的(joinable)</u>**_

当一个std::thread对象处于可结合状态时，意味它与一个实际的线程相关联，并且该线程可能正在运行或等待调度。即使线程已经完成其任务，只要没有调用 join() 或 detach()，std::thread 对象仍然被认为是可结合的。包括：

（1）线程启动后，但还未结束。

（2）线程已经结束执行，但还没有被其他线程通过 join() 方法回收资源。

在这种状态下，可以调用 join() 来等待线程完成并回收它的资源，或者调用detach()来让线程独立运行，不再与 std::thread 对象关联。

_**<u>不可结合的 (Unjoinable)</u>**_

如果一个 std::thread 对象是不可结合的，那么它不与任何实际的线程相关联。

（1）默认构造：当使用默认构造函数创建 std::thread 时，它不会与任何线程关联。这种情况下，它不持有任何线程资源，也不代表任何后台操作。

（2）移动之后：如果使用移动语义将一个 std::thread 转移到另一个 std::thread 对象，原对象会变成不可结合的状态。新的对象接管了对底层线程的所有权，而原来的对象则不再与任何线程关联。

已调用 join()：一旦调用了 join() 方法，当前线程就会等待关联的线程完成，并且回收其资源。之后，这个 std::thread 对象就不再与任何线程关联，因此它是不可结合的。

已调用 detach()：当你调用 detach() 方法时，它将分离 std::thread 和底层线程之间的关系，使得线程可以独立运行，而 std::thread 对象不再跟踪这个线程的状态。

join() 和 detach() 是 C++ 标准库中 std::thread 类提供的两种用于管理线程生命周期的方法。它们允许程序员控制一个线程在其启动后如何结束，以及主线程（或其它线程）何时可以继续执行。

**join()**

当调用 join() 方法时，调用此方法的线程（通常称为主线程）将阻塞并等待关联的 std::thread 完成其任务。主线程会暂停执行，直到被 join() 的线程完成。这种方式确保了资源的正确回收，因为主线程会在子线程结束后继续执行，从而避免了资源泄露。在多线程环境中，这有助于同步，保证某些操作在特定线程完成后才发生。当需要确保某个线程的任务在程序继续之前已经完成。希望有序地清理和释放线程使用的资源。

**detach()**

<u>调用 detach() 方法会使 std::thread 对象与其表示的操作系统线程分离，允许该线程独立运行。调用 detach() 后，std::thread 对象不再跟踪这个线程的状态，且不能再次通过这个对象来管理和访问该线程。线程将在后台独立运行，直到它完成自己的任务，而无需等待其他线程（如主线程）进行显式连接。</u>

线程结束后，操作系统会自动回收它的资源。

（1）创建一个线程，并且不关心它的完成时间，也不需要等待它完成。

（2）让线程在后台长时间运行，而不阻塞任何其他线程。

**std::thread 的可结合性**

（1）可结合状态：表示一个std::thread对象关联了一个可能正在运行或等待运行的异步执行线程。

（2）不可结合状态：指没有与任何底层执行线程相关联的std::thread对象，包括默认构造的、已被移动、已调用join或者detach方法后的std::thread对象。

**为什么可结合性重要？**

<u>当一个处于可结合状态的std::thread对象被销毁时（即其析构函数被调用），如果这个线程还没有被join或detach，那么程序将调用std::terminate，导致程序异常终止。</u>

原因：

std::thread的析构函数在对象销毁时如果该线程是可结合状态，则会调用std::terminate终止程序。这是为了防止更糟糕的情况发生：

（1）隐式join:这会导致不确定的行为，特别是当线程不应该继续执行的时候，例如条件不满足时。

（2）隐式detach:这可能导致严重的调试问题，因为被分离的线程可能会访问已经销毁的对象，造成未定义行为。

标准委员会选择了让程序立即终止作为保护机制，迫使程序员显式处理线程的生命周期。

给定的代码尝试并发地执行两个任务：过滤值和检查条件是否满足。

```cpp
constexpr auto tenMillion = 10000000;
bool doWork(std::function<bool(int)>filter,int maxVal = tenMillion)
{
    std::vector<int> goodVals;
    std::thread t([&filter, maxVal, &goodVals]
    {
        for (auto i = 0; i <= maxVal; ++i)
            if (filter(i)) goodVals.push_back(i);
    });
    auto nh = t.native_handle();
    // 设置t的优先级...
    if (conditionsAreSatisfied()) {
        t.join(); // 等待t完成
        performComputation(goodVals);
        return true;
    }
    return false;
}
```

代码中的问题

（1）未处理的可结合线程：如果conditionsAreSatisfied()返回false或抛出异常，则doWork函数会在t仍然处于可结合状态下退出。这会导致t的析构函数被调用，并由于t是可结合的，从而触发std::terminate，造成程序非正常终止。

设置线程优先级的时机：代码试图在线程启动后通过原生句柄设置线程优先级，但这种做法可能不是最佳实践，因为此时线程可能已经开始执行。更好的做法是在创建线程之前配置好所有需要的属性。

**改进建议**

为了解决上述问题，应该确保在函数结束前总是正确地处理线程。可以通过使用范围保护机制（如智能指针或专门设计的类）来保证这一点。此外，在创建线程之前就配置好所有的线程属性也是一个好的实践。例如，可以使用try-catch块来捕获可能发生的异常，并确保即使在异常情况下也能正确地调用join或detach。另一种方法是利用RAII（资源获取即初始化）原则，通过定义一个局部变量来管理线程的生命周期，确保在线程管理对象离开作用域时自动调用join或detach。

ThreadRAII 类解析

```cpp
class ThreadRAII {
public:
enum class DtorAction { join, detach };     // 指定析构时的动作
ThreadRAII(std::thread&& t, DtorAction a)   // 只接受右值引用以移动线程
:action(a), t(std::move(t)) {}
~ThreadRAII()                               // 析构函数中根据action进行相应操作
{                                          
    if (t.joinable()) {
        if (action == DtorAction::join) {
            t.join();
        } else {
            t.detach();
        }
    }
}
std::thread& get() { return t; }            // 提供对内部std::thread的访问
private:
DtorAction action;
std::thread t;
};
```

构造函数：接收一个std::thread右值引用和一个动作枚举，用于确定析构时的行为。

析构函数：检查线程是否可结合，如果是，则根据构造时提供的动作（join或detach）执行相应的操作。

get方法：提供对内部std::thread对象的访问，以便于外部代码可以对其执行其他操作。

使用 ThreadRAII 改进 doWork 函数。

通过引入ThreadRAII，可以确保doWork函数中的线程在任何情况下都能得到正确的处理

```cpp
bool doWork(std::function<bool(int)> filter, int maxVal = tenMillion){
    std::vector<int> goodVals;
    ThreadRAII t(
    std::thread([&filter, maxVal, &goodVals]
    {
        for(auto i=0;i<=maxVal;++i)
            if (filter(i))
                goodVals.push_back(i);
    }),
    ThreadRAII::DtorAction::join // 确保在析构时进行join
    );
    auto nh = t.get().native_handle();
    // 设置线程优先级...
    if (conditionsAreSatisfied()) {
        t.get().join(); // 显式调用join
        performComputation(goodVals);
        return true;
    }
    return false;
}
```

在这个改进后的版本中，无论doWork函数如何退出，ThreadRAII对象的析构函数都会保证join被调用，从而避免了程序非正常终止的风险。此外，通过选择join而不是detach，减少了潜在的未定义行为风险，尽管这可能带来一些性能上的延迟。

item17说明因为`ThreadRAII`声明了一个析构函数，因此不会有编译器生成移动操作，但是没有理由`ThreadRAII`对象不能移动。如果要求编译器生成这些函数，函数的功能也正确，所以显式声明来告诉编译器自动生成也是合适的。

```cpp
classThreadRAII {public:
    enumclassDtorAction { join, detach };         //跟之前一样
ThreadRAII(std::thread&& t, DtorAction a)       //跟之前一样
: action(a), t(std::move(t)) {}
~ThreadRAII()
{
    …    //跟之前一样
}
ThreadRAII(ThreadRAII&&) = default;
//支持移动
ThreadRAII& operator=(ThreadRAII&&) = default;
std::thread& get(){ return t; } 
//跟之前一样private: 
//as before
DtorAction action;
std::thread t;
};
```

<h4 id="9cf390e3">条款三十八：关注不同线程句柄的析构行为</h4>
**之前内容的总结：**

item37中说明了可结合的`std::thread`对应于执行的系统线程。

未延迟（non-deferred）任务的_future_（参见item36）与系统线程有相似的关系。

<u>因此，可以将</u>`<u>std::thread</u>`<u>对象和</u>_<u>future</u>_<u>对象都视作系统线程的</u>**<u>句柄</u>**<u>（</u>_<u>handles</u>_<u>）。</u>

<h6 id="8e6753f2">📌为什么future对象可以视为系统线程的句柄？future对象不是表示获取异步结果的对象吗？</h6>
`std::thread`和_future_在析构时有相当不同的行为。在[Item37](https://cntransgroup.github.io/EffectiveModernCppChinese/7.TheConcurrencyAPI/item37.html)中说明，可结合的`std::thread`析构会终止你的程序，因为两个其他的替代选择——隐式`join`或者隐式`detach`都是更加糟糕的。

在C++中，<font style="background-color:#fbf5b3;">std::future和std::promise是用于处理异步操作结果的模板类</font>。它们允许一个线程（调用者）等待另一个线程（被调用者）完成计算并获取其结果。这种机制背后的实现通过共享状态（shared state），是一个独立的对象，既不是std::promise也不是std::future，std::promise和std::future都引用了同一个共享状态,共享状态通常是基于堆的对象，但是标准并未指定其类型、接口和实现。标准库的作者可以通过任何他们喜欢的方式来实现共享状态。 当创建一个std::promise<T>对象时，可以通过调用它的get_future()成员函数来获得一个关联的std::future<T>对象。

std::future<T>对象可以用来等待或检查std::promise<T>是否已经设置了值，并最终获取那个值。

共享状态包含了以下信息：

（1）**结果存储：**实际的结果数据，即被调用者要传递给调用者的值或异常。

（2）**同步原语**：用于管理对结果的访问的锁或其他同步机制。

（3）**状态标志：**指示结果是否已经被设置、是否有错误发生等。

当被调用者完成了它的任务，它会调用std::promise的set_value()或者如果发生了异常则调用set_exception()方法，将结果写入共享状态。此时，任何持有与该std::promise相关联的std::future的线程都可以通过调用get()方法来检索结果，将阻塞直到结果可用。

（ 目前觉得类似管道 )

由于std::promise通常是局部变量并且会在被调用者结束时销毁，std::future可能被复制成多个std::shared_future实例，因此结果不能直接存储在这些对象内部。相反，它们都指向堆上的共享状态，这个状态负责保存结果直到所有相关的std::future和std::shared_future都被销毁为止。这样的设计使得即使原始的std::promise和第一个std::future被销毁后，其他std::shared_future实例仍然能够访问结果，只要还有至少一个std::future或std::shared_future存在，共享状态就会保持有效，确保只可移动类型的结果可以被正确处理，因为移动操作不会影响共享状态中的实际数据。

_future_是通信信道的一端，被调用者通过该信道将结果发送给调用者。被调用者（通常是异步执行）将计算结果写入通信信道中（通常通过`std::promise`对象），调用者使用_future_读取结果。

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742811189211-a53c455b-5b2d-47e8-9e43-b23045a1060b.png)

可以想象调用者，被调用者，共享状态之间关系如下图，虚线还是表示信息流方向：

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742811189561-7085dbda-f7ad-4074-b729-022eb1a29f8c.png)

共享状态非常重要，因为_future_的析构函数，取决于与_future_关联的共享状态。

在C++中，std::future 与 std::shared_future 类用于获取由异步操作产生的结果或异常。std::future 对象是独占的，而 std::shared_future 可以被多个对象共享。这些 future 对象关联着一个共享状态，这个状态包含未来将要得到的结果或异常。

**（1）未延迟任务（即立即执行的任务）**

当使用 std::async 启动一个任务时，默认情况下（如果没有指定 std::launch::deferred），任务会立即在一个独立线程中开始执行。返回的 std::future 对象引用了该任务的共享状态。如果这是最后一个引用该共享状态的 std::future 实例（例如，当它离开作用域并被销毁时），那么它的析构函数将会阻塞当前线程直到异步任务完成。这是因为最后一个 std::future 的销毁表示不再有其他地方等待这个异步操作的结果了，所以 C++ 标准库保证任务已经完成，并且结果或异常已经被处理。

**（2）其他所有 std::future 的析构**

如果有多个 std::future 或 std::shared_future 对象共享同一个状态，它们的销毁不会导致阻塞，除非它是最后一个引用该共享状态的对象。对于立即执行的任务，底层线程的行为类似于调用了 detach ，即使没有 std::future 对象再监视它，它也会继续运行。而对于延迟任务（通过 std::launch::deferred 指定），如果这是最后一个引用共享状态的 std::future 并且它被销毁了，那么该延迟任务永远不会被执行，因为它依赖于至少存在一个 std::future 来触发它的执行。

_<u>上面规则的解释：</u>_

真正要处理的是一个简单的“正常”行为以及一个单独的例外。正常行为是_future_析构函数销毁_future。_就是这样。意味着不`join`也不`detach`，也不运行什么，只销毁_future_的数据成员（当然，还做了另一件事，就是递减了共享状态中的引用计数，这个共享状态是由引用它的_future_和被调用者的`std::promise`共同控制的。引用计数见item19）

正常行为的例外情况仅在某个`future`同时满足下列所有情况下才会出现：

+ **它关联到由于调用**`**std::async**`**而创建出的共享状态**。
+ **任务的启动策略是**`**std::launch::async**`（参见[Item36](https://cntransgroup.github.io/EffectiveModernCppChinese/7.TheConcurrencyAPI/item36.html)），原因是运行时系统选择了该策略，或者在对`std::async`的调用中指定了该策略。
+ **这个**_**future**_**是关联共享状态的最后一个**_**future**_。

对于`std::future`，情况总是如此，对于`std::shared_future`，如果还有其他的`std::shared_future`，与要被销毁的_future_引用相同的共享状态，则要被销毁的_future_遵循正常行为（简单地销毁它的数据成员)。

只有当上面的三个条件都满足时，_future_的析构函数才会表现“异常”行为，就是在异步任务执行完之前阻塞住。实际上，这相当于对由于运行`std::async`创建出任务的线程隐式`join`。

_**<u>正常行为</u>**_

对于大多数情况下的 std::future 或 std::shared_future 的析构，行为是相当直接和简单的：

（1）销毁数据成员：当一个 std::future 被销毁时，会简单地清理自己的资源，包括它的内部数据成员。

（2）递减引用计数：更重要的是，它会对共享状态中的引用计数进行递减操作。这个引用计数用于追踪有多少个 std::future 或 std::shared_future 正在引用同一个共享状态。这是管理资源生命周期的一种方式，确保只有当没有其他 future 对象引用该状态时，相关资源才能被安全地释放。

_**<u>异常行为（特殊条件）</u>**_

异常行为仅在特定条件下发生，即当以下所有条件同时满足时：

（1）关联到由 std::async 创建的共享状态：意味该 future 是通过调用 std::async 来创建的，而 std::async 通常用于启动异步任务。

（2）任务使用 std::launch::async 策略：这里指的是异步任务确实是在一个独立的线程中执行的。如果策略是 std::launch::deferred，那么任务会在第一次访问结果时才被执行，并且不会立即开始。

（3）最后一个引用此共享状态的 future：如果这是一个与共享状态关联的最后一个 std::future 或 std::shared_future，则意味着不再有其他对象等待这个异步操作的结果。

异常行为的表现

当上述三个条件都满足时，std::future 的析构函数将表现出“异常”行为。它不会简单地销毁自己并递减引用计数，而是会阻塞当前线程，直到关联的异步任务完成。这实际上相当于对启动该任务的线程进行了隐式的 join 操作。这种设计确保了即使没有任何显式代码在等待结果，也能保证异步任务正确完成，防止潜在的资源泄漏或未定义行为。

_**<u>关于 std::future 析构时的行为不确定性</u>**_

问题核心：无法通过 std::future 的 API 确定它是否引用了由 std::async 创建的共享状态，因此不能提前知道析构函数是否会因为等待异步任务完成而阻塞。

```cpp
std::vector<std::future<void>> futs; 
// 可能在析构时阻塞，如果其中至少一个 future 引用了 std::async 启动的任务。
```

解决方案：当使用 std::packaged_task 创建共享状态时，可以明确知道该 std::future 不会因 std::async 而产生阻塞行为，因为它关联的是 std::packaged_task 创建的共享状态。

```cpp
int calcValue(); //函数定义
//包裹函数以异步运行
std::packaged_task<int()> pt(calcValue);
auto fut = pt.get_future(); //获取 future
```

std::thread 和 std::packaged_task 的结合使用

线程管理：当将 std::packaged_task 传递给 std::thread 执行时，需要确保正确地处理线程的生命周期（join 或 detach），这通常会在代码中显式地进行，从而避免 std::future 的析构函数阻塞。

```cpp
{
    std::packaged_task<int()> pt(calcValue); 
    auto fut = pt.get_future();
    // 将 task 移交给 thread 执行
    std::thread t(std::move(pt));
    // 此处可以对 t 进行 join 或 detach 操作
    // 如果不做任何操作，t 在作用域结束时会导致程序终止
    // 示例：
    // t.join(); // 显式 join
    // 或者
    // t.detach(); // 显式 detach
} // 结束代码块
```

当有一个与 std::packaged_task 创建的共享状态相关联的 std::future 时，不需要担心其析构函数会执行异常行为，因为通常会在代码中做出关于线程生命周期的决定（例如，调用 join 或 detach）。此外，如果要使用 std::async 来运行任务，就没有必要再额外使用 std::packaged_task，因为 std::async 已经涵盖了 std::packaged_task 的所有功能。

**请记住：**

+ _future_的正常析构行为就是销毁_future_本身的数据成员。
+ 引用了共享状态，使用`std::async`启动的未延迟任务建立的最后一个_future_的析构函数会阻塞住，直到任务完成。

<h4 id="7fa7cc4d">条款三十九：对于一次性事件通信考虑使用void的futures</h4>
<h5 id="6434c40d">前置：C++中条件变量的使用</h5>
std::condition_variable 的 wait 函数有多个重载版本。

基本形式，等待直到被通知：

```cpp
template< class Predicate >
void wait( std::unique_lock<std::mutex>& lock, Predicate pred );
```

等待直到被通知或直到指定的时间点：

```cpp
template< class Clock, class Duration >
cv_status wait_until( unique_lock<mutex>& lock,
const chrono::time_point<Clock, Duration>& abs_time );
template< class Clock, class Duration, class Predicate >
bool wait_until( unique_lock<mutex>& lock,
const chrono::time_point<Clock, Duration>& abs_time,
Predicate pred );
```

等待直到被通知或直到经过指定的时间间隔：

```cpp
template< class Rep, class Period >
cv_status wait_for( unique_lock<mutex>& lock,
const chrono::duration<Rep, Period>& rel_time );
template< class Rep, class Period, class Predicate >
bool wait_for( unique_lock<mutex>& lock,
const chrono::duration<Rep, Period>& rel_time,
Predicate pred );
```

最常用的两个是带有谓词（predicate）的 wait 和 wait_for/wait_until，它们能够处理虚假唤醒的问题，并且可以确保线程只会在条件真正满足时才继续执行。

对于带有谓词的 wait 方法，它的模板参数 Predicate 应该是一个可调用对象（比如函数、lambda 表达式、函数对象等），它接受零个参数并返回一个可转换为 bool 的值。谓词会被 wait 函数反复评估，只有当谓词返回 true 时，wait 才会终止，线程才会继续执行。

例如，使用带有谓词的 wait 方法：

```cpp
std::condition_variable cv;
std::mutex mtx;
bool ready = false;
//反应任务代码
{
    std::unique_lock<std::mutex> lk(mtx);
    cv.wait(lk, []{ return ready; }); // 等待直到ready为true
    // 对事件作出反应...
}
```

cv.wait(lk, []{ return ready; }); 会一直阻塞当前线程，直到 ready 变量变为 true 或者发生虚假唤醒。如果发生虚假唤醒，wait 会再次检查谓词，确保条件真正满足之后才继续执行。

_**<u>互斥锁+条件变量的不足：</u>**_

**互斥锁的使用**

条件变量总是与互斥锁一起使用，这是因为条件变量本身不提供同步保护。当一个线程等待一个条件变量时，它会释放关联的互斥锁，并进入等待状态。当条件变量被通知时，线程会被唤醒并重新获取互斥锁，然后继续执行。

即使检测任务和反应任务之间没有共享数据的竞争，我们仍然需要互斥锁来防止所谓的“竞态条件”，即检测任务可能在反应任务检查条件之前改变了条件，但又在反应任务开始等待之前再次改变条件。这种情况下，反应任务可能会错过通知。

**漏失的通知**

如果检测任务在反应任务调用 wait 之前调用了 notify_one() 或 notify_all()，那么这个通知就会被漏失，因为此时还没有任何线程在等待条件变量。为了避免这种情况，通常的做法是将条件变量与一个条件谓词（predicate）结合使用。条件谓词是一个布尔表达式，用来表示线程应该等待的条件是否满足。

**虚假唤醒**

虚假唤醒是指线程在没有收到实际通知的情况下从 wait 中醒来。

_**<u>简单的轮询方法</u>**_

使用共享的布尔型 flag 进行线程间通信的方法确实避免了条件变量设计中的一些复杂性，如互斥锁的需要和虚假唤醒的问题。这种方法有其自身的缺点，与性能和资源利用相关。

简单的轮询方法

在这种方法中，一个线程（检测线程）通过设置一个 std::atomic<bool> 类型的标志位来通知另一个线程（反应线程）。反应线程则通过不断地检查这个标志位来等待事件的发生：

```cpp
std::atomic<bool> flag(false);          // 共享的flag；使用原子操作保证线程安全
// 检测线程代码
{
    // 检测某个事件...
    flag = true;                        // 事件发生后将flag置位
}
// 反应线程代码
while (!flag.load(std::memory_order_acquire)); // 等待事件
// 对事件作出反应...
```

这种方法的优点是简单直接，不需要复杂的同步机制。但是，它存在明显的缺陷：

（1）CPU 资源浪费：反应线程在while (!flag)循环中不断检查 flag 的状态，这被称为“忙等”或“自旋”。即使没有实际的工作要做，该线程也会持续占用 CPU 时间片，从而导致 CPU 资源的浪费，并可能影响其他任务的执行。

（2）能量消耗：由于 CPU 核心不会进入节能模式，这种轮询方式会增加设备的能量消耗，尤其是在移动设备或服务器环境中，这对能效是一个重要的考虑因素。

_**<u>条件变量+条件谓词</u>**_

为了处理这个问题，应该始终在一个循环中使用条件变量的 wait 函数，并且传递一个条件谓词给它。这样可以确保线程只有在条件真正满足时才会继续执行，而不会因为虚假唤醒或漏失的通知而错误地继续执行。

正确的代码结构

```cpp
std::condition_variable cv;
std::mutex m;
bool ready = false; // 条件谓词：代表事件是否已发生
// 检测任务代码
{
    std::lock_guard<std::mutex> lk(m); // 简单锁定，自动解锁
    // 检测事件...
    ready = true; // 改变条件谓词的状态
}
cv.notify_one(); // 或者 notify_all() 如果有多个反应任务
// 反应任务代码
{
    std::unique_lock<std::mutex> lk(m);
    cv.wait(lk, []{ return ready; }); // 等待直到ready为true
    // 对事件作出反应...
}
```

ready 是条件谓词，用来表示事件是否已经发生。

cv.wait(lk, []{ return ready; }); 这一行保证了反应任务只会在线程确实准备好了（即 ready == true）之后才继续执行，从而解决了虚假唤醒的问题。同时，由于检测任务在改变 ready 的状态之前锁定了互斥锁，所以也避免了漏失的通知。

避免虚假唤醒：由于 wait 函数使用了谓词（lambda），即使出现虚假唤醒，反应任务也会检查 flag 是否为 true，只有当 flag 确实为 true 时才会继续执行。

防止漏失通知：无论检测任务是在反应任务进入 wait 之前还是之后调用 notify_one，只要 flag 在 wait 返回时为 true，反应任务就会正确地作出响应。

无需轮询：反应任务真正地阻塞在 wait 上，不会占用 CPU 资源。

虽然这种方法有效，但确实有些不那么优雅的地方，比如检测任务需要分别设置 flag 和调用 notify_one，这可能看起来有点冗余。然而，这种设计实际上是合理的，因为它保证了顺序性和原子性。

（1）顺序性：先设置 flag 再通知，确保反应任务看到的是最新的状态。

（2）原子性：虽然 flag 的设置和通知不是原子操作，但是由于 flag 是在互斥锁保护下修改的，所以对于其他线程来说，这两个操作是原子的。

_**<u>promise和future替代线程间通信</u>**_

简化代码：相比于条件变量和标志位的方法，使用 std::promise 和 std::future 可以显著简化代码。检测任务只需要设置 std::promise<void> 来通知事件的发生，而反应任务只需在对应的 std::future<void> 上等待。

```cpp
#include <future>
#include <iostream>
// 检测任务代码
void detecting_task(std::promise<void>& p) {
    // 模拟检测某个事件...
    std::cout << "事件被检测到\n";
    p.set_value(); // 通知反应任务
}
// 反应任务代码
void reacting_task(std::future<void> f) {
    std::cout << "准备作出反应...\n";
    f.wait(); // 等待事件
    std::cout << "对事件作出反应\n";
}
```

避免虚假唤醒：std::future::wait() 只会在 std::promise 被设置后返回，因此不存在虚假唤醒的问题。

真正的阻塞：与轮询方法不同，std::future::wait() 会真正阻塞反应任务，不会占用 CPU 资源，直到 std::promise 被设置。

不需要互斥锁：由于 std::promise 和 std::future 内部处理了同步问题，所以不再需要显式的互斥锁来保护共享数据。

```cpp
int main() {
    std::promise<void> p;
    std::future<void> f = p.get_future();
    // 启动反应任务（可以是另一个线程）
    std::thread react(reacting_task, std::move(f));
    // 模拟一些工作...
    std::this_thread::sleep_for(std::chrono::seconds(1));
    // 启动检测任务（可以在当前线程中执行）
    detecting_task(p);
    react.join();
}
```

std::promise 和 std::future 的缺点

1. 动态分配开销：std::promise 和 std::future 之间的共享状态是动态分配的，每次创建它们时都会产生堆内存分配和释放的开销。
2. 一次性通信：std::promise 一旦设置就不能再次设置，使得它只能用于一次性的通信。如果需要多次通信，则必须为每次通信创建新的 std::promise 和 std::future 对象，增加了管理的复杂度。
3. 不适合频繁通信：由于上述原因，对于需要频繁通信的场景，基于 std::promise 和 std::future 的设计可能不是最佳选择。

尽管 std::promise 和 std::future 通常用于异步调用模式，但也可以用于任何需要从程序的一个地方传递信息到另一个地方的情况。例如，创建一个挂起的系统线程，以便在实际使用之前对其进行配置（如设置优先级或核心亲和性）。这种情况下，std::promise 和 std::future 提供了一种简单的方式来进行初始化后的线程启动信号传递。

虽然 std::promise 和 std::future 提供了一种简洁且有效的线程间通信方式，并解决了条件变量和标志位设计中的某些问题，但它们也有自己的局限性和适用范围。选择哪种方法取决于具体的应用场景和需求。对于需要频繁通信或复用通信通道的场景，条件变量可能是更好的选择；而对于简单的、一次性通信，std::promise 和 std::future 则提供了更简洁的解决方案。

_**<u>std::promise<void> 和 std::future<void> 来挂起和取消挂起线程的方案</u>**_

单个线程挂起与取消挂起

对于仅需要对某线程进行一次挂起（即在线程创建后但在执行线程函数前），可以使用 std::promise<void> 和 std::future<void> 来实现：

```cpp
#include <iostream>
#include <thread>
#include <future>
void react() {
    std::cout << "Reacting to the event...\n";
}
void detect() {
    std::promise<void> p;
    std::thread t([&p]() {
        p.get_future().wait(); // 挂起t直到future被置位
        react();
    });
    // 这里，t在调用react前挂起
    p.set_value(); // 解除挂起t（因此调用react）
    // 做其他工作...
    t.join(); // 使t不可结合
}
```

使用 RAII 管理线程生命周期

为了确保所有离开 detect 函数的路径中线程都被正确地 join，可以使用 RAII 类来管理线程的生命周期。然而，这里有一个潜在的问题：如果在 p.set_value() 之前发生异常，lambda 中的 wait 将永远不会返回，导致线程不会结束，而 RAII 对象会在析构时尝试 join，从而造成死锁。

```cpp
class ThreadRAII {
public:
ThreadRAII(std::thread&& t, DtorAction action) : th(std::move(t)), action_(action) {}
~ThreadRAII() {
    if (th.joinable()) {
        if (action_ == DtorAction::join)
            th.join();
        else if (action_ == DtorAction::detach)
            th.detach();
    }
}
private:
std::thread th;
DtorAction action_;
};
enum class DtorAction { join, detach };
void detect_with_raii() {
    std::promise<void> p;
    ThreadRAII tr(std::thread([sf = p.get_future().share()]{
        sf.wait();
        react();
    }), ThreadRAII::DtorAction::join);
    // 如果这里抛出异常，p.set_value() 不会被调用，导致tr中的线程永远等待
    p.set_value(); // 解除挂起tr中的线程
}
```

支持多个反应线程

为了解决上述异常处理的问题并支持多个反应线程，可以使用 std::shared_future<void>。每个反应线程都拥有一个指向共享状态的 std::shared_future<void> 的副本，这允许它们共同等待同一个事件的发生。

```cpp
#include <iostream>
#include <thread>
#include <future>
#include <vector>
void react() {
    std::cout << "Reacting to the event...\n";
}
void detect_multiple_reactors(size_t threadsToRun) {
    std::promise<void> p;
    auto sf = p.get_future().share(); // 创建可共享的future
    std::vector<std::thread> vt;
    for (size_t i = 0; i < threadsToRun; ++i) {
        vt.emplace_back([sf]() { // 按值捕获shared_future
            sf.wait(); // 在局部副本上wait
            react();
        });
    }
    // 如果这里的“...”区域抛出异常，所有的线程将永久挂起！
    try {
        // 模拟一些工作...
        p.set_value(); // 所有线程解除挂起
    } catch (...) {
        // 异常处理逻辑
        throw; // 或者其他处理方式
    }
    for (auto& t : vt) {
        t.join(); // 确保所有线程完成
    }
}
int main() {
    detect_multiple_reactors(5); // 示例：启动5个反应线程
    return 0;
}
```

单次通信：std::promise<void> 和 std::future<void> 提供了一种简单的方法来进行一次性的线程间通信。

异常安全：通过 std::shared_future<void> 和 RAII 类，可以在异常情况下安全地管理多个线程的生命周期。

多线程支持：std::shared_future<void> 允许多个线程共同等待同一事件，简化了多线程同步的实现。

**请记住：**

+ 对于简单的事件通信，基于条件变量的设计需要一个多余的互斥锁，对检测和反应任务的相对进度有约束，并且需要反应任务来验证事件是否已发生。
+ 基于flag的设计避免的上一条的问题，但是是基于轮询，而不是阻塞。
+ 条件变量和flag可以组合使用，但是产生的通信机制很不自然。
+ 使用`std::promise`和_future_的方案避开了这些问题，但是这个方法使用了堆内存存储共享状态，同时有只能使用一次通信的限制。

<h4 id="f4626c61">条款四十：对于并发使用std::atomic，对于特殊内存使用volatile</h4>
其他编程语言中(Java和C#)，`volatile`是有并发含义的，即使在C++中，有些编译器在实现时也将并发的某种含义加入到了`volatile`关键字中。因此讨论`volatile`关键字的含义以消除异议。

_**<u>atomic</u>**_

`std::atomic`模板这种模板的实例化（比如，`std::atomic<int>`，`std::atomic<bool>`，`std::atomic<Widget*>`等）提供了一种在其他线程看来操作是原子性的的保证。一旦`std::atomic`对象被构建，在其上的操作表现得像操作是在互斥锁保护的关键区内，但是通常这些操作是使用特定的机器指令实现，这比锁的实现更高效。

分析如下使用`std::atmoic`的代码：

```cpp
std::atomic<int> ai(0);       //初始化ai为0
ai = 10;                        //原子性地设置ai为10std::cout << ai;             //原子性地读取ai的值
++ai;                           //原子性地递增ai到11
--ai;                           //原子性地递减ai到10
```

在这些语句执行过程中，其他线程读取`ai`，只能读取到0，10，11三个值其中一个。

这个例子中有两点值得注意。首先，在“`std::cout << ai;`”中，`ai`是一个`std::atomic`的事实只保证了对`ai`的读取是原子的。没有保证整个语句的执行是原子的。在读取`ai`的时刻与调用`operator<<`将值写入到标准输出之间，另一个线程可能会修改`ai`的值。这对于这个语句没有影响，因为`int`的`operator<<`是使用`int`型的传值形参来输出（所以输出的值就是读取到的`ai`的值），但是重要的是要理解原子性的范围只保证了读取`ai`是原子性的。

`ai`的递增递减都是读-改-写(read-modify-writeRMW）操作，它们整体作为原子执行。`std::atomic`类型的最优的特性之一：一旦`std::atomic`对象被构建，所有成员函数，包括RMW操作，从其他线程来看都是原子性的。

相反，使用`volatile`在多线程中实际上不保证任何事情：

```cpp
volatile int vi(0);          //初始化vi为0
vi = 10;                     //设置vi为10 std::cout << vi; //读vi的值
++vi;                        //递增vi到11
--vi;                        //递减vi到10
```

代码的执行过程中，如果其他线程读取`vi`，可能读到任何值，比如-12，68，409072，任何值。这份代码有未定义行为，因为这里的语句修改`vi`，所以如果同时其他线程读取`vi`，同时存在多个readers和writers读取没有`std::atomic`或者互斥锁保护的内存，这就是数据竞争的定义。

举一个关于在多线程程序中`std::atomic`和`volatile`表现不同的具体例子，考虑这样一个简单的计数器，通过多线程递增。把它们初始化为0：

```cpp
std::atomic<int> ac(0);         
//“原子性的计数器 
volatile int vc(0);             
//“volatile计数器”
```

然后我们在两个同时运行的线程中对两个计数器递增：

```cpp
/*----- Thread 1 ----- *//*------- Thread 2 ------- */
++ac;                              ++ac;
++vc;                              ++vc;
```

当两个线程执行结束时，`ac`的值（即`std::atomic`的值）肯定是2，因为每个自增操作都是不可分割的（原子性的）。另一方面，`vc`的值，不一定是2，因为自增不是原子性的。每个自增操作包括了读取`vc`的值，增加读取的值，然后将结果写回到`vc`。这三个操作对于`volatile`对象不能保证原子执行，所有可能是下面的交叉执行顺序：

1. Thread1读取`vc`的值，是0。
2. Thread2读取`vc`的值，还是0。
3. Thread1将读到的0加1，然后写回到`vc`。
4. Thread2将读到的0加1，然后写回到`vc`。

`vc`的最后结果是1，即使看起来自增了两次。

不仅只有这一种可能的结果，`vc`的最终结果是不可预测的，因为`vc`会发生数据竞争，对于数据竞争造成未定义行为，标准规定表示编译器生成的代码可能是任何逻辑。

RMW操作不是仅有的`std::atomic`在并发中有效而`volatile`无效的例子。假定一个任务计算第二个任务需要的一个重要的值。当第一个任务完成计算，必须传递给第二个任务。[Item39](https://cntransgroup.github.io/EffectiveModernCppChinese/7.TheConcurrencyAPI/item39.html)表明一种使用`std::atomic<bool>`的方法来使第一个任务通知第二个任务计算完成。计算值的任务的代码如下：

```cpp
std::atomic<bool> valVailable(false); 
auto imptValue = computeImportantValue();   //计算值
valAvailable = true;                        //告诉另一个任务，值可用了
```

人类读这份代码，能看到在`valAvailable`赋值之前对`imptValue`赋值很关键，但是所有编译器看到的是给相互独立的变量的一对赋值操作。通常来说，编译器会被允许重排这对没有关联的操作。这意味着，给定如下顺序的赋值操作（其中`a`，`b`，`x`，`y`都是互相独立的变量），

```cpp
a = b;
x = y;
```

编译器可能重排为如下顺序：

```cpp
x = y;
a = b;
```

即使编译器没有重排顺序，底层硬件也可能重排（或者可能使它看起来运行在其他核心上），因为有时这样代码执行更快。`std::atomic`会限制这种重排序，并且这样的限制之一是，在源代码中，对`std::atomic`变量写之前不会有任何操作。（这只在`std::atomic`使用**顺序一致性**时成立，对于使用在本书中展示的语法的`std::atomic`对象，这也是默认的和唯一的一致性模型。C++11也支持带有更灵活的代码重排规则的一致性模型。这样的**弱(**_weak)_模型使构建一些软件在某些硬件构架上运行的更快成为可能，但是使用这样的模型产生的软件**更加**难改正、理解、维护。在使用松散原子性的代码中微小的错误很常见，即使专家也会出错，所以应当尽可能坚持顺序一致性。）

```cpp
auto imptValue = computeImportantValue(); //计算值
valAvailable = true;                      //告诉另一个任务，值可用了
```

编译器不仅要保证`imptValue`和`valAvailable`的赋值顺序，还要保证生成的硬件代码不会改变这个顺序。结果就是，将`valAvailable`声明为`std::atomic`确保了必要的顺序——其他线程看到的是`imptValue`值的改变不会晚于`valAvailable`。将`valAvailable`声明为`volatile`不能保证上述顺序：

```cpp
volatile bool valVailable(false); 
auto imptValue = computeImportantValue();
val Available = true;                        
//其他线程可能看到这个赋值操作早于imptValue的赋值操作
```

这份代码编译器可能将`imptValue`和`valAvailable`赋值顺序对调，如果它们没这么做，可能不能生成机器代码，来阻止底部硬件在其他核心上的代码看到`valAvailable`更改在`imptValue`之前。

不保证操作的原子性以及对代码重排顺序没有足够限制，解释了为什么`volatile`在多线程编程中没用，但是没有解释它应该用在哪。它是用来告诉编译器，它们处理的内存有不正常的表现。“正常”内存应该有这个特性，在写入值之后，这个值会一直保持直到被覆盖。假设有这样一个正常的`int`

```cpp
int x;
```

编译器看到下列的操作序列：

```cpp
auto y = x;                             //读x
y = x;                                  //再次读x
```

编译器可通过忽略对`y`的一次赋值来优化代码，因为有了`y`初始化，赋值是冗余的。

正常内存还有一个特征，就是如果你写入内存没有读取，再次写入，第一次写就可以被忽略，因为值没被用过。给出下面的代码：

```cpp
x = 10;                                 //写x
x = 20;                                 //再次写x
```

编译器可以忽略第一次写入。这意味着如果写在一起：

```cpp
auto y = x;                             //读x
y = x;                                  //再次读x
x = 10;                                 //写x
x = 20;                                 //再次写x
```

编译器生成的代码是这样的：

```cpp
auto y = x;                             //读x
x = 20;                                 //写x
```

可能你会想谁会写这种重复读写的代码（技术上称为**冗余访问**（_redundant loads_）和**无用存储**（_dead stores_）），答案是开发者不会直接写——至少我们不希望开发者这样写。但是在编译器拿到看起来合理的代码，执行了模板实例化，内联和一系列重排序优化之后，结果会出现冗余访问和无用存储，所以编译器需要摆脱这样的情况并不少见。

这种优化仅仅在内存表现正常时有效，特殊的内存不行。最常见的“特殊”内存是用来做内存映射I/O的内存。这种内存实际上是与外围设备(比如传感器，显示器，打印机，网络端口)通信，而不是读写通常的内存(比如RAM)。这种情况下，再次考虑这看起来冗余的代码：

```cpp
auto y = x;                             //读x
y = x;                                  //再次读x
```

如果`x`的值是一个温度传感器上报的，第二次对于`x`的读取就不是多余的，因为温度可能在第一次和第二次读取之间变化。看起来冗余的写操作也类似。比如在这段代码中：

```cpp
x = 10;                                 //写x
x = 20;                                 //再次写x
```

如果`x`与无线电发射器的控制端口关联，则代码是给无线电发指令，10和20意味着不同的指令。优化掉第一条赋值会改变发送到无线电的指令流。

`volatile`是告诉编译器我们正在处理特殊内存。意味告诉编译器“不要对这块内存执行任何优化”。所以如果`x`对应于特殊内存，应该声明为`volatile`：

```cpp
volatile int x;
```

考虑对我们的原始代码序列有何影响：

```cpp
auto y = x;                             //读x
y = x;                                  //再次读x（不会被优化掉）
x = 10;                                 //写x（不会被优化掉）
x = 20;                                 //再次写x
```

如果`x`是内存映射的(或者已经映射到跨进程共享的内存位置等)，这正是我们想要的。

在最后一段代码中，`y`是什么类型：`int`还是`volatile int`？（`y`的类型使用`auto`类型推导，所以使用[Item2](https://cntransgroup.github.io/EffectiveModernCppChinese/1.DeducingTypes/item2.html)中的规则。规则上说非引用非指针类型的声明（就是`y`的情况），`const`和`volatile`限定符被拿掉。`y`的类型因此仅仅是`int`。这意味着对`y`的冗余读取和写入可以被消除。在例子中，编译器必须执行对`y`的初始化和赋值两个语句，因为`x`是`volatile`的，所以第二次对`x`的读取可能会产生一个与第一次不同的值）

在处理特殊内存时，必须保留看似冗余访问和无用存储的事实，顺便说明了为什么`std::atomic`不适合这种场景。编译器被允许消除对`std::atomic`的冗余操作。代码的编写方式与`volatile`那些不那么相同，但是如果我们暂时忽略它，只关注编译器执行的操作，则概念上可以说，编译器看到这个，

Volatile

volatile关键字用于告诉编译器某个变量可能会被外部因素（如硬件中断或并发线程）改变，因此每次访问该变量时都必须实际读取或写入内存，而不是从寄存器或缓存中获取值。这防止了编译器优化掉看似冗余的读写操作，确保了对特殊内存位置（例如I/O端口、硬件寄存器等）的正确处理。

```cpp
volatile int x;
auto y = x; // 读x
y = x;      // 再次读x，不会被优化掉
x = 10;     // 写x，不会被优化掉
x = 20;     // 再次写x
```

由于auto推导出的y是一个普通的int类型，它不受volatile的影响，因此对y的操作可以被优化。但是，因为x是volatile的，所以所有对x的访问都不能被优化。

然而，std::atomic并不阻止编译器进行某些类型的优化，比如合并连续的读操作。此外，std::atomic不允许拷贝构造或赋值，以防止无法实现真正的原子性操作。因此，要与普通变量交互，需要使用load()和store()方法：

```cpp
std::atomic<int> x;
int y = x.load();   // 读x
x.store(10);        // 写x
```

结合使用 volatile 和 std::atomic

对于既需要原子性又需要防止编译器优化的情况（例如访问共享内存中的硬件寄存器），可以将两者结合起来使用：

```cpp
volatile std::atomic<int> vai; // 对vai的操作是原子性的，且不能被优化
```

这样定义的变量既可以保证线程安全，也可以确保所有访问都不会被编译器优化掉。

因此情况很明显：

+ `std::atomic`用在并发编程中，对访问特殊内存没用。
+ `volatile`用于访问特殊内存，对并发编程没用。

因为`std::atomic`和`volatile`用于不同的目的，所以可以结合起来使用：

```cpp
volatilestd::atomic<int> vai;          //对vai的操作是原子性的，且不能被优化掉
```

**请记住：**

+ `std::atomic`用于在不使用互斥锁情况下，来使变量被多个线程访问的情况。是用来编写并发程序的一个工具。
+ `volatile`用在读取和写入不应被优化掉的内存上。是用来处理特殊内存的一个工具。

<h1 id="898fdffd">第8章 微调</h1>
<h4 id="1c462c3b">条款四十一：对于移动成本低且总是被拷贝的可拷贝形参，考虑按值传递</h4>
有些函数的形参是可拷贝的。比如说，`addName`成员函数可以拷贝自己的形参到一个私有容器。为了提高效率，应该拷贝左值，移动右值。

讨论C++中如何优化函数参数的传递方式，特别是当涉及到拷贝和移动语义时。对于`addName`成员函数，作者提出了三种不同的方法来处理左值（lvalue）和右值（rvalue），并分析了各自的优缺点。

**第一种方法：重载**

通过为左值引用和右值引用分别提供两个版本的`addName`函数：

```cpp
class Widget {
public:
void addName(const std::string& newName) // 接受左值；拷贝它
{ names.push_back(newName); }
void addName(std::string&& newName)      // 接受右值；移动它
{ names.push_back(std::move(newName)); }
private:
std::vector<std::string> names;
};
```

这种方法的优点是清晰直接，但对于维护代码来说，意味着要管理两个独立的函数实现。此外，如果这两个函数没有被内联，可能会增加目标代码的大小。

**第二种方法：通用引用与模板**

使用模板和完美转发来处理左值和右值：

```cpp
class Widget {
public:
template<typename T>
void addName(T&& newName) // 接受左值和右值；拷贝左值，移动右值
{ names.push_back(std::forward<T>(newName)); }
private:
std::vector<std::string> names;
};
```

这种方式减少了源代码中的重复，但增加了复杂性，因为`addName`现在是一个模板，必须放在头文件中，并且可能为每种类型实例化出多个版本。此外，某些实参类型不能通过通用引用传递，这可能导致编译错误。

**第三种方法：按值传递**

最后一种方法是简单地按值传递参数，这违反了传统上避免传值传递用户定义类型的建议，但在这个上下文中可能是合理的：

```cpp
class Widget {
public:
void addName(std::string newName) // 按值传递；自动处理拷贝或移动
{ names.push_back(std::move(newName)); }
private:
std::vector<std::string> names;
};
```

这种方法的优点在于它只需要一个函数声明和实现，简化了代码维护。同时，由于C++11引入的移动语义，当调用者传递一个临时对象（即右值）给`addName`时，编译器会自动选择更有效的移动构造函数而不是复制构造函数。而对于左值，依旧会发生复制操作。因此，这个单一的函数可以有效地处理两种情况。

_**<u>为什么按值传递合理？</u>**_

（1）简化代码：只编写一个函数，减少了源代码和目标代码中的重复，避免了维护多个函数声明和实现的问题。

（2）避免模板复杂性：不使用通用引用（万能引用）可以防止头文件膨胀、奇怪的失败情况以及令人困惑的编译错误信息。

（3）高效利用现代C++特性：得益于C++11的移动语义，当传入的是右值时，newName会通过移动构造而不是复制构造初始化，从而提高了性能。

效率对比

重载方式： 左值一次拷贝。右值一次移动

通用引用方式：左值一次拷贝。右值一次移动

按值传递方式：左值一次拷贝 + 一次移动。右值两次移动

从效率的角度来看，按值传递的方式在处理左值时确实引入了一次额外的移动操作。然而，由于移动操作通常比拷贝更轻量级，特别是对于大型对象或者资源管理类（如std::string），这种额外的开销通常是可接受的。对于右值来说，两次移动相比一次拷贝也可能是更优的选择。

1. 可拷贝类型的按值传递

```cpp
class Widget {
public:
void addName(std::string newName) {
    if ((newName.length() >= minLen) && (newName.length() <= maxLen)) {
        names.push_back(std::move(newName));
    }
}
private:
std::vector<std::string> names;
};
```

如果addName不总是将newName添加到names中，那么即使没有添加，也会有构造和销毁newName的开销。

2. 只可移动类型的重载

```cpp
class Widget {
public:
void setPtr(std::unique_ptr<std::string>&& ptr) {
    p = std::move(ptr);
}
private:
std::unique_ptr<std::string> p;
};
```

对于只可移动类型（如std::unique_ptr），只需要一个接受右值引用的重载函数即可，因为无法为左值提供有效的拷贝构造函数。

3. 赋值操作的复杂性

```cpp
class Password {
public:
explicit Password(std::string pwd) : text(std::move(pwd)) {}
void changeTo(std::string newPwd) { text = std::move(newPwd); }
private:
std::string text;
};
```

changeTo函数通过赋值来更新text，可能导致额外的内存管理开销，尤其是当旧对象的容量不足以容纳新对象时。

综合建议

对于可拷贝类型：仅当移动开销较小且总是需要拷贝时，才考虑按值传递。

对于只可移动类型：使用重载或通用引用来避免不必要的移动操作。

考虑实际使用场景：评估是否每次都需要构造和销毁形参，尤其是在条件判断之后可能不会使用的场景。

赋值操作的谨慎使用：当形参通过赋值操作进行复制时，注意潜在的内存管理开销，特别是在动态分配内存的情况下。

C++11没有从根本上改变C++98对于按值传递的智慧。通常，按值传递仍然性能下降，而且按值传递会导致切片问题。C++11中新的功能是区分了左值和右值实参。利用对于可拷贝类型的右值的移动语义，需要重载或者通用引用，尽管两者都有其缺陷。对于特殊的场景，可拷贝且移动开销小的类型，传递给总是会拷贝他们的一个函数，并且切片也不需要考虑，这时，按值传递就提供了一种简单的实现方式，效率接近传递引用的函数，但是避免了传引用方案的缺点。

**请记住：**

+ 对于可拷贝，移动开销低，而且无条件被拷贝的形参，按值传递效率基本与按引用传递效率一致，而且易于实现，还生成更少的目标代码。
+ 通过构造拷贝形参可能比通过赋值拷贝形参开销大的多。
+ 按值传递会引起切片问题，所说不适合基类形参类型。

<h4 id="ad9585ce">条款四十二：考虑使用置入代替插入</h4>
如果拥有一个容器，例如放着`std::string`，那么当你通过插入（insertion）函数（例如`insert`，`push_front`，`push_back`，或者对于`std::forward_list`来说是`insert_after`）添加新元素时，传入的元素类型应该是`std::string`。逻辑上看来如此，但是并非总是如此。

```cpp
std::vector<std::string> vs; //std::string的容器
vs.push_back("xyzzy");       //添加字符串字面量
```

容器里内容是`std::string`，但是实际上试图通过`push_back`加入的是字符串字面量，即引号内的字符序列。字符串字面量并不是`std::string`，这意味着你传递给`push_back`的实参并不是容器里的内容类型。

`std::vector`的`push_back`被按左值和右值分别重载：

```cpp
template <class T,                  //来自C++11标准
class Allocator = allocator<T>>
class vector {
public:
…
void push_back(const T& x);     //插入左值
void push_back(T&& x);          //插入右值
…
};
```

```cpp
vs.push_back("xyzzy");
```

这个调用中，编译器看到实参类型(`const char[6]`)和`push_back`采用的形参类型(`std::string`的引用）之间不匹配。它们通过从字符串字面量创建一个`std::string`类型的临时对象来消除不匹配，然后传递临时变量给`push_back`。换句话说，编译器处理的这个调用应该像这样：

```cpp
vs.push_back(std::string("xyzzy")); //创建临时std::string，把它传给push_back
```

代码可以编译并运行，皆大欢喜。除了对于性能执着的人意识到了这份代码不如预期的执行效率高。

为了在`std::string`容器中创建新元素，调用了`std::string`的构造函数，但是这份代码并不仅调用了一次构造函数，而是调用了两次，而且还调用了`std::string`析构函数。

1. 一个`std::string`的临时对象从字面量“`xyzzy`”被创建。这个对象没有名字，我们可以称为`temp`。`temp`的构造是第一次`std::string`构造。因为是临时变量，所以`temp`是右值。
2. `temp`被传递给`push_back`的右值重载函数，绑定到右值引用形参`x`。在`std::vector`的内存中一个`x`的副本被创建。这次构造——也是第二次构造——在`std::vector`内部真正创建一个对象。（将`x`副本拷贝到`std::vector`内部的构造函数是移动构造函数，因为`x`在它被拷贝前被转换为一个右值，成为右值引用。
3. 在`push_back`返回之后，`temp`立刻被销毁，调用了一次`std::string`的析构函数。

是否存在一种方法可以获取字符串字面量并将其直接传入到步骤2里在`std::vector`内构造`std::string`的代码中，可以避免临时对象`temp`的创建与销毁。

`emplace_back`就是像我们想要的那样做的：使用传递给它的任何实参直接在`std::vector`内部构造一个`std::string`。没有临时变量会生成

```cpp
vs.emplace_back("xyzzy");           //直接用“xyzzy”在vs内构造std::string
```

`emplace_back`使用完美转发，只要没有遇到完美转发的限制(参见item30)，就可以传递任何实参以及组合到`emplace_back`。

比如，如果想通过接受一个字符和一个数量的`std::string`构造函数，在`vs`中创建一个`std::string`，代码如下：

```cpp
vs.emplace_back(50, 'x');           //插入由50个“x”组成的一个std::string
```

`emplace_back`可以用于每个支持`push_back`的标准容器。类似每个支持`push_front`的标准容器都支持`emplace_front`。每个支持`insert`（除了`std::forward_list`和`std::array`）的标准容器支持`emplace`。关联容器提供`emplace_hint`来补充接受“hint”迭代器的`insert`函数，`std::forward_list`有`emplace_after`来匹配`insert_after`。

使得emplace函数功能优于插入函数的原因是它们有灵活的接口。插入函数接受**对象**去插入，而置入函数接受**对象的构造函数接受的实参**去插入。这种差异允许置入函数避免插入函数所必需的临时对象的创建和销毁。

因为可以传递容器内元素类型的实参给置入函数（因此该实参使函数执行复制或者移动构造函数），所以在插入函数不会构造临时对象的情况，也可以使用置入函数。在这种情况下，插入和置入函数做的是同一件事，比如：

```cpp
std::string queenOfDisco("Donna Summer");
```

下面的调用都是可行的，对容器的实际效果也一样:

```cpp
vs.push_back(queenOfDisco);         //拷贝构造queenOfDisco
vs.emplace_back(queenOfDisco);      //同上
```

因此置入函数可以完成插入函数的所有功能。并且有时效率更高，至少在理论上，不会更低效。那为什么不在所有场合使用它们？

因为只是“理论上”，在理论和实际上没有什么区别，但是实际上区别还是有的。在当前标准库的实现下，有些场景，就像预期的那样，置入执行性能优于插入，但是，有些场景反而插入更快。

这种场景不容易描述，因为依赖于传递的实参的类型、使用的容器、置入或插入到容器中的位置、容器中类型的构造函数的异常安全性，和对于禁止重复值的容器（即`std::set`，`std::map`，`std::unordered_set`，`set::unordered_map`）要添加的值是否已经在容器中。因此，大致的调用建议是：通过benchmark测试来确定置入和插入哪种更快。

如果下列条件都能满足，置入会优于插入：

+ **值是通过构造函数添加到容器，而不是直接赋值。** 例子就像本条款刚开始的那样（用“`xyzzy`”添加`std::string`到`std::vector`容器`vs`中），值添加到`vs`末尾——一个先前没有对象存在的地方。新值必须通过构造函数添加到`std::vector`。如果我们回看这个例子，新值放到已经存在了对象的一个地方，那情况就完全不一样了。考虑下：

```cpp
std::vector<std::string> vs;        //跟之前一样
…                                   //添加元素到vs
vs.emplace(vs.begin(), "xyzzy");    //添加“xyzzy”到vs头部
```

对于这份代码，没有实现会在已经存在对象的位置`vs[0]`构造这个添加的`std::string`。而是，通过移动赋值的方式添加到需要的位置。但是移动赋值需要一个源对象，所以这意味着一个临时对象要被创建，而置入优于插入的原因就是没有临时对象的创建和销毁，所以当通过赋值操作添加元素时，置入的优势消失殆尽。

向容器添加元素是通过构造还是赋值通常取决于实现。基于节点的容器实际上总是使用构造添加新元素，大多数标准库容器都是基于节点的。

例外的容器只有`std::vector`，`std::deque`，`std::string`。（`std::array`也不是基于节点的，但是它不支持置入和插入，所以它与这儿无关。）

在不是基于节点的容器中，可以依靠`emplace_back`来使用构造向容器添加元素，`std::deque`，`emplace_front`也是一样的。

+ **传递的实参类型与容器的初始化类型不同。**

置入优于插入通常基于以下事实：当传递的实参不是容器保存的类型时，接口不需要创建和销毁临时对象。当将类型为`T`的对象添加到`container<T>`时，没有理由期望置入比插入运行的更快，因为不需要创建临时对象来满足插入的接口。

+ **容器不拒绝重复项作为新值。** 容器要么允许添加重复值，要么你添加的元素大部分都是不重复的。这样要求的原因是为了判断一个元素是否已经存在于容器中，置入实现通常会创建一个具有新值的节点，以便可以将该节点的值与现有容器中节点的值进行比较。如果要添加的值不在容器中，则链接该节点。然后，如果值已经存在，置入操作取消，创建的节点被销毁，意味着构造和析构时的开销被浪费了。这样的节点更多的是为置入函数而创建，相比起为插入函数来说。

本条款开始的例子中下面的调用满足上面的条件。所以`emplace_back`比`push_back`运行更快。

```cpp
vs.emplace_back("xyzzy");              
//在容器末尾构造新值；不是传递的容器中元素的类型;没有使用拒绝重复项的容器
vs.emplace_back(50, 'x');//同上
```

资源管理与异常安全性

当你有一个容器，例如 std::list<std::shared_ptr<Widget>> ptrs;，并想添加一个带有自定义删除器的 std::shared_ptr 时，有几种方法可以实现。理想情况下，应使用 std::make_shared 来创建std::shared_ptr，因为它更高效且安全。然而，当需要指定自定义删除器时，这可能不可行。

在这种情况下，作者建议避免直接传递 new Widget 给 emplace_back 或 push_back，因为这样做可能导致资源泄漏问题，特别是在异常发生的情况下。相反，应该先创建一个临时的 std::shared_ptr 对象，并通过它来调用 push_back 或 emplace_back。这样可以确保即使在插入过程中发生了异常（如内存分配失败），std::shared_ptr 的析构函数也会被正确调用，从而避免资源泄漏。

```cpp
//推荐的方法：首先创建 std::shared_ptr 然后使用 push_back 或 emplace_back
std::shared_ptr<Widget> spw(new Widget, killWidget);
ptrs.push_back(std::move(spw)); // 或者
ptrs.emplace_back(std::move(spw));
```

Explicit 构造函数与置入函数

置入函数（如 emplace_back）使用直接初始化的方式，这意味着它们可以使用标记为 explicit 的构造函数。这不同于插入函数（如 push_back），后者使用拷贝初始化，不允许隐式转换或使用 explicit 构造函数。考虑一个例子，如果尝试将 nullptr 添加到 std::vector<std::regex> 中，emplace_back 会成功编译，而 push_back 则不会，因为 std::regex 的构造函数是 explicit 的，不允许从 nullptr 进行隐式转换。

```cpp
std::vector<std::regex> regexes;
regexes.emplace_back(nullptr); // 可以编译，但行为未定义
regexes.push_back(nullptr);    // 编译错误
```

结论

使用 std::make_shared 和 std::make_unique 是最佳实践，除非你需要自定义删除器。

如果必须手动创建 std::shared_ptr，则应先创建临时对象再使用 push_back 或 emplace_back，以确保异常安全。当使用置入函数时，要特别小心，因为它们可以调用 explicit 构造函数，可能会导致意外的行为或编译错误。

代码示例

```cpp
void killWidget(Widget* pWidget) {
    // 自定义删除逻辑
}
// 正确使用 push_back
{
    std::shared_ptr<Widget> spw(new Widget, killWidget);
    ptrs.push_back(std::move(spw));
}
// 正确使用 emplace_back
{
    std::shared_ptr<Widget> spw(new Widget, killWidget);
    ptrs.emplace_back(std::move(spw));
}
```

**请记住：**

+ 原则上，置入函数有时会比插入函数高效，并且不会更差。
+ 实际上，当以下条件满足时，置入函数更快：（1）值被构造到容器中，而不是直接赋值；（2）传入的类型与容器的元素类型不一致；（3）容器不拒绝已经存在的重复值。
+ 置入函数可能执行插入函数拒绝的类型转换。

