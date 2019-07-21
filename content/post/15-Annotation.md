---
title: 'Annotation'
tags: ["疯狂Java讲义第三版读书笔记","javaSE"]
date: 2019-06-15
draft: false
---

# 1.`JDK`的元`Annotation`

## 1.1`@Retention`

`@Retention`只能用于修饰`Annotation`定义,用于指定被修饰的`Annotation`可以保留多长时间.`@Retention`包含一个`RetentionPolicy`类型的`value`成员变量,所以使用`@Retention`时必须为该`value`成员变量指定值.

`value`成员变量的值只能是如下三个:

* `RetentionPolicy.CLASS`:编译器将把`Annotation`记录在`class`文件中.当运行`Java`程序时,`JVM`不可获取`Annotation`信息.这是默认值.
* `RetentionPolicy.RUNTIME`：编译器将把`Annotation`记录在`class`文件中.当运行`Java`程序时,`JVM`可获取`Annotation`信息,程序可以通过反射获取该`Annotation`信息.
* `RetentionPolicy.SOURCE`:`Annotation`只能保留在源代码中,编译器直接丢弃这种`Annotation`.

**示例代码:**

```java
    @Retention(RetentionPolicy.RUNTIME)
    @interface Testable {
    }
```

如果使用注解时只需要为`value`成员变量指定值,则使用该注解时可以直接在该注解后的括号里指定`value`成员变量的值,无须使用**`value`==变量值**的形式.

## 1.2`@Target`

`@Target`也只能修饰一个`Annotation`定义,它用于指定被修饰的`Annotation`能用于修饰哪些程序单元.`@Target`元`Annotation`也包含一个名为`value`的成员变量,该成员变量的值只能是如下几个:

* `ElementType.ANNOTATION_TYPE`：指定该策略的`Annotation`只能修饰`Annotation`.
* `ElementType.CONSTRUCTOR`：只能修饰构造器
* `ElementType.FIELD`：只能修饰成员变量
* `ElementType.LOCAL_VARIABLE`: 只能修饰局部变量
* `ElementType.METHOD`:只能修饰方法定义
* `ElementType.PACKAGE`:只能修饰包定义
* `ElementType.PARAMETER`:只能修饰参数
* `ElementType.TYPE`:指定该策略的`Annotation`可以修饰类,接口(包括注解类型)或枚举定义.

**示例代码:**

```java
    @Target(ElementType.FIELD)
    @interface OnlyForField {
    }
```

## 1.3`@Documented`

`@Documented`用于指定被该元`Annotation`修饰的`Annotation`类将被`javadoc`工具提取成文档,如果定义`Annotation`类时使用了`@Documented`修饰,则所有使用`Annotation`修饰的程序元素的`API`文档将会包含该`Annotation`说明.

## 1.4`@Inherited`

`@Inherited`元`Annotation`指定被它修饰的`Annotation`将具有继承性,如果某个类使用了`@Xxx`注解修饰(定义该`Annotation`时使用了`@Inherited`修饰)修饰,则其子类将自动被`@Xxx`修饰.

**示例代码:**

```java
	@Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Inherited
    @interface Inheritable {

    }

    @Inheritable
    class Base {
    }

    class Sub extends Base {
    }

    @Test
    public void test59() {
        System.out.println(Sub.class.isAnnotationPresent(Inheritable.class));
    }
```

**控制台输出:**

```
true
```

# 2.自定义`Annotation`

## 2.1定义`Annotation`

定义新的`Annotation`类型使用`@interface`关键字定义一个新的`Annotation`类型与定义一个接口非常像.

**定义Annotation**

```java
	@Retention(RetentionPolicy.RUNTIME)
    @interface MyAnnotation {
        //声明变量,使用此注解时需要声明此变量值
        String name();

        //设置默认值,使用注解时无需声明此变量值
        int age() default 24;
    }
```

根据`Annotation`是否可以包含成员变量,可以把`Annotation`分为如下两类.

