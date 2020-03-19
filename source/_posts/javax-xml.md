---
title: Java 通过注解生成 XML 示例
date: 2017-01-10 22:09:48
updated:
tags: Java
typora-root-url: ..
---

本文演示通过 JAXB 生成和解析 XML，主要涉及以下常用注解，更多注解可以自行尝试：

```Java
@XmlRootElement
@XmlElement
@XmlAttribute
@XmlValue
```

实体类如下：

```Java
// @Setter 会报错 com.sun.xml.internal.bind.v2.runtime.IllegalAnnotationsException: 4 counts of IllegalAnnotationExceptions
@Getter
@ToString
@Builder
@NoArgsConstructor
@AllArgsConstructor
@XmlRootElement(name = "head")
class Head {
    @XmlElement(name = "trade_code")
    private String tradeCode;

    @XmlElement(name = "trade_date")
    private String tradeDate;

    @XmlElement(name = "trade_time")
    private String tradeTime;

    @XmlElement(name = "serial_no")
    private String serialNo;

    @XmlElement(name = "parent")
    private Parent parent;
}

@Getter
@ToString
@Builder
@NoArgsConstructor
@AllArgsConstructor
@XmlRootElement
class Parent {
    @XmlElement
    private List<Child> child;
}

@Getter
@ToString
@Builder
@NoArgsConstructor
@AllArgsConstructor
class Child {
    @XmlAttribute
    private String key;

    @XmlValue
    private String value;
}
```

# XML 工具类

```Java
@Slf4j
public class XmlUtils {

    private static final Map<String, JAXBContext> JAXB_CONTEXT_MAP = new HashMap<>();

    /**
     * 传入一个对象，生成对应的 XML
     */
    public static <T> String toXml(T t) {
        if (t == null) {
            log.warn("[XmlUtils toXml] params is null");
            return null;
        }

        JAXBContext jaxbContext = getJAXBContext(t.getClass());
        Marshaller jaxbMarshaller;
        try (StringWriter stringWriter = new StringWriter()) {
            jaxbMarshaller = jaxbContext.createMarshaller();
            jaxbMarshaller.setProperty(Marshaller.JAXB_FORMATTED_OUTPUT, Boolean.TRUE);
            jaxbMarshaller.setProperty(Marshaller.JAXB_ENCODING, "UTF-8");
            jaxbMarshaller.marshal(t, stringWriter);
            return stringWriter.toString();
        } catch (JAXBException | IOException e) {
            log.error("[XmlUtils toXml] exception", e);
        }
        return null;
    }

    /**
     * 将 XML 解析成指定对象
     */
    public static <T> T unXml(String xml, Class<T> clazz) {
        if (xml == null || xml.isEmpty()) {
            log.warn("[XmlUtils unXml] xml is null");
            return null;
        }

        JAXBContext jaxbContext = getJAXBContext(clazz);
        try {
            Unmarshaller unmarshaller = jaxbContext.createUnmarshaller();
            StringReader reader = new StringReader(xml);
            @SuppressWarnings("unchecked")
            T unmarshal = (T) unmarshaller.unmarshal(reader);
            return unmarshal;
        } catch (JAXBException e) {
            log.error("[XmlUtils unXml] exception", e);
        }
        return null;
    }

    /**
     * 根据类名称获取 JAXBContext
     */
    private static JAXBContext getJAXBContext(Class clazz) {
        JAXBContext jaxbContext = JAXB_CONTEXT_MAP.get(clazz.getName());
        if (jaxbContext == null) {
            try {
                jaxbContext = JAXBContext.newInstance(clazz);
                JAXB_CONTEXT_MAP.put(clazz.getName(), jaxbContext);
            } catch (JAXBException e) {
                log.error(e.getMessage(), e);
            }
        }
        return jaxbContext;
    }

}
```

# 生成 XML

```Java
@Test
public void testToXml() {
    LocalDateTime now = LocalDateTime.now();
    Child child = Child.builder()
        .key("key")
        .value("value")
        .build();

    Parent parent = Parent.builder()
        .child(Arrays.asList(child, child))
        .build();

    Head head = Head.builder()
        .tradeCode("XT-001")
        .tradeDate(now.format(DateTimeFormatter.ofPattern("yyyy-MM-dd")))
        .tradeTime(now.format(DateTimeFormatter.ofPattern("HH:mm:ss")))
        .serialNo(UUID.randomUUID().toString())
        .parent(parent)
        .build();
    String xml = XmlUtils.toXml(head);
    log.info(xml);
}
```

输出结果：

```XML
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<head>
    <trade_code>XT-001</trade_code>
    <trade_date>2020-03-19</trade_date>
    <trade_time>18:39:28</trade_time>
    <serial_no>114d2c0c-30a8-4275-982c-f8eba86cbadb</serial_no>
    <parent>
        <child key="key">value</child>
        <child key="key">value</child>
    </parent>
</head>
```

# 解析 XML

```Java
@Test
public void testUnXml() {
    String xml = "<?xml version=\"1.0\" encoding=\"UTF-8\" standalone=\"yes\"?>\n" +
        "<head>\n" +
        "    <trade_code>XT-001</trade_code>\n" +
        "    <trade_date>2020-03-19</trade_date>\n" +
        "    <trade_time>18:39:28</trade_time>\n" +
        "    <serial_no>114d2c0c-30a8-4275-982c-f8eba86cbadb</serial_no>\n" +
        "    <parent>\n" +
        "        <child key=\"key\">value</child>\n" +
        "        <child key=\"key\">value</child>\n" +
        "    </parent>\n" +
        "</head>";
    Head head = XmlUtils.unXml(xml, Head.class);
    log.info(head.toString());
}
```

输出结果

```
Head(tradeCode=XT-001, tradeDate=2020-03-19, tradeTime=18:39:28, serialNo=114d2c0c-30a8-4275-982c-f8eba86cbadb, parent=Parent(child=[Child(key=key, value=value), Child(key=key, value=value)]))
```

