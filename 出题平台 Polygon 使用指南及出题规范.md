# 出题平台 Polygon 使用指南及出题规范

## 平台使用

本节部分参考自[codeforces的polygon平台使用指北](https://blog.csdn.net/jokerwyt/article/details/100145318)。Polygon 上使用 latex 语法，其本质是可视化 latex 编辑器和编译器。因而在使用 Polygon 平台需要掌握一定的 latex 使用能力。

### 题目内部

一道题目通常由题面（statement）、checker、validator、std、数据和 interactor（如果是交互题你需要写这一部分）所构成。下面将重点介绍 checker、validator 和 interactor 的作用和实现。下面的这些文件需要使用 ```Testlib.h``` 库，因而首先简要介绍如何使用 ```Testlib.h```。

#### Testlib.h

```Testlib.h``` 可以实现（或经常使用）的功能有：

1.   产生数据。
2.   从各个文件中输入，输出评测结果和其他信息。

在 ```Testlib.h``` 中产生在一定范围内的数据通常使用 ```rnd.next(l, r)``` 函数，其中 ```l,r``` 类型相同，为所选上下界。你可以通过 ```rnd.setSeed()``` 函数来给随机数生成器选定种子。**注意：使用了 ```Testlib.h``` 后禁止使用 ```rand``` 和 ```mt19937``` 等随机数产生器**。 

```Testlib.h``` 将输入和输出均包装成了一个类 ```Instream``` 和 ```Outstream```。下文中以 ```inf``` 作为该类的实例化，输入中常使用到的函数有：

```cpp
	int x = inf.readInt(l, r, "variable_name"); // 读入一个范围在 [l, r] 的整型变量 variable_name，并存放在 x 中。
	double x = inf.readDouble(l, r, "variable_name"); // 读入一个范围在 [l, r] 的变量 variable_name，并存放在 x 中。
	char c = inf.readChar(c); // 读入一个字符，c 可缺失，如果指定则必须读入一个 c，将结果保存进入变量 c 中。
	string s = inf.readLine("regex_pattern", "variable_name"); // 读入一个符合正则表达式 regex_pattern 的字符串变量 variable_name 并存放在 s 中。该函数同 readString
	// 不推荐使用 inf.readLines 和 inf.readStrings。
	inf.readSpace(); // 读入一个空格。
	inf.readEoln(); // 读入一个回车。
	inf.readEof(); // 文件结尾时需要读入一个 Eof 标志。
```

注意，对于上文中读入不合法的情况会返回 $3$（```_pe```），对于输入文件会提示题面错误，在输出中会返回 ```_pe```。

对于输出，通常使用 ```quitf``` 函数返回给评测机结果并进行一些输出，其他输出通常不会有任何地方给予提示。

```cpp
	quitf(res, "output strings", argv);
```

其中，输出结果```res``` 有列举变量 ```  _ok, _wa, _pe, _fail, _dirt, _points, _unexpected_eof, _partially```，但我们通常仅使用前三个。后面的 ```output strings``` 格式同 ```printf```，再次不再赘述。

#### checker

checker 是判断一个人的输出是不是正确的**唯一判据**。由于通常来说一道题通常仅有唯一答案，且为一个或若干个简单字符串所构成，仅需要比较选手输出和 std 输出的结果是否完全相同，因而 Polygon 平台内置了若干个 checker 的模板可以直接使用，点击 checker 页面中的 select 选项中列举了若干个 checker，不需要单独另写。

如果答案有多种，不一定和 std 输出相同，那么就需要单独写 checker。首先需要一个基本的头：

```cpp
#include "testlib.h"
#include <bits/stdc++.h>
using namespace std;
int main(int argc, char * argv[])
{
    // setName("compare two signed huge integers"); 可选，标注当前 checker 的工作内容
    registerTestlibCmd(argc, argv);
}
```

通过点击 checker 板块中的 ```Newfile``` 你就可以写一个 checker 然后再后面的 ```Select``` 中使用到你所写的文件。后文新增的文件同理。

接下来你可以从三个地方进行读入：```inf```、```ansf```、```ouf```，这三个均为一个读入类，分别从输入数据、std 产生的答案文件、选手文件进行读入，读入规则由 ```1.1.1``` 节进行了详细阐述。你最后只需要根据这三个文件的读入，给予选手的输出一个结果，通过 ```quitf``` 进行输出即可。

#### validator

validator 是判断所有的数据是否合法的程序。该程序是必须提供并且自己写的。同 checker，你也需要写一个头：

```cpp
#include <bits/stdc++.h>
#include "testlib.h"
using namespace std;
int main(int argc, char* argv[])
{
    registerValidation(argc, argv);
}
```

validator 的要求相当的严格！它精确控制到每一个输入文件的字符，因而在编写 validator 的时候要考虑每一个可能的字符并将其读入。

下面是一个示例：

>   第一行输入两个整数 $n,k$，第二行输入 $n$ 个整数 $\{a_n\}$。$1 \le n \le 10^5$，$1 \le k,a_i \le 10^9$。

那么你的输入文件是这样的：

```
整数 空格 整数 回车
整数 空格 整数 空格 ... 整数 回车
文件结束符
```

因而它对应的 validator 可以如下所示：

```cpp
#include <bits/stdc++.h>
#include "testlib.h"
using namespace std;
const int N = 1e5, K = 1e9;
int main(int argc, char* argv[])
{
    registerValidation(argc, argv);
    int n = inf.readInt(1, N, "n");
    inf.readSpace();
    int k = inf.readInt(1, K, "k");
    inf.readEoln();
    for (int i = 1; i <= n; i++)
    {
        int x = inf.readInt(1, K, "a_i");
        if (i != n)
            inf.readEoln();
        else
            inf.readSpace();
    }
    inf.readEof();
    return 0;
}
```

你可以通过生成题面（Preview）并编写几个题面样例来检查你的 validator 是否正确。

#### interactor

该程序是用于和选手进行交互的程序。同 checker，你也需要写一个头：

```cpp
#include "testlib.h"
#include <bits/stdc++.h>
using namespace std;
int main(int argc, char **argv)
{
    registerInteraction(argc, argv);
    cout.flush(); // to make sure output doesn't stuck in some buffer
    return 0;
}
```

interactor 规则与 checker 别无二致，其评测结果返回也是通过 ```quitf``` 函数的调用实现。读入选手内容通过 ```inf``` 实现。若要输出到选手的 Buffer，使用 ``cout``，若是输出到评测机的 Buffer，使用 ```tout```。

#### gernerator

该程序是用于生成数据的。通常来说，除了手造小数据，其他的数据都建议使用 generator 生成，因为过大的数据无法上传平台，且修改性很差。

同上文，该程序也需要你写一个头：

```cpp
#include <bits/stdc++.h>
#include "testlib.h"
using namespace std;
int main(int argc, char *argv[])
{
    registerGen(argc, argv, 1);
}
```

你可以在上边栏中的 Files 选项的 Source Files 中 Add Files 添加你的 generator。

除了如 ```1.1.1``` 节中所说的只能使用 ```rnd.next()``` 函数产生随机数外，没有任何禁忌。但是你需要保证你输出的格式能过通过 validator 的检查。你可以通过 custom invocation 中评测的结果是 Fail 来观察你生成的数据是不是格式错误的。

注意到此处的 ```main``` 函数是有参数的，通常是依靠 ```argv``` 参数来决定当前数据的规模。

写完了 generator 之后，你需要在 Tests 界面编写脚本以产生数据。其规则较为简单，下文为一示例：

```
<#list 1..5 as testNumber>
    gen 100 1 ${testNumber} > $
    gen 100 2 ${testNumber} > $
    gen 100 3 ${testNumber} > $
    gen 100 4 ${testNumber} > $
    gen 100 5 ${testNumber} > $
</#list>
<#list 1..5 as testNumber>
    gen 10 1 ${testNumber} > $
    gen 10 2 ${testNumber} > $
    gen 10 3 ${testNumber} > $
    gen 10 4 ${testNumber} > $
    gen 10 5 ${testNumber} > $
</#list>
```

```gen``` 为 generator 文件名，后面带着的三个为 main 函数的参数，最后固定输出一个 ```> $```。使用 ```list``` 环境产生循环，可以多重循环嵌套。使用 ```${}``` 环境表示对值的引用。

#### 题面

题面并不是 markdown 而是 latex，但是大体使用上与 markdown 没有太大区别。仅需要注意以下几个内容：

1.   行间公式块请使用 ```\begin{equation} \end{equation}``` 环境而非两个 ```$```。

2.   行间代码块请使用 ```\begin{lstlisting} \end{lstlisting}``` 环境。注意，latex 并不自带代码块环境，如果你需要使用代码块，请在上边栏中的 Files 中加入以下代码，并按 ```1.3``` 节中的提示进行：

     ```latex
     \usepackage{verbatim}
     \usepackage{color}
     \usepackage{listings}
     \lstset{
         basicstyle=\tt,
     }
     ```

同时注意，**哪怕是写中文题面，题面语言也请选择英语**，否则将会不正常输出。题面的具体格式在第二章进行详细讨论。

#### Solution

Polyon 平台允许上传多个 Solution（可以有错误）。在 Polygon 中，比对的答案都是通过 main correct solution 产生的。你可以通过选择当前的 Solution 是 Correct、Incorrect（允许任何错误）、Correct or Time Limit Exceed（暴力，仅允许 TLE 但不允许 WA）等来进行方便的对拍。对拍操作将在 ```1.2.1``` 节阐述。

### 题目组装

在配置完以上题目各个组成部分后，现在就可以将这些文件组装成一个完整的题目。Polygon 平台本质是一个 git，因而每次修改都需要进行 Commit 以进行同步，但不支持 branch。对于未同步内容，请确认清楚具体哪个版本才是当前需要的版本，然后再进行覆写。

注意上文中每个 files 的编写都是仅针对于当前题目的，当你切换一道题之后这些 files 就不再起作用了。同理，你可以修改当前 files 中的 ```olymp.sty``` 文件以实现仅调整当前题目中风格的目的。

下面是对于一道完整题目的一些操作：

#### 自测（Invocations）

该功能允许对所有或部分 solution 进行实测。如果发现对拍出错，可以通过在 Tests 页面中 preview tests 的功能选择对应的数据进行下载到本地调试。

#### Commit 和打包（Package）

当你完成一道题的修改操作时，你需要通过 Commit 以上传平台让其他人共享你当前的工作进度。

**注意：如果没有特殊需求，请在每次 commit 时选择 minor changes，否则平台将会对每次 commit 自动给别人发送邮件。**

只有当你本地没有额外修改时才允许 package 操作，将当前的题目文件（题面、数据、各种 Source File 等）组装成一个完整的 Package。组装完成后你就可以下载到本地然后进行后续的操作。通常来说使用 standard package 就足够了。

#### Verification

该操作通常是在进行 package 操作时一并进行的，主要内容是进行数据的检查、各个 Solution 的检查。如果出现问题则会拒绝打包。

### 比赛组装

当你造好每一道题，你就可以尝试将其组装成为一个完整的比赛了。

整个比赛中同样有 Properties/Files，它描述了整个比赛中每个题 latex 源文件的大 style。由于 latex 并不自带代码块，因而如果**某个题**中使用了代码块，你就需要在整个比赛的 Properties/Files 中的 ```olymp.sty``` 中加入下面描述代码块风格的代码：

```latex
\usepackage{verbatim}

\usepackage{listings}
\usepackage{xcolor}
\lstset{
    language=C++,
    basicstyle=\tt,
    breaklines,                                 % 自动将长的代码行换行排版
    extendedchars=false,                        % 解决代码跨页时，章节标题，页眉等汉字不显示的问题
    backgroundcolor=\color[rgb]{0.96,0.96,0.96},% 背景颜色
    keywordstyle=\color{blue}\bfseries,         % 关键字颜色
    identifierstyle=\color{black},              % 普通标识符颜色
    commentstyle=\color[rgb]{0,0.6,0},          % 注释颜色
    stringstyle=\color[rgb]{0.58,0,0.82},       % 字符串颜色
    showstringspaces=false,                     % 不显示字符串内的空格
    numberstyle=\tiny,                          % 设置数字字体
    captionpos=t,                               % title在上方(在bottom即为b)
    frame=single,                               % 设置代码框形式
    rulecolor=\color[rgb]{0.8,0.8,0.8},         % 设置代码框颜色
}
```

你可以直接在比赛界面 ``New problem`` 以创建一道题目，或者将已经有的题目拉到当前的比赛中来。

通过右上角的 preview 功能你可以预览当前比赛（HTML 格式或 PDF）。如果 PDF 报错，大概率是整个比赛的 style 文件配置错误，请根据错误提示自行调整该文件。

如果要上传 Codeforces Gym 平台，请邀请 codeforces 并给予只读权限，否则将无法读取题面的 descriptor。你只需要在 Codeforces Gym 平台中使用 coach mode 然后 ```create new training```，在 dashboard 的下方选择 ```New problems from contest```，输入 Polygon Contest ID（位于 Polygon 平台比赛界面右上角）即可导入。

如果是下载到本地，（下载到本地进行配置部分待补）

## 出题规范

一场好的比赛需要满足以下几个要求：

从题目本身上看：

1.   赛中无公开 clarification 更新题面错误
2.   赛中无重测
3.   赛后无明显 hack

从题目难度上看：

1.   每队都能过题
2.   每题都有人过
3.   没人能过所有题

本节规范部分参考自[洛谷主题库规范](https://zhuanlan.zhihu.com/p/556417143)，[格式手册](https://oi-wiki.org/intro/format/)。请特别注意 [公式要求](https://oi-wiki.org/intro/format/#latex-%E5%85%AC%E5%BC%8F%E7%9A%84%E6%A0%BC%E5%BC%8F%E8%A6%81%E6%B1%82) 部分。

约定：使用 ```$``` 环绕的称为行内公式环境，使用 ` 符号环绕的称为行内代码块，使用 ```\begin{equation} \end{equation}``` 环绕的是行间公式块。下文中举例文本均使用 ```""``` 环绕。

### 公式类

#### 数字类

##### 使用公式体的场合

需要以公式体呈现的数字内容：可以使用字母替换的变量、数学公式。

应当使用公式环境的数字举例：“第 $1$ 个盒子中取出 $5$ 个球”中的 ```1``` 与 ```5```。

不应当使用公式环境的数字举例：“2023-2024 赛季”中的 ```2023-2024```。

##### 数字规范

1.   对于整数，不超过 $1000$ 的数字直接写出。
2.   对于大于 $1000$ 的整数，如果后面 $0$ 的数目较多，请使用科学计数法。此外，如果该数字以 $1$ 开头，请省略 $1\times $。如 $2\times 10^5$，$10^9$，$2.5\times 10^3$。建议不要使用 $2.5e3$ 等写法。
3.   如果没有什么 $0$，请每隔三位数字打一个空格。如 $998\ 244\ 353$。
4.   对于小数，使用科学计数法。 

#### 字母类

输出的特定字符串请用行内代码块，如

>   如果无解，请输出 ```-1```。
>
>   如果答案为无穷大，请输出 ```inf```。

禁止以公式体呈现的内容：非函数的字母组合或单词，如算法名称、人名、地名或其他专有名词。

应当使用公式环境呈现的字母情况：“语法元素 $S$ 的 $\rm First$ 集合为 ${\rm First}(S)=\{\cdots\}$。”中的 ${\rm First}$，因为它是数学公式、函数或符号。

禁止使用公式环境呈现的字母情况：“我的第一次 $Accepted$”。中的 Accepted。因为它是单词，不是数学符号。

#### 公式

##### 公式块本身

使用行内公式时，请勿使用 ```\dfrac```、```\displaystyle``` 等破坏行高的函数。如果真的想放大，请使用行间公式块。

正确：“请输出 $\sum_{d|n} \varphi(d)$。”

错误：“考虑 $\displaystyle \sum_{i=1}^n \binom{n}{i}$”。

##### 括号

当公式中有破坏行高的内容（如 ```\sum```，```\prod```，```\frac```，```\binom``` 等大型运算符），请使用 ```\left,\right``` 包络括号。

示例：
$$
\left(\sum_{i=1}^n \left \lfloor \dfrac{ai+b}{c}\right \rfloor\right)\left(\sum_{i=1}^n \left \lceil \dfrac{ai+b}{c}\right \rceil\right)
$$
对应的公式为 ```\left(\sum_{i=1}^n \left \lfloor \dfrac{ai+b}{c}\right \rfloor\right)\left(\sum_{i=1}^n \left \lceil \dfrac{ai+b}{c}\right \rceil\right)```。

### 字体

公式环境中默认为罗马斜体：$abcdefghijklmnopqrstuvwxyz$。

示例：“$a+b-c$”，“$f(x)$”。

对于有名字的规范函数和特定常数（如自然常数 $\rm e$），请使用罗马正体：$\rm abcdefghijklmnopqrstuvwxyz$。

示例：$\log,\ln,\min,\max,\gcd,{\operatorname{argmin}},{\operatorname{First}}$：```\log,\ln,\min,\max,\gcd,{\operatorname{argmin}},{\operatorname{First}}```。如果环境中无此函数，请使用 ```\operatorname{}``` 来呈现。

对于特定的域，请使用反斜杠或黑板体（```\mathbb```）。

示例：$\Q,\R,{\mathbb F},\Z$：```\Q,\R,{\mathbb F},\Z```。

对于特定的变换，请使用公式花体（```\mathscr```）。

示例：$\mathscr{L,F}$：```\mathscr{L,F}```。

### 空格

在书写中文题面时，行内公式块、行内代码块与汉字之间均需使用一个空格分隔，而全角标点符号不用。特别的，使用汉语文本时，禁止使用半角标点符号。

举例：“Walk_alone 想要你计算 $\sum_{d|n} \varphi(d)$。如果 $n$ 为负数，请输出 ```-1```。”。注意本例中空格的情况，下面为源代码：

```markdown
Walk_alone 想要你计算 $\sum_{d|n} \varphi(d)$。如果 $n$ 为负数，请输出 ```-1```。
```

### 其他特殊环境

在 latex 中还有若干的特殊环境如 ```enumerate,tabular,itemize``` 等，可以创建有序表、无序表等。由于 Polygon 平台支持的 latex 语法仅为全部 latex 语法的子集，因而具体环境请根据其[官方文档](https://polygon.codeforces.com/docs/statements-tex-manual)具体分析。

## 审计工作

（待补）
