---
title:         注解的那点事儿
date:        2017-10-15 18:00
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - 后端
    - Java
tags:
    - 后端
    - Java
    - 2017
---



### 什么是注解?


----------



**注解是`JDK1.5`引入的一个语法糖，它主要用来当作元数据，简单的说就是用于解释数据的数据**。在Java中，类、方法、变量、参数、包都可以被注解。很多开源框架都使用了注解，例如`Spring`、`MyBatis`、`Junit`。我们平常最常见的注解可能就是`@Override`了，该注解用来标识一个重写的函数。

注解的作用：

 - 配置文件：替代`xml`等文本文件格式的配置文件。使用注解作为配置文件可以在代码中实现动态配置，相比外部配置文件，注解的方式会减少很多文本量。但缺点也很明显，更改配置需要对代码进行重新编译，无法像外部配置文件一样进行集中管理（所以现在基本都是外部配置文件+注解混合使用）。

 - 数据的标记：注解可以作为一个标记（例如：被`@Override`标记的方法代表被重写的方法）。

 - 减少重复代码：注解可以减少重复且乏味的代码。比如我们定义一个`@ValidateInt`，然后通过反射来获得类中所有成员变量，只要是含有`@ValidateInt`注解的成员变量，我们就可以对其进行数据的规则校验。

定义一个注解非常简单，只需要遵循以下的语法规则：

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
@Documented
public @interface ValidateInt {
	// 它们看起来像是定义一个函数，但其实这是注解中的属性
    int maxLength();

    int minLength();

}
```

我们发现上面的代码在定义注解时也使用了注解，这些注解被称为元注解。**作用于注解上的注解称为元注解（元注解其实就是注解的元数据）**，`Java`中一共有以下元注解。

 - `@Target`：用于描述注解的使用范围（注解可以用在什么地方）。

     - ElementType.CONSTRUCTOR：构造器。

     - ElementType.FIELD：成员变量。

     - ElementType.LOCAL_VARIABLE：局部变量。

     - ElementType.PACKAGE：包。

     - ElementType.PARAMETER：参数。

     - ElementType.METHOD：方法。

     - ElementType.TYPE：类、接口(包括注解类型) 或enum声明。

 - `@Retention`：注解的生命周期，用于表示该注解会在什么时期保留。

     - RetentionPolicy.RUNTIME：运行时保留，这样就可以通过反射获得了。

     - RetentionPolicy.CLASS：在class文件中保留。

     - RetentionPolicy.SOURCE：在源文件中保留。

 - `@Documented`：表示该注解会被作为被标注的程序成员的公共API，因此可以被例如javadoc此类的工具文档化。

 - `@Inherited`：表示该注解是可被继承的（如果一个使用了@Inherited修饰的annotation类型被用于一个class，则这个annotation将被用于该class的子类）。

了解了这些基础知识之后，接着完成上述定义的`@ValidateInt`，我们定义一个`Cat`类然后在它的成员变量中使用`@ValidateInt`，并通过反射进行数据校验。

```java
public class Cat {

    private String name;

    @ValidateInt(minLength = 0, maxLength = 10)
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public static void main(String[] args) throws IllegalAccessException {
        Cat cat = new Cat();
        cat.setName("楼楼");
        cat.setAge(11);

        Class<? extends Cat> clazz = cat.getClass();
        Field[] fields = clazz.getDeclaredFields();
        if (fields != null) {
            for (Field field : fields) {
                ValidateInt annotation = field.getDeclaredAnnotation(ValidateInt.class);
                if (annotation != null) {
                    field.setAccessible(true);
                    int value = field.getInt(cat);
                    if (value < annotation.minLength()) {
                        // ....
                    } else if (value > annotation.maxLength()) {
                        // ....
                    }
                }
            }
        }
    }

}
```

> 本文作者为:[SylvanasSun(sylvanas.sun@gmail.com)][1]，首发于[SylvanasSun's Blog][2]。
> 原文链接：https://sylvanassun.github.io/2017/10/15/2017-10-15-JavaAnnotation/
> （转载请务必保留本段声明，并且保留超链接。）


### 注解的实现


----------



注解其实只是`Java`的一颗语法糖（语法糖是一种方便程序员使用的语法规则，但它其实并没有表面上那么神奇的功能，只不过是由编译器帮程序员生成那些繁琐的代码）。在`Java`中这样的语法糖还有很多，例如`enum`、泛型、`forEach`等。

通过阅读[JLS(Java Language Specification][3]（当你想了解一个语言特性的实现时，最好的方法就是阅读官方规范）发现，**注解是一个继承自`java.lang.annotation.Annotation`接口的特殊接口**，原文如下：

```text
An annotation type declaration specifies a new annotation type, a special kind of interface type. To distinguish an annotation type declaration from a normal interface declaration, the keyword interface is preceded by an at-sign (@).

