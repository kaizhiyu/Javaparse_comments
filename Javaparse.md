# Javaparse book example



> Javaparse官方写了一本书用于介绍使用，github上有书中所有代码样例的代码。
>
> 用于学习Javaparse书中的使用样例，并注解。



## chapter2.ModifyingVisitorStarter

> 用于获取全部代码段

```
import com.github.javaparser.JavaParser;
import com.github.javaparser.ast.CompilationUnit;

import java.io.FileInputStream;

public class ModifyingVisitorStarter {

    private static final String FILE_PATH = "src/main/java/org/javaparser/samples/ReversePolishNotation.java";

    public static void main(String[] args) throws Exception {

        CompilationUnit cu = JavaParser.parse(new FileInputStream( FILE_PATH ));
        System.out.println( cu );
    }
}
```

```
//System.out.println( cu );
import java.util.Stack;
import java.util.stream.Stream;

/**
 * A Simple Reverse Polish Notation calculator with memory function.
 */
public class ReversePolishNotation {

    // What does this do?
    public static int ONE_BILLION = 1000000000;

    private double memory = 0;

//省略了很多代码.......//
    public void memoryStore(double value) {
        memory = value;
    }
}
/* EOF */
```



## chapter2.ModifyingVisitorComplete

> 用于获取所有的代码，将数字进行替换，将原来的1000000000替换为1_000_000_000

```
import com.github.javaparser.JavaParser;
import com.github.javaparser.ast.CompilationUnit;
import com.github.javaparser.ast.body.FieldDeclaration;
import com.github.javaparser.ast.expr.IntegerLiteralExpr;
import com.github.javaparser.ast.visitor.ModifierVisitor;

import java.io.FileInputStream;
import java.util.regex.Pattern;

public class ModifyingVisitorComplete {

    private static final String FILE_PATH = "src/main/java/org/javaparser/samples/ReversePolishNotation.java";

    private static final Pattern LOOK_AHEAD_THREE = Pattern.compile("(\\d)(?=(\\d{3})+$)");
	//正则式解析：（?=(\d{3})+$)表达式满足（\d{3})+$的规则
	//			 $：必须以表达式的规则为结尾
	//			 d{3}+ ：必须反复出现3次 6次 9次
	//			 (\d)(?=(\d{3})+$) :以一位数字开头，三位、六位。。。数字结尾   1234、1234567
    public static void main(String[] args) throws Exception {

        CompilationUnit cu = JavaParser.parse(new FileInputStream(FILE_PATH));

        ModifierVisitor<?> numericLiteralVisitor = new IntegerLiteralModifier();
        numericLiteralVisitor.visit(cu, null);

        System.out.println(cu.toString());
    }

    private static class IntegerLiteralModifier extends ModifierVisitor<Void> {

        @Override
        public FieldDeclaration visit(FieldDeclaration fd, Void arg) {
            super.visit(fd, arg);
            fd.getVariables().forEach(v ->
                    v.getInitializer().ifPresent(i -> {
                        if (i instanceof IntegerLiteralExpr) {
                            v.setInitializer(formatWithUnderscores(((IntegerLiteralExpr) i).getValue()));
                        }
                    }));
            return fd;
        }
    }

    static String formatWithUnderscores(String value) {
        String withoutUnderscores = value.replaceAll("_", "");
        return LOOK_AHEAD_THREE.matcher(withoutUnderscores).replaceAll("$1_");
    }
}
```

```
import java.util.Stack;
import java.util.stream.Stream;

/**
 * A Simple Reverse Polish Notation calculator with memory function.
 */
public class ReversePolishNotation {

    // What does this do?
    public static int ONE_BILLION = 1_000_000_000;//主要区别

    private double memory = 0;
    /**
     * Memory Clear sets the memory to 0.
     */
    public void memoryClear() {
        memory = 0;
    }

    public void memoryStore(double value) {
        memory = value;
    }
}
/* EOF */
```



## chapter2.CommentReporterStarter

> 用于获取所有的代码注释

```
import com.github.javaparser.JavaParser;
import com.github.javaparser.ast.CompilationUnit;
import com.github.javaparser.ast.comments.Comment;

import java.io.FileInputStream;
import java.util.List;


public class CommentReporterStarter {

    private static final String FILE_PATH = "src/main/java/org/javaparser/samples/ReversePolishNotation.java";

    public static void main(String[] args) throws Exception {

        CompilationUnit cu = JavaParser.parse(new FileInputStream(FILE_PATH));

        List<Comment> comments = cu.getAllContainedComments();
        comments.forEach(System.out::println);
    }
}
```

```
//comments.forEach(System.out::println);
/* EOF */

/**
 * A Simple Reverse Polish Notation calculator with memory function.
 */

// What does this do?

/**
 * Takes reverse polish notation style string and returns the resulting calculation.
 *
 * @param input mathematical expression in the reverse Polish notation format
 * @return the calculation result
 */

/**
 * Memory Recall uses the number in stored memory, defaulting to 0.
 *
 * @return the double
 */

/**
 * Memory Clear sets the memory to 0.
 */
```



## chapter2.CommentReporterComplete

> 是对于CommentReporterStarter的进一步操作，用于获取所有的代码注释，并进行整理打印。

