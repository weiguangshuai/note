# Java异常总结

结构图:

![](https://github.com/weiguangshuai/note/blob/master/%E5%9B%BE%E5%BA%8A/exception.png)

## Error


>Error用来表示编译时和系统错误，一般我们不关系。异常是发生错误时被抛出的一个通知，所以Error是在编译时和系统错误时发出的通知

## Exception

>这是我们平时所关心的异常，因为这个异常发生的时候，说明我们的程序出现了问题，我们需要处理这些问题，Exception又分为`检查异常`和`不检查异常`

### 不检查异常

我们平时见到的`RuntimeException`就是不检查异常,不检查异常不需要我们去手动抛出异常，系统会自动帮你检测并在出现异常的时候抛出异常,当抛出不检查异常时，程序会自动停止

### 检查异常

这一类异常需要我们去捕获它，是我们平时代码中经常遇到的，如SQLException、XMLStreamException等异常。至于说如何处理，根据具体的业务逻辑来编写处理的代码

#### 处理检查异常

有两种方式去处理异常：

1.使用try块去捕获异常，这种方式处理异常不会导致程序的停止，后面的代码可以正常运行，可以在catch块中增加处理方式。当程序不抛出异常时，将不会运行catch块中代码

2.将异常抛出去，交给上一层去处理，这种方式会导致异常后面的代码不能正常运行

#### 继承中方法说明限制

在继承了某个类时，如果父类中某个方法有异常说明，子类在覆盖方法时，异常说明只能和父类一样或者无异常说明，不能添加父类没有的异常说明

```Java
class A{
    void a() throws AExcetpion{
        ...
    }
}

class B extends A {
    @Override
    void a(){
        ...
    }
    //或者
    void a() throws AExcetpion {
        ...
    }
}
```
#### 异常链

把异常发生的原因一个个串起来，把底层的异常信息传给上层，这样逐层抛出;通过异常链，我们可以提高代码的可理解性、系统的可维护性和友好性。

```Java
public class ExceptionChain {
    public static void main(String[] args) {
        try {
            B();
        } catch (MyException e) {
            e.printStackTrace();
        }
    }

    public static void A() throws MyException {
        try {
            FileReader fileReader = new FileReader("ddd.txt");
        } catch (FileNotFoundException e) {
            throw new MyException("文件没有找到--01", e);
        }
    }

    public static void B() throws MyException {
        try {
            A();
        } catch (MyException e) {
            throw new MyException("文件没有找到--02", e);
        }
    }
}

class MyException extends Exception {

    public MyException() {
    }

    public MyException(String message, Throwable cause) {
        super(message, cause);
    }
}
```
解决异常被抑制的情况，如下代码：

```Java
public class ExceptionTest{
    public void test() throws InterruptedException {
        try {
            Exception exception = null;
            try {
                A();
            } catch (Exception e) {
                exception = e;
            } finally {
                try {
                    B();
                } catch (BException e) {
                    if (exception != null) {
                        exception.addSuppressed(e);
                    } else {
                        exception = e;
                    }
                }
                if (exception != null) {
                    throw exception;
                }
            }
        } catch (Exception e) {
            System.out.println(e);
            for (Throwable throwable : e.getSuppressed()) {
                System.out.println(throwable);
            }
        }
    }

    static void A() throws AException {
        throw new AException();
    }

    static void B() throws BException {
        throw new BException();
    }
}

class AException extends Exception {
    @Override
    public String toString() {
        return "AException{}";
    }
}

class BException extends Exception {
    public BException(Throwable cause) {
        super(cause);
    }

    public BException() {

    }

    @Override
    public String toString() {
        return "BException{}";
    }
}


输出结果：
AException{}
BException{}
```

#### try、catch、finally语句块的执行顺序

1.当try没有捕获到异常时：try语句块中的语句逐一被执行，程序将跳过catch语句块，执行finally语句块和其后的语句

2.当try捕获到异常，catch语句块里没有处理此异常的情况：当try语句块里的某条语句出现异常时，而没有处理此异常的catch语句块时，此异常将会抛给JVM处理，finally语句块里的语句还是会被执行，但finally语句块后的语句不会被执行

3.当try捕获到异常，catch语句块里有处理此异常的情况：在try语句块中是按照顺序来执行的，当执行到某一条语句出现异常时，程序将跳到catch语句块，并与catch语句块逐一匹配，找到与之对应的处理程序，其他的catch语句块将不会被执行，而try语句块中，出现异常之后的语句也不会被执行，catch语句块执行完后，执行finally语句块里的语句，最后执行finally语句块后的语句

#### try、catch、finally语句中的return

1.try和finally中有return，最后返回的值是finally中return返回的值
```Java
public class ReturnAboutTryCatchFinally {
    @Test
    public void testTryFinally {
        System.out.println(tryFinally());
    }

    public int tryFinally() {
        try {
            return 1;
        } finally {
            return 2;
        }
    }
}
```

2.try、catch和finally都有return，最后返回值还是finally中return返回的值

```Java
public class Test{

    @Test
    public void testTryCatchFinally() {
        System.out.println(tryCatchFinally());
    }

    public int tryCatchFinally() {
        try {
            int i = 10 / 0;
            return 1;
        } catch (Exception e) {
            return 2;
        } finally {
            return 3;
        }
    }
}
```
