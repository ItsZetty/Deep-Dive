# Intro
앞으로 해당 디렉토리 안에서 데이터 접근 기술에 대해서 공부할 예정이다.

## 🤷🏻‍♂️ 데이터 접근 기술이 무엇인가요?
데이터 접근 기술은 애플리케이션에서 데이터베이스나 다른 데이터 저장소에 데이터를 저장하고 조회하는 방법을 의미합니다.

## 🤷🏻‍♂️ 데이터 접근 기술에는 무엇들이 있나요?

데이터 접근 기술에는 아래와 같이 있습니다.
- JdbcTemplate
- MyBatis
- JPA, Hibernate
- Spring Data JPA
- Querydsl

그리고 위 기술들을 크게 2가지로 분류해 보았습니다.

**SQLMapper**
- JdbcTemplate
- MyBatis

SQLMapper는 개발자가 SQL만 작성하면, 해당 SQL의 결과를 객체로 편리하게 매핑해주는 역할을 합니다. 그리고 JDBC를 직접 사용할 때 발생하는 여러가지 중복을 제거해주며, 개발자에게 여러가지 편리한 기능을 제공합니다.

>JDBC란? <br>
>JDBC는 Java Database Connectivity의 약자로, 자바 애플리케이션에서 데이터베이스에 접근할 수 있도록 해주는 표준 API입니다.

**ORM**
- JPA, Hibernate
- Spring Data JPA
- Querydsl

JdbcTemplate이나 MyBatis 같은 SQL 매퍼 기술은 SQL을 개발자가 직접 작성해야 한다는 불편함이 존재합니다. 하지만 JPA를 사용하면 기본적인 SQL은 JPA가 대신 작성하고 처리해줍니다. 

개발자는 저장하고 싶은 객체를 마치 자바 컬렉션에 저장하고, 조회하듯이 사용하면 ORM 기술이 데이터베이스에 해당 객체를 저장하고 조회해줍니다.

>JPA란? <br>
> JPA는 자바 진영의 ORM 표준이며, Hibernate는 JPA에서 가장 많이 사용하는 구현체입니다. 자바에서 ORM을 사용할 때는 JPA 인터페이스를 사용하고, 그 구현체로 Hibernate를 사용한다고 생각하면 됩니다.

또 Spring Data JPA, Querydsl은 JPA를 더 편리하게 사용할 수 있게 도와주는 기술입니다. 만약 실무에서는 JPA를
사용하면, 해당 기술들을 활용하는 것이 필수적으로 필요합니다.