## Java 面试题

记录日常遇到的 Java 问题。

## 访问权限

```java
public > protected > default > private
```

## 重写与重载

+ 重写（Override）：发生于父类与子类之间，要求函数名、参数列表均相同，返回值有限制，并使用 `@Override` 关键字。子类中的重写方法访问权限要大于等于父类的方法。也称为「覆盖率」（无语的翻译）。
+ 重载（Overload）：发生于同一个类当中，要求函数名相同，参数列表不同，返回值可以不同。

对比总结：

|          |                       重写（Override）                       | 重载（Overload） |
| :------: | :----------------------------------------------------------: | :--------------: |
| 发生范围 |                        父类与子类之间                        |    同一个类中    |
|  函数名  |                             相同                             |       相同       |
| 参数列表 |                             相同                             |     必须不同     |
|  返回值  | JDK8 之前要求必须相同，后面允许子类重写的方法的**返回值是父类方法返回值的子类** |     允许不同     |
| 访问权限 |                  子类重写的方法 >= 父类方法                  |     没有要求     |
|   异常   |                      规则与「返回值」同                      |     没有要求     |

重写例子（注意返回值）：

```java
public class Test {
    public static class Father {
        public Object say() {
            System.out.println("father");
            return new Object();
        }
    }

    public static class Son extends Father {
        @Override
        public Integer say() {
            System.out.println("Son");
            return Integer.parseInt("0");
        }
    }

    public static void main(String[] args) {
        Son son = new Son();
        son.say();
    }
}
```

重载例子：

```java
public class Test {
    public void say()
    {
        System.out.println("Hello");
    }

    public int say(String name)
    {
        System.out.println("Hello by " + name);
        return 0;
    }

    public static void main(String[] args) {
        Test test = new Test();
        test.say();
        test.say("sinkinben");
    }
}
```

面试题：重载（Overload）与重写（Override）的区别。

答：方法的重载和重写都是实现多态的方式，区别在于前者实现的是编译时的多态性，而后者实现的是运行时的多态性。重载发生在一个类中，同名的方法如果有不同的参数列表（参数类型不同、参数个数不同或者二者都不同）则视为重载；重写发生在子类与父类之间，重写要求子类被重写方法与父类被重写方法有相同的参数列表，有兼容的返回类型，比父类被重写方法更好访问，不能比父类被重写方法声明更多的异常（里氏代换原则）。重载对返回类型没有特殊的要求，不能根据返回类型进行区分。