# Spring Session

## 使用

* 引入maven

```
        <dependency>
            <groupId>org.springframework.session</groupId>
            <artifactId>spring-session-data-redis</artifactId>
        </dependency>
```

* 代码

```
    @RequestMapping("/doLogin")
    public Object doLogin(@RequestBody Map<String,String> map, HttpSession httpSession){
        System.out.println(JSON.toJSONString(map));
        httpSession.setAttribute("username",map.get("username"));
        return "SUCCESS";
    }
```

* 效果

```
127.0.0.1:6379> keys *
1) "spring:session:expirations:1595005920000"
2) "spring:session:sessions:expires:23926a94-eb3a-435b-8ab4-04885ae461a7"
3) "spring:session:sessions:23926a94-eb3a-435b-8ab4-04885ae461a7"
```

