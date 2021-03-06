---
title: C语言assert断言函数完全攻略
tags:
  - 笔试面经
  - C/C++编程
categories:
  - 学习笔记
date: 2021-09-15 14:51:15
---

## C语言assert断言函数完全攻略

对于断言，相信大家都不陌生，大多数编程语言也都有断言这一特性。简单地讲，断言就是对某种假设条件进行检查。在 C 语言中，断言被定义为宏的形式（assert(expression)），而不是函数，其原型定义在<assert.h>文件中。其中，assert 将通过检查表达式 expression 的值来决定是否需要终止执行程序。也就是说，如果表达式 expression 的值为假（即为 0），那么它将首先向标准错误流 stderr 打印一条出错信息，然后再通过调用 abort 函数终止程序运行；否则，assert 无任何作用。

默认情况下，assert 宏只有在 Debug 版本（内部调试版本）中才能够起作用，而在 Release 版本（发行版本）中将被忽略。当然，也可以通过定义宏或设置编译器参数等形式来在任何时候启用或者禁用断言检查（不建议这么做）。同样，在程序投入运行后，最终用户在遇到问题时也可以重新起用断言。这样可以快速发现并定位软件问题，同时对系统错误进行自动报警。对于在系统中隐藏很深，用其他手段极难发现的问题也可以通过断言进行定位，从而缩短软件问题定位时间，提高系统的可测性。

<!--more-->

### 尽量利用断言来提高代码的可测试性

在讨论如何使用断言之前，先来看下面一段示例代码：

```c++
void *Memcpy(void *dest, const void *src, size_t len)
{
    char *tmp_dest = (char *)dest;
    char *tmp_src = (char *)src;
    while(len --)
            *tmp_dest ++ = *tmp_src ++;
    return dest;
}
```

对于上面的 Memcpy 函数，毋庸置疑，它能够通过编译程序的检查成功编译。从表面上看，该函数并不存在其他任何问题，并且代码也非常干净。

但遗憾的是，在调用该函数时，如果不小心为 dest 与 src 参数错误地传入了 NULL 指针，那么问题就严重了。轻者在交付之前这个潜在的错误导致程序瘫痪，从而暴露出来。否则，如果将该程序打包发布出去，那么所造成的后果是无法估计的。

由此可见，不能够简单地认为“只要通过编译程序成功编译的就都是安全的程序”。当然，编译程序也很难检查出类似的潜在错误（如所传递的参数是否有效、潜在的算法错误等）。面对这类问题，一般首先想到的应该是使用最简单的if语句进行判断检查，如下面的示例代码所示：

```c++
void *Memcpy(void *dest, const void *src, size_t len)
{
    if(dest == NULL)
    {
        fprintf(stderr,"dest is NULL\n");
        abort();
    }
    if(src == NULL)
    {
        fprintf(stderr,"src is NULL\n");
        abort();
    }
    char *tmp_dest = (char *)dest;
    char *tmp_src = (char *)src;
    while(len --)
        *tmp_dest ++ = *tmp_src ++;
    return dest;
}
```

现在，通过“if（dest==NULL）与if（data-src==NULL）”判断语句，只要在调用该函数的时候为 dest 与 src 参数错误地传入了NULL指针，这个函数就会检查出来并做出相应的处理，即先向标准错误流 stderr 打印一条出错信息，然后再调用 abort 函数终止程序运行。W

从表面看来，上面的解决方案应该堪称完美。但是，随着函数参数或需要检查的表达式不断增多，这种检查测试代码将占据整个函数的大部分（这一点从上面的 Memcpy 函数中就不难看出）。这样代码看起来非常不简洁，甚至可以说很“糟糕”，而且也降低了函数的执行效率。

面对上面的问题，或许可以利用 C 的预处理程序有条件地包含或不包含相应的检查部分进行解决，如下面的代码所示：

