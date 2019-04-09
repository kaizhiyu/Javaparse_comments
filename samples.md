# Javaparse book samples



> Javaparse官方写了一本书用于介绍使用，github上有书中所有代码样例的代码。
>
> 用于学习Javaparse书中的使用样例，并注解。
>
> 加注了samples部分的代码



## samples.CommentRemover

> 将选择的注释从节点上进行删除

```
import com.github.javaparser.JavaParser;
import com.github.javaparser.ast.CompilationUnit;
import com.github.javaparser.ast.Node;
import com.github.javaparser.ast.comments.Comment;
import com.github.javaparser.ast.comments.LineComment;

import java.io.File;
import java.util.List;
import java.util.stream.Collectors;

public class CommentRemover {

    private static final String FILE_PATH = "src/main/java/org/javaparser/samples/ReversePolishNotation.java";

    public static void main(String[] args) throws Exception {
        CompilationUnit cu = JavaParser.parse(new File(FILE_PATH));

        List<Comment> comments = cu.getAllContainedComments();
        
        List<Comment> unwantedComments = comments
                .stream()
                //筛选条件，只选择一行的注释进行删除，或者可以不选择，全部从节点上摘掉
                //.filter(p -> !p.getCommentedNode().isPresent() || p instanceof LineComment)
                .collect(Collectors.toList());
        unwantedComments.forEach(Node::remove);

        System.out.println(cu.toString());

    }
}
```



## samples.CommentGenerator

> 生成每个类的方法注释，在方法名上边生成类名注释，将原来的注释替换掉

```
import com.github.javaparser.JavaParser;
import com.github.javaparser.ast.CompilationUnit;
import com.github.javaparser.ast.body.MethodDeclaration;
import com.github.javaparser.ast.visitor.VoidVisitorAdapter;

import java.io.File;
import java.util.ArrayList;
import java.util.List;
import java.util.regex.Pattern;

public class CommentGenerator {

    private static final String FILE_PATH = "src/main/java/org/javaparser/samples/ReversePolishNotation.java";

    private static final Pattern FIND_UPPERCASE = Pattern.compile("(.)(\\p{Upper})");

    public static void main(String[] args) throws Exception {

        CompilationUnit cu = JavaParser.parse(new File(FILE_PATH));

        List<MethodDeclaration> methodDeclarations = new ArrayList<>();
        VoidVisitorAdapter<List<MethodDeclaration>> unDocumentedMethodCollector = new UnDocumentedMethodCollector();
        unDocumentedMethodCollector.visit(cu, methodDeclarations);

        methodDeclarations.forEach(md -> md.setJavadocComment(generateJavaDoc(md)));

        System.out.println(cu.toString());
    }

    private static class UnDocumentedMethodCollector extends VoidVisitorAdapter<List<MethodDeclaration>> {

        @Override
        public void visit(MethodDeclaration md, List<MethodDeclaration> collector) {
            super.visit(md, collector);
            if(md.getJavadoc()!= null) {
                collector.add(md);
            }
        }
    }

    private static String generateJavaDoc(MethodDeclaration md){
        return " " + camelCaseToTitleFormat(md.getNameAsString()) + " ";
    }

    private static String camelCaseToTitleFormat(String text){
        String split = FIND_UPPERCASE.matcher(text).replaceAll("$1 $2");
        return split.substring(0,1).toUpperCase() + split.substring(1);
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
    public static int ONE_BILLION = 1000000000;

    private double memory = 0;

    /**
     * Calc
     */
    public Double calc(String input) {
        String[] tokens = input.split(" ");
        Stack<Double> numbers = new Stack<>();
        Stream.of(tokens).forEach(t -> {
            double a;
            double b;
            switch(t) {
                case "+":
                    b = numbers.pop();
                    a = numbers.pop();
                    numbers.push(a + b);
                    break;
                case "/":
                    b = numbers.pop();
                    a = numbers.pop();
                    numbers.push(a / b);
                    break;
                case "-":
                    b = numbers.pop();
                    a = numbers.pop();
                    numbers.push(a - b);
                    break;
                case "*":
                    b = numbers.pop();
                    a = numbers.pop();
                    numbers.push(a * b);
                    break;
                default:
                    numbers.push(Double.valueOf(t));
            }
        });
        return numbers.pop();
    }

    /**
     * Memory Recall
     */
    public double memoryRecall() {
        return memory;
    }

    /**
     * Memory Clear
     */
    public void memoryClear() {
        memory = 0;
    }

    /**
     * Memory Store
     */
    public void memoryStore(double value) {
        memory = value;
    }
}
/* EOF */
```



## samples.ReversePolishNotation

> 用于其他样例使用此样例进行注释的解析与分析

```
import java.util.Stack;
import java.util.stream.Stream;


/**
 * A Simple Reverse Polish Notation calculator with memory function.
 */
public class ReversePolishNotation {

    // What does this do?
    public static int ONE_BILLION = 1000000000;

    private double memory = 0;

    /**
     * Takes reverse polish notation style string and returns the resulting calculation.
     *
     *
     * @param input mathematical expression in the reverse Polish notation format
     * @return the calculation result
     */
    public Double calc(String input) {

        String[] tokens = input.split(" ");
        Stack<Double> numbers = new Stack<>();

        Stream.of(tokens).forEach(t -> {
            double a;
            double b;
            switch(t){
                case "+":
                    b = numbers.pop();
                    a = numbers.pop();
                    numbers.push(a + b);
                    break;
                case "/":
                    b = numbers.pop();
                    a = numbers.pop();
                    numbers.push(a / b);
                    break;
                case "-":
                    b = numbers.pop();
                    a = numbers.pop();
                    numbers.push(a - b);
                    break;
                case "*":
                    b = numbers.pop();
                    a = numbers.pop();
                    numbers.push(a * b);
                    break;
                default:
                    numbers.push(Double.valueOf(t));
            }
        });
        return numbers.pop();
    }

    /**
     * Memory Recall uses the number in stored memory, defaulting to 0.
     *
     * @return the double
     */
    public double memoryRecall(){
        return memory;
    }

    /**
     * Memory Clear sets the memory to 0.
     */
    public void memoryClear(){
        memory = 0;
    }


    public void memoryStore(double value){
        memory = value;
    }

}
/* EOF */
```



## samples.LexicalPreservation

> 用于测试

```
// Hey, this is a comment

// Another one
public class LexicalPreservation {
}
```