* 标记`Annotation`:没有定义成员变量的`Annotation`类型被称为标记.这种`Annotation`仅利用自身的存在与否来提供信息,如`@Override`,`@Test`.
* 元数据`Annotation`:包含成员变量的`Annotation`,因为它们可以接受更多的元数据.

## 2.2提取`Annotation`信息

主要方法介绍:

* `<A extends Annotation> A getAnnotation(Class<A> annotationClass)`:返回该程序元素上存在的,指定类型的注解,如果该类型的注解不存在,则返回`null`.
* `<A extends Annotation> A getDeclaredAnnotation(Class<A> annotationClass)`:这是`Java`8新增的方法,该方法尝试获取直接修饰该程序元素,制定类型的`Annotation`.如果该类型的注解不存在,则返回`null`.
* `Annotation[] getAnnotations()`:返回该程序元素上存在的所有注解.
* `Annotaton[] getDeclaredAnnotations()`:返回直接修饰该程序元素的所有`Annotation`.
* `boolean isAnnotationPresent(Class<? extends Annotation> annotationClass)`:判断该程序元素上是否存在指定类型的注解,如果存在则返回`true`,否则返回`false`.
* `<A extends Annotation> A[] getAnnotationByType(Class<A> annotationClass)` :该方法的功能与`getAnnotation()`方法基本类似.但此方法可用于获取`Java8`添加的重复注解数据.
* `<A extends Annotation> A[] getDeclaredAnnotationByType(Class<A> annotationClass)` :该方法的功能与`getDeclaredAnnotation()`方法基本类似.但此方法可用于获取`Java8`添加的重复注解数据.

### 2.2.1提取`Annotation`信息

```java
	@MyAnnotation(name = "Alex")
    @Test
    public void test60() {
        try {
            //通过反射获取方法
            Method method = Class.forName("PersonalTest").getMethod("test60");
            //获取并输出方法所有的注解
            Annotation[] annotations = method.getAnnotations();
            for (Annotation annotation : annotations) {
                System.out.println(annotation);
            }
            System.out.println("**************************");
            //获取特定Annotation
            MyAnnotation myAnnotation = method.getDeclaredAnnotation(MyAnnotation.class);
            System.out.println(myAnnotation);
            System.out.println("myAnnotation's name is " + myAnnotation.name());
            System.out.println("myAnnotation's age is " + myAnnotation.age());
        } catch (ClassNotFoundException | NoSuchMethodException e) {
            e.printStackTrace();
        }
    }
```

**控制台输出:**

```
@PersonalTest$MyAnnotation(age=24, name=Alex)
@org.junit.Test(timeout=0, expected=class org.junit.Test$None)
**************************
@PersonalTest$MyAnnotation(age=24, name=Alex)
myAnnotation's name is Alex
myAnnotation's age is 24
```

### 2.2.2`getAnnotations`和`getDeclaredAnnotations`的区别

`AnnotationParent.java`

```java
    @MyAnnotation(name = "annotationTest")
    @Inheritable
    class AnnotationParent {

    }

```

`AnnotationChild.java`

```java
	@MyAnnotation(name = "annotationTest")
    class AnnotationChild extends AnnotationParent {

    }
```

`Test`代码

```java
	@Test
    public void test61() {
        try {
            Class<?> aClass = Class.forName("PersonalTest$AnnotationChild");
            //获取当前类及从父类继承而来的注解
            Annotation[] annotations = aClass.getAnnotations();
            for (Annotation annotation : annotations) {
                System.out.println(annotation);
            }
            System.out.println("***************************");
            //仅获取当前类的注解
            Annotation[] declaredAnnotation = aClass.getDeclaredAnnotations();
            for (Annotation annotation : declaredAnnotation) {
                System.out.println(annotation);
            }
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
```

**控制台输出:**

```
@PersonalTest$Inheritable()
@PersonalTest$MyAnnotation(age=24, name=annotationTest)
***************************
@PersonalTest$MyAnnotation(age=24, name=annotationTest)
```

### 2.2.3不同类上相同类型注解的比较

`AnnotationTest.java`

