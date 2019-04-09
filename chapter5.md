# chapter5

## chapter5.GetTypeOfReference

> 获得所有使用到的变量的名称以及类型，JavaSymbolSolver类。

```
import com.github.javaparser.JavaParser;
import com.github.javaparser.ast.CompilationUnit;
import com.github.javaparser.ast.expr.AssignExpr;
import com.github.javaparser.resolution.types.ResolvedType;
import com.github.javaparser.symbolsolver.JavaSymbolSolver;
import com.github.javaparser.symbolsolver.model.resolution.TypeSolver;
import com.github.javaparser.symbolsolver.resolution.typesolvers.CombinedTypeSolver;

import java.io.File;
import java.io.FileNotFoundException;

public class GetTypeOfReference {

    private static final String FILE_PATH = "src/main/java/org/javaparser/examples/chapter5/Bar.java";

    public static void main(String[] args) throws FileNotFoundException {
        TypeSolver typeSolver = new CombinedTypeSolver();

        JavaSymbolSolver symbolSolver = new JavaSymbolSolver(typeSolver);
        JavaParser.getStaticConfiguration().setSymbolResolver(symbolSolver);

        CompilationUnit cu = JavaParser.parse(new File(FILE_PATH));

        cu.findAll(AssignExpr.class).forEach(ae -> {
            ResolvedType resolvedType = ae.calculateResolvedType();
            System.out.println(ae.toString() + " is a: " + resolvedType);
        });
    }
}
```

```
a = a + 1 is a: PrimitiveTypeUsage{name='int'}
```

```
class Bar {

    private String a;

    void aMethod() {
        while (true) {
            int a = 0;
            a = a + 1;
        }
    }
}
```



## chapter5.ResolveMethodCalls

> 获得所有使用到的方法调用，以及参数类型

```
import com.github.javaparser.JavaParser;
import com.github.javaparser.ast.CompilationUnit;
import com.github.javaparser.ast.expr.MethodCallExpr;
import com.github.javaparser.symbolsolver.JavaSymbolSolver;
import com.github.javaparser.symbolsolver.model.resolution.TypeSolver;
import com.github.javaparser.symbolsolver.resolution.typesolvers.ReflectionTypeSolver;

import java.io.File;

public class ResolveMethodCalls {

    private static final String FILE_PATH = "src/main/java/org/javaparser/examples/chapter5/A.java";

    public static void main(String[] args) throws Exception {
        TypeSolver typeSolver = new ReflectionTypeSolver();

        JavaSymbolSolver symbolSolver = new JavaSymbolSolver(typeSolver);
        JavaParser.getStaticConfiguration().setSymbolResolver(symbolSolver);

        CompilationUnit cu = JavaParser.parse(new File(FILE_PATH));

        cu.findAll(MethodCallExpr.class).forEach(mce ->
                System.out.println(mce.resolveInvokedMethod().getQualifiedSignature()));
    }
}
```

```
class A {

    public void foo(Object param) {
        System.out.println(1);
        System.out.println("hi");
        System.out.println(param);
    } 
}
```

```
java.io.PrintStream.println(int)
java.io.PrintStream.println(java.lang.String)
java.io.PrintStream.println(java.lang.Object)
```



## chapter5.ResolveTypeInContext

> 获得所有使用到的类别，原例子是只获得最上边的类别，加了findAllNodesOfGivenClass方法，但是原方法已经弃用。