Note that the at-sign (@) and the keyword interface are distinct tokens. It is possible to separate them with whitespace, but this is discouraged as a matter of style.

The rules for annotation modifiers on an annotation type declaration are specified in §9.7.4 and §9.7.5.

The Identifier in an annotation type declaration specifies the name of the annotation type.

It is a compile-time error if an annotation type has the same simple name as any of its enclosing classes or interfaces.

The direct superinterface of every annotation type is java.lang.annotation.Annotation.
```

```java
package java.lang.annotation;

/**
 * The common interface extended by all annotation types.  Note that an
 * interface that manually extends this one does <i>not</i> define
 * an annotation type.  Also note that this interface does not itself
 * define an annotation type.
 *
 * More information about annotation types can be found in section 9.6 of
 * <cite>The Java&trade; Language Specification</cite>.
 *
 * The {@link java.lang.reflect.AnnotatedElement} interface discusses
 * compatibility concerns when evolving an annotation type from being
 * non-repeatable to being repeatable.
 *
 * @author  Josh Bloch
 * @since   1.5
 */
public interface Annotation {
    ...
}
```

我们将上节定义的`@ValidateInt`注解进行反编译来验证这个说法。

```java
  Last modified Oct 14, 2017; size 479 bytes
  MD5 checksum 2d9dd2c169fe854db608c7950af3eca7
  Compiled from "ValidateInt.java"
public interface com.sun.annotation.ValidateInt extends java.lang.annotation.Annotation
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_INTERFACE, ACC_ABSTRACT, ACC_ANNOTATION
Constant pool:
   #1 = Class              #18            // com/sun/annotation/ValidateInt
   #2 = Class              #19            // java/lang/Object
   #3 = Class              #20            // java/lang/annotation/Annotation
   #4 = Utf8               maxLength
   #5 = Utf8               ()I
   #6 = Utf8               minLength
   #7 = Utf8               SourceFile
   #8 = Utf8               ValidateInt.java
   #9 = Utf8               RuntimeVisibleAnnotations
  #10 = Utf8               Ljava/lang/annotation/Retention;
  #11 = Utf8               value
  #12 = Utf8               Ljava/lang/annotation/RetentionPolicy;
  #13 = Utf8               RUNTIME
  #14 = Utf8               Ljava/lang/annotation/Target;
  #15 = Utf8               Ljava/lang/annotation/ElementType;
  #16 = Utf8               FIELD
  #17 = Utf8               Ljava/lang/annotation/Documented;
  #18 = Utf8               com/sun/annotation/ValidateInt
  #19 = Utf8               java/lang/Object
  #20 = Utf8               java/lang/annotation/Annotation
{
  public abstract int maxLength();
    descriptor: ()I
    flags: ACC_PUBLIC, ACC_ABSTRACT

  public abstract int minLength();
    descriptor: ()I
    flags: ACC_PUBLIC, ACC_ABSTRACT
}
SourceFile: "ValidateInt.java"
RuntimeVisibleAnnotations:
  0: #10(#11=e#12.#13)
  1: #14(#11=[e#15.#16])
  2: #17()
```

`public interface com.sun.annotation.ValidateInt extends java.lang.annotation.Annotation`，很明显`ValidateInt`继承自`java.lang.annotation.Annotation`。

那么，如果注解只是一个接口，又是如何实现对属性的设置呢？这是**因为`Java`使用了动态代理对我们定义的注解接口生成了一个代理类，而对注解的属性设置其实都是在对这个代理类中的变量进行赋值**。所以我们才能用反射获得注解中的各种属性。

为了证实注解其实是个动态代理对象，接下来我们使用`CLHSDB(Command-Line HotSpot Debugger)`来查看`JVM`的运行时数据。如果有童鞋不了解怎么使用的话，可以参考R大的文章[借HSDB来探索HotSpot VM的运行时数据 - Script Ahead, Code Behind - ITeye博客][4]。

```java
0x000000000257f538 com/sun/proxy/$Proxy1
```

注解的类型为`com/sun/proxy/$Proxy1`，这正是动态代理生成代理类的默认类型，`com/sun/proxy`为默认包名，`$Proxy`是默认的类名，`1`为自增的编号。


### 实践-包扫描器


----------



我们在使用`Spring`的时候，只需要指定一个包名，框架就会去扫描该包下所有带有`Spring`中的注解的类。实现一个包扫描器很简单，主要思路如下：

 - 先将传入的包名通过类加载器获得项目内的路径。

 - 然后遍历并获得该路径下的所有class文件路径（需要处理为包名的格式）。

 - 得到了class文件的路径就可以使用反射生成Class对象并获得其中的各种信息了。

定义包扫描器接口：

```java
public interface PackageScanner {

