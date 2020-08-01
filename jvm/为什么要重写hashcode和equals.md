# 为什么要重写hashcode和equals

## 为什么要重写equals方法

因为object中的equals()方法比较的是对象引用地址是否相等,如果你要判断对象里面的内容是否相等,则需要重写equals方法

## java中那些类重写了equals方法

java中绝大多数类都重写了equals方法,没有重写的类大部分都是自己定义的类

## hashcode方法的作用

hashcode生成散列值,根据散列值可以判断对象中是否存在该值,如果不存在则插入,如果存在,则再通过equals方法判断,这样可以提高效率.

## 重写equals方法为什么要同时重写hashcode方法

为了保证当两个对象通过equals方法比较相等时,那么他们的hashcode值也一定要保持相等.

## 未重写hashcode和equals方法的案例

```
public class Person {
    private String code;
    private String name;

    public Person() {
    }

    public Person(String code, String name) {
        this.code = code;
        this.name = name;
    }

    public String getCode() {
        return code;
    }

    public void setCode(String code) {
        this.code = code;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```



```
public class PersonService {

    public static void countNum(List<Person> list){
        Map<Person,Integer> map = new HashMap<>();
        for(Person person : list){
            if(map.containsKey(person)){
                map.put(person,map.get(person)+1);
            }else{
                map.put(person,1);
            }
        }
        System.out.println(map.size());
    }

    public static void main(String[] args) {
        Person p1 = new Person("1","aaa");
        Person p2 = new Person("2","bbbb");
        Person p3 = new Person("1","aaa");

        countNum(Arrays.asList(p1,p2,p3));
    }
}
```

如果不重写hashcode和equals方法,则返回值是3条;

如果重写了hashcode和equals方法,则返回值是2条;