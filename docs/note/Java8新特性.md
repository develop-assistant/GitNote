# Java8新特性

## Lambda表达式

## 函数式接口

## 接口的默认方法和静态方法

**默认方法**使得开发者可以在 不破坏二进制兼容性的前提下，往现存接口中添加新的方法，即不强制那些实现了该接口的类也同时实现这个新加的方法。

默认方法和抽象方法之间的区别在于抽象方法需要实现，而默认方法不需要。接口提供的默认方法会被接口的实现类继承或者覆写，例子代码如下：

```java
public interface Interface8 {

    default String defaultInterf() {
        return "default";
    }
}
```

```java
public class Java8Demo implements Interface8 {

    /**
     * 该方法非必须实现，不实现走父类默认方法，覆盖则走子类实现。
     * @return
     */
    @Override
    public String defaultInterf() {
        return "child";
    }

    public static void main(String[] args) {
        Java8Demo demo = new Java8Demo();
        String tes = demo.defaultInterf();
        System.out.println(tes);
    }
}
```



## 方法和构造函数引用

## Streams(流)

- Filter(过滤)
- Sorted(排序)
- Map(映射)
- Match(匹配)
- Count(计数)
- Reduce(规约)

## Data API(日期相关API)