```
import com.github.javaparser.JavaParser;
import com.github.javaparser.ast.CompilationUnit;
import com.github.javaparser.ast.body.FieldDeclaration;
import com.github.javaparser.symbolsolver.JavaSymbolSolver;
import com.github.javaparser.symbolsolver.javaparser.Navigator;
import com.github.javaparser.symbolsolver.model.resolution.TypeSolver;
import com.github.javaparser.symbolsolver.resolution.typesolvers.CombinedTypeSolver;
import com.github.javaparser.symbolsolver.resolution.typesolvers.JavaParserTypeSolver;
import com.github.javaparser.symbolsolver.resolution.typesolvers.ReflectionTypeSolver;

import java.io.File;
import java.util.List;

public class ResolveTypeInContext {

    private static final String FILE_PATH = "src/main/java/org/javaparser/examples/chapter5/Foo.java";
    private static final String SRC_PATH = "src/main/java";

    public static void main(String[] args) throws Exception {
        TypeSolver reflectionTypeSolver = new ReflectionTypeSolver();
        TypeSolver javaParserTypeSolver = new JavaParserTypeSolver(new File(SRC_PATH));
        reflectionTypeSolver.setParent(reflectionTypeSolver);

        CombinedTypeSolver combinedSolver = new CombinedTypeSolver();
        combinedSolver.add(reflectionTypeSolver);
        combinedSolver.add(javaParserTypeSolver);

        JavaSymbolSolver symbolSolver = new JavaSymbolSolver(combinedSolver);
        JavaParser.getStaticConfiguration().setSymbolResolver(symbolSolver);

        CompilationUnit cu = JavaParser.parse(new File(FILE_PATH));

        FieldDeclaration fieldDeclaration = Navigator.findNodeOfGivenClass(cu, FieldDeclaration.class);

        System.out.println("Field type: " + fieldDeclaration.getVariables().get(0).getType()
                .resolve().asReferenceType().getQualifiedName());

        List allNodesOfGivenClass = Navigator.findAllNodesOfGivenClass(cu, FieldDeclaration.class);
        for (int i = 0; i < allNodesOfGivenClass.size(); i++){
            FieldDeclaration fieldDeclaration1 = (FieldDeclaration) allNodesOfGivenClass.get( i );
            System.out.println( "Field type: " + fieldDeclaration1.getVariables().get( 0 ).getType()
                    .resolve().asReferenceType().getQualifiedName());
        }
    }
}
```



## chapter5.UsingTypeSolver

> 获得类内所有的定义变量以及方法

```
import com.github.javaparser.resolution.declarations.ResolvedReferenceTypeDeclaration;
import com.github.javaparser.symbolsolver.model.resolution.TypeSolver;
import com.github.javaparser.symbolsolver.resolution.typesolvers.ReflectionTypeSolver;


import java.io.IOException;

public class UsingTypeSolver {

    private static void showReferenceTypeDeclaration(ResolvedReferenceTypeDeclaration resolvedReferenceTypeDeclaration){

        System.out.println(String.format("== %s ==",
                resolvedReferenceTypeDeclaration.getQualifiedName()));
        System.out.println(" fields:");
        resolvedReferenceTypeDeclaration.getAllFields().forEach(f ->
                System.out.println(String.format("    %s %s", f.getType(), f.getName())));
        System.out.println(" methods:");
        resolvedReferenceTypeDeclaration.getAllMethods().forEach(m ->
                System.out.println(String.format("    %s", m.getQualifiedSignature())));
        System.out.println();
    }

    public static void main(String[] args) {
        TypeSolver typeSolver = new ReflectionTypeSolver();

        //showReferenceTypeDeclaration(typeSolver.solveType("java.lang.Object"));
        showReferenceTypeDeclaration(typeSolver.solveType("java.lang.String"));
        //showReferenceTypeDeclaration(typeSolver.solveType("java.util.List"));

    }
}
```

