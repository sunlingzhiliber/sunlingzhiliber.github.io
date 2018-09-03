---
title: "有趣的Java类库: Lombok"
categories:
  - Java
tags:
  - 有趣的Java类库
---

>Project Lombok is a java library that automatically plugs into your editor and build tools, spicing up your java.
Never write another getter or equals method again.
#简介
Lombok的目的是减少（POJO）代码的重复编写，并提供比较好的解决方案。



# 常用注解
## 类注解
### @Data
   该注解使用在类上，回提供get/set/equals/hashcode/toString等方法，该注解很有用，但是没法使用其他注解所拥有的控制粒度

### @NoArgsConstructor@AllArgsConstructor和@RequiredArgsConstructor
- @NoArgsConstructor会生成一个无参构造函数
- @AllArgsConstructor会生成一个全参构造函数
- @RequiredArgsConstructor会选择需要进行构造的参数形成构造函数，这里的参数包括：以`final`修饰但是没有初始化的字段或者@NonNull注解的并未初始化的字段

### @log和@Slf4j和@Log4j
可以使用变量log

### @Setter和@Getter
当然也可以作用于变量

### @ToString
- 可以通过`exclude`,排除特定字段 `@ToString(exclude = {"id","name","sex"})`
- 可以通过`includeFieldNames`,设置输出是否包含key值`@ToString(includeFieldNames = true)

### @EqualsAndHashCode
   添加equals和hashCode

### @Value
和@Data的区别在于它不回提供setter方法，应用场景是不可变对象
所有对象都是私有的，且默认是final的，无setter。


### @RequiredArgsConstructor
会生成 包含常量（final）的和标识了@NotNull的变量的构造方法，应用场景是。。。。

### @Builder
提供bulder模式的方法构建对象


##属性注解
### @NonNull 
自动为属性进行非空检测

### @CleanUp
自动关闭流
```java
 @Cleanup InputStream in = new FileInputStream(args[0]);
```
当然你可以使用`try with resources`来自动回收资源
```java
try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
```

# 使用注意
由于Lombok是采用的注解方式，只有在编译之后，才会自动生成方法，为了Idea的代码检验出错，你需要安装Lombok Plugin插件。


# 实现原理
利用javac的编译器利用注解来完成对class文件的修改，具体流程如下：
1)javac对源代码进行分析，生成一棵抽象语法树(AST)
2)运行过程中调用实现了"JSR 269 API"的lombok程序
3)此时lombok就对第一步骤得到的AST进行处理，找到@Data注解所在类对应的语法树(AST)，然后修改该语法树(AST)，增加getter和setter方法定义的相应树节点
4)javac使用修改后的抽象语法树(AST)生成字节码文件，即给class增加新的节点（代码块）