```
    @MyAnnotation(name = "AnnotationTest")
    class AnnotationTest {

    }
```

**测试代码:**

```java
	@Test
    public void test62() {
        try {
            //不同类上相同内容注解的比较
            MyAnnotation parent = Class.forName("PersonalTest$AnnotationParent").getDeclaredAnnotation(MyAnnotation.class);
            MyAnnotation child = Class.forName("PersonalTest$AnnotationChild").getDeclaredAnnotation(MyAnnotation.class);
            System.out.println(parent.equals(child));
            System.out.println("parent hashCode -> " + parent.hashCode());
            System.out.println("child hashCode -> " + child.hashCode());

            //不同类上不同内容注解的比较
            MyAnnotation myAnnotationTest = Class.forName("PersonalTest$AnnotationTest").getDeclaredAnnotation(MyAnnotation.class);
            System.out.println(parent.equals(myAnnotationTest));
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
```

**控制台输出:**

```
true
parent hashCode -> 1348781357
child hashCode -> 1348781357
false
```

**结论:**

* 不同类上相同内容的同一类型注解相同
* 不同类上不同内容的同一类型注解不同

## 2.3使用`Annotation`的示例

### 2.3.1标记`Annotation`

`Testable.java`

```java
/**
 * @Author Alex
 * @Date 2019/5/30 16:05
 * @Desc 定义一个标记注解,不包含任何成员变量
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Testable {
}
```

`TestableTest.java`

```java
/**
 * @Author Alex
 * @Date 2019/5/30 16:05
 * @Desc Testable测试类
 */
public class TestableTest {

    public void m1() {}

    @Testable
    public void m2() {
        throw new RuntimeException("系统出错了");
    }

    @Testable
    public void m3() {}

    public void m4() {}

    @Testable
    public void m5() {
        throw new RuntimeException("业务系统出错");
    }

    @Testable
    public void m6() {}

}
```

**注解处理方法(获取`TestableTest`中被`Testable`注解修饰的方法并进行执行和统计)**

```java
	@Test
    public void test63() throws ClassNotFoundException {
        int total = 0;
        int passed = 0;
        int failed = 0;

        Class<?> aClass = Class.forName("personal.TestableTest");
        Method[] methods = aClass.getMethods();
        for (Method method : methods) {
            if (method.isAnnotationPresent(annotations.Testable.class)) {
                total++;
                try {
                    //调用方法时需要传入实例
                    method.invoke(aClass.newInstance());
                    passed++;
                } catch (IllegalAccessException | InvocationTargetException | InstantiationException e) {
                    System.out.println("方法" + method + "运行失败,异常:" + e.getCause());
                    failed++;
                }
            }
        }
        System.out.println("Testable修饰的方法共有:" + total);
        System.out.println("passed:" + passed);
        System.out.println("failed:" + failed);
    }
```

**控制台输出:**

```
方法public void personal.TestableTest.m5()运行失败,异常:java.lang.RuntimeException: 业务系统出错
方法public void personal.TestableTest.m2()运行失败,异常:java.lang.RuntimeException: 系统出错了
Testable修饰的方法共有:4
passed:2
failed:2
```

### 2.3.2元数据`Annotation`

`ActionListenerFor.java`

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface ActionListenerFor {
    //定义一个成员变量,用于设置元数据.
    Class<? extends ActionListener> listener();
}
```

`ActionListenerInstaller.java`

```java
/**
 * 处理注解工具类
 */