```
== java.lang.String ==
 fields:
    ResolvedArrayType{PrimitiveTypeUsage{name='char'}} value
    PrimitiveTypeUsage{name='int'} hash
    PrimitiveTypeUsage{name='long'} serialVersionUID
    ResolvedArrayType{ReferenceType{java.io.ObjectStreamField, typeParametersMap=TypeParametersMap{nameToValue={}}}} serialPersistentFields
    ReferenceType{java.util.Comparator, typeParametersMap=TypeParametersMap{nameToValue={java.util.Comparator.T=ReferenceType{java.lang.String, typeParametersMap=TypeParametersMap{nameToValue={}}}}}} CASE_INSENSITIVE_ORDER
 methods:
    java.lang.String.substring(int)
    java.lang.String.valueOf(char[])
    java.lang.String.replace(char, char)
    java.lang.String.format(java.lang.String, java.lang.Object...)
    java.lang.String.matches(java.lang.String)
    java.lang.String.join(java.lang.CharSequence, java.lang.Iterable<? extends java.lang.CharSequence>)
    java.lang.String.compareToIgnoreCase(java.lang.String)
    java.lang.String.charAt(int)
    java.lang.String.lastIndexOf(java.lang.String, int)
    java.lang.String.indexOf(java.lang.String)
    java.lang.CharSequence.chars()
    java.lang.String.isEmpty()
    java.lang.String.valueOf(java.lang.Object)
    java.lang.Object.notify()
    java.lang.String.codePointAt(int)
    java.lang.String.regionMatches(boolean, int, java.lang.String, int, int)
    java.lang.String.indexOf(char[], int, int, java.lang.String, int)
    java.lang.String.toString()
    java.lang.String.contentEquals(java.lang.CharSequence)
    java.lang.String.indexOf(int, int)
    java.lang.String.indexOf(java.lang.String, int)
    java.lang.String.codePointCount(int, int)
    java.lang.String.concat(java.lang.String)
    java.lang.Object.wait(long, int)
    java.lang.String.indexOf(char[], int, int, char[], int, int, int)
    java.lang.String.getBytes(java.lang.String)
    java.lang.String.compareTo(java.lang.String)
    java.lang.String.getBytes()
    java.lang.String.toUpperCase()
    java.lang.String.replaceFirst(java.lang.String, java.lang.String)
    java.lang.String.startsWith(java.lang.String, int)
    java.lang.String.valueOf(int)
    java.lang.String.getBytes(java.nio.charset.Charset)
    java.lang.String.toLowerCase()
    java.lang.String.toLowerCase(java.util.Locale)
    java.lang.Object.clone()
    java.lang.String.codePointBefore(int)
    java.lang.String.lastIndexOf(int)
    java.lang.String.copyValueOf(char[], int, int)
    java.lang.String.valueOf(boolean)
    java.lang.Object.notifyAll()
    java.lang.String.getChars(int, int, char[], int)
    java.lang.String.getChars(char[], int)
    java.lang.String.subSequence(int, int)
    java.lang.String.format(java.util.Locale, java.lang.String, java.lang.Object...)
    java.lang.String.startsWith(java.lang.String)
    java.lang.String.endsWith(java.lang.String)
    java.lang.Comparable.compareTo(T)
    java.lang.String.getBytes(int, int, byte[], int)
    java.lang.String.valueOf(double)
    java.lang.Object.wait(long)
    java.lang.String.contains(java.lang.CharSequence)
    java.lang.String.split(java.lang.String)
    java.lang.String.replaceAll(java.lang.String, java.lang.String)
    java.lang.String.checkBounds(byte[], int, int)
    java.lang.String.valueOf(char[], int, int)
    java.lang.Object.registerNatives()
    java.lang.String.regionMatches(int, java.lang.String, int, int)
    java.lang.String.equalsIgnoreCase(java.lang.String)
    java.lang.String.lastIndexOf(char[], int, int, java.lang.String, int)
    java.lang.String.indexOfSupplementary(int, int)
    java.lang.String.substring(int, int)
    java.lang.String.toCharArray()
    java.lang.Object.finalize()
    java.lang.String.offsetByCodePoints(int, int)
    java.lang.String.replace(java.lang.CharSequence, java.lang.CharSequence)
    java.lang.String.lastIndexOfSupplementary(int, int)
    java.lang.Object.getClass()
    java.lang.String.trim()
    java.lang.String.copyValueOf(char[])
    java.lang.String.length()
    java.lang.String.equals(java.lang.Object)
    java.lang.String.nonSyncContentEquals(java.lang.AbstractStringBuilder)
    java.lang.String.valueOf(long)
    java.lang.String.valueOf(char)
    java.lang.CharSequence.codePoints()
    java.lang.String.lastIndexOf(int, int)
    java.lang.String.intern()
    java.lang.String.split(java.lang.String, int)
    java.lang.String.join(java.lang.CharSequence, java.lang.CharSequence...)
    java.lang.String.lastIndexOf(char[], int, int, char[], int, int, int)
    java.lang.String.hashCode()
    java.lang.String.indexOf(int)
    java.lang.String.lastIndexOf(java.lang.String)
    java.lang.String.contentEquals(java.lang.StringBuffer)
    java.lang.String.toUpperCase(java.util.Locale)
    java.lang.Object.wait()
    java.lang.String.valueOf(float)
```



## chapter5.UsingCombinedTypeSolver

> 对于UsingTypeSolver的一个扩展，使用联合类型器，就是将一些类联合起来构成CombinedTypeSolver。

