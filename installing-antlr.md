# 安装ANTLR

ANTLR是由Java写成的，所以在安装ANTLR前必须保证已经安装有Java 1.6或以上版本。你可以到[这里](http://www.antlr.org/download.html)下载ANTLR的最新版本，或者也可使用命令行工具下载：

```
$ curl -O https://www.antlr.org/download/antlr-4.7.1-complete.jar
```

归档文件包含运行ANTLR工具的所有必要依赖，以及编译和执行由ANTLR生成的识别器所需的运行库。简而言之，就是ANTLR工具将文法转换成识别程序，然后识别程序利用ANTLR运行库中的某些支持类识别由该文法描述的语言的句子。此外，该归档文件还包含两个支持库：<a href="https://github.com/abego/treelayout">TreeLayout（一个复杂的树布局库）</a>和<a href="http://www.stringtemplate.org/">StringTemplate（一个用于生成代码和其它结构化文本的模板引擎）</a>。

现在来测试下ANTLR工具是否工作正常：

```
$ java -jar antlr-4.7.1-complete.jar  # 启动org.antlr.v4.Tool
```

如果正常的话会看到以下帮助信息：

```
ANTLR Parser Generator  Version 4.7.1
 -o ___              specify output directory where all output is generated
 -lib ___            specify location of grammars, tokens files
 -atn                generate rule augmented transition network diagrams
 -encoding ___       specify grammar file encoding; e.g., euc-jp
 -message-format ___ specify output style for messages in antlr, gnu, vs2005
 -long-messages      show exception details when available for errors and warnings
 -listener           generate parse tree listener (default)
 -no-listener        don't generate parse tree listener
 -visitor            generate parse tree visitor
 -no-visitor         don't generate parse tree visitor (default)
 -package ___        specify a package/namespace for the generated code
 -depend             generate file dependencies
 -D<option>=value    set/override a grammar-level option
 -Werror             treat warnings as errors
 -XdbgST             launch StringTemplate visualizer on generated code
 -XdbgSTWait         wait for STViz to close before continuing
 -Xforce-atn         use the ATN simulator for all predictions
 -Xlog               dump lots of logging info to antlr-timestamp.log
 -Xexact-output-dir  all output goes into -o dir regardless of paths/package
```

每次运行ANTLR工具都要输入这么长的命令是不是有些痛苦？写个脚本来解放我们的手指吧！

```
#!/bin/sh
java -cp antlr-4.7.1-complete.jar org.antlr.v4.Tool $*
```

把它保存为antlr，以后就可以使用下列命令来运行ANTLR工具：

```
$ ./antlr
```
