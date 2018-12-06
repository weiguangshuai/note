# Java注解

> 注解，也被称为元数据，为我们在代码中添加信息提供一种形式化的方法，使我们在稍后某个时刻非常方便的使用这些数据。
>
> 注解是在实际的源代码级别保存所有的信息，而不是某种注释性的文字，这使得代码更整洁，且便于维护。

### 注解的语法

#### 定义

注解的定义很和接口的定义很像，只是把定义接口的饰符`intertface`换成了`@interface`。而且，与其他的`Java`接口一样，注解也会编译成`class`文件。下面是一个注解的示例：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {

}
```

我们定义了一个名为`TestAnnotation`的注解，注解中一般会包含一些元素以表示某些值，当分析这些注解时，程序可以使用到这些值，`TestAnnotation`注解没有元素，我们称这种注解为`标记注解`；下面是一个有元素的注解：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface UserCase {
    public int id();
    public String desc() default "no desc";
}
```

#### 元注解

从上面两个注解中我们发现除了`@interface`以外，我们还发现上面有两个注解，这两个注解为`元注解`，它的作用是负责注解其他的注解，`Java`内部内置了四种元注解。这四种注解分别是：`@Target`、`@Retention`、`@Documented`和`@Inherited`；

- `@Target`：表示注解可以用于什么地方，可能的`ElemenType`参数包括：
  - `TYPE`：用于类、接口或者`enum`上
  - `PARAMETER`：用于参数上
  - `PACKAGE`：用于包上
  - `METHOD`：用于方法上
  - `FIELD`：用于域声明
  - `LOCAL_VARIABLE`：用于局部变量声明
  - `CONSTRUCTOR`：用于构造器的声明
- `@Retention`：表示需要在什么级别保存该注解信息，可选的`RetentionPolicy`参数如下
  - `SOURCE`：注解将被编译器丢弃
  - `CLASS`：注解在`class`文件中可用，但是会被`VM`丢弃
  - `RUNTIME`：`VM`将在运行期间保留注解，因此可以通过反射机制读取注解的信息
- `@Documented`：将此注解包含在`Javadoc`中
- `@Inherited`：允许子类继承父类中的注解

### 注解处理器

> 如果没有用来读取注解的工具，那么注解连注释都不如。在使用注解的过程中，很重要的一个部分就是创建和使用`注解处理器`

下面我们先编写一个简单的注解处理器，然后通过对注解处理的讲解进一步认识和学习怎么编写一个注解处理器，以及编写注解处理器时的注意事项

```java
//注解
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface UserCase {
    public int id();
    public String desc() default "no desc";
}
//注解处理器
public class UseCaseTracker {
    private static void trackUseCases(List<Integer> useCases, Class<?> cl) {
        for (Method method : cl.getDeclaredMethods()) {
            UseCase useCase = method.getAnnotation(UseCase.class);
            if (useCase != null) {
                System.out.println("Found Use Case: " + useCase.id() + " " + useCase.desc());
                //由于list中是Integer类型，所以移除的时候需要使用以下方式，不然会被认定为角标，从而出错
                useCases.remove(new Integer(useCase.id()));
            }
        }
        for (int i : useCases) {
            System.out.println("Warning: Missing use case-" + i);
        }
    }

    public static void main(String[] args) {
        List<Integer> useCases = new ArrayList<>();
        Collections.addAll(useCases, 47, 48, 49, 50);
        trackUseCases(useCases, PasswordUtils.class);
    }
}

```

上面代码是一个注解和一个注解处理器，从注解上看，此注解包含`int`元素`id`，以及一个`String`元素`desc`，然而注解的元素不单只有这两种类型，注解元素可用的类型如下：

- 所有的基本类型(`int`, `float`，`boolean`等)
- `String`
- `Class`
- `enum`
- `Annotation`
- 以上类型的数组

在注解的第二个元素我们发现，`desc`后有一个`default`修饰符，这是对注解元素的默认值的限制。对于注解来说，元素要么使用默认值，要么必须在使用的时候提供元素的值；

像上面的`UseCase`注解，元素`id`没有使用默认值，那么在使用的时候就必须要提供一个元素的值；而对于元素`desc`则使用了默认值，那么我们可以在使用的时候不提供元素的值，如果在使用的时候提供元素的值，那么将会覆盖掉默认值。

> 注意：不能以null作为默认值，如果需要表示某个元素不存在，可以使用空字符串或负数来表示





