---
title: Java 开发必备类库 Apache Commons Lang
date: 2017-12-25 22:29:16
updated:
tags: Java
---

Apache Commons 是一个 Apache 项目，专注于可重用 Java 组件的方方面面。

Apache Commons 项目由三个部分组成：

![Apache Commons](/img/java/apache/apache-commons.png)   

其中，Apache Commons Lang 是 Java 开发过程中很常用的一个类库，可以理解为它是对 Java Lang and Util Base  Libraries 的增强。

Commons Lang 安装方法：

```xml
<!-- https://mvnrepository.com/artifact/org.apache.commons/commons-lang3 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>x.x.x</version>
</dependency>
```

Commons Lang 总览：

![Commons Lang](/img/java/apache/commons-lang/commons-lang.png)

Commons Lang 提供了以下 package：

- `org.apache.commons.lang3`
- `org.apache.commons.lang3.builder`
- `org.apache.commons.lang3.concurrent`
- `org.apache.commons.lang3.event`
- `org.apache.commons.lang3.exception`
- `org.apache.commons.lang3.math`
- `org.apache.commons.lang3.mutable`
- `org.apache.commons.lang3.reflect`
- `org.apache.commons.lang3.text`
- `org.apache.commons.lang3.text.translate`
- `org.apache.commons.lang3.time`
- `org.apache.commons.lang3.tuple`

下面重点来看下最常用的几个工具：

`org.apache.commons.lang3`

- `StringUtils`
- `ArrayUtils`
- `BooleanUtils`

# StringUtils

![StringUtils](/img/java/apache/commons-lang/StringUtils.png)

## 判空函数

API：

```java
StringUtils.isEmpty(String str)
StringUtils.isNotEmpty(String str)
StringUtils.isBlank(String str)
StringUtils.isNotBlank(String str)
StringUtils.isAnyBlank(CharSequence… css)
StringUtils.isAnyEmpty(CharSequence… css)
StringUtils.isNoneBlank(CharSequence… css)
StringUtils.isNoneEmpty(CharSequence… css)
StringUtils.isWhitespace(CharSequence cs)
```

看下 `isBlank` 和 `isEmpty` 的区别：

```java
StringUtils.isBlank(null) // true
StringUtils.isEmpty(null) // true

StringUtils.isBlank("") // true
StringUtils.isEmpty("") // true

StringUtils.isBlank(" ") // true
StringUtils.isEmpty(" ") // false

StringUtils.isBlank("\n\t") // true
StringUtils.isEmpty("\n\t") // false

isNotEmpty = !isEmpty, isBlank同理
```

使用 `isAnyBlank` 和 `isAnyEmpty` 进行多维判空：

```java
StringUtils.isAnyBlank("", "bar", "foo"); // true
StringUtils.isAnyEmpty(" ", "bar", "foo"); // false

isNoneBlank = !isAnyBlank；isNoneEmpty同理
```

使用 `isWhitespace` 判断空白：

```java
StringUtils.isWhitespace(null); // false
StringUtils.isWhitespace(""); // true
StringUtils.isWhitespace(" "); // true
```

## 判断是否相等函数

API：

```java
equals(CharSequence cs1,CharSequence cs2)
equalsIgnoreCase(CharSequence str1, CharSequence str2)
```

例子：

```java
StringUtils.equals("abc", null)  = false
StringUtils.equals("abc", "abc") = true
StringUtils.equals("abc", "ABC") = false
```

忽略大小写判断：

```java
StringUtils.equalsIgnoreCase("abc", null)  = false
StringUtils.equalsIgnoreCase("abc", "abc") = true
StringUtils.equalsIgnoreCase("abc", "ABC") = true
```

## 是否包含函数

API：

```java
containsOnly(CharSequence cs,char… valid)
containsNone(CharSequence cs,char… searchChars)

startsWith(CharSequence str,CharSequence prefix)
startsWithIgnoreCase(CharSequence str,CharSequence prefix)
startsWithAny(CharSequence string,CharSequence… searchStrings)
```

