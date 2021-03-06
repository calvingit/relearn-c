# 重学 C 语言

- 教材：[《C 语言程序设计-现代方法》第二版](https://book.douban.com/subject/4279678/)
- C 语言版本: C99

其他参考资料：

- [C 语言中文网](http://c.biancheng.net/)

## 1. 格式化整数"%d"和 "%i" 的区别

- 对于`printf`来说没有区别
- 对于`scanf` 有区别："%d"只能用于接收 10 进制数字输入，"%i"可以接收 10 进制、8 进制、16 进制输入

例子：

```c
int a, b, c;
printf("Enter three integers: a b c\n");
scanf("%d%i%i", &a, &b, &c);
printf("Your integers: %i, %d, %i\n", a, b, c);
```

执行:

```bash
Enter two integers: a b c
12 010 0xFF
Your integers: 12, 8, 255
```

## 2. `scanf`输入格式不正确时会跳过匹配

scanf 匹配出错时，会把剩余的内容返回给缓冲区，留给下一次`scanf`匹配。

```c
void testScanf2(void)
{
  int a;
  float b;
  char c[20] = {0};
  scanf("%d%f%s", &a, &b, c);
  printf("%d %f %s\n", a, b, c);
}
```

执行:

```bash
3.1415926 abcd
3 0.141593 abcd
```

## 3. C99 有 bool 类型

C89 没有定义 bool 类型，只能用 int 代替，比如厂家的宏定义：

```c
#define BOOL int
#define TRUE 1
#define FALSE 0

BOOL flag = TRUE;

if (flag) {
	// do something
}
```

C99 定义了 `_Bool`基本类型， 本质上还是 unsigned int。C99 增加了头文件`stdbool.h`，头文件里定义了`bool`宏。还有 `true` 、`false`宏，分别代表 1 和 0。

```c
#define bool _Bool
```

`_Bool`类型只接受 0 或 1，赋值非 0 的整数都会变成 1，当然 print 也只会是 0 和 1

## 4. 类型定义 typedef 代替 #define

typedef 在可读性和可移植性上会更好，重命名类型时尽量用 typedef。

```c
typedef float Dollars;

Dollars cash_in, cash_out;
```

类似的还有标准库的`size_t`, `wchar_t`等。
C99 里面的头文件`stdint.h`使用 typedef 定义不同长度的整型，如`int32_t`表示 32 位有符号整型。

**数组和指针类型不能定义为宏**，比如一个“指向整数的指针”：

```c
// 定义宏
#define PTR_TO_INT int *

// 声明
PTR_TO_INT p, q, r;
```

宏替换之后变成`int * p, q, r;`，其中只有 p 是指针，q 和 r 是整型。typedef 不会有这样的问题

## 5. sizeof 运算符

`sizeof` 返回类型是 `size_t`，在 C89 中是`unsigned long`，在 C99 中 可以更长。

格式化时有点区别，C89 需要强制转换成 `unsigned long`，C99 不需要。但是格式化说明符有点区别：

```c
printf("%lu\n", (unsigned long)sizeof(int));
printf("%zu\n", sizeof(int)); // C99才可以
```

## 6. 数组初始化可以指定下标

C89 的数组初始化一般是下面几种:

```c
int a[5] = {1,2,3,4,5} // 全部初始化
int a[10] = {1,2,3,4,5} // 部分初始化，剩余部分为0
int a[20] = {0} // 全部初始化为0
int a[] = {1,2,3,4,5,6,7,8,9,10} // 不指定长度，由初始化长度决定
```

有时候比较大的数组只需要初始化较少的元素，其他默认赋值。C89 只能做到下面这种，冗余且难看：

```c
int a[15] = {0,1,2,3,0,5,0,0,0,0,10,0,0,13,14}
```

C99 可以指定序号初始化，而且先后顺序没有关系:

```c
int a[15] = {[2] = 29, [14] = 34, [10] = 22}
```

如果不指定数组长度，由最大的序号决定数组长度，即最大的序号是 n，长度是 n+1，如下面的长度是 24：

```c
int a[] = {[5] = 9, [23] = 89, [10] = 78}
```

**这种方式对多维数组也有效**，如下：

```c
double ident[2][3] = {[1][1] = 2.0, [0][2] = 5.0 }
```

## 7. C99 的 变长数组

C89 的数组变量的长度必需是常量表达式，需要指定长度，或者添加初始化表达式。

```c
int a[10]; // 长度为0
int a[] = {0,0,0}; // 长度为3
int a[]; // error: definition of variable with array type needs an explicit size or an initializer
```

C99 增加了变长数组，即可以用一个变量来表示数组的长度:

```c
int n = 9;
int dd[n * 12];
```

**不能初始化变长数组**:

```c
int dd[n * 2] = {0}; // error: variable-sized object may not be initialized
```

## 8. 数组作为函数参数

一般情况下，数组不会单独作为参数，会有长度参数：

```c
void store_zeros(int a[], int n)￼
```

数组当参数时需要明确两个重要论点：

- **函数无法检测传入的数组长度的正确性**

  > n 可以比数组实际长度更大，或者更小
  >
  > ```c
  > void store_zeros(int a[], int n)￼ {
  >
  > }
  >
  > int n = 10;
  > int arr[5] = {1,2,3,4,5};
  > store_zeros(arr, n)
  > ```
  >
  > 即使数组带了长度，如`void store_zeros(int a[5], int n)￼`，编译器也会忽略掉长度 5，并没有什么用处，因为实际上可以传递任意长度的数组。

- 函数可以改变数组参数的元素，且改变会在相应的实际参数中体现出来

  > 函数体内可以重新赋值新元素给数组
  >
  > ```c
  > void store_zeros(int a[], int n)￼
  > {￼
  >   	for (int i = 0; i < n; i++)￼
  > 		a[i] = 0;￼
  > }
  > ```

C99 数组作为函数参数时，同样可以使用变长数组:

```c
void fun(int n, int a[n])
{
  // do something...
}

int concatenate(int m, int n, int a[m], int b[n], int c[m+n])￼
{
  // 编写一个函数来连接两个数组a和b，要求先复制a的元素，再复制b的元素，把结果写入第三个数组c
}
```

注意：**长度变量 n 必须在数组之前，因为需要先声明**。

多维数组的时候，这个特性会更有效，可以指定行数和列数：

```c
int sum_two_dimensional_array(int n, int m, int a[n][m])￼
{￼
  int i, j, sum = 0;￼
  for (i = 0; i < n; i++)￼
    for (j = 0; j < m; j++)￼
      sum += a[i][j];￼
  return sum;￼
}
```

C99 可以使用复合字面量来初始化数组参数，当你的数组变量没有其他地方使用的时候，这样可以方便一些:

```c
// C89
int b[] = {3, 0, 3, 4, 1};￼
total = sum_array(b, 5);

// C99
total = sum_array((int []){3, 0, 3, 4, 1},5);
```

也可以部分初始化`sum_array((int []){3},5);`，第一个初始化为 3，其他为 0。
当然，复合字面量的元素也可以在函数内部被改变，如要要只读，加上`const`，如`(const int[]){4,5}`。

## 9. main 函数

早期的 main 函数常常省略返回类型，这是利用了函数返回类型默认为 int 类型的传统，甚至连参数有时都会省略:

```c
main()
{
}
```

C99 规定上面这种是不合法的，main 函数返回的值是状态码，在某些操作系统中程序终止时可以检测到状态码。￼

**程序退出**

如果程序正常终止，main 函数应该返回 0；为了表示异常终止，main 函数应该返回非 0 的值。

```c
int main(void)
{
  return 0;
}
```

当然，程序退出使用`<stdlib.h>`头文件里的`exit(int)`函数也是一样的效果，参数和 return 是一样的效果：

```c
exit(0);
exit(EXIT_SUCCESS); // #define	EXIT_SUCCESS	0
exit(EXIT_SUCCESS); // #define	EXIT_FAILURE	1
```

`return` 只能在 main 里面用，`exit` 可以在任何地方调用。

## 10. 数组与指针

- 对于任意一维数组 a 来说，`a[i]` 等价于` *(a+i)`

- 当做形式参数时，数组参数和指针参数是一样的

  ```c
  void find_largest(int a[], n)
  void find_largest(int *a, n)
  ```

- 数组当形式参数时，不一定要从 0 开始，可以从中间位置传递:
  ```c
  int largest = find_largest(&b[5], 10)
  ```

**二维数组**

```c
int a[NUM_ROWS][NUM_COLS];
```

一个重要的观点：数组是按行存储的，对于二维数组是 `a[行数][列数]`。

对 a 来说，`&a[i][0]`指向第 i 行的第一个元素, 等同于`&(*(a[i] + 0))`, 等价于`&*a[i]`，&和\*抵消之后，等同于`a[i]`。

也可以将二维数组看作一维数组，这个一维数组的每个元素又是一维数组。当指针时，a 的类型是`int (*)[NUM_COLS]`，指向长度为 NUM_COLS 的整型数组的指针。

例如，把数组 a 的第 i 列清零：

```c
int a[3][5] = {{1, 2, 3, 4, 5}, {6, 7, 8, 9, 10}, {11, 12, 13, 14, 15}};
int(*p)[5]; // 指针类型
int i = 2;
for (p = &a[0]; p < &a[3]; p++)
{
	(*p)[i] = 0;
}
```

## 11. 判断字符串结束符'\0'

判断字符串结束的标志是最后一个字符是否为等于 '\0'。在 ASCii 码表里面，第一个字符就是空字符。所以，`*s != '\0'`就等于`*s != 0`，也等于`*s`

```c
size_t strlen(const char *s)
{
  size_t len = 0;
  while (*s != '\0')
  // 或者 while (*s != 0)
  // 或者 while (*s)
  {
    len++;
    s++;
  }

  return len;
}
```

指针也可以做加减运算，结束的位置地址减去第 0 个地址即可算出字符串的长度，所以长度 len 的自增可以省略掉，简化成下面的样子：

```c
size_t strlen(const char *s)
{
  const char *p = s;
  while (*s)
  {
    s++;
  }

  return s - p;
}
```

## 12. 运算符优先级

没有必要死记硬背所有的优先级，但是要注意最顶级的两类运算符：

- 优先级 1 的运算符，结合方向是从左到右

| 运算符 | 名称或含义       | 使用形式                  |
| ------ | ---------------- | ------------------------- |
| []     | 数组下标         | 数组名[常量表达式]        |
| ()     | 圆括号           | （表达式）/函数名(形参表) |
| .      | 成员选择（对象） | 对象.成员名               |
| ->     | 成员选择（指针） | 对象指针->成员名          |

- 优先级 2 的运算符，结合方向是从右到左

| 运算符 | 名称或含义     | 使用形式          |
| ------ | -------------- | ----------------- |
| (类型) | 强制类型转换   | (数据类型)表达式  |
| ++     | 自增运算符     | ++变量名/变量名++ |
| --     | 自减运算符     | --变量名/变量名-- |
| \*     | 取值运算符     | \*指针变量        |
| &      | 取地址运算符   | &变量名           |
| !      | 逻辑非运算符   | !表达式           |
| ~      | 按位取反运算符 | ~表达式           |
| sizeof | 长度运算符     | sizeof(表达式)    |
| -      | 负号运算符     | -表达式           |

**容易出错的问题：**

| 优先级问题            | 表达式              | 错误结果                                        | 实际结果                                                |
| --------------------- | ------------------- | ----------------------------------------------- | ------------------------------------------------------- |
| .的优先级高于\*       | \*p.f               | p 所指对象的字段 f，(\*p).f                     | 对 p 取 f 偏移，作为指针，然后进行解除引用操作。\*(p.f) |
| []高于\*              | int \*ap[]          | ap 是个指向 int 数组的指针, int (\*ap)[]        | ap 是个元素为 int 指针的数组, int \*(ap[])              |
| 函数()高于\*          | int \*fp()          | fp 是个函数指针，所指函数返回 int。int (\*fp)() | fp 是个函数，返回 int _。int _(fp())                    |
| == 和 != 高于位操作   | (val & mask != 0)   | (Val & mask) != 0                               | Val & (mask != 0)                                       |
| == 和 != 高于赋值操作 | c= getchar() != EOF | (c = getchar()) != EOF                          | c = (getchar() != EOF)                                  |

## 13. 指针与自增运算\*p++

++和\*都是同一优先级，但是结合性是从右往左。

| A =     | 第一步 | 第二步  |                                |
| ------- | ------ | ------- | ------------------------------ |
| \*p++   | \*p    | p++     | 先取值，后指针++，A = \*p      |
| \*++p   | ++p    | \*(++p) | 先指针++，后取值，A = \*(++p)  |
| ++\*p   | \*p    | (\*p)+1 | 先取值，后值++，A = (\*p)+1    |
| (\*p)++ | \*p    | (\*p)+1 | 先取值，后值++，A = (\*p) + 1; |

我们实现一个字符串拼接的函数`strcat`：

```c
char *strcat(char *s1, const char *s2)
{
  char *p = s1;
  while (*p) // 找到s1的最后一个位置
    p++;

  while ((*p++ = *s2++))
    ;
  return s1;
}
```

我们注意第二个`while`循环，首先是后缀`++`操作，这两个可以先忽略，变成：

```c
*p = *s2
```

这个表达式就是将 s2 指向的字符复制到 p 指向的地方，一直重复操作。

循环终止的条件是 \*s2 为空字符，并且是赋值之后才终止，所以循环体内不需要用一条语句来在新字符串的末尾添加空字符'\0'。

赋值运算之后才进行自增操作。

多加了一对圆括号是为了消除 gcc 的 `Wparentheses`警告，gcc 编辑器在-Wall 选项下，会明确用户在判断语句中使用"="的真正意图。当你想在判断语句中使用"="时，要加上括号：`if((a = b) !=0)`。

## 14. struct 结构体

c89 的 struct 初始化需要按照字段声明顺序进行初始化，但 C99 可以指定每个字段进行初始化，并且不限制顺序，类似数组的初始化，比如：

```c
struct student
{
  int age;
  char parents[20];
  char *name;
};
struct student s = {.name = "zhangsan", .age = 18};
```

**struct 数组**

假设有一个结构体:

```c
struct dialing_code {
  char *country;
  int code;
}
```

然后有一个数组`country_codes`，元素类型是`dialing_code`, 初始化可以像这样：

```c
const struct dialing_code country_codes[] = {
  {"China", 86},
  {"France", 33},
  {"Russia", 7}
};
```

相应的，C99 里面也可以指定初始化式：

```c
struct student students[100] = {
  [0].age = 15,
  [0].parents[0] = '\0',
  [0].name = "Jack"
}
```

**内存对齐**

默认情况下编译器将按照实际结构体中占内存字节数最大的成员的整数倍进行分配，该最大的整数倍应正好大于或者等于结构体实际所占的内存字节数。

```c
struct {
  char a;
  int b;
} s;
```

内存对齐的原因，使用`sizeof(s)`计算时是 8 个字节，其中`a`多占了 3 个空的位置。

如果 s 里面包含`double`类型，那么内存大小是 16。

> 可以用 `#pragma pack(N)`来指定 N 个字节对齐，常见于嵌入式开发中。

**复制**

当 struct 里面有数组时，会把数组内容一起复制过去。但是如果是指针，只会复制地址。

```c
int n = 33;
struct student {
  int age;
  char parents[40];
  char *name;
};
struct student s = {.name = "zhangsan", .age = n, .parents = "father,mother"};
struct student s2 = s;
strcpy(s.parents, "f,m");

printf("%d, %s, %s\n", s.age, s.name, s.parents);
printf("%d, %s, %s\n", s2.age, s2.name, s2.parents);
```

上面例子打印如下:

```bash
33, zhangsan, f,m
33, zhangsan, father,mother
```

## 15. union 联合

编译器只为 union 中最大的成员分配足够的内存空间，初始化的时候只能初始化一个成员。

> **类似的 Swift 的 enum 了，每个 case 就是一个成员，每个成员指定不同的类型。**

C99 的 union 的初始化也和 struct 类似，可以指定成员：

```c
union {
	int i;
  double d;
} u = { .d = 10.0 }
```

**union 的作用主要用来节省空间，但是不容易确定 union 最后改变的成员，所以可以用一个 struct 再封装一层，增加一个标记字段。**

例如下面比较复杂的例子中的`item_type`：

```c
struct catalog_item {
  int stock_number;
  double price;
  int item_type;
  union {
    struct {
      char title[TITLE_LEN + 1];
      char author[AUTHOR_LEN + 1];
      int num_pages;
    } book;
    struct {
      char design[DESIGN_LEN + 1];
    } mug;
    struct {
      char design[DESIGN_LEN + 1];
      int colors;
      int sizes;
    } shirt;
  } item;
}
```

赋值的时候同时改变`item`和`item_type`，调用的时候可以这样判断:

```c
if (catalog.item_type == BOOK) {
  printf("%s", catalog.item.book.title);
} else {
  printf("%s", catalog.item.mug.design);
}
```

**注意：union 的成员之间虽然是不同大小的内存长度，但是内存是对齐的，即起始位置一样，结束位置当然是最大那个成员的长度了。**

这个特性会带来一些副作用，对某一个成员赋值时会影响到其他成员，比如：

```c
union data{
  int n;
  char ch;
  short m;
};
```

对`n`赋值为 100，`ch`和`m`的值也会变成`100`；对`n`赋值 128, `ch`是超出范围，变成-128，`m`还是 128。
即使 union 成员是结构体，一样会受影响：

```c
union {
struct {
int a;
} i;

struct {
char b1;
char b2;
char b3;
char b4;
} c;

struct {
char cs[4];
} ch;

struct {
float f;
} f;

} u;
u.i.a = 0x12345678;
printf("%x\n", u.i.a);
printf("%x, %x, %x, %x\n", u.c.b1, u.c.b2, u.c.b3, u.c.b4);
printf("%x\n", u.ch.cs[0]);
printf("%f\n", u.f.f);
```

上面例子输出结果如下：

```bash
12345678
78, 56, 34, 12
78
0.000000
```

浮点类型存储方式和整型不一样，所以不受影响。

## 16. enum 枚举

很多 C 开发者喜欢用宏来定义各种常量，这可以增加代码的可读性，但是如果有多个具有相同类型(特别是整型)的值，定义成 enum 会更好。尤其是在调试的时候，宏定义被替换成具体的值，擦除了类型。

C 语言是把 enum 当做整数来处理的，所以可以对不同的 enum 值指明任意整数，可以相同，并且顺序无关。

```c
enum EGA_colors {
  BLACK,
  LT_GRAY = 7,
  DK_GRAY,
  WHITE = 15,
}
```

> C99 中允许 enum 的最后一个元素后面增加","结尾

**用 enum 来标记字段**

可以用来确定 union 中最后一个被赋值的成员，如：

```c
typedefu struct {
  enum { INT_KIND, DOUBLE_KIND } kind;
  union {
    int i;
    double d;
  } u;
} Number;
```

**用作数组下标**
因为是整数是原因，又是默认从 0 开始，所以是很理想的下标：

```c
enum weekdays {MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY};￼
const char *daily_specials[] = {￼
  [MONDAY] = "Beef ravioli",￼
  [TUESDAY] = "BLTs",￼
  [WEDNESDAY] = "Pizza",￼
  [THURSDAY] = "Chicken fajitas",￼
  [FRIDAY] = "Macaroni and cheese"￼
};
```

## 17. 左值和右值

值是表达式，到底是左还是右，是相对赋值运算符来说的：左值 = 右值。

通常来说，一个左值肯定可以解析出对应对象的地址。生成左值的运算符包括下标运算符`[]`和间接运算符`*`，如下表：

| 表达式    | 是左值吗                                     |
| --------- | -------------------------------------------- |
| array[1]  | 是；一个数组元素是一个具有位置的对象         |
| &array[1] | 否；此对象的位置，并非一个具有位置的对象     |
| ptr       | 是；此指针变量是一个具有位詈的对象           |
| \*ptr     | 是；指针所指的地方是一个具有位置的对象       |
| ptr+1     | 否；此加法产生一个新的地址值，但不是一个对象 |
| \*ptr+l   | 否；此加法产生一个新的算术值，但不是一个对象 |

被`const`修饰的常量对象，尽管是左值，但是不能位于赋值运算的左边。

> 赋值运算左边的操作数，以及任何自增或自减运算符（++ 和 --）的操作数，不仅应该是左值，还应该是可修改的左值。可修改的左值，其类型不可以被声明为限定符 const，并且可修改的左值不能是数组类型。如果可修改的左值所表示的对象是结构或联合类型，那么它的元素都不可以被声明（不管是直接地或间接地）为具有限定符 const 的类型

## 18. 指针的高级应用

### 动态内存分配

主要用到 3 个函数，都定义在 <stdlib.h> 里面：

- malloc：分配内存，不初始化
- calloc：分配内存，初始化为 0
- realloc：调整内存大小
- free: 释放内存

当使用`malloc`函数给字符串分配内存空间时，长度应该是 n + 1，给最后的空字符留出空间。

### 链表

**插入结点**

```c
struct node *add_to_list(struct node *list, int n)￼
{￼
  struct node *new_node;￼
  new_node = malloc(sizeof(struct node));￼
  if (new_node == NULL)  {￼
    printf("Error: malloc failed in add_to_list\n");￼
    exit(EXIT_FAILURE);￼
  }￼
  new_node->value = n;￼
  new_node->next = 1ist;￼
  return new_node;￼
}
```

**搜索链表结点**

到达链表末尾处时 list 为 NULL，所以即使找不到 n，返回 list 也是正确的

```c
struct node *search_list(struct node *list, int n) {
	while(list != NULL && list->value != n) {
    list = list->next;
  }
  return list;
}
```

**删除链表结点**

删除结点也包含 3 个步骤：

1. 定位要删除的结点；
2. 改变前一个结点，从而使它“绕过”删除结点；
3. 调用 free 函数收回删除结点占用的内存空间。

在第 1 步搜索链表时，将保留一个指向前一个结点的指针（prev），还有指向当前结点的指针（cur）。

```c
struct node *delete_from_list(struct node *list, int n)￼
{￼
  struct node *cur, *prev;￼
  for (cur = list, prev = NULL;￼
       cur != NULL && cur->value != n;￼
       prev = cur,  cur = cur->next)￼
    ;￼
  if (cur == NULL)￼
    return list;                /* n was not found */￼
  if (prev == NULL)￼
    list = list->next;          /* n is in the first node */￼
  else￼
    prev->next = cur->next;     /* n is in some other node */￼
  free (cur);￼
  return list;￼
}
```

**指向指针的指针**

以链表插入结点为例，如果不想要返回`new_node`，可以改成以下版本：

```c
void_add_to_list(struct node **list, int n)￼
{￼
   struct node *new_node;￼
   new_node = malloc(sizeof(struct node));￼
   if (new_node == NULL) {￼
     printf("Error: malloc failed in add_to_list\n");￼
     exit(EXIT_FAILURE);￼
   }￼
   new_node->value = n;￼
   new_node->next = *list;￼
   *list = new_node;￼
}
```

当调用新版本的函数 add_to_list 时，第一个实际参数将会是 first 的地址：

```c
add_to_list(&first, 10);
```

既然给 list 赋予了 first 的地址，那么可以使用*list 作为 first 的别名。特别是，把 new_node 赋值给*list 将会修改 first 的内容。

**指向函数的指针**

通常声明函数指针是下面这样的格式：

```c
double integrate(double (*f)(double), double a, double b);
```

也可以直接用函数的声明形式:

```c
double integrate(double f(double), double a, double b);
```

调用`integrate`函数：

```c
result = integrate(sin, 0.0, PI / 2);
```

`integrate`函数体可以这样使用参数`f`:

```c
y = (*f)(x);
```

最经典的例子当属`stdlib.h`里面的`qsort`函数，声明如下：

```c
void qsort(void *__base, size_t __nel, size_t __width,
	    int (* _Nonnull __compar)(const void *, const void *));
```

## 19. 声明

几个关键词：

- **static**: 静态声明，全局变量。也用于作用域，文件内或块内
- **extern**: 常用于头文件，提供给别的文件引用变量
- **const**: 只读变量，但是不能用于常量表达式，比如数组声明时的长度不用使用 const 变量。

  比如下面是错误的：

  ```c
  const int x=10;
  char arr[x] = {0};
  ```

  没有绝对的原则说明何时使用#define 以及何时使用 const。这里建议对表示数或字符的常量使#define。这样就可以把这些常量作为数组维数，并且在 switch 语句或其他要求常量表达式的地方使用它们

- **volatile**: 只用在底层编程，可忽略
- **restrict**：C99 特性，只用于指针

## 20. <ctype.h>判断字符类型的函数

| 函数    | 说明                                                                  |
| ------- | --------------------------------------------------------------------- |
| isalnum | 是否字母或数字                                                        |
| isalpha | 是否字母                                                              |
| isblank | 是否空格和 水平制表符\t (c99)                                         |
| iscntrl | 是否控制字符 \x00 到\x1f ，和\x7f                                     |
| isdigit | 是否十进制数字                                                        |
| isgraph | 是否可显示字符(除空格外)                                              |
| islower | 是否小写字母                                                          |
| issuper | 是否大写字母                                                          |
| isprint | 是否可打印字符(包括空格)                                              |
| isspace | 是否空白字符(换页\f， 换行符\n, 回车符\r, 水平制表符\t, 垂直制表符\v) |

这些函数常用来判断字符串里面的每个字符，比如校验用户输入的数据需要纯字母或纯数字。可以直接判断 char 类型的 ASCII 值，但是没有这些函数这么清晰明了。
