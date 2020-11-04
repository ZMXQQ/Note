# EXPLAIN命令介绍及使用

## 0.概念

EXPLAIN命令可以查看查询优化器是如何执行查询的。



## 1.使用

要使用explain，只需再select关键字之前加上explain，例如：

```mysql
EXPLAIN SELECT * 
FROM staff s
WHERE s.staff_id = '11111111'
```

这样就可以返回执行计划中每一步的信息和执行的次序，而并不会执行它。

![image-20201029170413462](C:%5CUsers%5C%E6%9D%8E%E4%BD%B3%E5%BA%86%5CDesktop%5C%E7%AC%94%E8%AE%B0%E6%9C%AC%5C%E5%9B%BE%E7%89%87%E5%BA%93%5Cimage-20201029170413462.png)

