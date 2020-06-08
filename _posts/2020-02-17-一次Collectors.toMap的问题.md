---
layout:     post
title:      一次Collectors.toMap的问题
subtitle:   Collectors.toMap
date:       2020-02-17
author:     Square
header-img: img/wallhaven-eypero.jpg
catalog: true
tags:
    - Java8
---

## Collections.toMap作用
将list按照规则转成map
```java
        UserVO userVO1 = new UserVO("", "b1", "c1", "d1");
        UserVO userVO2 = new UserVO(null, "b1", "c1", "d1");
        UserVO userVO3 = new UserVO("A1", "b1", "c1", "d1");
        UserVO userVO4 = new UserVO("A2", "b1", "c1", "d1");
        UserVO userVO5 = null;
        UserVO userVO6 = new UserVO();
        UserVO userVO7 = new UserVO();
        List<UserVO> voList = Arrays.asList(userVO1, userVO2, userVO3,userVO4,userVO5,userVO6,userVO7);
        Map<String, String> stringMap = Optional.ofNullable(voList)
                .map(Collection::stream)
                .orElse(Stream.empty())
                .collect(Collectors.toMap(UserVO::getId, UserVO::getUsername));
```

## Collections.toMap 存在的问题
1. 空指针（map中value是null导致,map中key可以为null）
2. key值重复（Map中的key不能重复）
### 1.解决方式
使用stream的collect的重载方法：
```java
        Map<String, String> stringHashMap = Optional.ofNullable(voList)
                .map(Collection::stream)
                .orElse(Stream.empty())
                .collect(HashMap::new, (m, v) ->
                        m.put(v.getId(), v.getUsername()), HashMap::putAll);
        System.err.println(new Gson().toJson(stringHashMap));
        //{"":"b1","A1":"b1","A2":"b1"}
```
### 2.解决方式
过滤Map中key，value不为空
```java
          Map<String, String> stringMap2 = Optional.ofNullable(voList)
                        .map(Collection::stream)
                        .orElse(Stream.empty())
                        .filter(Objects::nonNull)
                        .filter(x -> StrUtil.isNotBlank(x.getId()))
                        .filter(x -> StrUtil.isNotBlank(x.getUsername()))
                        .collect(Collectors.toMap(UserVO::getId,
                                UserVO::getUsername,
                                (left, right) -> right));
                System.err.println(new Gson().toJson(stringMap));
                //{"A1":"b1","A2":"b1"}
```
### 3.解决方式
```java
     Map<String, String> hashMap = Optional.ofNullable(voList)
                .map(Collection::stream)
                .orElse(Stream.empty())
               .filter(Objects::nonNull)
                .filter(x -> StrUtil.isNotBlank(x.getId()))
                .filter(x -> StrUtil.isNotBlank(x.getUsername()))
                .collect(Collectors.toMap(
                        UserVO::getId,
                        UserVO::getUsername,
                        (left, right) -> right,
                        HashMap::new));
        System.err.println(new Gson().toJson(hashMap));
        //{"A1":"b1","A2":"b1"}
```