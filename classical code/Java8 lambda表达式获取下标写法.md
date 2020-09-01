# Java8 lambda表达式获取下标写法

方法1:

```
List<String> list = new ArrayList<>();
list.add("1");
list.add("2");
list.add("3");
list.add("4");
list.add("5");
 
Stream.iterate(0, i -> i + 1).limit(list.size()).forEach(i -> {
	System.out.println(list.get(i));
});
```



方法2:

```
List<String> list = new ArrayList<>();
list.add("1");
list.add("2");
list.add("3");
list.add("4");
list.add("5");
 
IntStream.range(0,list.size()).forEach(i->{
    System.out.println(list.get(i));
});
```

