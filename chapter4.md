# chapter4

## chapter4.LexicalPreservationStarter

> 将获取到的代码进行打印，同时进行词汇提取与保存，通过对LexicalPreservingPrinter类的定义。

```
import com.github.javaparser.ParseResult;
import com.github.javaparser.ParseStart;
import com.github.javaparser.StringProvider;
import com.github.javaparser.ast.CompilationUnit;
import com.github.javaparser.printer.lexicalpreservation.LexicalPreservingPrinter;
import com.github.javaparser.utils.Pair;

public class LexicalPreservationStarter {

    public static void main(String[] args) {
        String code = "class A { }";
        Pair<ParseResult<CompilationUnit>, LexicalPreservingPrinter> result = LexicalPreservingPrinter.setup(
            ParseStart.COMPILATION_UNIT, new StringProvider(code));
            CompilationUnit cu = result.a.getResult().get();
            LexicalPreservingPrinter lpp = result.b;
            System.out.println(lpp.print(cu));
    }
}
```

```
class A { }
```



## chapter4.LexicalPreservationComplete

> 将获取到的代码进行词汇的提取与保存，可以修改类名、类名类型、以及引用的包来源，然后进行打印。使用cu.getClassByName("MyNewClassName").get()来获取代码段中的类名，然后可以进行更改。

```
import com.github.javaparser.ParseResult;
import com.github.javaparser.ParseStart;
import com.github.javaparser.StringProvider;
import com.github.javaparser.ast.CompilationUnit;
import com.github.javaparser.ast.Modifier;
import com.github.javaparser.ast.body.ClassOrInterfaceDeclaration;
import com.github.javaparser.printer.lexicalpreservation.LexicalPreservingPrinter;
import com.github.javaparser.utils.Pair;

public class LexicalPreservationComplete {

    public static void main(String[] args) {
        String code = "// Hey, this is a comment// Another one\nclass A { }";
        Pair<ParseResult<CompilationUnit>, LexicalPreservingPrinter> result = LexicalPreservingPrinter.setup(
                ParseStart.COMPILATION_UNIT, new StringProvider(code));
        CompilationUnit cu = result.a.getResult().get();
        LexicalPreservingPrinter lpp = result.b;
        System.out.println(lpp.print(cu));

        System.out.println("----------------");

        ClassOrInterfaceDeclaration myClass = cu.getClassByName("A").get();
        myClass.setName("MyNewClassName");
        System.out.println(lpp.print(cu));

        System.out.println("----------------");

        myClass = cu.getClassByName("MyNewClassName").get();
        myClass.setName("MyNewClassName");
        myClass.addModifier(Modifier.Keyword.PUBLIC);
        System.out.println(lpp.print(cu));

        System.out.println("----------------");

        myClass = cu.getClassByName("MyNewClassName").get();
        myClass.setName("MyNewClassName");
        myClass.addModifier(Modifier.Keyword.PUBLIC);
        cu.setPackageDeclaration("org.javaparser.samples");
        System.out.println(lpp.print(cu));
    }
}

```

```
// Hey, this is a comment
// Another one

class A { }
----------------
// Hey, this is a comment
// Another one

class MyNewClassName { }
----------------
// Hey, this is a comment
// Another one

public class MyNewClassName { }
----------------
package org.javaparser.samples;

// Hey, this is a comment
// Another one

public class MyNewClassName { }
```



## chapter4.PrettyPrintStarter

> 定义一个类别，使用ClassOrInterfaceDeclaration类进行定义

```
import com.github.javaparser.ast.body.ClassOrInterfaceDeclaration;
import com.github.javaparser.ast.comments.LineComment;

public class PrettyPrintStarter {

    public static void main(String[] args) {
        ClassOrInterfaceDeclaration myClass = new ClassOrInterfaceDeclaration();
        myClass.setComment(new LineComment("A very cool class!"));
        myClass.setName("MyClass");
        myClass.addField("String", "foo");
        System.out.println(myClass);
    }
}
```

```
// A very cool class!
class MyClass {

    String foo;
}

```





## chapter4.PrettyPrintStarter

> 定义一个类别，并进行漂亮的格式打印，通过PrettyPrinterConfiguration进行规则定义，PrettyPrinter进行打印。

```
import com.github.javaparser.ast.body.ClassOrInterfaceDeclaration;
import com.github.javaparser.ast.comments.LineComment;
import com.github.javaparser.printer.PrettyPrinter;
import com.github.javaparser.printer.PrettyPrinterConfiguration;

public class PrettyPrintComplete {

    public static void main(String[] args) {
        ClassOrInterfaceDeclaration myClass = new ClassOrInterfaceDeclaration();
        myClass.setComment(new LineComment("A very cool class!"));
        myClass.setName("MyClass");
        myClass.addField("String", "foo");

        PrettyPrinterConfiguration conf = new PrettyPrinterConfiguration();
        //conf.setIndent(" ");
        conf.setPrintComments(false);
        PrettyPrinter prettyPrinter = new PrettyPrinter(conf);
        System.out.println(prettyPrinter.print(myClass));
    }
}
```

```
class MyClass {

    String foo;
}

```



## chapter4.PrettyPrintVisitorComplete

> 定义一个类别，并进行漂亮的打印，暂时不清楚跟PrettyPrinter区别

```
import com.github.javaparser.ast.body.ClassOrInterfaceDeclaration;
import com.github.javaparser.ast.comments.LineComment;
import com.github.javaparser.ast.expr.MarkerAnnotationExpr;
import com.github.javaparser.ast.expr.NormalAnnotationExpr;
import com.github.javaparser.ast.expr.SingleMemberAnnotationExpr;
import com.github.javaparser.printer.PrettyPrintVisitor;
import com.github.javaparser.printer.PrettyPrinter;
import com.github.javaparser.printer.PrettyPrinterConfiguration;

public class PrettyPrintVisitorComplete {

    public static void main(String[] args) {
        ClassOrInterfaceDeclaration myClass = new ClassOrInterfaceDeclaration();
        myClass.setComment(new LineComment("A very cool class!"));
        myClass.setName("MyClass");
        myClass.addField("String", "foo");
        myClass.addAnnotation("MySecretAnnotation");

        System.out.println( myClass );
        PrettyPrinterConfiguration conf = new PrettyPrinterConfiguration();
        //conf.setIndent("  ");
        conf.setPrintComments(false);
        conf.setVisitorFactory(prettyPrinterConfiguration -> new PrettyPrintVisitor(conf) {

            @Override
            public void visit(MarkerAnnotationExpr n, Void arg) {
                // ignore
            }

            @Override
            public void visit(SingleMemberAnnotationExpr n, Void arg) {
                // ignore
            }

            @Override
            public void visit(NormalAnnotationExpr n, Void arg) {
                // ignore
            }

        });
        PrettyPrinter prettyPrinter = new PrettyPrinter(conf);
        System.out.println(prettyPrinter.print(myClass));
    }
}
```

```
// A very cool class!
@MySecretAnnotation()
class MyClass {

    String foo;
}

class MyClass {

    String foo;
}
```

