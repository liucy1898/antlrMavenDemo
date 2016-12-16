# 在IDEA中使用ANTLR4教程
Antlr 是一个基于 Java 开发的功能强大的语言识别工具，Antlr 以其简介的语法和高速的运行效率在这类工具中出类拔萃。当你需要开发一种领域语言时，语言可能像 Excel 中的公式一样复杂，也可能像本文中的例子一样简单（只有算术运算），这时你可以考虑使用 Antlr 来处理你的语言。

---

## Antlr 简介
1. ANTLR 语言识别的一个工具 (ANother Tool for Language Recognition ) 是一种语言工具，它提供了一个框架，可以通过包含 Java, C++, 或 C# 动作（action）的语法描述来构造语言识别器，编译器和解释器。 计算机语言的解析已经变成了一种非常普遍的工作，在这方面的理论和工具经过近 40 年的发展已经相当成熟，使用 Antlr 等识别工具来识别，解析，构造编译器比手工编程更加容易，同时开发的程序也更易于维护。
2. 语言识别的工具有很多种，比如大名鼎鼎的 Lex 和 YACC，Linux 中有他们的开源版本，分别是 Flex 和 Bison。在 Java 社区里，除了 Antlr 外，语言识别工具还有 JavaCC 和 SableCC 等。
3. 和大多数语言识别工具一样，Antlr 使用上下文无关文法描述语言。最新的 Antlr 是一个基于 LL(*) 的语言识别器。在 Antlr 中通过解析用户自定义的上下文无关文法，自动生成词法分析器 (Lexer)、语法分析器 (Parser) 和树分析器 (Tree Parser)。

---

## Antlr 能做什么
### 编程语言处理
识别和处理编程语言是 Antlr 的首要任务，编程语言的处理是一项繁重复杂的任务，为了简化处理，一般的编译技术都将语言处理工作分为前端和后端两个部分。其中前端包括词法分析、语法分析、语义分析、中间代码生成等若干步骤，后端包括目标代码生成和代码优化等步骤。

Antlr 致力于解决编译前端的所有工作。使用 Anltr 的语法可以定义目标语言的词法记号和语法规则，Antlr 自动生成目标语言的词法分析器和语法分析器；此外，如果在语法规则中指定抽象语法树的规则，在生成语法分析器的同时，Antlr 还能够生成抽象语法树；最终使用树分析器遍历抽象语法树，完成语义分析和中间代码生成。整个工作在 Anltr 强大的支持下，将变得非常轻松和愉快。
文本处理

### 文本处理
当需要文本处理时，首先想到的是正则表达式，使用 Anltr 的词法分析器生成器，可以很容易的完成正则表达式能够完成的所有工作；除此之外使用 Anltr 还可以完成一些正则表达式难以完成的工作，比如识别左括号和右括号的成对匹配等。

---

## 在IDEA中安装使用Antlr
1. 在Settings-Plugins中安装ANTLR v4 grammar plugin
2. 新建一个Maven项目，在pom.xml文件中添加ANTLR4插件和运行库的依赖。注意一定要用最新版的，依赖，不知道最新版本号的可以自己google一下maven antlr4。
```
<dependencies>

        <dependency>
            <groupId>org.antlr</groupId>
            <artifactId>antlr4-runtime</artifactId>
            <version>4.5.3</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.antlr</groupId>
                <artifactId>antlr4-maven-plugin</artifactId>
                <version>4.3</version>
                <executions>
                    <execution>
                        <id>antlr</id>
                        <goals>
                            <goal>antlr4</goal>
                        </goals>
                        <phase>none</phase>
                    </execution>
                </executions>
                <configuration>
                    <outputDirectory>src/test/java</outputDirectory>
                    <listener>true</listener>
                    <treatWarningsAsErrors>true</treatWarningsAsErrors>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

`antlr4-maven-plugin`用于生产Java代码，`antlr4-runtime`则是运行时所需的依赖库。把`antlr4-maven-plugin`的`phase`设置成none，这样在Maven 的lifecycle种就不会调用ANTLR4。如果你希望每次构建生成文法可以将这个配置去掉。

3. 我们定义一个最简单的领域语言，从一个简单的完成算术运算的例子出发，详细说明 Antlr 的使用。首先我们需要在src\main\java中新建一个 Antlr 的文法文件， 一般以 .g4 为文件名后缀，命名为 Demo.g4 。


---

## 表达式定义
### 文法定义
在这个文法文件 Demo.g4 中根据 Antlr 的语法规则来定义算术表达式的文法，文件的头部是 grammar 关键字，定义文法的名字，必须与文法文件文件的名字相同：
```
grammar Demo;
```

为了简单起见，假设我们的自定义语言只能输入一个算术表达式。从而整个程序有一个语句构成，语句有表达式或者换行符构成。如清单 1 所示：

**清单1.程序和语句**
```
prog: stat 
; 
stat: expr 
  |NEWLINE 
;
```

在 Anltr 中，算法的优先级需要通过文法规则的嵌套定义来体现，加减法的优先级低于乘除法，表达式 expr 的定义由乘除法表达式 multExpr 和加减法算符 ('+'|'-') 构成；同理，括号的优先级高于乘除法，乘除法表达式 multExpr 通过原子操作数 atom 和乘除法算符 ('*'|'/') 构成。整个表达的定义如清单 2 所示：

**清单2.表达式**
```
expr : multExpr (('+'|'-') multExpr)* 
; 
multExpr : atom (('*'|'/') atom)* 
; 
atom:  '(' expr ')' 
      | INT  
   | ID  