例子：

```java
//判断字符串中所有字符，是否都是出自参数2中
StringUtils.containsOnly("ab", "")      = false
StringUtils.containsOnly("abab", "abc") = true
StringUtils.containsOnly("ab1", "abc")  = false
StringUtils.containsOnly("abz", "abc")  = false

//判断字符串中所有字符，都不在参数2中。
StringUtils.containsNone("abab", 'xyz') = true
StringUtils.containsNone("ab1", 'xyz')  = true
StringUtils.containsNone("abz", 'xyz')  = false

//判断字符串是否以第二个参数开始
StringUtils.startsWith("abcdef", "abc") = true
StringUtils.startsWith("ABCDEF", "abc") = false
```

## 索引下标函数

API：

```java
indexOf(CharSequence seq,CharSequence searchSeq)
indexOf(CharSequence seq,CharSequence searchSeq,int startPos)
indexOfIgnoreCase/lastIndexOfIgnoreCase(CharSequence str,CharSequence searchStr)
lastIndexOf(CharSequence seq,int searchChar)
```

例子：

```java
//返回第二个参数开始出现的索引值
StringUtils.indexOf("aabaabaa", "a")  = 0
StringUtils.indexOf("aabaabaa", "b")  = 2
StringUtils.indexOf("aabaabaa", "ab") = 1

//从第三个参数索引开始找起，返回第二个参数开始出现的索引值
StringUtils.indexOf("aabaabaa", "a", 0)  = 0
StringUtils.indexOf("aabaabaa", "b", 0)  = 2
StringUtils.indexOf("aabaabaa", "ab", 0) = 1
StringUtils.indexOf("aabaabaa", "b", 3)  = 5
StringUtils.indexOf("aabaabaa", "b", 9)  = -1  

//返回第二个参数出现的最后一个索引值        
StringUtils.lastIndexOf("aabaabaa", 'a') = 7
StringUtils.lastIndexOf("aabaabaa", 'b') = 5

StringUtils.lastIndexOfIgnoreCase("aabaabaa", "A", 8)  = 7
StringUtils.lastIndexOfIgnoreCase("aabaabaa", "B", 8)  = 5
StringUtils.lastIndexOfIgnoreCase("aabaabaa", "AB", 8) = 4
StringUtils.lastIndexOfIgnoreCase("aabaabaa", "B", 9)  = 5
```

## 截取函数

API：

```java
substring(String str,int start)
substringAfter(String str,String separator)
substringBeforeLast(String str,String separator)
substringAfterLast(String str,String separator)
substringBetween(String str,String tag)
```

例子：

```java
//start>0表示从左向右, start<0表示从右向左, start=0则从左第一位开始
StringUtils.substring("abcdefg", 0)  = "abcdefg"
StringUtils.substring("abcdefg", 2)  = "cdefg"
StringUtils.substring("abcdefg", 4)  = "efg"
StringUtils.substring("abcdefg", -2) = "fg"
StringUtils.substring("abcdefg", -4) = "defg"
```

// start>0&&end>0从左开始(包括左)到右结束(不包括右),

//start<0&&end<0从右开始(包括右),再向左数到end结束(包括end)

![substring](/img/java/apache/commons-lang/substring.png)

```java
//从第二个参数字符串开始截取，排除第二个字符串
StringUtils.substringAfter("abc", "a")   = "bc"
StringUtils.substringAfter("abcba", "b") = "cba"
StringUtils.substringAfter("abc", "c")   = ""

//从最后一个字母出现开始截取
StringUtils.substringBeforeLast("abcba", "b") = "abc"
StringUtils.substringBeforeLast("abc", "c")   = "ab"
StringUtils.substringBeforeLast("a", "a")     = ""
StringUtils.substringBeforeLast("a", "z")     = "a"

StringUtils.substringAfterLast("abc", "a")   = "bc"
StringUtils.substringAfterLast("abcba", "b") = "a"
StringUtils.substringAfterLast("abc", "c")   = ""

StringUtils.substringBetween("tagabctag", null)  = null
StringUtils.substringBetween("tagabctag", "")    = ""
StringUtils.substringBetween("tagabctag", "tag") = "abc"
```