    List<Class<?>> scan(String packageName);

    List<Class<?>> scan(String packageName, ScannedClassHandler handler);

}
```

函数2需要传入一个`ScannedClassHandler`接口，该接口是我们定义的回调函数，用于在扫描所有类文件之后执行的处理操作。

```java
@FunctionalInterface // 这个注解表示该接口为一个函数接口，用于支持Lambda表达式
public interface ScannedClassHandler {

    void execute(Class<?> clazz);

}
```

我想要包扫描器可以识别和支持不同的文件类型，定义一个枚举类`ResourceType`：

```java
public enum ResourceType {

    JAR("jar"),
    FILE("file"),
    CLASS_FILE("class"),
    INVALID("invalid");

    private String typeName;

    public String getTypeName() {
        return this.typeName;
    }

    private ResourceType(String typeName) {
        this.typeName = typeName;
    }

}
```

`PathUtils`是一个用来处理路径和包转换等操作的工具类：

```java
public class PathUtils {

    private static final String FILE_SEPARATOR = System.getProperty("file.separator");

    private static final String CLASS_FILE_SUFFIX = ".class";

    private static final String JAR_PROTOCOL = "jar";

    private static final String FILE_PROTOCOL = "file";

    private PathUtils() {
    }
	
	// 去除后缀名
    public static String trimSuffix(String filename) {
        if (filename == null || "".equals(filename))
            return filename;

        int dotIndex = filename.lastIndexOf(".");
        if (-1 == dotIndex)
            return filename;
        return filename.substring(0, dotIndex);
    }

    public static String pathToPackage(String path) {
        if (path == null || "".equals(path))
            return path;

        if (path.startsWith(FILE_SEPARATOR))
            path = path.substring(1);
        return path.replace(FILE_SEPARATOR, ".");
    }

    public static String packageToPath(String packageName) {
        if (packageName == null || "".equals(packageName))
            return packageName;
        return packageName.replace(".", FILE_SEPARATOR);
    }

    /**
     * 根据URL的协议来判断资源类型
     */
    public static ResourceType getResourceType(URL url) {
        String protocol = url.getProtocol();
        switch (protocol) {
            case JAR_PROTOCOL:
                return ResourceType.JAR;
            case FILE_PROTOCOL:
                return ResourceType.FILE;
            default:
                return ResourceType.INVALID;
        }
    }

    public static boolean isClassFile(String path) {
        if (path == null || "".equals(path))
            return false;
        return path.endsWith(CLASS_FILE_SUFFIX);
    }

    /**
     * 抽取URL中的主要路径.
     * Example:
     * "file:/com/example/hello" to "/com/example/hello"
     * "jar:file:/com/example/hello.jar!/" to "/com/example/hello.jar"
     */
    public static String getUrlMainPath(URL url) throws UnsupportedEncodingException {
        if (url == null)
            return "";
		
		// 如果不使用URLDecoder解码的话，路径会出现中文乱码问题
        String filePath = URLDecoder.decode(url.getFile(), "utf-8");
        // if file is not the jar
        int pos = filePath.indexOf("!");
        if (-1 == pos)
            return filePath;

        return filePath.substring(5, pos);
    }

    public static String concat(Object... args) {
        if (args == null || args.length == 0)
            return "";

        StringBuilder stringBuilder = new StringBuilder();
        for (int i = 0; i < args.length; i++)
            stringBuilder.append(args[i]);

        return stringBuilder.toString();
    }

}
```

定义了这些辅助类之后，就可以去实现包扫描器了。

```java
public class SimplePackageScanner implements PackageScanner {

    protected String packageName;

    protected String packagePath;

    protected ClassLoader classLoader;

    private Logger logger;

    public SimplePackageScanner() {
        this.classLoader = Thread.currentThread().getContextClassLoader();
        this.logger = LoggerFactory.getLogger(SimplePackageScanner.class);
    }

    @Override
    public List<Class<?>> scan(String packageName) {
        return this.scan(packageName, null);
    }

