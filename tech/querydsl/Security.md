# Security in QueryDSL

QueryDSL은 타입 안전성과 동적 쿼리 생성을 지원하여 SQL Injection에 강한 도구입니다.  

그러나 잘못 사용하면 여전히 보안 취약점이 발생할 수 있습니다.

## 🤷🏻‍♂️ SQL Injection이 무엇인가요?

SQL Injection은 악의적인 사용자가 SQL 쿼리에 악성 코드를 삽입하여 데이터베이스를 조작하는 공격 기법입니다. 

```java
String sql = "SELECT * FROM users WHERE name = '" + userInput + "'";
```
만약 위와 같은 sql을 서버에 작성을 했고, 누군가 userInput에 "'; DROP TABLE users; --"를 넣었다고 가정을 해봅시다.

그럼 `SELECT * FROM users WHERE name = ''; DROP TABLE users`와 같은 결과가 발생합니다.

이런 공격으로 인해 데이터 유출, 데이터 삭제, 권한 상승 등의 보안 문제가 발생할 수 있습니다.

## 🤷🏻‍♂️ 왜 QueryDSL이 SQL Injection에 강한 도구인가요?

QueryDSL은 다음과 같은 이유로 SQL Injection에 강합니다:

### 1. 타입 안전성 (Type Safety)
```java
// QueryDSL 사용
QUser user = QUser.user;
List<User> users = queryFactory
    .selectFrom(user)
    .where(user.name.eq(userInput))  // 자동으로 파라미터 바인딩
    .fetch();
```

### 2. 자동 파라미터 바인딩
QueryDSL은 사용자 입력을 자동으로 `PreparedStatement`의 파라미터로 바인딩합니다. 이는 SQL 문자열에 직접 삽입되지 않아 SQL Injection을 방지합니다.

>**PreparedStatement란?** <br>
>SQL문을 미리 컴파일해서 변수 부분에 값을 안전하게 넣어 실행하는 방식입니다.

### 3. 컴파일 타임 검증
QueryDSL은 컴파일 시점에 쿼리 구조를 검증하여 런타임 에러를 줄이고 안전성을 높입니다.

## 🤷🏻‍♂️ 그럼 어떻게 잘못 사용하면, 보안 취약점이 발생하나요?

### 1. `Expressions.stringTemplate()` 직접 사용
```java
// 위험한 예시 - 사용자 입력을 직접 문자열 템플릿에 전달
String userInput = request.getParameter("condition");
StringExpression condition = Expressions.stringTemplate(userInput);

List<User> users = queryFactory
    .selectFrom(user)
    .where(condition)  // SQL Injection 위험!
    .fetch();
```

### 2. 동적 테이블명/컬럼명 사용
```java
// 위험한 예시 - 사용자 입력으로 테이블명 결정
String tableName = request.getParameter("table");
StringExpression table = Expressions.stringTemplate(tableName);

List<?> data = queryFactory
    .select()
    .from(table)  // SQL Injection 위험!
    .fetch();
```

### 3. 동적 정렬 필드 사용
```java
// 위험한 예시 - 사용자 입력으로 정렬 필드 결정
String sortField = request.getParameter("sort");
StringExpression sortExpression = Expressions.stringTemplate(sortField);

List<User> users = queryFactory
    .selectFrom(user)
    .orderBy(sortExpression.asc())  // SQL Injection 위험!
    .fetch();
```

### 4. 실제 공격 예시
```java
// 사용자가 입력: "name = 'admin' OR 1=1"
String userCondition = request.getParameter("condition");
StringExpression condition = Expressions.stringTemplate(userCondition);

// 결과: 모든 사용자 정보가 조회됨
List<User> users = queryFactory
    .selectFrom(user)
    .where(condition)
    .fetch();
```

>**💡 핵심** : QueryDSL의 타입 안전한 메서드(`eq()`, `contains()` 등)는 안전하지만, `Expressions.stringTemplate()`에 사용자 입력을 직접 전달하면 SQL Injection 위험이 있습니다.

## 🤷🏻‍♂️ 안전한 QueryDSL 사용법은 무엇인가요?

### 1. 항상 파라미터 바인딩 사용
```java
// 안전한 예시
QUser user = QUser.user;
String searchTerm = request.getParameter("search");

List<User> users = queryFactory
    .selectFrom(user)
    .where(user.name.contains(searchTerm))  // 자동 파라미터 바인딩
    .fetch();
```

### 2. 동적 쿼리 시 조건문 사용
```java
// 안전한 동적 쿼리
public List<User> searchUsers(String name, String email) {
    QUser user = QUser.user;
    
    BooleanBuilder builder = new BooleanBuilder();
    
    if (name != null && !name.isEmpty()) {
        builder.and(user.name.contains(name));
    }
    
    if (email != null && !email.isEmpty()) {
        builder.and(user.email.contains(email));
    }
    
    return queryFactory.selectFrom(user).where(builder).fetch();
}
```

### 3. 입력값 검증
```java
// 입력값 검증 추가
public List<User> searchUsers(String searchTerm) {
    // 입력값 검증
    if (searchTerm == null || searchTerm.trim().isEmpty()) {
        return Collections.emptyList();
    }
    
    // 특수문자 필터링 (필요시)
    searchTerm = searchTerm.replaceAll("[^a-zA-Z0-9가-힣\\s]", "");
    
    QUser user = QUser.user;
    return queryFactory
        .selectFrom(user)
        .where(user.name.contains(searchTerm))
        .fetch();
}
```

## 🤷🏻‍♂️ 추가 보안 고려사항은 무엇인가요?

### 1. 권한 기반 쿼리
```java
// 사용자 권한에 따른 쿼리 제한
public List<User> getUsersByRole(String userRole) {
    QUser user = QUser.user;
    
    BooleanBuilder builder = new BooleanBuilder();
    
    if ("ADMIN".equals(userRole)) {
        // 관리자는 모든 사용자 조회 가능
        return queryFactory.selectFrom(user).fetch();
    } else if ("USER".equals(userRole)) {
        // 일반 사용자는 자신의 정보만 조회 가능
        builder.and(user.id.eq(getCurrentUserId()));
    }
    
    return queryFactory.selectFrom(user).where(builder).fetch();
}
```

### 2. 쿼리 결과 제한
```java
// 결과 개수 제한으로 DoS 공격 방지
public List<User> searchUsers(String searchTerm) {
    QUser user = QUser.user;
    
    return queryFactory
        .selectFrom(user)
        .where(user.name.contains(searchTerm))
        .limit(100)  // 최대 100개 결과만 반환
        .fetch();
}
```

## 💡 결론

QueryDSL은 기본적으로 SQL Injection에 강하지만, 잘못된 사용법으로 인해 여전히 보안 취약점이 발생할 수 있습니다. 

**안전한 사용을 위한 핵심 원칙:**
1. 항상 QueryDSL의 타입 안전한 메서드 사용
2. 사용자 입력을 직접 SQL 문자열로 조합하지 않기
3. 입력값 검증 및 필터링 추가
4. 권한 기반 쿼리 제한
5. 결과 개수 제한으로 DoS 공격 방지