## 删除函数

API：

```java
StringUtils.remove(String str, char remove)
StringUtils.remove(String str, String remove)
StringUtils.removeEnd(String str, String remove)
StringUtils.removeEndIgnoreCase(String str, String remove)
StringUtils.removePattern(String source, String regex)
StringUtils.removeStart(String str, String remove)
StringUtils.removeStartIgnoreCase(String str, String remove)
StringUtils.deleteWhitespace(String str)
```

例子：

```java
//删除字符
StringUtils.remove("queued", 'u') = "qeed"

//删除字符串
StringUtils.remove("queued", "ue") = "qd"

//删除结尾匹配的字符串     
StringUtils.removeEnd("www.domain.com", ".com")   = "www.domain"

//删除结尾匹配的字符串,找都不到返回原字符串
StringUtils.removeEnd("www.domain.com", "domain") = "www.domain.com"

//忽略大小写的
StringUtils.removeEndIgnoreCase("www.domain.com", ".COM") = "www.domain")

//删除所有空白（好用）
StringUtils.deleteWhitespace("abc")        = "abc"
StringUtils.deleteWhitespace("   ab  c  ") = "abc"
```

## 删除空白函数

API：

```java
trim(String str)
trimToEmpty(String str)
trimToNull(String str)
deleteWhitespace(String str)
```

例子：

```java
StringUtils.trim("     ")       = ""
StringUtils.trim("abc")         = "abc"
StringUtils.trim("    abc    ") = "abc"

StringUtils.trimToNull("     ")       = null
StringUtils.trimToNull("abc")         = "abc"
StringUtils.trimToNull("    abc    ") = "abc"
StringUtils.trimToEmpty("     ")       = ""
StringUtils.trimToEmpty("abc")         = "abc"
StringUtils.trimToEmpty("    abc    ") = "abc"

StringUtils.deleteWhitespace("")           = ""
StringUtils.deleteWhitespace("abc")        = "abc"
StringUtils.deleteWhitespace("   ab  c  ") = "abc"
```

## 替换函数

API：

```java
replace(String text, String searchString, String replacement)
replace(String text, String searchString, String replacement, int max)
replaceChars(String str, char searchChar, char replaceChar)
replaceChars(String str, String searchChars, String replaceChars)
replaceEach(String text, String[] searchList, String[] replacementList)
replaceEachRepeatedly(String text, String[] searchList, String[] replacementList)
replaceOnce(String text, String searchString, String replacement)
replacePattern(String source, String regex, String replacement)
overlay(String str,String overlay,int start,int end)
```

`replace` 例子：

```java
StringUtils.replace("aba", "a", "")    = "b"
StringUtils.replace("aba", "a", "z")   = "zbz"    

//数字就是替换个数，0代表不替换，1代表从开始数起第一个，-1代表全部替换
StringUtils.replace("abaa", "a", "", -1)   = "b"
StringUtils.replace("abaa", "a", "z", 0)   = "abaa"
StringUtils.replace("abaa", "a", "z", 1)   = "zbaa"
StringUtils.replace("abaa", "a", "z", 2)   = "zbza"
StringUtils.replace("abaa", "a", "z", -1)  = "zbzz"
```

`replaceEach` 是对 `replace` 的增强版，用于一次性替换多个字符。搜索列表和替换长度必须一致，否则报 `IllegalArgumentException` 异常：

```java
StringUtils.replaceEach("abcde", new String[]{"ab", "d"}, new String[]{"w", "t"})  = "wcte"
StringUtils.replaceEach("abcde", new String[]{"ab", "d"}, new String[]{"d", "t"})  = "dcte"
```

`replaceChars` 用于对单个字符逐一替换，其操作如下：

