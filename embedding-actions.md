# 在语法中嵌入任意的操作

如果我们不想付出构建语法分析树的开销，或者想要在分析期间动态地计算值或把东西打印出来，那么可以通过在语法中嵌入任意代码实现。它的比较困难的，因为我们必须明白在语法分析器上的操作的影响，以及在哪里放置这些操作。

为了解释嵌入在语法中的操作，让我们先来看下文件rows.txt中的数据：

```
parrt   Terence Parr    101
tombu   Tom Burns       020
bke     Kevin Edgar     008
```

这些列是由TAB分隔的，每一行用一个换行结束。匹配这种类型的输入在语法上还是相当简单的。下面是此语法文件Rows.g的内容：

```
file : (row NL)+ ;    // NL is newline token: '\r'? '\n'
row  : STUFF+ ;
```

我们需要创建一个构造器以便我们能传递我们想要的列号（从1开始计数），所以我们需要在规则中添加一些操作来做这些事情：

```
grammar Rows;

@parser::members {    // add members to generated RowsParser
    int col;
    public RowsParser(TokenStream input, int col) {    // custom constructor
        this(input);
        this.col = col;
    }
}

file: (row NL)+ ;

row
locals [int i=0]
    : ( STUFF
        {
        $i++;
        if ( $i == col ) System.out.println($STUFF.text);
        }
      )+
    ;

TAB  :  '\t' -> skip ;    // match but don't pass to the parser
NL   :  '\r'? '\n' ;      // match and pass to the parser
STUFF:  ~[\t\r\n]+ ;      // match any chars except tab, newline
```

在上述语法中，操作是被花括号括起来的代码片段；members操作的代码将会被注入到生成的语法分析器类中的成员区；在规则row中的操作访问的$i是由locals子句定义的局部变量，该操作也用$STUFF.text获取最近匹配的STUFF记号的文本内容。STUFF词法规则匹配任何非TAB或换行的字符，这意味着在列中可以有空格字符。

现在，是时候去思考如何使用定制的构造器传递一个列号给语法分析器，并且告诉语法分析器不要构建语法分析树了：

```
public class Rows {

    public static void main(String[] args) throws Exception {
        ANTLRInputStream input = new ANTLRInputStream(System.in);
        RowsLexer lexer = new RowsLexer(input);
        CommonTokenStream tokens = new CommonTokenStream(lexer);
        int col = Integer.valueOf(args[0]);
        RowsParser parser = new RowsParser(tokens, col);    // pass column number!
        parser.setBuildParseTree(false);    // don't waste time bulding a tree
        parser.file();
    }
}
```

现在，让我们核实下我们的语法分析器能否正确匹配一些示例输入：

```
antlr -no-listener Rows.g  # don't need the listener
compile *.java
run Rows 1 < rows.txt
```

这时你会看到rows.txt文件的第1列内容被输出：

```
parrt
tombu
bke
```

如果将上面命令中的1换成2，你会看到rows.txt文件的第2列内容被输出；如果换成3，那么rows.txt文件的第3列内容将会被输出。