;
```

最后需要考虑的词法的定义，在 Antlr 中语法定义和词法定义通过规则的第一个字符来区别， 规定语法定义符号的第一个字母小写，而词法定义符号的第一个字母大写。算术表达式中用到了 4 类记号 ( 在 Antlr 中被称为 Token)，分别是标识符 ID，表示一个变量；常量 INT，表示一个常数；换行符 NEWLINE 和空格 WS，空格字符在语言处理时将被跳过，skip() 是词法分析器类的一个方法。如清单 3 所示：

**清单 3. 记号定义**
```
ID:('a'..'z'|'A'..'Z')+;
INT:'0'..'9'+;
NEWLINE:'\r'?'\n';
WS:(' '|'\t'|'\n'|'\r')+{skip();};
```

Antlr 支持多种目标语言，可以把生成的分析器生成为 Java，C#，C，Python，JavaScript 等多种语言，默认目标语言为 Java，通过 options {language=?;} 来改变目标语言。我们的例子中目标语言为 Java。

整个Demo.g4文件内容如下：
```
grammar Demo;

//parser
prog:stat
;
stat:expr|NEWLINE
;

expr:multExpr(('+'|'-')multExpr)*
;
multExpr:atom(('*'|'/')atom)*
;
atom:'('expr')'
    |INT
    |ID
;

//lexer
ID:('a'..'z'|'A'..'Z')+;
INT:'0'..'9'+;
NEWLINE:'\r'?'\n';
WS:(' '|'\t'|'\n'|'\r')+{skip();};

```

## 运行ANTLR
1. 右键Demo.g4，选择Configure ANTLR，配置output路径。

![](https://img3.doubanio.com/view/photo/photo/en2IjTkvxMclQWMdMnzxXA/124898500/x2405823644.jpg)

2. 右键Demo.g4，选择Generate ANTLR Recognizer。可以看到生成结果结果。
其中Demo.tokens为文法中用到的各种符号做了数字化编号，我们可以不关注这个文件。DemoLexer是Antlr生成的词法分析器，DemoParser是Antlr 生成的语法分析器。

![](https://img1.doubanio.com/view/photo/photo/txb6xdb77rCUwllTjozhJw/124898500/x2405823647.jpg)

3. 调用分析器。新建一个Main.java。
```
public static void run(String expr) throws Exception{

        //对每一个输入的字符串，构造一个 ANTLRStringStream 流 in
        ANTLRInputStream in = new ANTLRInputStream(expr);

        //用 in 构造词法分析器 lexer，词法分析的作用是产生记号
        DemoLexer lexer = new DemoLexer(in);

        //用词法分析器 lexer 构造一个记号流 tokens
        CommonTokenStream tokens = new CommonTokenStream(lexer);

        //再使用 tokens 构造语法分析器 parser,至此已经完成词法分析和语法分析的准备工作
        DemoParser parser = new DemoParser(tokens);

        //最终调用语法分析器的规则 prog，完成对表达式的验证
        parser.prog();
    }
```

完整Main.java代码：

```
import org.antlr.v4.runtime.CommonTokenStream;
import org.antlr.v4.runtime.ANTLRInputStream;

public class Main {

    public static void run(String expr) throws Exception{

        //对每一个输入的字符串，构造一个 ANTLRStringStream 流 in
        ANTLRInputStream in = new ANTLRInputStream(expr);

        //用 in 构造词法分析器 lexer，词法分析的作用是产生记号
        DemoLexer lexer = new DemoLexer(in);

        //用词法分析器 lexer 构造一个记号流 tokens
        CommonTokenStream tokens = new CommonTokenStream(lexer);

        //再使用 tokens 构造语法分析器 parser,至此已经完成词法分析和语法分析的准备工作
        DemoParser parser = new DemoParser(tokens);

        //最终调用语法分析器的规则 prog，完成对表达式的验证
        parser.prog();
    }

    public static void main(String[] args) throws Exception{

        String[] testStr={
                "2",
                "a+b+3",
                "(a-b)+3",
                "a+(b*3"
        };

        for (String s:testStr){
            System.out.println("Input expr:"+s);
            run(s);
        }
    }
}
```

4. 运行Main.java
当输入合法的的表达式时，分析器没有任何输出，表示语言被分析器接受；当输入的表达式违反文法规则时，比如“a + (b * 3”，分析器输出 line 0:-1 mismatched input '<EOF>' expecting ')'；提示期待一个右括号却遇到了结束符号。
![](https://img1.doubanio.com/view/photo/photo/PMOQz21qT_zFsp8nh9lE7A/124898500/x2405823648.jpg)

---

## 文法可视化
1. 打开Antlr Preview。
2. 在Demo.g4中选中一个语法定义符号，如expr。右键选中的符合，选择Text Rule expr。
![](https://img1.doubanio.com/view/photo/photo/0zTTzvFbOiai_8GRwOho8g/124898500/x2405823649.jpg)
3. 在ANTLR Preview中选择input,输入表达式，如a+b*c+4/2。则能显示出可视化的文法。
![](https://img3.doubanio.com/view/photo/photo/kfLbHe2Ts0PlkRHgysgXIw/124898500/x2405823651.jpg)
