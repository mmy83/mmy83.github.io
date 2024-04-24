---
title: java基础-注解处理器：Processor
author: mmy83
date: 2024-04-24 20:13:00 +0800
categories: [编程, java]
tags: [注解, Annotation, Processor, java]
math: true
mermaid: true
image:
  path: /images/2024-04-24/java基础-注解处理器：Processor/java基础-注解处理器：Processor-00.jpg
  lqip: data:image/webp;base64,UklGRlQAAABXRUJQVlA4IEgAAADQAQCdASoIAAUAAUAmJQBOgB8xitA9gAD+CyM/07vlY32ZX2F0u+fo9rdR0rnr3X3ptNmxgXpzuIBSWpkEkjwfvtrACQYAAAA=
  alt: java基础-注解处理器：Processor
---

## 前言

&emsp;&emsp;在[java基础-注解](/posts/java基础-注解/)中介绍过，注解的处理有两种办法，但是里面只讲了通过反射来处理，并没有讲过如何通过```Processor```来实现对注解的处理。这里将详细介绍如何通过```Processor```来实现对注解的处理。

&emsp;&emsp; ```Processor``` 不光能处理 ```RetentionPolicy.RUNTIME```的注解，还能处理```RetentionPolicy.SOURCE```和```RetentionPolicy.CLASS```的注解。这个很重要，毕竟反射方式只能处理```RetentionPolicy.RUNTIME```的注解。

## Processor

```java
public interface Processor {

    /**
     * 用于获取当前注解处理器支持的额外参数，也可以通过@SupportedOptions指定
     */
    Set<String> getSupportedOptions();

    /**
     * 指定注解处理哪个注解，必须指定，也可以通过@SupportedAnnotationTypes指定
     */
    Set<String> getSupportedAnnotationTypes();

    /**
     * 指定 Java 版本，通常返回 SourceVersion.latestSupported()，可以通过@SupportedSourceVersion指定
     */
    SourceVersion getSupportedSourceVersion();

    /**
     * 初始化操作，方法会被注解处理工具调用
     */
    void init(ProcessingEnvironment processingEnv);

    // 每个处理器的主要方法，这里实现扫描、解析和处理等逻辑
    boolean process(Set<? extends TypeElement> annotations,
                    RoundEnvironment roundEnv);

    // 主要供 IDE 使用，用于在使用注解时向用户提供关于注解的建议
    Iterable<? extends Completion> getCompletions(Element element,
                                                AnnotationMirror annotation,
                                                ExecutableElement member,
                                                String userText);
}
```

* __init__: init 方法会被注解处理工具调用，并传入 ProcessingEnviroment 参数。ProcessingEnviroment 类提供了很多有用的工具类：Elements, Types 和 Filer。

* __process__: process 方法相当于每个处理器的主函数。可以在这里实现扫描、解析和处理注解的逻辑，以及生成 Java 文件。方法会传入参数 RoundEnviroment，可以查询出包含特定注解的被注解元素。如果结果返回了 true，则表示该注解处理器处理了注解，其他注解处理器不会再运行。如要要其他处理器能够继续运行，此方法应返回 false。

* __getSupportedOptions__: 获取当前注解处理器支持的额外参数。

* __getSupportedAnnotationTypes__: 指定这个注解处理器处理哪个注解的，必须指定。注意，方法的返回值是一个字符串集合，包含处理器想要处理的注解类型的合法全称。

* __getSupportedSourceVersion__: 指定我们使用的 Java 版本，通常这里返回SourceVersion.latestSupported()。但是如果我们有足够的理由只支持 Java 6 的话，也可以返回 SourceVersion.RELEASE_6。但是推荐使用前者。

* __getCompletions__: 主要供 IDE 使用，用于在使用注解时向用户提供关于注解的建议。

> 注：
> 在 Java 7 及更高版本，可以使用注解来代替 getSupportedOptions() 方法、getSupportedAnnotationTypes() 方法、getSupportedSourceVersion() 方法。但是因为兼容的原因，特别是针对 Android 平台，建议使用重载 getSupportedAnnotationTypes() 方法、getSupportedOptions() 方法、getSupportedSourceVersion() 方法代替 @SupportedAnnotationTypes 注解、@SupportedOptions注解、 @SupportedSourceVersion 注解。
{: .prompt-tip }

&emsp;&emsp;```Processor```接口是所有注解处理器的基类，```AbstractProcessor``` 类是其直接实现类，每一个注解处理器都是继承于 ```AbstractProcessor```。每一个注解处理器类都必须有一个空的构造函数。

## ProcessingEnvironment