```
import com.github.javaparser.JavaParser;
import com.github.javaparser.ast.CompilationUnit;

import java.io.File;
import java.util.List;
import java.util.stream.Collectors;

public class CommentReporterComplete {

    private static final String FILE_PATH = "src/main/java/org/javaparser/samples/ReversePolishNotation.java";

    public static void main(String[] args) throws Exception {

        CompilationUnit cu = JavaParser.parse(new File(FILE_PATH));

        List<CommentReportEntry> comments = cu.getAllContainedComments()
                .stream()
                .map(p -> new CommentReportEntry(p.getClass().getSimpleName(),
                        p.getContent(),
                        p.getRange().get().begin.line,
                        !p.getCommentedNode().isPresent()))
                .collect(Collectors.toList());

        comments.forEach(System.out::println);
    }


    private static class CommentReportEntry {
        private String type;
        private String text;
        private int lineNumber;
        private boolean isOrphan;

        CommentReportEntry(String type, String text, int lineNumber, boolean isOrphan) {
            this.type = type;
            this.text = text;
            this.lineNumber = lineNumber;
            this.isOrphan = isOrphan;
        }
        
        @Override
        public String toString() {
            return lineNumber + "|" + type + "|" + isOrphan + "|" + text.replaceAll("\\n","").trim();
        }
    }
}
```

```
//comments.forEach(System.out::println);
81|BlockComment|true|EOF
7|JavadocComment|false|* A Simple Reverse Polish Notation calculator with memory function.
12|LineComment|false|What does this do?
17|JavadocComment|false|* Takes reverse polish notation style string and returns the resulting calculation.     *     * @param input mathematical expression in the reverse Polish notation format     * @return the calculation result
59|JavadocComment|false|* Memory Recall uses the number in stored memory, defaulting to 0.     *     * @return the double
68|JavadocComment|false|* Memory Clear sets the memory to 0.
```



## chapter2.VoidVisitorStarter

> 用于获取函数的调用关系，代码同ModifyingVisitorStarter.java，最初始的代码

```
import com.github.javaparser.JavaParser;
import com.github.javaparser.ast.CompilationUnit;

import java.io.FileInputStream;

public class VoidVisitorStarter {

    private static final String FILE_PATH = "src/main/java/org/javaparser/samples/ReversePolishNotation.java";

    public static void main(String[] args) throws Exception {

        CompilationUnit cu = JavaParser.parse(new FileInputStream(FILE_PATH));
    }
}
```



## chapter2.VoidVisitorComplete

> 用于获取函数名，两种方法，一种是在方法体里边进行对每个函数名的操作，一种是获取到返回值进行操作。

```
import com.github.javaparser.JavaParser;
import com.github.javaparser.ast.CompilationUnit;
import com.github.javaparser.ast.body.MethodDeclaration;
import com.github.javaparser.ast.visitor.VoidVisitor;
import com.github.javaparser.ast.visitor.VoidVisitorAdapter;

import java.io.FileInputStream;
import java.util.ArrayList;
import java.util.List;

public class VoidVisitorComplete {

    private static final String FILE_PATH = "src/main/java/org/javaparser/samples/ReversePolishNotation.java";

    public static void main(String[] args) throws Exception {

        CompilationUnit cu = JavaParser.parse(new FileInputStream(FILE_PATH));

        VoidVisitor<?> methodNameVisitor = new MethodNamePrinter();
        methodNameVisitor.visit(cu, null);
        List<String> methodNames = new ArrayList<>();
        VoidVisitor<List<String>> methodNameCollector = new MethodNameCollector();
        methodNameCollector.visit(cu, methodNames);
        methodNames.forEach(n -> System.out.println("Method Name Collected: " + n));

    }

    private static class MethodNamePrinter extends VoidVisitorAdapter<Void> {

        @Override
        public void visit(MethodDeclaration md, Void arg) {
            super.visit(md, arg);
            System.out.println("Method Name Printed: " + md.getName());
        }
    }

    private static class MethodNameCollector extends VoidVisitorAdapter<List<String>> {

        @Override
        public void visit(MethodDeclaration md, List<String> collector) {
            super.visit(md, collector);
            collector.add(md.getNameAsString());
        }
    }
}
```

```
Method Name Printed: calc
Method Name Printed: memoryRecall
Method Name Printed: memoryClear
Method Name Printed: memoryStore
Method Name Collected: calc
Method Name Collected: memoryRecall
Method Name Collected: memoryClear
Method Name Collected: memoryStore
```



## chapter3.ConfigurationOptions

> 获取文档中的代码时，取消注释、取消注释中的空行。其他的筛选选项在ParserConfiguration文件中有。

```
import com.github.javaparser.JavaParser;
import com.github.javaparser.ParserConfiguration;
import com.github.javaparser.ast.CompilationUnit;

import java.io.FileInputStream;

public class ConfigurationOptions {

    public void ignoreComments() {

        ParserConfiguration parserConfiguration = new ParserConfiguration()
                .setAttributeComments(false);

        JavaParser.setStaticConfiguration(parserConfiguration);

    }

    public void ignoreFloatingComments() {

        ParserConfiguration parserConfiguration = new ParserConfiguration()
                .setDoNotAssignCommentsPrecedingEmptyLines(true);

        JavaParser.setStaticConfiguration(parserConfiguration);
    }
}

class ConfigurationOptionsTest {

    private static final String FILE_PATH = "src/main/java/org/javaparser/samples/ReversePolishNotation.java";

    public static void main ( String[] args ) throws Exception{

        ConfigurationOptions consiguretionOptions = new ConfigurationOptions();

        //忽略注释中的空行
        consiguretionOptions.ignoreFloatingComments();
        //忽略注释
        consiguretionOptions.ignoreComments();

        CompilationUnit cu = JavaParser.parse(new FileInputStream(FILE_PATH));

        System.out.println( cu );
    }
}
```