```c++
void *MemCopy(void *dest, const void *src, size_t len)
{
    #ifdef DEBUG
    if(dest == NULL)
    {
        fprintf(stderr,"dest is NULL\n");
        abort();
    }
    if(src == NULL)
    {
        fprintf(stderr,"src is NULL\n");
        abort();
    }
    #endif
    char *tmp_dest = (char *)dest;
    char *tmp_src = (char *)src;
    while(len --)
        *tmp_dest ++ = *tmp_src ++;
    return dest;
}
```

这样，通过条件编译“#ifdef DEBUG”来同时维护同一程序的两个版本（内部调试版本与发行版本），即在程序编写过程中，编译其内部调试版本，利用其提供的测试检查代码为程序自动查错。而在程序编完之后，再编译成发行版本。

上面的解决方案尽管通过条件编译“#ifdef DEBUG”能产生很好的结果，也完全符合我们的程序设计要求，但是仔细观察会发现，这样的测试检查代码显得并不那么友好，当一个函数里这种条件编译语句很多时，代码会显得有些浮肿，甚至有些糟糕。

因此，对于上面的这种情况，多数程序员都会选择将所有的调试代码隐藏在断言 assert 宏中。其实，assert 宏也只不过是使用条件编译“#ifdef”对部分代码进行替换，利用 assert 宏，将会使代码变得更加简洁，如下面的示例代码所示：

```c++
void *MemCopy(void *dest, const void *src, size_t len)
{
    assert(dest != NULL && src !=NULL);
    char *tmp_dest = (char *)dest;
    char *tmp_src = (char *)src;
    while(len --)
            *tmp_dest ++ = *tmp_src ++;
    return dest;
}
```

现在，通过“assert(dest !=NULL&&src !=NULL)”语句既完成程序的测试检查功能（即只要在调用该函数的时候为 dest 与 src 参数错误传入 NULL 指针时都会引发 assert），与此同时，对 MemCopy 函数的代码量也进行了大幅度瘦身，不得不说这是一个两全其美的好办法。

实际上，在编程中我们经常会出于某种目的（如把 assert 宏定义成当发生错误时不是中止调用程序的执行，而是在发生错误的位置转入调试程序，又或者是允许用户选择让程序继续运行等）需要对 assert 宏进行重新定义。

但值得注意的是，不管断言宏最终是用什么样的方式进行定义，其所定义宏的主要目的都是要使用它来对传递给相应函数的参数进行确认检查。如果违背了这条宏定义原则，那么所定义的宏将会偏离方向，失去宏定义本身的意义。与此同时，为不影响标准 assert 宏的使用，最好使用其他的名字。例如，下面的示例代码就展示了用户如何重定义自己的宏 ASSERT：

```
/*使用断言测试*/
#ifdef DEBUG
/*处理函数原型*/
void Assert(char * filename, unsigned int lineno);
#define ASSERT(condition)\
if(condition)\
    NULL; \
else\
    Assert(__FILE__ , __LINE__)
/*不使用断言测试*/
#else
#define ASSERT(condition) NULL
#endif
void Assert(char * filename, unsigned int lineno)
{
    fflush(stdout);
    fprintf(stderr,"\nAssert failed： %s, line %u\n",filename, lineno);
    fflush(stderr);
    abort();
}
```

如果定义了 DEBUG，ASSERT 将被扩展为一个if语句，否则执行“#define ASSERT(condition) NULL”替换成 NULL。

这里需要注意的是，因为在编写 C 语言代码时，在每个语句后面加一个分号“；”已经成为一种约定俗成的习惯，因此很有可能会在“Assert（__FILE__，__LINE__）”调用语句之后习惯性地加上一个分号。实际上并不需要这个分号，因为用户在调用 ASSERT 宏时，已经给出了一个分号。面对这种问题，我们可以使用“do{}while(0)”结构进行处理，如下面的代码所示：

```c++
#define ASSERT(condition)\
do{   \
    if(condition)\
       NULL; \
    else\
       Assert(__FILE__ , __LINE__);\
}while(0)

现在，将不再为分号“；”而担心了，调用示例如下：
void Test(unsigned char *str)
{
    ASSERT(str != NULL);
    /*函数处理代码*/
}
int main(void)
{
    Test(NULL);
    return 0;
}
```