public class ActionListenerInstaller {
    public static void processAnnotations(Object object) {
        //获取Class和属性数组
        Class<?> aClass = object.getClass();
        Field[] declaredFields = aClass.getDeclaredFields();

        for (Field field : declaredFields) {
            //将field中private设置为可访问
            field.setAccessible(true);
            //获取修饰属性的ActionListenerFor注解
            ActionListenerFor annotation = field.getAnnotation(ActionListenerFor.class);
            try {
                //获取Field对象
                Object fieldObj = field.get(object);
                //如果Field对象是AbstractButton的子类,并且被ActionListenerFor修饰,进入判断体
                if (annotation != null && fieldObj instanceof AbstractButton) {
                    //获取注解中的Listener class
                    Class<? extends ActionListener> listenerClazz = annotation.listener();
                    //生成Listener实例并将其绑定到Button中
                    ActionListener actionListener = listenerClazz.newInstance();
                    AbstractButton abstractButton = (AbstractButton) fieldObj;
                    abstractButton.addActionListener(actionListener);
                }
            } catch (IllegalAccessException | InstantiationException e) {
                e.printStackTrace();
            }

        }
    }
}
```

`AnnotationTest.java`

```java
/**
 * ActionListenerFor测试方法
 */
public class AnnotationTest {
    private JFrame mainWin = new JFrame("使用注解绑定事件监听器");

    @ActionListenerFor(listener = OKListener.class)
    private JButton ok = new JButton("确定");

    @ActionListenerFor(listener = CancelListener.class)
    private JButton cancel = new JButton("取消");

    //测试方法
    public void init() {
        JPanel jp = new JPanel();
        jp.add(ok);
        jp.add(cancel);
        mainWin.add(jp);
        ActionListenerInstaller.processAnnotations(this);
        mainWin.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        mainWin.pack();
        mainWin.setVisible(true);
    }

    //注意Swing编程需要在main方法中启动
    public static void main(String[] args) {
        new AnnotationTest().init();
    }
}

class OKListener implements ActionListener {

    @Override
    public void actionPerformed(ActionEvent e) {
        JOptionPane.showMessageDialog(null, "单击了确认按钮!");
    }
}

class CancelListener implements ActionListener {

    @Override
    public void actionPerformed(ActionEvent e) {
        JOptionPane.showMessageDialog(null, "单击了取消按钮");
    }
}
```

**运行`main`方法,点击按钮,查看对应弹窗**

## 2.4`Java8`新增的重复注解

`CJTag.java`

```java
/**
 * 定义可重复注解
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
//若要重复使用该注解,需要使用如下注解指明其容器注解
@Repeatable(CJTags.class)
public @interface CJTag {
    String name();
    int age();
}
```

`CJTags.java`

```java
/**
 * CJTag的容器注解
 */
//容器注解的保留期必须比它所包含的注解的保留期更长,否则编译器会报错.
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface CJTags {
    //数组方法名必须为value()
    CJTag[] value();
}
```

**调用重复注解,`Java8`之前的写法**

```java
@CJTags({@CJTag(name = "Alex", age = 24), @CJTag(name = "Wang", age = 25)})
class CJTagTest1 {
}
```

**`Java8`新增的重复注解**

```java
@CJTag(name = "Alex", age = 24)
@CJTag(name = "Wang", age = 25)
class CKTagTest2 {

}
```

**测试代码:**

```java
	@Test
    public void test64() {
        try {
            Class<?> clazz = Class.forName("PersonalTest$CKTagTest2");
            //getDeclaredAnnotationsByType是getDeclaredAnnotation的升级版,可以获取多个重复注解
            CJTag[] cjTags = clazz.getDeclaredAnnotationsByType(CJTag.class);
            for (CJTag cjTag : cjTags) {
                System.out.println(cjTag);
            }
            System.out.println("================");
            //只能获取一个注解
            CJTags cjtag = clazz.getDeclaredAnnotation(CJTags.class);
            System.out.println(cjtag);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
```

```
@annotations.CJTag(name=Alex, age=24)
@annotations.CJTag(name=Wang, age=25)
================
@annotations.CJTags(value=[@annotations.CJTag(name=Alex, age=24), @annotations.CJTag(name=Wang, age=25)])
```

## 2.5`Java8`新增的`Type Annotation`

**自定义`Type Annotation`**

```java
	//自定义类型注解,设置Target为ElementType.TYPE_USE
    @Target(ElementType.TYPE_USE)
    @interface NotNull {
    }
```

**使用示例:**

```java
public void test65(@NotNull String string) throws @NotNull FileNotFoundException {
	String str = new @NotNull String("123412");
}
```