```
import com.github.javaparser.resolution.declarations.ResolvedReferenceTypeDeclaration;
import com.github.javaparser.symbolsolver.model.resolution.TypeSolver;
import com.github.javaparser.symbolsolver.resolution.typesolvers.CombinedTypeSolver;
import com.github.javaparser.symbolsolver.resolution.typesolvers.JarTypeSolver;
import com.github.javaparser.symbolsolver.resolution.typesolvers.JavaParserTypeSolver;
import com.github.javaparser.symbolsolver.resolution.typesolvers.ReflectionTypeSolver;

import java.io.File;
import java.io.IOException;

public class UsingCombinedTypeSolver {

    private static void showReferenceTypeDeclaration(ResolvedReferenceTypeDeclaration resolvedReferenceTypeDeclaration){

        System.out.println(String.format("== %s ==",
                resolvedReferenceTypeDeclaration.getQualifiedName()));
        System.out.println(" fields:");
        resolvedReferenceTypeDeclaration.getAllFields().forEach(f ->
                System.out.println(String.format("    %s %s", f.getType(), f.getName())));
        System.out.println(" methods:");
        resolvedReferenceTypeDeclaration.getAllMethods().forEach(m ->
                System.out.println(String.format("    %s", m.getQualifiedSignature())));
        System.out.println();
    }

    public static void main(String[] args) throws IOException {
        TypeSolver myTypeSolver = new CombinedTypeSolver(
                new ReflectionTypeSolver(),
                JarTypeSolver.getJarTypeSolver( "C:/Program Files/Java/jdk1.8.0_201/jre/lib/deploy.jar" ),
                JarTypeSolver.getJarTypeSolver( "C:/Program Files/Java/jdk1.8.0_201/jre/lib/jsse.jar" ),
                JarTypeSolver.getJarTypeSolver( "C:/Program Files/Java/jdk1.8.0_201/jre/lib/jfr.jar" ),
                new JavaParserTypeSolver(new File("src/main/java"))
        );
        // using myTypeSolver
        showReferenceTypeDeclaration(myTypeSolver.solveType("java.lang.Object"));
    }
}
```

```
== java.lang.Object ==
 fields:
 methods:
    java.lang.Object.equals(java.lang.Object)
    java.lang.Object.notifyAll()
    java.lang.Object.toString()
    java.lang.Object.clone()
    java.lang.Object.wait()
    java.lang.Object.finalize()
    java.lang.Object.notify()
    java.lang.Object.wait(long)
    java.lang.Object.hashCode()
    java.lang.Object.getClass()
    java.lang.Object.registerNatives()
    java.lang.Object.wait(long, int)
```



## chapter5.MemoryTypeSolverComplete

> 对于类型获取的操作，暂时不知道为何这么设计。

```
import com.github.javaparser.JavaParser;
import com.github.javaparser.ast.CompilationUnit;
import com.github.javaparser.resolution.declarations.ResolvedReferenceTypeDeclaration;
import com.github.javaparser.resolution.declarations.ResolvedTypeDeclaration;
import com.github.javaparser.symbolsolver.core.resolution.Context;
import com.github.javaparser.symbolsolver.javaparsermodel.contexts.CompilationUnitContext;
import com.github.javaparser.symbolsolver.model.resolution.SymbolReference;
import com.github.javaparser.symbolsolver.resolution.typesolvers.MemoryTypeSolver;
import org.easymock.EasyMock;
import org.junit.Test;
import java.io.File;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertTrue;

public class MemoryTypeSolverComplete {

    private static final String FILE_PATH = "src/main/java/org/javaparser/examples/chapter5/Foo.java";

    @Test
    public void solveTypeInSamePackage() throws Exception {
        CompilationUnit cu = JavaParser.parse(new File(FILE_PATH));

        ResolvedReferenceTypeDeclaration otherClass = EasyMock.createMock(ResolvedReferenceTypeDeclaration.class);
        EasyMock.expect(otherClass.getQualifiedName()).andReturn("org.javaparser.examples.chapter5.Bar");

        /* Start of the relevant part */
        MemoryTypeSolver memoryTypeSolver = new MemoryTypeSolver();
        memoryTypeSolver.addDeclaration(
                "org.javaparser.examples.chapter5.Bar", otherClass);
        Context context = new CompilationUnitContext(cu, memoryTypeSolver);

        /* End of the relevant part */

        EasyMock.replay(otherClass);

        SymbolReference<ResolvedTypeDeclaration> ref = context.solveType("Bar");

        assertTrue(ref.isSolved());
        assertEquals("org.javaparser.examples.chapter5.Bar", ref.getCorrespondingDeclaration().getQualifiedName());
        //System.out.println( ref.getCorrespondingDeclaration().getQualifiedName() );
    }
}
```

