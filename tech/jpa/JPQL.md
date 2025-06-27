# JPQL

JPQL은 JPA 전용 쿼리 언어입니다. SQL과 비슷하지만 `테이블/컬럼` 대신 `엔티티/필드`를 대상으로 쿼리합니다. 그리고 Hibernate 등 JPA 구현체가 JPQL -> SQL로 변환해 DB에 실행시킵니다.

## 🤷🏻‍♂️ JPA, JPQL, Querydsl, JDBC, Native Query, MyBatis에 대한 개념들이 헷갈려요!


| 기술         | 설명 |
|------------|------------|
| **JPA**     | 자바 ORM 표준 인터페이스. 엔티티를 영속화/조회/삭제하기 위한 API 제공 |
| **JPQL**    | JPA에서 사용하는 쿼리 언어. 엔티티와 필드를 대상으로 쿼리 |
| **Querydsl**| 타입 안전한 쿼리 빌더. JPQL을 자바 코드로 동적 생성 가능 |
| **JDBC**    | 자바가 DB와 소통하기 위한 가장 낮은 수준의 API. SQL을 수동 작성 후 `ResultSet`을 다뤄야 함 |
| **Native Query** | SQL을 문자열로 직접 작성하여 실행. DB 종속적 |
| **MyBatis** | SQL 중심의 매퍼 프레임워크. SQL을 XML 또는 애노테이션으로 관리하고 자바 객체와 매핑해 줌 |

한 줄로 정리한다면, 아래와 같습니다.

- **JPA**: ORM 표준 인터페이스
- **JPQL**: 엔티티 기반 쿼리 언어
- **Querydsl**: 동적 쿼리를 타입 안전하게
- **JDBC**: 가장 원초적 DB 연동 수단
- **Native Query**: DB SQL을 직접 날리는 방식
- **MyBatis**: SQL을 수작업 관리 + 객체 매핑까지