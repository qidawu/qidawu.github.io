---
title: JSON 框架系列之 Jackson 总结
date: 2017-02-10 22:09:48
updated:
tags: Java
typora-root-url: ..
---

# FasterXML Jackson

https://github.com/FasterXML/jackson

## JsonNode 的使用

`com.fasterxml.jackson.databind.JsonNode` 表示一个 JSON 节点，可以通过 `ObjectMapper#readTree` 方法解析出来，也可以通过 `JsonNode` 的子类 API 自定义构建：

![JsonNode](/img/java/jackson/JsonNode.png)

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

`JsonNode` 构建完成后，可以灵活的读取其值，例如：

```java
// [value0, value1]
log.info(jsonNode.get("hello").findValuesAsText("key").toString());
// value3
log.info(jsonNode.get("test").get("key3").asText());
```
## ObjectMapper 工具类

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
        //     fromJson(java.lang.String, com.fasterxml.jackson.databind.JavaType)
        return fromJson(text, OBJECT_MAPPER.getTypeFactory().constructParametricType(ArrayList.class, type));
    }

}
```

## JavaType 的使用

工具类的方法 `fromJsonToList`，使用了方法 `OBJECT_MAPPER.getTypeFactory().constructParametricType(...)`，其值调试如下：

![](/img/java/jackson/CollectionType.png)

这个值为 `JavaType` 的子类 `CollectionType`，可用于调用方法 `ObjectMapper#readValue(java.lang.String, com.fasterxml.jackson.databind.JavaType)` 进行集合类型的 JSON 转换：

![JavaType](/img/java/jackson/JavaType.png)

## 使用例子

本例中，我们需要获取以下两个方法的泛型返回值中的实际类型参数 `XxxRespDTO` 的 `Class` 类型，以用于 JSON 转换：

```java
public interface ApiService {

    /**
     * 接口一
     */
    RespDTO<XxxRespDTO> get(XxxReqDTO reqDTO);

    /**
     * 接口二
     */
    RespDTO<List<XxxRespDTO>> list(XxxReqDTO reqDTO);
}
```

定义一个方法，用于转换 JSON：

```java
private Object getObject(Method method, String json) {
    Object result;
    
    // 获取该方法的泛型返回值，然后获取其第一个实际类型参数
    ParameterizedType returnType = (ParameterizedType) method.getGenericReturnType();
    Type type = returnType.getActualTypeArguments()[0];
    
    // 处理接口一的情况
    if (type instanceof Class) {
        Class<?> clazz = (Class<?>) type;
        result = JsonUtils.fromJson(json, clazz);  // ObjectMapper#readValue(String, Class<T>)
    } else if (type instanceof ParameterizedType) {
        ParameterizedType nestedReturnType = (ParameterizedType) type;
        // 处理接口二的情况
        if (nestedReturnType.getRawType() == List.class) {
            Class<?> clazz = (Class<?>) nestedReturnType.getActualTypeArguments()[0];
            result = JsonUtils.fromJsonToList(decryptedRespData, clazz);
        } else {
            throw new IllegalStateException("未实现的 JSON 解析！");
        }
    } else {
        throw new IllegalStateException("未实现的 JSON 解析！");
    }
    return result;
}
```

这种用法常常出现在框架之中。下面来看下调试效果：

### 接口一

下图展示了变量 `returnType` 为参数化类型 `ParameterizedType`，其实际类型参数 `type` 为 `Class` 类型，值为 `XxxRespDTO` ：

![JsonNode](/img/java/jackson/ObjectMapper_example.png)

### 接口二

下图展示了变量 `returnType` 的实际类型参数 `type` 与接口一为 `Class` 类型不同，接口二为 `ParameterizedType` 参数化类型，值为 `List<XxxRespDTO>`：

![](/img/java/jackson/ObjectMapper_example_2.png)

## 常见报错

### Unrecognized field, not marked as ignorable

该错误的意思是说，不能够识别的字段没有标示为可忽略。出现该问题的原因就是 JSON 中包含了目标 Java 对象没有的属性。

解决方法有如下几种：

1. 保证传入的 JSON 串不包含目标对象的没有的属性。
2. `@JsonIgnoreProperties(ignoreUnknown = true)` 在目标对象的类级别上加上该注解和属性，则 Jackson 在反序列化的时候，会忽略该目标对象不存在的属性。
3. 全局 `DeserializationFeature` 配置
   `objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES,false)` 配置该 `objectMapper` 在反序列化时，忽略目标对象没有的属性。凡是使用该 `objectMapper` 反序列化时，都会拥有该特性。

# Google Gson

https://github.com/google/gson

```java
JsonObject jsonObject = new JsonObject();
jsonObject.addProperty("time", LocalDateTime.now().toString());
jsonObject.addProperty("ip", ip);
jsonObject.addProperty("rspMillis", rspMillis);
jsonObject.addProperty("rspCode", rspCode);
jsonObject.addProperty("method", method);
jsonObject.addProperty("path", uriPath);
// Deprecated
// new JsonParser().parse(jsonTrace);
jsonObject.add("body", JsonParser.parseString(jsonTrace));
requestJsonLogger.info(jsonObject.toString());
```