    @Override
    public List<Class<?>> scan(String packageName, ScannedClassHandler handler) {
        this.initPackageNameAndPath(packageName);
        if (logger.isDebugEnabled())
            logger.debug("Start scanning package: {} ....", this.packageName);
        URL url = this.getResource(this.packagePath);
        if (url == null)
            return new ArrayList<>();
        return this.parseUrlThenScan(url, handler);
    }

    private void initPackageNameAndPath(String packageName) {
        this.packageName = packageName;
        this.packagePath = PathUtils.packageToPath(packageName);
    }
	
}	
```

函数`getResource()`会根据包名来通过类加载器获得当前项目下的URL对象，如果这个URL为空则直接返回一个空的`ArrayList`。

```java
    protected URL getResource(String packagePath) {
        URL url = this.classLoader.getResource(packagePath);
        if (url != null)
            logger.debug("Get resource: {} success!", packagePath);
        else
            logger.debug("Get resource: {} failed,end of scan.", packagePath);
        return url;
    }
```

函数`parseUrlThenScan()`会解析URL对象并进行扫描，最终返回一个类列表。

```java
    protected List<Class<?>> parseUrlThenScan(URL url, ScannedClassHandler handler) {
        String urlPath = "";
        try {
		    // 先提取出URL中的路径（不含协议名等信息）
            urlPath = PathUtils.getUrlMainPath(url);
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
            logger.debug("Get url path failed.");
        }

        // 判断URL的类型
        ResourceType type = PathUtils.getResourceType(url);
        List<Class<?>> classList = new ArrayList<>();

        try {
            switch (type) {
                case FILE:
                    classList = this.getClassListFromFile(urlPath, this.packageName);
                    break;
                case JAR:
                    classList = this.getClassListFromJar(urlPath);
                    break;
                default:
                    logger.debug("Unsupported file type.");
            }
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
            logger.debug("Get class list failed.");
        }

		// 执行回调函数
        this.invokeCallback(classList, handler);
        logger.debug("End of scan <{}>.", urlPath);
        return classList;
    }
```

函数`getClassListFromFile()`会扫描路径下的所有class文件，并拼接包名生成Class对象。

```java
    protected List<Class<?>> getClassListFromFile(String path, String packageName) throws ClassNotFoundException {
        File file = new File(path);
        List<Class<?>> classList = new ArrayList<>();

        File[] listFiles = file.listFiles();
        if (listFiles != null) {
            for (File f : listFiles) {
                if (f.isDirectory()) {
					// 如果是一个文件夹，则继续递归调用，注意传递的包名
                    List<Class<?>> list = getClassListFromFile(f.getAbsolutePath(),
                            PathUtils.concat(packageName, ".", f.getName()));
                    classList.addAll(list);
                } else if (PathUtils.isClassFile(f.getName())) {
                    // 我们不添加名字带有$的class文件，这些都是JVM动态生成的
                    String className = PathUtils.trimSuffix(f.getName());
                    if (-1 != className.lastIndexOf("$"))
                        continue;

                    String finalClassName = PathUtils.concat(packageName, ".", className);
                    classList.add(Class.forName(finalClassName));
                }
            }
        }

        return classList;
    }
```

函数`getClassListFromJar()`会扫描Jar中的class文件。

```java
    protected List<Class<?>> getClassListFromJar(String jarPath) throws IOException, ClassNotFoundException {
        if (logger.isDebugEnabled())
            logger.debug("Start scanning jar: {}", jarPath);

        JarInputStream jarInputStream = new JarInputStream(new FileInputStream(jarPath));
        JarEntry jarEntry = jarInputStream.getNextJarEntry();
        List<Class<?>> classList = new ArrayList<>();

        while (jarEntry != null) {
            String name = jarEntry.getName();
            if (name.startsWith(this.packageName) && PathUtils.isClassFile(name))
                classList.add(Class.forName(name));
            jarEntry = jarInputStream.getNextJarEntry();
        }

        return classList;
    }
```

函数`invokeCallback()`遍历类对象列表，然后执行回调函数。

```java
    protected void invokeCallback(List<Class<?>> classList, ScannedClassHandler handler) {
        if (classList != null && handler != null) {
            for (Class<?> clazz : classList) {
                handler.execute(clazz);
            }
        }
    }
```

本节中实现的包扫描器源码地址：https://gist.github.com/SylvanasSun/6ab31dcfd9670f29a46917decdba36d1



[1]: https://github.com/SylvanasSun
[2]: https://sylvanassun.github.io/
[3]: http://docs.oracle.com/javase/specs/jls/se8/html/jls-9.html#jls-9.6
[4]: http://rednaxelafx.iteye.com/blog/1847971