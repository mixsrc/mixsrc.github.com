---
title: Java8 中 Arrays.sort 注意事项
categories: 开发
toc: false
permalink: /java8-arrays-sort.html
---

有一个比较复杂的业务排序方法，原来在 Java 6 上运行正常，但升级到 Java 8 后，偶尔出现以下异常：

```text
Exception in thread "main" java.lang.IllegalArgumentException: Comparison method violates its general contract!
    at java.util.TimSort.mergeHi(TimSort.java:895)
    at java.util.TimSort.mergeAt(TimSort.java:512)
    at java.util.TimSort.mergeCollapse(TimSort.java:437)
    at java.util.TimSort.sort(TimSort.java:241)
    at java.util.Arrays.sort(Arrays.java:1438)
```

查找资料，发现 JDK7+ 版本默认使用了 TimSort，要求 Comparator 要满足自反性，传递性，对称性：

- 自反性：`comparator(x, y) = -comparator(y, x)`
- 传递性：`x > y, y > z , 则 x > z`
- 对称性：`x = y, 则 comparator(x, z) == comparator(y, z)`

网上收集到两个可以复现的示例，`x > y ? 1 : -1` 的问题在于当 x == y 时，不满足自反性。示例如下：

```java
Integer[] array = {5, 1, 9, 5, 1, 5, 11, 5, 5, 1, 9, 5, 1, 5, 11, 5, 5, 1, 9, 5, 1, 5, 11, 5, 5, 1, 9, 5, 1, 5, 11, 5, 5, 1, 9, 5, 1, 5, 11, 5};

Integer[] array = {0, 0, 0, 0, 0, 0, 0, 3, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 0, 0, 1, 0, 1, 0, 0, 0, 0, 1, 0, 0, 1, 0, 0, 0, 2, 1, 0, 0, 0, 2, 30, 0, 3};

Arrays.sort(array, (a, b) -> a > b ? 1 : -1);
```

查看 Arrays.sort 的源码：

```java
public static <T> void sort(T[] a, Comparator<? super T> c) {
    if (c == null) {
        sort(a);
    } else {
        if (LegacyMergeSort.userRequested)
            legacyMergeSort(a, c);
        else
            TimSort.sort(a, 0, a.length, c, null, 0, 0);
    }
}

static final class LegacyMergeSort {
    private static final boolean userRequested =
        java.security.AccessController.doPrivileged(
            new sun.security.action.GetBooleanAction(
                "java.util.Arrays.useLegacyMergeSort")).booleanValue();
}
```

那么解决办法有三：

1. 重新编写 Comparator 方法，使之满足自反性，传递性，对称性
2. JDK启动参数加 `-Djava.util.Arrays.useLegacyMergeSort=true` 使用 JDK6 的比较算法；
3. 代码中使用 `System.setProperty(“java.util.Arrays.useLegacyMergeSort”, “true”)` 使用 JDK6 的比较算法，因为 LegacyMergeSort.userRequested 是一个静态常量，所以必须在LegacyMergeSort加载前setProperty，最好在 Main 方法第一行就设置。
