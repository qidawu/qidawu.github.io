---
title: JSON 框架系列之 Jackson 总结
date: 2020-12-10 22:09:48
updated:
tags: Java
typora-root-url: ..
---

https://github.com/FasterXML/jackson

https://github.com/FasterXML/jackson-docs

# Core modules

> Core modules are the foundation on which extensions (modules) build upon. There are 3 such modules currently (as of Jackson 2.x):
>
> - [Streaming](https://github.com/FasterXML/jackson-core) ([docs](https://github.com/FasterXML/jackson-core/wiki)) ("jackson-core") defines low-level streaming API, and includes JSON-specific implementations
> - [Annotations](https://github.com/FasterXML/jackson-annotations) ([docs](https://github.com/FasterXML/jackson-annotations/wiki)) ("jackson-annotations") contains standard Jackson annotations
> - [Databind](https://github.com/FasterXML/jackson-databind) ([docs](https://github.com/FasterXML/jackson-databind/wiki)) ("jackson-databind") implements data-binding (and object serialization) support on `streaming` package; it depends both on `streaming` and `annotations` packages

## Annotations

https://github.com/FasterXML/jackson-annotations

[这个页面](https://github.com/FasterXML/jackson-annotations/wiki/Jackson-Annotations)列出了所有 Jackson 2.0 通用注解，按功能分组。

常用注解如下：

* `@JsonProperty`
* `@JsonIgnore`
* `@JsonIgnoreProperties`
* `@JsonInclude`
* `@JsonFormat`
* `@JsonSerialize`
* `@JsonDeserialize`
* ...

## Databind

For all data-binding, we need a [`com.fasterxml.jackson.databind.ObjectMapper`](https://fasterxml.github.io/jackson-databind/javadoc/2.9/com/fasterxml/jackson/databind/ObjectMapper.html) instance:

```java
ObjectMapper mapper = new ObjectMapper(); // create once, reuse
```

参考：

《[ObjectMapper，别再像个二货一样一直 new 了！](https://mp.weixin.qq.com/s/SHbt1jmgGaHQs1eeyJQ-qA)》

### Configuration

https://github.com/FasterXML/jackson-databind/wiki/#reference-manual

### POJO

The most common usage is to take piece of JSON, and construct a Plain Old Java Object ("POJO") out of it.

Serialization:

```java
MyValue value = mapper.readValue(new File("data.json"), MyValue.class);
// or:
MyValue value = mapper.readValue(new URL("http://some.com/api/entry.json"), MyValue.class);
// or:
MyValue value = mapper.readValue("{\"name\":\"Bob\", \"age\":13}", MyValue.class);
```

Deserialization:

```java
mapper.writeValue(new File("result.json"), myResultObject);
// or:
byte[] jsonBytes = mapper.writeValueAsBytes(myResultObject);
// or:
String jsonString = mapper.writeValueAsString(myResultObject);
```

### Generic Collections

You can also handle JDK `List`s, `Map`s.

Serialization:

```java
Map<String, Integer> scoreByName = mapper.readValue(jsonSource, Map.class);
List<String> names = mapper.readValue(jsonSource, List.class);
```

Deserialization:

```java
// and can obviously write out as well
mapper.writeValue(new File("names.json"), names);
```

### Generic Type

如果需要将 JSON 字符串反序列化为泛型，有两种方式：

#### 方式一：TypeReference

方式一：使用 [`com.fasterxml.jackson.core.type.TypeReference<T>`](https://fasterxml.github.io/jackson-core/javadoc/2.9/com/fasterxml/jackson/core/type/TypeReference.html)：

> This generic abstract class is used for obtaining full generics type information by sub-classing; it must be converted to [`ResolvedType`](https://fasterxml.github.io/jackson-core/javadoc/2.9/com/fasterxml/jackson/core/type/ResolvedType.html) implementation (implemented by `JavaType` from "databind" bundle) to be used. Class is based on ideas from http://gafter.blogspot.com/2006/12/super-type-tokens.html, Additional idea (from a suggestion made in comments of the article) is to require bogus implementation of `Comparable` (any such generic interface would do, as long as it forces a method with generic type to be implemented). to ensure that a Type argument is indeed given.
>
> Usage is by sub-classing: here is one way to instantiate reference to generic type `List<Integer>`:
>
> ```
>   TypeReference ref = new TypeReference<List<Integer>>() { };
> ```
>
> which can be passed to methods that accept TypeReference, or resolved using `TypeFactory` to obtain [`ResolvedType`](https://fasterxml.github.io/jackson-core/javadoc/2.9/com/fasterxml/jackson/core/type/ResolvedType.html).

代码如下：

```java
TypeReference<RespDTO<XxxRespDTO>> typeRef = new TypeReference<RespDTO<XxxRespDTO>>() {};
RespDTO<XxxRespDTO> resp = mapper.readValue(JSON_A, typeRef);

TypeReference<List<XxxRespDTO>> typeRef1 = new TypeReference<List<XxxRespDTO>>() {};
List<XxxRespDTO> resp2 = mapper.readValue(JSON_B, typeRef1);
```

#### 方式二：JavaType

方式二：使用 [`com.fasterxml.jackson.databind.JavaType`](https://fasterxml.github.io/jackson-databind/javadoc/2.9/com/fasterxml/jackson/databind/JavaType.html)：

> Base class for type token classes used both to contain information and as keys for deserializers.
>
> Instances can (only) be constructed by `com.fasterxml.jackson.databind.type.TypeFactory`.

`JavaType` 的继承结构如下图：

![com.fasterxml.jackson.databind.JavaType](/img/java/jackson/JavaType.png)

代码如下：

```java
JavaType valueType = mapper.getTypeFactory().constructParametricType(RespDTO.class, XxxRespDTO.class);
RespDTO<XxxRespDTO> resp = mapper.readValue(JSON_A, valueType);

JavaType valueType2 = mapper.getTypeFactory().constructParametricType(List.class, XxxRespDTO.class);
List<XxxRespDTO> resp2 = mapper.readValue(JSON_B, valueType2);
```

`JavaType` 的调试结果如下图，其值为 `JavaType` 的子类 `CollectionType`：

![com.fasterxml.jackson.databind.type.CollectionType](/img/java/jackson/CollectionType.png)

### Tree Model (JsonNode)

> Tree Model can be more convenient than data-binding, especially in cases where structure is highly dynamic, or does not map nicely to Java classes.

[`com.fasterxml.jackson.databind.JsonNode`](https://fasterxml.github.io/jackson-databind/javadoc/2.9/com/fasterxml/jackson/databind/JsonNode.html) 表示一个 JSON 树节点，可以通过 `ObjectMapper#readTree` 方法反序列化出来，也可以通过 `JsonNode` 的子类 API 自定义构建：

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

也可以修改其值：

```java
((ObjectNode) jsonNode).put("key", "value");
```

#### 使用场景

一、对接口响应的 JSON 原文进行验签：

```json
{
    "response": {
        "head": {
            ...
        },
        "body": {
            ...
        }
    },
    "signature": "..."
}
```

```java
JsonNode jsonNode = objectMapper.readTree(responseJson);
String response = jsonNode.get("response").toString();
String signature = jsonNode.get("signature").toString();
return RsaUtil.verify(response, publicKey, signature);
```

二、<[Compare Two JSON Objects with Jackson](https://www.baeldung.com/jackson-compare-two-json-objects)>

> using the [*JsonNode.equals*](https://static.javadoc.io/com.fasterxml.jackson.core/jackson-databind/2.9.8/com/fasterxml/jackson/databind/JsonNode.html#equals-java.lang.Object-) method. **The `equals()` method performs a full (deep) comparison.**

### 例子

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

#### 接口一

下图展示了变量 `returnType` 为参数化类型 `ParameterizedType`，其实际类型参数 `type` 为 `Class` 类型，值为 `XxxRespDTO` ：

![JsonNode](/img/java/jackson/ObjectMapper_example.png)

#### 接口二

下图展示了变量 `returnType` 的实际类型参数 `type` 与接口一为 `Class` 类型不同，接口二为 `ParameterizedType` 参数化类型，值为 `List<XxxRespDTO>`：

![](/img/java/jackson/ObjectMapper_example_2.png)

## 常见报错

### Unrecognized field, not marked as ignorable

该错误的意思是说，不能够识别的字段没有标示为可忽略。出现该问题的原因就是 JSON 中包含了目标 Java 对象没有的属性。

解决方案：

1. 保证传入的 JSON 串不包含目标对象的没有的属性。

2. On deserialization, `@JsonIgnoreProperties(ignoreUnknown=true)` ignores properties that don't have getter/setters

3. [`Deserialization Features`](https://github.com/FasterXML/jackson-databind/wiki/Deserialization-Features#failure-handling) 全局配置：

   ```
   // 配置该 `objectMapper` 在反序列化时，忽略目标对象没有的属性。
   objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
   ```

### 缺少默认构造方法

问题：

```java
com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Cannot construct instance of `com.***.RespBody` (no Creators, like default construct, exist): cannot deserialize from Object value (no delegate- or property-based Creator)
```

解决方案：

* POJO 加上 `@NoArgsConstructor`

### Using Java inner classes for Jackson serialization

https://dev.to/pavel_polivka/using-java-inner-classes-for-jackson-serialization-4ef8

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

Deserialization to Generic Type: `com.google.gson.reflect.TypeToken`

```java
RespDTO<XxxRespDTO> data = new Gson().fromJson(json, new TypeToken<RespDTO<XxxRespDTO>>() {}.getType());
List<XxxRespDTO> data = new Gson().fromJson(json, new TypeToken<ArrayList<XxxRespDTO>>() {}.getType());
```

# 参考

https://www.baeldung.com/category/json/jackson/

* https://www.baeldung.com/jackson-vs-gson
* https://www.baeldung.com/jackson-exception

https://github.com/qidawu/java-api-test/tree/master/src/main/java/json
