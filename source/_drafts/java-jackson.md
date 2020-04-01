---
title: JSON 框架系列之 Jackson 总结
date: 2017-02-10 22:09:48
updated:
tags: Java
typora-root-url: ..
---

# JsonNode

`com.fasterxml.jackson.databind.JsonNode` 表示。。。，可以通过 `ObjectMapper#readTree` 方法解析出来，也可以通过 `JsonNode` 的子类自定义构建：

![JsonNode](/img/jackson/JsonNode.png)

构建代码如下：

```java
JsonNode jsonNode =
    new ObjectNode(JsonNodeFactory.instance, ImmutableMap.of(
        "hello",
        new ArrayNode(JsonNodeFactory.instance, ImmutableList.of(
            new ObjectNode(JsonNodeFactory.instance, ImmutableMap.of("key", new TextNode("value0"))),
            new ObjectNode(JsonNodeFactory.instance, ImmutableMap.of("key", new TextNode("value1"))),
            new ObjectNode(JsonNodeFactory.instance, ImmutableMap.of("key2", new TextNode("value2")))
        )),
        "test",
        new ObjectNode(JsonNodeFactory.instance, ImmutableMap.of(
            "key3", new TextNode("value3")))));

// {"hello":[{"key":"value0"},{"key":"value1"},{"key2":"value2"}],"test":{"key3":"value3"}}
log.info(jsonNode.toString());
```

`JsonNode` 构建完成后，可以通过灵活的读取其值，例如：

```java
// [value0, value1]
log.info(jsonNode.get("hello").findValuesAsText("key").toString());
// value3
log.info(jsonNode.get("test").get("key3").asText());
```

# 工具类

```java
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.*;
import lombok.experimental.UtilityClass;
import lombok.extern.slf4j.Slf4j;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;

/**
 * jackson工具类
 **/
@Slf4j
@UtilityClass
public final class JsonUtils {
    private static final ObjectMapper OBJECT_MAPPER = new ObjectMapper();

    static {
        //false: 空对象不抛异常 e.g. handler{ }
        OBJECT_MAPPER.configure(SerializationFeature.FAIL_ON_EMPTY_BEANS, false);
        //false: 解析JSON时忽略未知属性
        OBJECT_MAPPER.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        //true :空字符串可以解析为空对象
        OBJECT_MAPPER.configure(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT, true);
    }

    private ObjectMapper getObjectMapper() {
        return OBJECT_MAPPER;
    }

    /**
     * @param obj POJO
     * @param <T> 泛型
     * @return JSON
     */
    public static <T> String toJson(T obj) {
        try {
            return getObjectMapper().writeValueAsString(obj);
        } catch (Exception e) {
            log.error("to json failure", e);
            throw new IllegalArgumentException(e);
        }
    }

    /**
     * @param text JSON
     * @param type Class<?>
     * @param <T>  泛型
     * @return POJO
     */
    public static <T> T fromJson(String text, Class<T> type) {
        try {
            return getObjectMapper().readValue(text, type);
        } catch (IOException e) {
            log.error("fromJson failure", e);
            throw new IllegalArgumentException(e);
        }
    }

    public static JsonNode fromJson(String text) {
        try {
            return getObjectMapper().readTree(text);
        } catch (IOException e) {
            log.error("from json failure", e);
            throw new IllegalArgumentException(e);
        }
    }

    public static <T> T fromJson(String text, JavaType javaType) {
        try {
            return getObjectMapper().readValue(text, javaType);
        } catch (IOException e) {
            log.error("from json failure", e);
            throw new IllegalArgumentException(e);
        }
    }

    public static <T> T fromJson(String text, TypeReference<T> valueTypeRef) {
        try {
            return getObjectMapper().readValue(text, valueTypeRef);
        } catch (IOException e) {
            log.error("from json failure", e);
            throw new IllegalArgumentException(e);
        }
    }

    /**
     * @param src  JSON字节
     * @param type Class<?>
     * @param <T>  泛型
     * @return POJO
     */
    public static <T> T fromByte(byte[] src, Class<T> type) {
        try {
            return getObjectMapper().readValue(src, OBJECT_MAPPER.constructType(type));
        } catch (IOException e) {
            log.error("from byte failure", e);
            throw new IllegalArgumentException(e);
        }
    }

    /**
     * @param map  Map
     * @param type Class<?>
     * @param <T>  泛型
     * @return POJO
     */
    public static <T> T fromMap(Map<String, Object> map, Class<T> type) {
        try {
            return getObjectMapper().convertValue(map, OBJECT_MAPPER.getTypeFactory().constructType(type));
        } catch (IllegalArgumentException e) {
            log.error("fromMap failure", e);
            throw new IllegalArgumentException(e);
        }
    }

    /**
     * @param text JSON
     * @param type Class<?>
     * @param <T>  T
     * @return List
     */
    public static <T> List<T> fromJsonToList(String text, Class<T> type) {
        return fromJson(text, OBJECT_MAPPER.getTypeFactory().constructParametricType(ArrayList.class, type));
    }

}
```

