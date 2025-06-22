# JdbcTemplate

JdbcTemplate란 간단히 JDBC를 매우 편리하게 사용할 수 있게 도와주는 방법 중 하나의 선택지이다. 장점과 단점은 아래와 같다.

**장점**
- 설정이 매우 편리합니다.
    - JdbcTemplate은 `spring-jdbc` 라이브러리에 포함되어 있는데, 이 라이브러리는 스프링으로 JDBC를 사용할 때 기본으로 사용되는 라이브러리입니다. 또 별도의 복잡한 설정 없이 바로 사용할 수 있습니다.
- 반복적인 문제를 해결할 수 있습니다.
    - JdbcTemplate은 템플릿 콜백 패턴을 사용해서, JDBC를 직접 사용할 때 발생하는 대부분의 반복 작업을 대신 처리해줍니다.
    - 개발자는 SQL을 작성하고, 전달할 파리미터를 정의하고, 응답 값을 매핑하기만 하면 됩니다.
    - 대부분의 반복 작업을 대신 처리해줍니다.
        - 커넥션 획득
        - `statement` 를 준비하고 실행
        - 결과를 반복하도록 루프를 실행
        - 커넥션 종료, `statement`, `resultset` 종료
        - 트랜잭션 다루기 위한 커넥션 동기화
        - 예외 발생시 스프링 예외 변환기 실행 

**단점**
- 동적 SQL을 해결하기 어렵습니다.

## 🤷🏻‍♂️ 그럼 JdbcTemplate를 어떻게 활용하나요?
우선, 단계별로 JdbcTemplate를 어떻게 활용하는지 알아보겠습니다.

**save**
```java
@Override
public Item save(Item item) {
    String sql = "insert into item (item_name, price, quantity) values (?, ?, ?)";
    KeyHolder keyHolder = new GeneratedKeyHolder();
    
    template.update(connection -> {
        // 자동 증가 키
        PreparedStatement ps = connection.prepareStatement(sql, new String[]{"id"});
        ps.setString(1, item.getItemName());
        ps.setInt(2, item.getPrice());
        ps.setInt(3, item.getQuantity());
        return ps;
    }, keyHolder);
    
    long key = keyHolder.getKey().longValue();
    item.setId(key);
    return item;
}
```
save는 데이터를 저장하는 메서드입니다. 데이터를 변경할 때는 `template.update()`를 사용하면 됩니다.

해당 update()는 SQL에서 INSERT, UPDATE, DELETE를 사용할 때 사용합니다. 그리고 반환 값은 int인데, 이는 영향받은 row의 개수를 뜻합니다.

만약 PK 생성에 `identity` (auto increment) 방식을 사용한다면, 데이터를 저장할 때 PK 부분은 비워두고 저장해야 합니다. 그러면 데이터베이스에서 자동으로 PK 값을 생성해 줍니다.

그리고 해당 PK 값은 `KeyHolder`를 활용해 INSERT 이후에 데이터베이스에 생성된 데이터의 PK 값을 확인할 수 있습니다. 

**update**
```java
@Override
public void update(Long itemId, ItemUpdateDto updateParam) {
    String sql = "update item set item_name=?, price=?, quantity=? where id=?";
    
    template.update(sql,
        updateParam.getItemName(),
        updateParam.getPrice(),
        updateParam.getQuantity(),
        itemId);
}
```
update 또한 save와 똑같이 `template.update()`를 사용하면 됩니다. 여기서 '?'에 바인딩할 파라미터를 순서대로 전달하면 됩니다. 하지만 이는 바인딩할 순서를 "무조건" 지켜야 합니다.

반환 값 역시, save와 똑같이 영향받은 row의 개수입니다.

**findById**
```java
@Override
public Optional<Item> findById(Long id) {
    String sql = "select id, item_name, price, quantity from item where id = ?";
    
    try {
        Item item = template.queryForObject(sql, itemRowMapper(), id);
        return Optional.of(item);
    } catch (EmptyResultDataAccessException e) {
        return Optional.empty();
    }
}

private RowMapper<Item> itemRowMapper() {
    return ((rs, rowNum) -> {
        Item item = new Item();
        item.setId(rs.getLong("id"));
        item.setItemName(rs.getString("item_name"));
        item.setPrice(rs.getInt("price"));
        item.setQuantity(rs.getInt("quantity"));
        return item;
    });
}
```

`template.queryForObject()`는 결과 로우가 하나일 때 사용합니다.  

`RowMapper` 는 데이터베이스의 반환 결과인 `ResultSet` 을 객체로 변환합니다.

결과가 없을 때는 `Optional` 을 반환해야 합니다. 그래서 결과가 없으면 예외를 잡아서 `Optional.empty` 를 대신 반환하면 됩니다.

