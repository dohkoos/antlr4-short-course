# 使用Visitor模式计算结果

为了给前面的算术表达式语法分析器计算出结果，我们还需要做些其它的事情。

ANTLR v4鼓励我们保持语法的整洁，使用语法分析树Visitor和其它遍历器来实现语言应用。不过在接触这些之前，我们需要对语法做些修改。

首先，我们需要用标签标明规则的选项，标签可以是和规则名没有冲突的任意标志符。如果选项上没有标签，ANTLR只会为每个规则生成一个visit方法。

在本例中，我们希望为每个选项生成一个不同的visit方法，以便每种输入短语都能得到不同的事件。在新的语法中，标签出现在选项的右边缘，且以“#”符号开头：

```
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
```

接下来，让我们为运算符字面量定义一些记号名字，以便以后可以在visit方法中引用作为Java常量的它们：

```
MUL : '*' ;

DIV : '/' ;

ADD : '+' ;

SUB : '-' ;
```

现在，我们有了一个增强型的语法。接下来要做的事情是实现一个EvalVisitor类，它通过遍历表达式语法分析树计算和返回值。

执行下面的命令，让ANTLR生成Visitor接口和它的默认实现，其中-no-listener参数是告诉ANTLR不再生成Listener相关的代码：

```
antlr -no-listener -visitor Calc.g
```

所有被标签标明的选项在生成的Visitor接口中都定义了一个visit方法：

```
public interface CalcVisitor<T> extends ParseTreeVisitor<T> {
    T visitProg(CalcParser.ProgContext ctx);
    T visitPrintExpr(CalcParser.PrintExprContext ctx);
    T visitAssign(CalcParser.AssignContext ctx);
    ...
}
```

接口定义使用的是Java泛型，visit方法的返回值为参数化类型，这允许我们根据表达式计算返回值的类型去设定实现的泛型参数。因为表达式的计算结果是整型，所以我们的EvalVisitor应该继承`CalcBaseVisitor<Integer>`类。为计算语法分析树的每个节点，我们需要覆写与语句和表达式选项相关的方法。这里是全部的代码：

```
public class EvalVisitor extends CalcBaseVisitor<Integer> {
    /** "memory" for our calculator; variable/value pairs go here */
    Map<String, Integer> memory = new HashMap<String, Integer>();

    /** ID '=' expr */
    @Override
    public Integer visitAssign(CalcParser.AssignContext ctx) {
        String id = ctx.ID().getText();  // id is left-hand side of '='
        int value = visit(ctx.expr());   // compute value of expression on right
        memory.put(id, value);           // store it in our memory
        return value;
    }

    /** expr */
    @Override
    public Integer visitPrintExpr(CalcParser.PrintExprContext ctx) {
        Integer value = visit(ctx.expr()); // evaluate the expr child
        System.out.println(value);         // print the result
        return 0;                          // return dummy value
    }

    /** INT */
    @Override
    public Integer visitInt(CalcParser.IntContext ctx) {
        return Integer.valueOf(ctx.INT().getText());
    }

    /** ID */
    @Override
    public Integer visitId(CalcParser.IdContext ctx) {
        String id = ctx.ID().getText();
        if ( memory.containsKey(id) ) return memory.get(id);
        return 0;
    }

    /** expr op=('*'|'/') expr */
    @Override
    public Integer visitMulDiv(CalcParser.MulDivContext ctx) {
        int left = visit(ctx.expr(0));  // get value of left subexpression
        int right = visit(ctx.expr(1)); // get value of right subexpression
        if ( ctx.op.getType() == CalcParser.MUL ) return left * right;
        return left / right; // must be DIV
    }

    /** expr op=('+'|'-') expr */
    @Override
    public Integer visitAddSub(CalcParser.AddSubContext ctx) {
        int left = visit(ctx.expr(0));  // get value of left subexpression
        int right = visit(ctx.expr(1)); // get value of right subexpression
        if ( ctx.op.getType() == CalcParser.ADD ) return left + right;
        return left - right; // must be SUB
    }

    /** '(' expr ')' */
    @Override
    public Integer visitParens(CalcParser.ParensContext ctx) {
        return visit(ctx.expr()); // return child expr's value
    }
}
```

以前开发和测试语法都是使用的TestRig，这次我们试着编写计算器的主程序来启动代码：

```
public class Calc {

    public static void main(String[] args) throws Exception {
        InputStream is = args.length > 0 ? new FileInputStream(args[0]) : System.in;

        ANTLRInputStream input = new ANTLRInputStream(is);
        CalcLexer lexer = new CalcLexer(input);
        CommonTokenStream tokens = new CommonTokenStream(lexer);
        CalcParser parser = new CalcParser(tokens);
        ParseTree tree = parser.prog();

        EvalVisitor eval = new EvalVisitor();
        // 开始遍历语法分析树
        eval.visit(tree);

        System.out.println(tree.toStringTree(parser));
    }
}
```

创建一个运行主程序的脚本：

```
#!/bin/sh
java -cp .:./antlr-4.5.1-complete.jar:$CLASSPATH $*
```

把它保存为run.sh后，执行以下命令：

```
compile *.java
run Calc calc.txt
```

然后你就会看到文本形式的语法分析树以及计算结果：

```
193
17
9
(prog (stat (expr 193)) (stat a = (expr 5)) (stat b = (expr 6))
 (stat (expr (expr a) + (expr (expr b) * (expr 2)))) (stat (expr
 (expr ( (expr (expr 1) + (expr 2)) )) * (expr 3))))
```
