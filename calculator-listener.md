# 使用Listener模式计算结果

在上一节中的计算器是以解释的方式执行的，现在我们想要把它转换成以编译的方式执行。编译执行和解释执行相比，需要依赖于特定的目标机器。在这里我们假设有一台这样的机器，它用堆栈进行运算，支持如下表所示的几种指令：

指令 | 说明          | 操作数个数 | 用途
---- | ------------- | ---------- | ----------------------------
LDV  | Load Variable | 1          | 变量入栈
LDC  | Load Constant | 1          | 常量入栈
STR  | Store Value   | 1          | 栈顶一个元素存入指定变量
ADD  | Add           | 0          | 栈顶两个元素出栈，求和后入栈
SUB  | Subtract      | 0          | 栈顶两个元素出栈，求差后入栈
MUL  | Multiply      | 0          | 栈顶两个元素出栈，求积后入栈
DIV  | Divide        | 0          | 栈顶两个元素出栈，求商后入栈
RET  | Return        | 0          | 栈顶一个元素出栈，计算结束

做这个最简单的方法是使用ANTLR的语法分析树Listener机制实现DirectiveListener类，然后它通过监听来自树遍历器触发的事件，输出对应的机器指令。

Listener机制的优势是我们不必要自己去做任何树遍历，甚至我们不必要知道遍历语法分析树的运行时如何调用我们的方法，我们只要知道我们的DirectiveListener类得到通知，在与语法规则匹配的短语开始和结束时。这种方法减少了我们学习ANTLR必须要花费的时间，让我们回到我们所熟悉的编程语言领域。

这里不需要创建新的语法规则，还是继续沿用前文Calc.g所包含的语法，标签也要保留：

```
grammar Calc;

prog
    : stat+
    ;

stat
    : expr                   # printExpr
    | ID '=' expr            # assign
    ;

expr
    : expr op=(MUL|DIV) expr # MulDiv
    | expr op=(ADD|SUB) expr # AddSub
    | INT                    # int
    | ID                     # id
    | '(' expr ')'           # parens
    ;

MUL : '*' ;

DIV : '/' ;

ADD : '+' ;

SUB : '-' ;

ID  : [a-zA-Z]+ ;

INT : [0-9]+ ;

WS  : [ \t\r\n]+ -> skip ;    // toss out whitespace
```

然后，我们可以运行ANTLR工具：

```
antlr Calc.g
```

它会生成后缀名为tokens和java的六个文件：

```
Calc.tokens         CaclLexer.java          CalcParser.java
CalcLexer.tokens    CalcBaseListener.java   CalcListener.java
```

正如这里我们看到的，ANTLR会为我们自动生成Listener基础设施。其中CalcListener是语法和Listener对象之间的关键接口，描述我们可以实现的回调方法：

```
public interface CalcListener extends ParseTreeListener {
	void enterProg(CalcParser.ProgContext ctx);
	void exitProg(CalcParser.ProgContext ctx);
	void enterPrintExpr(CalcParser.PrintExprContext ctx);
    ...
}
```

CalcBaseListener则是ANTLR生成的一组空的默认实现。ANTLR内建的树遍历器会去触发在Listener中像enterProg()和exitProg()这样的一串回调方法，如同它对语法分析树执行了一次深度优先遍历。为响应树遍历器触发的事件，我们的DirectiveListener需要继承CalcBaseListener并实现一些方法。我们不需要实现全部的接口方法，我们也不需要去覆写每个enter和exit方法，我们只需要去覆写那些我们感兴趣的回调方法。

在本例中，我们需要通过覆写6个方法对6个事件——当树遍历器exit那些有标签的选项时触发——作出响应。我们的基本策略是当这些事件发生时打印出已转换的指令。以下是完整的实现代码：

```
public class DirectiveListener extends CalcBaseListener {

    @Override
    public void exitPrintExpr(CalcParser.PrintExprContext ctx) {
        System.out.println("RET\n");
    }

    @Override
    public void exitAssign(CalcParser.AssignContext ctx) {
        String id = ctx.ID().getText();
        System.out.println("STR " + id);
    }

    @Override
    public void exitMulDiv(CalcParser.MulDivContext ctx) {
        if (ctx.op.getType() == CalcParser.MUL) {
            System.out.println("MUL");
        } else {
            System.out.println("DIV");
        }
    }

    @Override
    public void exitAddSub(CalcParser.AddSubContext ctx) {
        if (ctx.op.getType() == CalcParser.ADD) {
            System.out.println("ADD");
        } else {
            System.out.println("SUB");
        }
    }

    @Override
    public void exitId(CalcParser.IdContext ctx) {
        System.out.println("LDV " + ctx.ID().getText());
    }

    @Override
    public void exitInt(CalcParser.IntContext ctx) {
        System.out.println("LDC " + ctx.INT().getText());
    }
}
```

为了让它运行起来，余下我们唯一需要做的事是创建一个主程序去调用它：

```
public class Calc {

    public static void main(String[] args) throws Exception {
        InputStream is = args.length > 0 ? new FileInputStream(args[0]) : System.in;

        ANTLRInputStream input = new ANTLRInputStream(is);
        CalcLexer lexer = new CalcLexer(input);
        CommonTokenStream tokens = new CommonTokenStream(lexer);
        CalcParser parser = new CalcParser(tokens);
        ParseTree tree = parser.prog();

        ParseTreeWalker walker = new ParseTreeWalker();
        walker.walk(new DirectiveListener(), tree);

        // print LISP-style tree
        System.out.println(tree.toStringTree(parser));
    }
}
```

这个程序和前文Calc.java中的代码极度相似，区别只在12-13行。这两行代码负责创建树遍历器，然后让树遍历器去遍历那颗从语法分析器返回的语法分析树，当树遍历器遍历时，它就会触发调用到我们的DirectiveListener中实现的方法。此外，通过传入一个不同的Listener实现我们能简单地生成完全不同的输出。Listener机制有效地隔离了语法和语言应用，使语法可以被其它应用再次使用。

现在一切完备，让我们尝试着去编译和运行它吧！下面是完整的命令序列：

```
compile *.java
run Calc calc.txt
```

编译的输出结果如下所示：

```
LDC 19
RET

LDC 5
STR a
LDC 6
STR b
LDV a
LDV b
LDC 2
MUL
ADD
RET

LDC 1
LDC 2
ADD
LDC 3
MUL
RET
```