**findAll**
```java
@Override
public List<Item> findAll(ItemSearchCond cond) {
    String itemName = cond.getItemName();
    Integer maxPrice = cond.getMaxPrice();
    String sql = "select id, item_name, price, quantity from item";
    
    // 동적 쿼리
    if (StringUtils.hasText(itemName) || maxPrice != null) {
        sql += " where";
    }
    
    boolean andFlag = false;
    List<Object> param = new ArrayList<>();
    
    if (StringUtils.hasText(itemName)) {
        sql += " item_name like concat('%',?,'%')";
        param.add(itemName);
        andFlag = true;
    }
    
    if (maxPrice != null) {
        if (andFlag) {
            sql += " and";
        }
        sql += " price <= ?";
        param.add(maxPrice);
    }
    
    log.info("sql={}", sql);
    return template.query(sql, itemRowMapper(), param.toArray());
}

private RowMapper<Item> itemRowMapper() {
    return ((rs, rowNum) -> {
        Item item = new Item();
        item.setId(rs.getLong("id"));
        item.setItemName(rs.getString("item_name"));
        item.setPrice(rs.getInt("price"));
        item.setQuantity(rs.getInt("quantity"));
        return item;
    });
}
```

`template.query()`는 결과가 하나 이상일 때 사용합니다. 만약 결과가 없으면 빈 컬렉션을 반환합니다.

## 🤷🏻‍♂️ 동적 쿼리는 왜 문제가 되나요?

위 findAll()에서 어려운 부분은 사용자가 검색하는 값에 따라서 실행하는 SQL이 동적으로 달려져야 한다는 점입니다.

여러 조건 검색을 다양한 상황에 따라 쿼리를 작성한다면, 아래와 같은 쿼리들이 발생할 수 있습니다.

```sql
select id, item_name, price, quantity from item
```

```sql
select id, item_name, price, quantity from item where item_name like concat('%',?,'%')
```

```sql
select id, item_name, price, quantity from item
where price <= ?
```

```sql
select id, item_name, price, quantity from item
where item_name like concat('%',?,'%')
and price <= ?
```

눈으로 언뜻보면 간단해 보이지만, 동적 쿼리를 생성할 때 고려해야 할 점은 무수히 많습니다.

예를 들면, 어떤 경우에는 `where` 를 앞에 넣고 어떤 경우에는 `and` 를 넣어야 하는지 등을 모두 계산해야 합니다. 그리고 각 상황에 맞추어 파라미터도 생성해야 합니다.

그리고 이는 간단한 공부에서 나온 쿼리이고, 실무에서는 더욱 더 복잡한 쿼리문이 발생하게 됩니다.

## 🤷🏻‍♂️ 바인딩 문제들은 어떻게 해결해야 하나요?

JdbcTemplate는 기본으로 사용하면 파라미터를 순서대로 바인딩합니다.

```sql
String sql = "update item set item_name=?, price=?, quantity=? where id=?";
template.update(sql,
    itemName,
    price,
    quantity,
    itemId);
```

순서를 잘 지키면 정말 다행이지만, 누군가 순서를 바꾸는 일이 발생한다면 문제가 생깁니다. 실무에서 파라미터는 10-20개 정도라고 하며, 가장 힘든 일이 데이터베이스에 데이터가 잘못 들어갔을 때라고 합니다.

그래서 이를 어떻게 해결할까?

**1️⃣ NamedParameterJdbcTemplate**
```java
@Override
public Item save(Item item) {
    String sql = "insert into item (item_name, price, quantity) " +
    "values (:itemName, :price, :quantity)";
    SqlParameterSource param = new BeanPropertySqlParameterSource(item);
    KeyHolder keyHolder = new GeneratedKeyHolder();
    template.update(sql, param, keyHolder);
    Long key = keyHolder.getKey().longValue();
    item.setId(key);
    return item;
}
```

위 코드를 보면 "?" 대신, `:파라미터 이름`을 사용하면 됩니다.

**2️⃣ Map**
단순히 Map을 사용하면 됩니다.

```java
Map<String, Object> param = Map.of("id", id);
Item item = template.queryForObject(sql, param, itemRowMapper());
```

**3️⃣ MapSqlParameterSource**
이는 `SqlParameterSource` 인터페이스의 구현체입니다. 그리고 `Map` 과 유사한데, SQL 타입을 지정할 수 있는 등 SQL에 좀 더 특화된 기능을 제공합니다. 또 메서드 체인을 통해 편리한 사용법도 제공합니다.

```java
SqlParameterSource param = new MapSqlParameterSource()
    .addValue("itemName", updateParam.getItemName())
    .addValue("price", updateParam.getPrice())
    .addValue("quantity", updateParam.getQuantity())
    .addValue("id", itemId); //이 부분이 별도로 필요하다.
    template.update(sql, param);
```

**4️⃣ BeanPropertySqlParameterSource**
자바빈 프로퍼티 규약을 통해서 자동으로 파라미터 객체를 생성합니다. 예를 들어서 `getItemName()` , `getPrice()` 가 있으면 다음과 같은 데이터를 자동으로 만들어냅니다.

