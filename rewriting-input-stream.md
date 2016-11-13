# 重写输入流

现在准备要构建一个工具，用来把前面idata.txt里的数据按group分行显示，就像这样：

```
2 9 10
3 1 2 3
```

我们可以借助语法分析树的Listener机制来对词法分析结束后生成的记号流进行改写，我们不需要实现每一个Listener接口方法，只需要在捕获到group的时候把换行符插到它末尾就行。实现改写的代码如下所示：

```
import org.antlr.v4.runtime.TokenStream;
import org.antlr.v4.runtime.TokenStreamRewriter;

public class RewriteListener extends IDataBaseListener {
    TokenStreamRewriter rewriter;

    public RewriteListener(TokenStream tokens) {
        rewriter = new TokenStreamRewriter(tokens);
    }

    @Override
    public void enterGroup(IDataParser.GroupContext ctx) {
        rewriter.insertAfter(ctx.stop, '\n');
    }
}
```

接着就是写一个小程序来调用我们上面的改写类：

```
import org.antlr.v4.runtime.*;
import org.antlr.v4.runtime.tree.*;
import java.io.FileInputStream;
import java.io.InputStream;

public class IData {

    public static void main(String[] args) throws Exception {
        InputStream is = args.length > 0 ? new FileInputStream(args[0]) : System.in;

        ANTLRInputStream input = new ANTLRInputStream(is);
        IDataLexer lexer = new IDataLexer(input);
        CommonTokenStream tokens = new CommonTokenStream(lexer);
        IDataParser parser = new IDataParser(tokens);
        ParseTree tree = parser.file();

        RewriteListener listener = new RewriteListener(tokens);

        System.out.println("Before Rewriting");
        System.out.println(listener.rewriter.getText());

        ParseTreeWalker walker = new ParseTreeWalker();
        walker.walk(listener, tree);

        System.out.println("After Rewriting");
        System.out.println(listener.rewriter.getText());
    }
}
```

这里的关键是TokenStreamRewriter对象知道如何在不修改流的情况下提供一个记号流的修改过的视图。它把所有的操作方法当作指令并把它们排进队列，等到在遍历记号流把它作为文本渲染回去的时候延迟执行。每次我们调用getText()时rewriter就会执行那些指令。

最后就是构建和测试应用：

```
antlr IData.g
compile *.java
run IData idata.txt
```

以下是输出结果：

```
Before Rewriting
29103123
After Rewriting
2910
3123
```

仅用几行代码，我们就能够没有任何烦恼地对某些内容做轻微的调整。这种策略对于源代码检测或重构这类一般性的问题是非常有效的。TokenStreamRewriter是一个非常强大且有效的操作记号流的方法。