很显然，因为调用语句“Test（NULL）”为参数 str 错误传入一个 NULL 指针的原因，所以 ASSERT 宏会自动检测到这个错误，同时根据宏 __FILE__ 和 __LINE__ 所提供的文件名和行号参数在标准错误输出设备 stderr 上打印一条错误消息，然后调用 abort 函数中止程序的执行。运行结果如图 1 所示。

![img](C%E8%AF%AD%E8%A8%80assert%E6%96%AD%E8%A8%80%E5%87%BD%E6%95%B0%E5%AE%8C%E5%85%A8%E6%94%BB%E7%95%A5/2-1PZ6151UA46-1631674767386.jpg)

图 1 调用自定义 ASSERT 宏的运行结果

如果这时候将自定义 ASSERT 宏替换成标准 assert 宏结果会是怎样的呢？如下面的示例代码所示：

```c++
void Test(unsigned char *str)
{
    assert(str != NULL);
    /*函数处理代码*/
}
```

毋庸置疑，标准 assert 宏同样会自动检测到这个 NULL 指针错误。与此同时，标准 assert 宏除给出以上信息之外，还能够显示出已经失败的测试条件。运行结果如图 2 所示。

![img](C%E8%AF%AD%E8%A8%80assert%E6%96%AD%E8%A8%80%E5%87%BD%E6%95%B0%E5%AE%8C%E5%85%A8%E6%94%BB%E7%95%A5/2-1PZ615193Dc-1631674767405.jpg)
图 2 调用标准 assert 宏的运行结果

从上面的示例中不难发现，对标准的 assert 宏来说，自定义的 ASSERT 宏将具有更大的灵活性，可以根据自己的需要打印输出不同的信息，同时也可以对不同类型的错误或者警告信息使用不同的断言，这也是在工程代码中经常使用的做法。当然，如果没有什么特殊需求，还是建议使用标准 assert 宏。

### 尽量在函数中使用断言来检查参数的合法性

在函数中使用断言来检查参数的合法性是断言最主要的应用场景之一，它主要体现在如下 3 个方面：

1. 在代码执行之前或者在函数的入口处，使用断言来检查参数的合法性，这称为前置条件断言。
2. 在代码执行之后或者在函数的出口处，使用断言来检查参数是否被正确地执行，这称为后置条件断言。
3. 在代码执行前后或者在函数的入出口处，使用断言来检查参数是否发生了变化，这称为前后不变断言。

例如，在上面的 Memcpy 函数中，除了可以通过“`assert(dest !=NULL&&src!=NULL);`”语句在函数的入口处检查 dest 与 src 参数是否传入 NULL 指针之外，还可以通过`assert(tmp_dest>=tmp_src+len||tmp_src>=tmp_dest+len);`  语句检查两个内存块是否发生重叠。如下面的示例代码所示：

```c++
void *Memcpy(void *dest, const void *src, size_t len)
{
    assert(dest!=NULL && src!=NULL);
    char *tmp_dest = (char *)dest;
    char *tmp_src = (char *)src;
    /*检查内存块是否重叠*/
    assert(tmp_dest>=tmp_src+len||tmp_src>=tmp_dest+len);
    while(len --)
            *tmp_dest ++ = *tmp_src ++;
    return dest;
}
```

除此之外，建议每一个 assert 宏只检验一个条件，这样做的好处就是当断言失败时，便于程序排错。试想一下，如果在一个断言中同时检验多个条件，当断言失败时，我们将很难直观地判断哪个条件失败。因此，下面的断言代码应该更好一些，尽管这样显得有些多此一举：

```c++
assert(dest!=NULL);
assert(src!=NULL);
```

最后，**建议 assert 宏后面的语句应该空一行**，以形成逻辑和视觉上的一致感，让代码有一种视觉上的美感。同时为复杂的断言添加必要的注释，可澄清断言含义并减少不必要的误用。

### 避免在断言表达式中使用改变环境的语句

默认情况下，因为 assert 宏只有在 Debug 版本中才能起作用，而在 Release 版本中将被忽略。因此，在程序设计中应该避免在断言表达式中使用改变环境的语句。如下面的示例代码所示：

