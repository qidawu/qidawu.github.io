---
title: Java 常用类型系列（六）枚举类型总结
date: 2018-04-08 22:56:42
updated:
tags: Java
typora-root-url: ..
---

# 例子

```java
import com.google.common.collect.Maps;
import lombok.Getter;
import lombok.RequiredArgsConstructor;

import java.util.Arrays;
import java.util.Map;

@RequiredArgsConstructor
@Getter
public enum PayMethodEnum {

    /**
     * 卡支付
     */
    CARDS(0),

    /**
     * 借记支付
     */
    BANK_DEBITS(1),

    /**
     * 网银支付
     */
    BANK_REDIRECTS(2),

    /**
     * 银行转账
     */
    BANK_TRANSFERS(3),

    /**
     * 分期付款
     */
    PAY_LATER(4),

    /**
     * 现金支付
     */
    VOUCHERS(5),

    /**
     * 电子钱包
     */
    WALLETS(6);

    /**
     * 支付方式编码
     */
    private final int code;
    public static final Map<Integer, PayMethodEnum> MAP = Maps.newHashMapWithExpectedSize(PayMethodEnum.values().length);

    static {
        Arrays.stream(PayMethodEnum.values()).forEach(aEnum ->
                MAP.put(aEnum.getCode(), aEnum)
        );
    }

    public static PayMethodEnum valueOfCode(int code) {
        return MAP.get(code);
    }

}
```

