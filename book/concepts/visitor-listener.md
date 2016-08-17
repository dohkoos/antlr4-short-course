# Visitor和Listener

ANTLR在它的运行库中为两种树遍历机制提供支持。默认情况下，ANTLR生成一个语法分析树Listener接口，在其中定义了回调方法，用于响应被内建的树遍历器触发的事件。

在Listener和Visitor机制之间最大的不同是：Listener方法被ANTLR提供的遍历器对象调用；而Visitor方法必须显式的调用visit方法遍历它们的子节点，在一个节点的子节点上如果忘记调用visit方法就意味着那些子树没有得到访问。

让我们首先从Listener开始。在我们了解Listener之后，我们也将看到ANTLR如何生成遵循Visitor设计模式的树遍历器。

### 语法分析树Listener

在Calc.java中有这样两行代码：

```
ParseTreeWalker walker = new ParseTreeWalker();
walker.walk(new DirectiveListener(), tree);
```

类ParseTreeWalker是ANTLR运行时提供的用于遍历语法分析树和触发Listener中回调方法的树遍历器。ANTLR工具根据Calc.g中的语法自动生成ParseTreeListener接口的子接口CalcListener和默认实现CalcBaseListener，其中含有针对语法中每个规则的enter和exit方法。DirectiveListener是我们编写的继承自CalcBaseListener的包含特定应用代码的实现，把它传递给树遍历器后，树遍历器在遍历语法分析树时就会触发DirectiveListener中的回调方法。

![](http://codemany.com/uploads/calc-listener-hierachy.png)

下图左边的语法分析树显示ParseTreeWalker执行了一次深度优先遍历，由粗虚线表示，箭头方向代表遍历方向。右边显示的是语法分析树的完整调用序列，它们由ParseTreeWalker触发调用。当树遍历器遇到规则assign的节点时，它触发enterAssign()并且给它传递AssignContext语法分析树节点。在树遍历器访问完assign节点的所有子节点后，它触发exitAssign()。

![](http://codemany.com/uploads/listener-call-sequence.png)

Listener机制的强大之处在于所有都是自动的。我们不必要写语法分析树遍历器，而且我们的Listener方法也不必要显式地访问它们的子节点。

### 语法分析树Visitor

有些情况下，我们实际想要控制的是遍历本身，在那里我们可以显式地调用visit方法去访问子树节点。选项-visitor告诉ANTLR工具从相应语法生成Visitor接口和默认实现，其中含有针对语法中每个规则的visit方法。

下图是我们熟悉的Visitor模式操作在语法分析树上。左边部分的粗虚线表示语法分析树的深度优先遍历，箭头方向代表遍历方向。右边部分指明Visitor中的方法调用序列。

![](http://codemany.com/uploads/visitor-call-sequence.png)

下面是Calc.java中的两行代码：

```
EvalVisitor eval = new EvalVisitor();
// To start walking the parse tree
eval.visit(tree);
```

我们首先初始化自制的树遍历器EvalVisitor，然后调用visit()去访问整棵语法分析树。ANTLR运行时提供的Visitor支持代码会在看到根节点时调用visitProg()。在那里，visitProg()会把子树作为参数调用visit方法继续遍历，如此等等。

![](http://codemany.com/uploads/calc-visitor-hierachy.png)

ANTLR自动生成的Visitor接口和默认实现可以让我们为Visitor方法编写自己的实现，让我们避免必须覆写接口中的每个方法，让我们仅仅聚焦在我们感兴趣的方法上。这种方法减少了我们学习ANTLR必须要花费的时间，让我们回到我们所熟悉的编程语言领域。