```c++
int Test(int i)
{
    assert(i++);
    return i;
}
int main(void)
{
    int i=1;
    printf("%d\n",Test(i));
    return 0;
}
```

对于上面的示例代码，由于“assert（i++）”语句的原因，将导致不同的编译版本产生不同的结果。如果是在 Debug 版本中，因为这里向变量 i 所赋的初始值为 1，所以在执行“assert（i++）”语句的时候将通过条件检查，进而继续执行“i++”，最后输出的结果值为 2；如果是在 Release 版本中，函数中的断言语句“assert（i++）”将被忽略掉，这样表达式“i++”将得不到执行，从而导致输出的结果值还是 1。

因此，应该避免在断言表达式中使用类似“i++”这样改变环境的语句，使用如下代码进行替换：

```c++
int Test(int i)
{    
	assert(i);    
	i++;    
	return i;
}
```

现在，无论是 Debug 版本，还是 Release 版本的输出结果都将为 2。

### 避免使用断言去检查程序错误

**在对断言的使用中，一定要遵循这样一条规定：对来自系统内部的可靠的数据使用断言，对于外部不可靠数据不能够使用断言，而应该使用错误处理代码。换句话说，断言是用来处理不应该发生的非法情况，而对于可能会发生且必须处理的情况应该使用错误处理代码，而不是断言。**

在通常情况下，系统外部的数据（如不合法的用户输入）都是不可靠的，需要做严格的检查（如某模块在收到其他模块或链路上的消息后，要对消息的合理性进行检查，此过程为正常的错误检查，不能用断言来实现）才能放行到系统内部，这相当于一个守卫。而对于系统内部的交互（如子程序调用），如果每次都去处理输入的数据，也就相当于系统没有可信的边界，这样会让代码变得臃肿复杂。事实上，在系统内部，传递给子程序预期的恰当数据应该是调用者的责任，系统内的调用者应该确保传递给子程序的数据是恰当且可以正常工作的。这样一来，就隔离了不可靠的外部环境和可靠的系统内部环境，降低复杂度。

但是在代码编写与测试阶段，代码很可能包含一些意想不到的缺陷，也许是处理外部数据的程序考虑得不够周全，也许是调用系统内部子程序的代码存在错误，造成子程序调用失败。这个时候，断言就可以发挥作用，用来确诊到底是哪部分出现了问题而导致子程序调用失败。在清理所有缺陷之后，就建立了内外有别的信用体系。等到发行版的时候，这些断言就没有存在的必要了。**因此，不能用断言来检查最终产品肯定会出现且必须处理的错误情况。**

看下面一段示例代码：

```c++
char * Strdup(const char * source)
{
    assert(source != NULL);
    char * result=NULL;
    size_t  len = strlen(source) +1;
    result = (char *)malloc(len);
    assert(result != NULL);
    strcpy(result, source);
    return  result;
}
```

对于上面的 Strdup 函数，相信大家都不陌生。其中，第一个断言语句“assert(source!=NULL)”用来检查该程序正常工作时绝对不应该发生的非法情况。换句话说，在调用代码正确的情况下传递给 source 参数的值必然不为 NULL，如果断言失败，说明调用代码中有错误，必须修改。因此，它属于断言的正常使用情况。

而第二个断言语句“assert(result!=NULL)”的用法则不同，它测试的是错误情况，是在其最终产品中肯定会出现且必须对其进行处理的错误情况。即对 malloc 函数而言，当内存不足导致内存分配失败时就会返回 NULL，因此这里不应该使用 assert 宏进行处理，而应该使用错误处理代码。如下面问题将使用 if 判断语句进行处理：

```c++
char * Strdup(const char * source)
{
    assert(source != NULL);
    char * result=NULL;
    size_t  len = strlen(source)+1;
    result = (char *)malloc(len);
    if (result != NULL)
    {
            strcpy(result, source);
    }
    return  result;
}
```

