# String.valueOf

## 🤷🏻‍♂️ 자바에서 Object 타입인 value를 String으로 타입 캐스팅하는 것과 String.valueOf()를 사용하는 것의 차이점은 무엇인가요?

두 방식 모두 String 타입으로 변환하는 것은 동일하지만, 동작 방식과 예외 처리에서 차이가 있습니다.

`(String) value`로 타입 캐스팅 하는 것은 value가 String 타입이 아닌 경우 ClassCastException이 발생하며, value가 null인 경우 그대로 null을 반환하여 이후 메서드를 호출할 때 NullPointerException이 발생합니다. 타입 캐스팅은 타입 안정성이 부족하기 때문에 캐스팅하는 타입이 확실할 때만 사용해야 합니다.

```java
Object intValue = 10;
String str1 = (String) intValue; // ClassCastException

Object nullValue = null;
String str2 = (String) nullValue; // null
str2.concat("maeilmail"); // NullPointerException
```

`String.valueOf(value)`는 value가 String 타입이 아닌 경우 `value.toString()`을 호출하여 String으로 변환하며, value가 null인 경우 "null" 문자열을 반환합니다.

```java
Object intValue = 10;
String str1 = String.valueOf(intValue); // "10"

Object nullValue = null;
String str2 = String.valueOf(nullValue); // "null"
str2.concat("maeilmail"); // "nullmaeilmail"
```

타입 캐스팅에서 발생하는 예외는 런타임 시점에 발생하기 때문에 `String.valueOf()`가 더 안전하고 예외를 방지할 수 있습니다.

## 🤷🏻‍♂️ 타입 캐스팅할 때 ClassCastException을 방지하는 방법은 무엇이 있을까요?

캐스팅할 타입과 맞는지 먼저 확인 후 캐스팅하면 ClassCastException을 방지할 수 있습니다. 이때 `instanceof`를 사용하면 안전하게 변환할 수 있습니다.

```java
Object intValue = 10;

if (intValue instanceof String str) {
    System.out.println(str);
} else {
    // ...
}
```

## 🤷🏻‍♂️ String.valueOf(null)이 "null"을 반환하는 것은 문제가 될 수도 있지 않나요?

"null"이라는 문자열과 null 자체는 다른 의미를 가질 수 있기 때문에 문제가 될 수 있습니다. 특히, JSON 변환이나 데이터베이스에 저장할 때 null이 "null" 문자열로 저장되어서 오류가 발생할 가능성이 있습니다. 원치 않는 "null" 문자열을 방지하려면 미리 null 여부를 검증하고 따로 처리하거나, `Objects.toString()`을 사용해서 null일 경우 다른 문자열로 처리하는 방법을 사용할 수 있습니다.