```java
public interface ProcessingEnvironment {

    /**
     * 获取 Options
     */
    Map<String,String> getOptions();

    /**
     * 获取 Messager
     */
    Messager getMessager();

    /**
     * 获取 Filer
     */
    Filer getFiler();

    /**
     * 获取 Elements
     */
    Elements getElementUtils();

    /**
     * 获取 Types
     */
    Types getTypeUtils();

    /**
     * 获取 SourceVersion
     */
    SourceVersion getSourceVersion();

    /**
     * 获取 Locale
     */
    Locale getLocale();

    /**
     * 是否开启预览
     */
    default boolean isPreviewEnabled() {
        return false;
    }

}
```

## Filer

```java
public interface Filer {

    /**
     * 创建一个新的源码文件，并返回一个 JavaFileObject 对象
     */
    JavaFileObject createSourceFile(CharSequence name,
                                    Element... originatingElements) throws IOException;

    /**
     * 创建一个新的 class 文件，并返回一个 JavaFileObject 对象
     */
    JavaFileObject createClassFile(CharSequence name,
                                   Element... originatingElements) throws IOException;

    /**
     * 创建一个新的资源文件，并返回一个 FileObject 对象
     */
   FileObject createResource(JavaFileManager.Location location,
                             CharSequence moduleAndPkg,
                             CharSequence relativeName,
                             Element... originatingElements) throws IOException;

    /**
     * 获取资源文件，参数限制同 createResource 方法
     */
    FileObject getResource(JavaFileManager.Location location,
                           CharSequence moduleAndPkg,
                           CharSequence relativeName) throws IOException;
}
```

## 实例

&emsp;&emsp;创建一个注解，为类创造构建器

### 创建注解

```java
//BuildGetter.java

package online.mmy83.processor;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD) // 注解用在方法上
@Retention(RetentionPolicy.SOURCE) // 尽在Source处理期间可用，运行期不可用
public @interface BuildGetter {
}
```

### 创建注解处理器

```java
// MyProcessor.java
package online.mmy83.processor;

import javax.annotation.processing.AbstractProcessor;
import javax.annotation.processing.ProcessingEnvironment;
import javax.annotation.processing.RoundEnvironment;
import javax.annotation.processing.SupportedAnnotationTypes;
import javax.lang.model.SourceVersion;
import javax.lang.model.element.Element;
import javax.lang.model.element.TypeElement;
import javax.lang.model.type.ExecutableType;
import javax.tools.Diagnostic;
import javax.tools.JavaFileObject;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.*;
import java.util.stream.Collectors;

@SuppressWarnings("unchecked")
@SupportedAnnotationTypes("online.mmy83.processor.BuildGetter") // 只处理这个注解；
public class MyProcessor extends AbstractProcessor {

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        System.out.println("MyProcessor.process ;");

        for (TypeElement annotation : annotations) {
            // 获取所有被该注解 标记过的实例
            Set<? extends Element> annotatedElements = roundEnv.getElementsAnnotatedWith(annotation);

            // 按照需求 检查注解使用的是否正确 以set开头，并且参数只有一个
            Map<Boolean, List<Element>> annotatedMethods = annotatedElements.stream().collect(
                    Collectors.partitioningBy(element ->
                            ((ExecutableType) element.asType()).getParameterTypes().size() == 1
                                    && element.getSimpleName().toString().startsWith("set")));

            List<Element> setters = annotatedMethods.get(true);
            List<Element> otherMethods = annotatedMethods.get(false);

            // 打印注解使用错误的case
            otherMethods.forEach(element ->
                    processingEnv.getMessager().printMessage(Diagnostic.Kind.ERROR,
                            "@BuilderProperty 注解必须放到方法上并且是set开头的单参数方法", element));

            if (setters.isEmpty()) {
                continue;
            }


            Map<String ,List<Element>> groupMap = new HashMap();

            // 按照全限定类名分组。一个类创建一个Build
            setters.forEach(setter ->{
                // 全限定类名
                String className = ((TypeElement) setter
                        .getEnclosingElement()).getQualifiedName().toString();
                List<Element> elements = groupMap.get(className);
                if(elements != null){
                    elements.add(setter);
                }else {
                    List<Element> newElements = new ArrayList<>();
                    newElements.add(setter);
                    groupMap.put(className,newElements);
                }
            });


            groupMap.forEach((groupSetterKey,groupSettervalue)->{
                //获取 类名SimpleName 和 set方法的入参
                Map<String, String> setterMap = groupSettervalue.stream().collect(Collectors.toMap(
                        setter -> setter.getSimpleName().toString(),
                        setter -> ((ExecutableType) setter.asType())
                                .getParameterTypes().get(0).toString()
                ));
                try {
                    // 组装XXXBuild类。并创建对应的类文件
                    writeBuilderFile(groupSetterKey,setterMap);
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }

            });
        }

        // 返回false 表示 当前处理器处理了之后 其他的处理器也可以接着处理，返回true表示，我处理完了之后其他处理器不再处理
        return true;
    }

    private void writeBuilderFile(
            String className, Map<String, String> setterMap)
            throws IOException {

        String packageName = null;
        int lastDot = className.lastIndexOf('.');
        if (lastDot > 0) {
            packageName = className.substring(0, lastDot);
        }

        String simpleClassName = className.substring(lastDot + 1);
        String builderClassName = className + "Builder";
        String builderSimpleClassName = builderClassName
                .substring(lastDot + 1);

        JavaFileObject builderFile = processingEnv.getFiler()
                .createSourceFile(builderClassName);

        try (PrintWriter out = new PrintWriter(builderFile.openWriter())) {

            // package xxx;
            if (packageName != null) {
                out.print("package ");
                out.print(packageName);
                out.println(";");
                out.println();
            }

            // public class xxxBuilder {
            out.print("public class ");
            out.print(builderSimpleClassName);
            out.println(" {");
            out.println();

            // private xxxBuilder object = new Builder();
            out.print("    private ");
            out.print(simpleClassName);
            out.print(" object = new ");
            out.print(simpleClassName);
            out.println("();");
            out.println();

            // public xxxBuilder build(){renturn object;}
            out.print("    public ");
            out.print(simpleClassName);
            out.println(" build() {");
            out.println("        return object;");
            out.println("    }");
            out.println();

            // public xxxBuilder yyy( A value){object.yyy(value);return this;}  // }
            setterMap.entrySet().forEach(setter -> {
                String methodName = setter.getKey();
                String argumentType = setter.getValue();

                out.print("    public ");
                out.print(builderSimpleClassName);
                out.print(" ");
                out.print(methodName);

                out.print("(");

                out.print(argumentType);
                out.println(" value) {");
                out.print("        object.");
                out.print(methodName);
                out.println("(value);");
                out.println("        return this;");
                out.println("    }");
                out.println();
            });

            out.println("}");
        }
    }

    @Override
    public synchronized void init(ProcessingEnvironment processingEnv) {
        super.init(processingEnv);
        System.out.println("----------");

        System.out.println(processingEnv.getOptions());

    }
    @Override
    public SourceVersion getSupportedSourceVersion() {
        return SourceVersion.latestSupported();
    }

}
```

