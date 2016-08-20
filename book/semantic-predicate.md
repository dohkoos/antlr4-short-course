# 使用语义谓词改变语法分析

有一个读入整数序列的语法，它的玄机是由输入的部分指定有多少个整数组合在一起，所以我们必须等到运行时才能知道有多少整数被匹配。这里是示例输入文件idata.txt的内容：

```
2 9 10 3 1 2 3
```

第1个数字表示匹配后续两个数字9和10；紧跟10的数字3表示匹配接下来的三个数字。我们的目的是设计一个语法IData.g，把9和10组合在一起，把1、2和3组合在一起。在语法上执行以下命令后显示的语法分析树能够清楚地标识出整数的分组，就像下图显示的那样：

```
antlr -no-listener IData.g
compile *.java
grun IData file -gui idata.txt
```

![](http://codemany.com/uploads/idata-parse-tree.png)

要达成这个目标，以下语法中的关键是一个被称为语义谓词的布尔值操作：{$i <= $n}?。当谓词计算结果为true时，语法分析器匹配整数直到超过序列规则参数n要求的数量；当计算结果为false时，谓词让相关的选项从生成的语法分析器中“消失”。
在这个案例中，值为false的谓词让(...)*循环从规则序列里终止并返回。

```
grammar IData;

file : group+ ;

group: INT sequence[$INT.int] ;

sequence[int n]
locals [int i = 1;]
     : ( {$i<=$n}? INT {$i++;} )*    // match n integers
     ;

INT  : [0-9]+ ;  // match integers
WS   : [ \t\n\r]+ -> skip ;    // toss out all whitespace
```

被语法分析器使用的规则序列的内部语法表示看起来就像下图这样：

![](http://codemany.com/uploads/idata-rule-sequence.png)

虚线表明谓词可以剪断那条路径，只给语法分析器留下一个选择：退出的路径。

虽然大部分时间我们不需要这样的微管理，但它至少让我们知道我们有这样的武器可以处理病理分析问题。