总之记住一句话：**断言是用来检查非法情况的，而不是测试和处理错误的**。因此，不要混淆非法情况与错误情况之间的区别，后者是必然存在且一定要处理的。

### 尽量在防错性程序设计中使用断言来进行错误报警

对于防错性程序设计，相信有经验的程序员并不陌生，大多数教科书也都鼓励程序员进行防错性程序设计。在程序设计过程中，总会或多或少产生一些错误，这些错误有些属于设计阶段隐藏下来的，有些则是在编码中产生的。为了避免和纠正这些错误，可在编码过程中有意识地在程序中加进一些错误检查的措施，这就是防错性程序设计的基本思想。其中，它又可以分为主动式防错程序设计和被动式防错程序设计两种。

主动式防错程序设计是指周期性地对整个程序或数据库进行搜查或在空闲时搜查异常情况。它既可以在处理输入信息期间使用，也可以在系统空闲时间或等待下一个输入时使用。如下面所列出的检查均适合主动式防错程序设计。

- 内存检查：如果在内存的某些块中存放了一些具有某种类型和范围的数据，则可对它们做经常性检查。
- 标志检查：如果系统的状态是由某些标志指示的，可对这些标志做单独检查。
- 反向检查：对于一些从一种代码翻译成另一种代码或从一种系统翻译成另一种系统的数据或变量值，可以采用反向检查，即利用反向翻译来检查原始值的翻译是否正确。
- 状态检查：对于某些具有多个操作状态的复杂系统，若用某些特定的存储值来表示这些状态，则可通过单独检查存储值来验证系统的操作状态。
- 连接检查：当使用链表结构时，可检查链表的连接情况。
- 时间检查：如果已知道完成某项计算所需的最长时间，则可用定时器来监视这个时间。
- 其他检查：程序设计人员可经常仔细地对所使用的数据结构、操作序列和定时以及程序的功能加以考虑，从中得到要进行哪些检查的启发。

被动式防错程序设计则是指必须等到某个输入之后才能进行检查，也就是达到检查点时才能对程序的某些部分进行检查。一般所要进行的检查项目如下：

- 来自外部设备的输入数据，包括范围、属性是否正确。
- 由其他程序所提供的数据是否正确。
- 数据库中的数据，包括数组、文件、结构、记录是否正确。
- 操作员的输入，包括输入的性质、顺序是否正确。
- 栈的深度是否正确。
- 数组界限是否正确。
- 表达式中是否出现零分母情况。
- 正在运行的程序版本是否是所期望的（包括最后系统重新组合的日期）。
- 通过其他程序或外部设备的输出数据是否正确。

虽然防错性程序设计被誉为有较好的编码风格，一直被业界强烈推荐。但防错性程序设计也是一把双刃剑，从调试错误的角度来看，它把原来简单的、显而易见的缺陷转变成晦涩的、难以检测的缺陷，而且诊断起来非常困难。从某种意义上讲，防错性程序设计隐瞒了程序的潜在错误。

当然，对于软件产品，希望它越健壮越好。但是调试脆弱的程序更容易帮助我们发现其问题，因为当缺陷出现的时候它就会立即表现出来。因此，在进行防错性程序设计时，如果“不可能发生”的事情的确发生了，则需要使用断言进行报警，这样，才便于程序员在内部调试阶段及时对程序问题进行处理，从而保证发布的软件产品具有良好的健壮性。

一个很常见的例子就是无处不在的 for 循环，如下面的示例代码所示：

```c++
for(i=0;i<count;i++)
{    
	/*处理代码*/
}
```

在几乎所有的 for 循环示例中，其行为都是迭代从 0 开始到“count-1”，因此，大家也都自然而然地编写成了上面这种防错性版本。但存在的问题是：如果 for 循环中的索引 i 值确实大于 count，那么极有可能意味着代码中存在着潜在的缺陷问题。

由于上面的 for 循环示例采用了防错性程序设计方式，因此，就算是在内部测试阶段中出现了这种缺陷也很难发现其问题的所在，更加不可能出现系统报警提示。同时，因为这个潜在的程序缺陷，极有可能会在以后让我们吃尽苦头，而且非常难以诊断。