![replaceChars](/img/java/apache/commons-lang/replaceChars.png)

```java
StringUtils.replaceChars("dabcba", "bcd", "yzx") = "xayzya"
StringUtils.replaceChars("abcba", "bc", "y")   = "ayya"
```

`replaceOnce` 例子：

```java
StringUtils.replaceOnce("aba", "a", "")    = "ba"
StringUtils.replaceOnce("aba", "a", "z")   = "zba"
```

`overlay` 例子：

```java
StringUtils.overlay("abcdef", "zzzz", 2, 4)   = "abzzzzef"
StringUtils.overlay("abcdef", "zzzz", 4, 2)   = "abzzzzef"
StringUtils.overlay("abcdef", "zzzz", -1, 4)  = "zzzzef"
StringUtils.overlay("abcdef", "zzzz", 2, 8)   = "abzzzz"
StringUtils.overlay("abcdef", "zzzz", -2, -3) = "zzzzabcdef"
StringUtils.overlay("abcdef", "zzzz", 8, 10)  = "abcdefzzzz"  
```

## 反转函数

API：

```java
reverse(String str)
reverseDelimited(String str, char separatorChar)
```

例子：

```java
StringUtils.reverse("bat") = "tab"
StringUtils.reverseDelimited("a.b.c", 'x') = "a.b.c"
StringUtils.reverseDelimited("a.b.c", ".") = "c.b.a"
```

## 分隔函数

API：

```java
split(String str)
split(String str, char separatorChar)
split(String str, String separatorChars)
split(String str, String separatorChars, int max)
splitByCharacterType(String str)
splitByCharacterTypeCamelCase(String str)
splitByWholeSeparator(String str, String separator)
splitByWholeSeparator(String str, String separator, int max)
splitByWholeSeparatorPreserveAllTokens(String str, String separator)
splitByWholeSeparatorPreserveAllTokens(String str, String separator, int max)
splitPreserveAllTokens(String str)
splitPreserveAllTokens(String str, char separatorChar)
splitPreserveAllTokens(String str, String separatorChars)
splitPreserveAllTokens(String str, String separatorChars, int max)
```

例子：

```java
//用空白符做空格
StringUtils.split("abc def")  = ["abc", "def"]
StringUtils.split("abc  def") = ["abc", "def"]
StringUtils.split("a..b.c", '.')   = ["a", "b", "c"]

//用字符分割
StringUtils.split("a:b:c", '.')    = ["a:b:c"]

//0 或者负数代表没有限制
StringUtils.split("ab:cd:ef", ":", 0)    = ["ab", "cd", "ef"]

//分割字符串 ,可以设定得到数组的长度，限定为2
StringUtils.split("ab:cd:ef", ":", 2)    = ["ab", "cd:ef"]

//null也可以作为分隔
StringUtils.splitByWholeSeparator("ab de fg", null)      = ["ab", "de", "fg"]
StringUtils.splitByWholeSeparator("ab   de fg", null)    = ["ab", "de", "fg"]
StringUtils.splitByWholeSeparator("ab:cd:ef", ":")       = ["ab", "cd", "ef"]
StringUtils.splitByWholeSeparator("ab-!-cd-!-ef", "-!-") = ["ab", "cd", "ef"]

//带有限定长度的分隔
StringUtils.splitByWholeSeparator("ab:cd:ef", ":", 2)       = ["ab", "cd:ef"]

```



## 合并函数

API：

```java
join(byte[] array,char separator)
join(Object[] array,char separator)
join(Object[] array,char separator,int startIndex,int endIndex)
```

例子

```java
//只有一个参数的join，简单合并在一起
StringUtils.join(["a", "b", "c"]) = "abc"
StringUtils.join([null, "", "a"]) = "a"    

//null的话，就是把字符合并在一起
StringUtils.join(["a", "b", "c"], null) = "abc"

//从index为0到3合并，注意是排除3的
StringUtils.join([null, "", "a"], ',', 0, 3)   = ",,a"
StringUtils.join(["a", "b", "c"], "--", 0, 3)  = "a--b--c"

//从index为1到3合并，注意是排除3的
StringUtils.join(["a", "b", "c"], "--", 1, 3)  = "b--c"
StringUtils.join(["a", "b", "c"], "--", 2, 3)  = "c"
```

