# Java8新特性

## Lambda表达式

## 函数式接口

## 接口的默认方法和静态方法

**默认接口**

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