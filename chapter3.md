# chapter3

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

```
//System.out.println( cu );
import java.util.Stack;
import java.util.stream.Stream;

public class ReversePolishNotation {

    public static int ONE_BILLION = 1000000000;

    private double memory = 0;

//省略了很多代码.......//
    public void memoryStore(double value) {
        memory = value;
    }
}
```



## chapter3.PrisonRules

> 不清楚是干嘛的。。

```
public class PrisonRules /* Maybe use Runnable? */ extends Thread {

    @Deprecated
    // What is
    // dibbs?
    private final String dibbs;

    public PrisonRules(final String dibbs) {
        this.dibbs = dibbs;
    }

    @Override // Is this really overridden?
    public void run() {
        System.out.println(dibbs);
    }

    public static void main(String[/* We should pass arguments in here */] args) {
        Thread t1 = new Thread(new PrisonRules("mine!") /* This should run first */);
        Thread t2 = new Thread(new PrisonRules("yoink!"));
        Thread t3 = new Thread(new PrisonRules("finders keepers!"));

        // Here Be Dragons
        t1/*(^_^)*/.start();
        t2/*(@_@)*/./*(¬_¬)*/start();
        /** We need more JavaDocs! */
        t3.start();

        t1.run();
        t2.run();
        t3.run();

    }
}
```

```
mine!
yoink!
finders keepers!
mine!
yoink!
finders keepers!
```



## chapter3.TimePrinterWithComments

> 带有注释的timeprinter

```
import java.time.LocalDateTime;

// Orphaned 1
// Attributed 1
public class TimePrinterWithComments {

    // Orphaned 2
    // Attributed 2
    public static void main(String args[]){
        System.out.print(LocalDateTime.now());
    }
    // Orphaned 3
}
```