## 大小写转换和判断

API：

```java
StringUtils.capitalize(String str)
StringUtils.uncapitalize(String str)
StringUtils.upperCase(String str)
StringUtils.upperCase(String str,Locale locale)
StringUtils.lowerCase(String str)
StringUtils.lowerCase(String str,Locale locale)
StringUtils.swapCase(String str)

StringUtils.isAllUpperCase(CharSequence cs)
StringUtils.isAllLowerCase(CharSequence cs)
```

大小写转换：

- `capitalize` 首字母大写
- `upperCase` 全部转化为大写
- `swapCase` 大小写互转

大小写判断：

- `isAllUpperCase` 是否全部大写
- `isAllLowerCase` 是否全部小写

## 缩短省略函数

API：

```java
abbreviate(String str, int maxWidth)
abbreviate(String str, int offset, int maxWidth)
abbreviateMiddle(String str, String middle, int length)
```

例子：

```java
StringUtils.abbreviate("abcdefg", 6) = "abc..."
StringUtils.abbreviate("abcdefg", 7) = "abcdefg"
StringUtils.abbreviate("abcdefg", 8) = "abcdefg"
StringUtils.abbreviate("abcdefg", 4) = "a..."
StringUtils.abbreviate("abcdefg", 3) = IllegalArgumentException

StringUtils.abbreviate("abcdefghijklmno", 6, 10)  = "...ghij..."

StringUtils.abbreviateMiddle("abcdef", ".", 4)     = "ab.f"
```

字符串的长度小于或等于最大长度，返回该字符串。

运算规律：`(substring(str, 0, max-3) + “…”)`

如果最大长度小于 4，则抛出异常 `IllegalArgumentException`。

## 相似度函数

API：

```java
difference(String str1,String str2)
```

例子：

```java
//在str1中寻找str2中没有的的字符串，并返回     
StringUtils.difference("", "abc") = "abc"
StringUtils.difference("abc", "") = ""
StringUtils.difference("abc", "abc") = ""
StringUtils.difference("abc", "ab") = ""
StringUtils.difference("ab", "abxyz") = "xyz"
StringUtils.difference("abcde", "abxyz") = "xyz"
StringUtils.difference("abcde", "xyz") = "xyz"
```

![difference](/img/java/apache/commons-lang/difference.png)

# BooleanUtils

![BooleanUtils](/img/java/apache/commons-lang/BooleanUtils.png)

# ArrayUtils

## 添加方法

add(boolean[] array,boolean element)
add(T[] array,int index,T element)
addAll(boolean[] array1,boolean… array2)

//添加元素到数组中        
ArrayUtils.add([true, false], true) = [true, false, true]    

//将元素插入到指定位置的数组中
ArrayUtils.add(["a"], 1, null)     = ["a", null]
ArrayUtils.add(["a"], 1, "b")      = ["a", "b"]
ArrayUtils.add(["a", "b"], 3, "c") = ["a", "b", "c"]
ArrayUtils.add(["a", "b"], ["c", "d"]) = ["a", "b", "c","d"]

## 克隆方法

clone(boolean[] array)等等

ArrayUtils.clone(new int[] { 3, 2, 4 }); = {3,2,4}

## 包含方法

contains(boolean[] array,boolean valueToFind)

// 查询某个Object是否在数组中
ArrayUtils.contains(new int[] { 3, 1, 2 }, 1); =  true

## 获取长度方法

getLength(Object array)

ArrayUtils.getLength(["a", "b", "c"]) = 3

## 获取索引方法

indexOf(boolean[] array,boolean valueToFind)
indexOf(boolean[] array,boolean valueToFind,int startIndex)