### 注册注解处理器

&emsp;&emsp;在```resource/META-INF.services```文件夹下创建一个名为```javax.annotation.processing.Processor```的文件；里面的内容就是注解处理器的全限定类名```online.mmy83.processor.MyProcessor```.

![注册注解处理器](/images/2024-04-24/java基础-注解处理器：Processor/java基础-注解处理器：Processor-01.png)

&emsp;&emsp;设置编译期间禁止处理 Process，之所以这样做是因为，如果你不禁止 Process，ServiceLoader 就会去加载你刚刚设置的注解处理器，但是因为是在编译期，Class 文件被没有被成功加载，所以会抛出下面的异常

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.5.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
                <executions>
                    <execution>
                        <id>default-compile</id>
                        <configuration>
                            <!-- 主要是这里 -->
                            <compilerArgument>-proc:none</compilerArgument>
                        </configuration>
                    </execution>
                    <execution>
                        <id>compile-project</id>
                        <phase>compile</phase>
                        <goals>
                            <goal>compile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

![编译时忽略注解处理器](/images/2024-04-24/java基础-注解处理器：Processor/java基础-注解处理器：Processor-02.png)

> 注:
>
> 可以使用使用 @AutoService 自动配置 SPI 的配置文件。
> @AutoService 是 Google 开源的一个小插件，它可以自动的帮我们生成META-INF/services 的文件, 也就不需要你去手动的创建配置文件了，而且不需要设置编译期间忽略注解处理器。
>
{: .prompt-tip }

### 使用

```java
// User.java
package online.mmy83.processor;

public class User {
    private String name;

    @BuildGetter
    public void setName(String name) {
        this.name = name;
    }

//    @BuildGetter
    public void say(){
        System.out.println("hello "+name);
    }
}
```

### 编译

```shell
mvn install
```

### 查看

![编译结果](/images/2024-04-24/java基础-注解处理器：Processor/java基础-注解处理器：Processor-03.png)

## 感谢

&emsp;&emsp;写本博文期间大量参考了网上资料，核心代码均从网上摘抄,在此感谢各位的无私奉献！！！

## 参考

[Annotation Processing 101](https://hannesdorfmann.com/annotation-processing/annotationprocessing101/)

[Java注解编译期处理AbstractProcessor详解](https://blog.csdn.net/agonie201218/article/details/130940854)

[注解处理器 2：java 注解处理器](https://www.cnblogs.com/wellcherish/p/17087711.html)
