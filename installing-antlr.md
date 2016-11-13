# 安装ANTLR

ANTLR是由Java写成的，所以在安装ANTLR前必须保证已经安装有Java 1.6或以上版本。你可以到 http://www.antlr.org/download.html 下载ANTLR的最新版本，或者也可使用命令行工具下载：

```
curl -O http://www.antlr.org/download/antlr-4.5.1-complete.jar
```

antlr-4.5.1-complete.jar包含运行ANTLR工具的所有必要依赖，以及编译和执行由ANTLR生成的识别器所需的运行库。ANTLR工具将由语法文件描述的语法转换成识别程序，识别程序利用ANTLR运行库中的某些支持类识别输入的语句。该jar包还包含两个支持库：<a href="https://github.com/abego/treelayout">TreeLayout（一个复杂的树布局库）</a>和<a href="http://www.stringtemplate.org/">StringTemplate（一个用于生成代码和其它结构化文本的模板引擎）</a>。

现在来测试下ANTLR工具是否工作正常：

```
java -jar antlr-4.5.1-complete.jar  # 启动org.antlr.v4.Tool
```

如果正常的话会看到以下帮助信息：

```
ANTLR Parser Generator  Version 4.5.1
 -o ___              specify output directory where all output is generated
 -lib ___            specify location of grammars, tokens files
 ...
```

每次运行ANTLR工具都要输入这么长的命令是否有些痛苦？写个脚本来解放我们的手指吧！

```
#!/bin/sh
java -cp .:./antlr-4.5.1-complete.jar:$CLASSPATH org.antlr.v4.Tool $*
```

把它保存为antlr.sh，以后就可以使用下列命令来运行ANTLR工具了：

```
antlr
```