//查询某个Object在数组中的位置,可以指定起始搜索位置,找不到返回-1
//从正序开始搜索,搜到就返回当前的index否则返回-1
ArrayUtils.indexOf(new int[] { 1, 3, 6 }, 6); =  2
ArrayUtils.indexOf(new int[] { 1, 3, 6 }, 2); =  -1

//从逆序开始搜索,搜到就返回当前的index,否则返回-1
ArrayUtils.lastIndexOf(new int[] { 1, 3, 6 }, 6); =  2

//从逆序索引为2开始搜索,,搜到就返回当前的index,否则返回-1
ArrayUtils.lastIndexOf(new Object[]{"33","yy","uu"}, "33",2 ) = 0

## 判空方法

isEmpty(boolean[] array)等等
isNotEmpty(T[] array)

//判断数组是否为空(null和length=0的时候都为空)
ArrayUtils.isEmpty(new int[0]); =  true
ArrayUtils.isEmpty(new Object[] { null }); = false

## 长度相等判断方法

isSameLength(boolean[] array1,boolean[] array2)

//判断两个数组的长度是否相等
ArrayUtils.isSameLength(new Integer[] { 1, 3, 5 }, new Long[] { "1", "3", "5"}); = true      

## 空数组转换

nullToEmpty(Object[] array)等等

//讲null转化为相应数组
int [] arr1 = null;
int [] arr2 = ArrayUtils.nullToEmpty(arr1);

## 删除元素方法

remove(boolean[] array,int index)等等
removeElement(boolean[] array,boolean element)
removeAll(T[] array,int… indices)
removeElements(T[] array,T… values)

//删除指定下标的元素        
ArrayUtils.remove([true, false], 1)       = [true]
ArrayUtils.remove([true, true, false], 1) = [true, false]

//删除第一次出现的元素
ArrayUtils.removeElement([true, false], false)      = [true]
ArrayUtils.removeElement([true, false, true], true) = [false, true]

//删除所有出现的下标的元素
ArrayUtils.removeAll(["a", "b", "c"], 0, 2) = ["b"]
ArrayUtils.removeAll(["a", "b", "c"], 1, 2) = ["a"]

//删除数组出现的所有元素
ArrayUtils.removeElements(["a", "b"], "a", "c")      = ["b"]
ArrayUtils.removeElements(["a", "b", "a"], "a")      = ["b", "a"]
ArrayUtils.removeElements(["a", "b", "a"], "a", "a") = ["b"]

## 反转方法

reverse(boolean[] array)等等
reverse(boolean[] array,int startIndexInclusive,int endIndexExclusive)

//反转数组
int[] array =new int[] { 1, 2, 5 };
ArrayUtils.reverse(array);// {5,2,1}

//指定范围的反转数组，排除endIndexExclusive的
int[] array =new int[] {1, 2, 5 ,3,4,5,6,7,8};
ArrayUtils.reverse(array,2,5);
System.out.println(ArrayUtils.toString(array)); = {1,2,4,3,5,5,6,7,8}

## 截取数组

subarray(boolean[] array,int startIndexInclusive,int endIndexExclusive)

//截取数组
ArrayUtils.subarray(newint[] { 3, 4, 1, 5, 6 }, 2, 4); =  {1,5}
//起始index为2(即第三个数据)结束index为4的数组
ArrayUtils.subarray(newint[] { 3, 4, 1, 5, 6 }, 2, 10); =  {1,5,6}
//如果endIndex大于数组的长度,则取beginIndex之后的所有数据  

## 打印数组方法

toString(Object array)
toString(Object array,String stringIfNull)

//打印数组
ArrayUtils.toString(newint[] { 1, 4, 2, 3 }); =  {1,4,2,3}
ArrayUtils.toString(new Integer[] { 1, 4, 2, 3 }); =  {1,4,2,3}
//如果为空，返回默认信息
ArrayUtils.toString(null, "I'm nothing!"); =  I'm nothing!

# 参考

https://commons.apache.org/