## ConcurrentModificationException

​	java.util.ConcurrentModificationException：并发修改异常

### 1、测试代码

```
public static void main(String[] args) {
    List<String> list = new ArrayList<>();
    list.add("11");
    for (String str : list) {
        if (str.equals("11")) list.remove(str);
    }
}
```

![image-20200708225824319](img\image-20200708225824319.png)

### 2、出现的原因



### 3、单线程环境下的解决办法



### 4、多线程环境下的解决方法

> 1）在使用iterator迭代的时候使用synchronized或者Lock进行同步；



> 2）使用并发容器CopyOnWriteArrayList代替ArrayList或Vector