```java
SqlParameterSource param = new BeanPropertySqlParameterSource(item);
KeyHolder keyHolder = new GeneratedKeyHolder();
template.update(sql, param, keyHolder);
```

지금까지 나온 내용들을 보면,  `BeanPropertySqlParameterSource` 가 많은 것을 자동화 해주기 때문에 가장 좋아보이지만, 이는 항상 사용할 수 있는 것은 아닙니다.

예를 들어서 `update()` 에서는 SQL에 `:id` 를 바인딩 해야 하는데, `update()` 에서 사용하는 `ItemUpdateDto` 에는 `itemId` 가 없다고 가정해 봅시다. 그렇다면 `BeanPropertySqlParameterSource` 를 사용할 수 없고, 대신에 `MapSqlParameterSource` 를 사용해야 합니다.

## 🤷🏻‍♂️ itemRowMapper()를 간략화할 수는 없나요?
```java
private RowMapper<Item> itemRowMapper() {
    return (rs, rowNum) -> {
    Item item = new Item();
    item.setId(rs.getLong("id"));
    item.setItemName(rs.getString("item_name"));
    item.setPrice(rs.getInt("price"));
    item.setQuantity(rs.getInt("quantity"));
    return item;
    };
}
```

위 코드에서 아래 코드처럼 간략화할 수 있습니다.

```java
private RowMapper<Item> itemRowMapper() {
    return BeanPropertyRowMapper.newInstance(Item.class); //camel 변환 지원
}
```

`BeanPropertyRowMapper` 는 `ResultSet` 의 결과를 받아서 자바빈 규약에 맞추어 데이터를 변환해 줍니다. 실제로 리플렉션과 같은 기능을 사용합니다.

대부분 자바와 DB로 매핑하는 과정에서 변수명 떄문에 문제를 겪습니다. 예를 들어, 자바에서는 userId라고 저장을 했지만, DB에는 user_id로 되어 있어서 매핑하는 과정에서 이름이 달라 에러가 발생합니다. 

그래서 이를 "as"를 활용해서 별칭을 만들어 해결할 수 있습니다. 그리고 `BeanPropertyRowMapper` 는 언더스코어 표기법을 카멜로 자동 변환해 주는 기능이 있습니다. 

## 🤷🏻‍♂️ INSERT 문을 작성하기 너무 귀찮은데 어떡하죠?
JdbcTemplate은 INSERT SQL를 직접 작성하지 않아도 되도록 `SimpleJdbcInsert` 라는 편리한 기능을 제공합니다.

```java
public class JdbcTemplateItemRepositoryV3 implements ItemRepository {
    private final NamedParameterJdbcTemplate template;
    private final SimpleJdbcInsert jdbcInsert;
    public JdbcTemplateItemRepositoryV3(DataSource dataSource) {
        this.template = new NamedParameterJdbcTemplate(dataSource);
        this.jdbcInsert = new SimpleJdbcInsert(dataSource)
            .withTableName("item")
            .usingGeneratedKeyColumns("id");
            //.usingColumns("item_name", "price", "quantity"); //생략 가능
    }
    @Override
    public Item save(Item item) {
        SqlParameterSource param = new BeanPropertySqlParameterSource(item);
        Number key = jdbcInsert.executeAndReturnKey(param);
        item.setId(key.longValue());
        return item;
    }
}
```

하나씩 코드를 살펴보면 아래와 같습니다.

- `withTableName` : 데이터를 저장할 테이블 명을 지정합니다.
- `usingGeneratedKeyColumns` : `key` 를 생성하는 PK 컬럼 명을 지정합니다.
- `usingColumns` : INSERT SQL에 사용할 컬럼을 지정한다. 특정 값만 저장하고 싶을 때 사용합니다. 그리고 생략할 수 있습니다.

`SimpleJdbcInsert` 는 생성 시점에 데이터베이스 테이블의 메타 데이터를 조회합니다. 따라서 어떤 컬럼이 있는지 확인 할 수 있으므로 `usingColumns` 을 생략할 수 있습니다. 만약 특정 컬럼만 지정해서 저장하고 싶다면 `usingColumns`를 사용하면 됩니다.

## 💡 결론
JdbcTemplate이 제공하는 주요 기능은 다음과 같습니다.

- `JdbcTemplate`
    - 순서 기반 파라미터 바인딩을 지원합니다.
- `NamedParameterJdbcTemplate`
    - 이름 기반 파라미터 바인딩을 지원합니다. (권장)
- `SimpleJdbcInsert`
    - INSERT SQL을 편리하게 사용할 수 있습니다.
- `SimpleJdbcCall`
    - 스토어드 프로시저를 편리하게 호출할 수 있습니다.

>스토어드 프로시저(Stored Procedure)는 데이터베이스에 저장된 SQL 문들의 집합으로, 하나의 프로그램처럼 실행할 수 있는 데이터베이스 객체입니다.

하지만 여전히 동적 쿼리의 문제를 해결하지는 못했다. 이를 해결하기 위해 사용하는 기술이 바로 MyBatis이다.