那么，不采用防错性程序设计会是什么样子呢？如下面的示例代码所示：

```c++
for(i=0;i!=count;i++)
{    
	/*处理代码*/
}
```

很显然，这种写法肯定是不行的，当 for 循环中的索引 i 值确实大于 count 时，它还是不会停止循环。

对于上面的问题，断言为我们提供了一个非常简单的解决方法，如下面的示例代码所示：

```c++
for(i=0;i<count;i++)
{    
	/*处理代码*/
}
assert(i==count);
```

不难发现，通过断言真正实现了一举两得的目的：健壮的产品软件和脆弱的开发调试程序，即在该程序的交付版本中，相应的程序防错代码可以保证当程序的缺陷问题出现的时候，用户可以不受损失；而在该程序的内部调试版本中，潜在的错误仍然可以通过断言预警报告。

因此，“无论你在哪里编写防错性代码，都应该尽量确保使用断言来保护这段代码”。当然，也不必过分拘泥于此。例如，如果每次执行 for 循环时索引 i 的值只是简单地增 1，那么要使索引i的值超过 count 从而引起问题几乎是不可能的。在这种情况下，相应的断言也就没有任何存在的意义，应该从程序中删除。但是，如果索引 i 的值有其他处理情况，则必须使用断言进行预警。由此可见，在防错性程序设计中是否需要使用断言进行错误报警要视具体情况而定，在编码之前都要问自己：“在进行防错性程序设计时，程序中隐瞒错误了吗？”如果答案是肯定的，就必须在程序中加上相应的断言，以此来对这些错误进行报警。否则，就不要多此一举了。

### 用断言保证没有定义的特性或功能不被使用

在日常软件设计中，如果原先规定的一部分功能尚未实现，则应该使用断言来保证这些没有被定义的特性或功能不被使用。例如，某通信模块在设计时，准备提供“无连接”和“连接”这两种业务。但当前的版本中仅实现了“无连接”业务，且在此版本的正式发行版中，用户（上层模块）不应产生“连接”业务的请求，那么在测试时可用断言来检查用户是否使用了“连接”业务。如下面的示例代码所示：

```c++
/*无连接业务*/
#define CONNECTIONLESS 0
/*连接业务*/
#define CONNECTION     1
int MessageProcess(MESSAGE *msg)
{
    assert(msg != NULL);
    unsigned char service;
    service = GetMessageService(msg);
    /*使用断言来检查用户是否使用了“连接”业务*/
    assert(service != CONNECTION);
    /*处理代码*/
}
```

### 谨慎使用断言对程序开发环境中的假设进行检查

在程序设计中，不能够使用断言来检查程序运行时所需的软硬件环境及配置要求，它们需要由专门的处理代码进行检查处理。而断言仅可对程序开发环境（OS/Compiler/Hardware）中的假设及所配置的某版本软硬件是否具有某种功能的假设进行检查。例如，某网卡是否在系统运行环境中配置了，应由程序中正式代码来检查；而此网卡是否具有某设想的功能，则可以由断言来检查。

除此之外，对编译器提供的功能及特性的假设也可以使用断言进行检查，如下面的示例代码所示：

```c++
/*int类型占用的内存空间是否为2*/
assert(sizeof(int)== 2);
/*long类型占用的内存空间是否为4*/
assert(sizeof(long)==4);
/*byte的宽度是否为8*/
assert(CHAR_BIT==8);
```

之所以可以这样使用断言，那是因为软件最终发行的 Release 版本与编译器已没有任何直接关系。

最后，**必须保证软件的 Debug 与 Release 两个版本在实现功能上的一致性，同时可以使用调测开关来切换这两个不同的版本，以便统一维护，切记不要同时存在 Debug 版本与 Release 版本两个不同的源文件。**

当然，因为频繁调用 assert 会极大影响程序的性能，增加额外的开销。因此，应该在正式软件产品（即 Release 版本）中将断言及其他调测代码关掉（尤其是针对自定义的断言宏）。

原文链接：http://c.biancheng.net/c/assert/