# Shallow Copy And Deep Copy

**얕은 복사(Shallow Copy)**와 **깊은 복사(Deep Copy)**는 객체를 복사하는 두 가지 방법입니다.

### 1. 얕은 복사 (Shallow Copy)
원본 객체의 **참조값만 복사**하는 방식입니다.

**특징**
- 원본 객체와 복사된 객체가 **같은 메모리 주소를 참조**합니다.
- 원본 객체의 변경이 복사된 객체에 영향을 줍니다.
- **빠르고 메모리 효율적**입니다.

### 2. 깊은 복사 (Deep Copy)
원본 객체의 **모든 데이터를 새로운 메모리 공간에 복사**하는 방식입니다.

**특징**
- 원본 객체와 복사된 객체가 **완전히 독립적**입니다.
- 원본 객체의 변경이 복사된 객체에 영향을 주지 않습니다.
- **느리고 메모리를 더 사용**합니다.

조금 더 이해를 돕기 위해 Bookr과 Author라는 두 클래스를 예제로 사용하겠습니다.

### 예제 코드
```java
class Book {
    private String name; // 책 이름
    private Author author; // 저자

    public Book(String name, Author author) {
        this.name = name;
        this.author = author;
    }

    public Book shallowCopy() { // 얕은 복사
        return new Book(this.name, this.author);
    }

    public Book deepCopy() { // 깊은 복사
        Author copiedAuthor = new Author(this.author.getName());
        return new Book(this.name, copiedAuthor);
    }

    public void changeAuthor(String name) { // 저자 이름 변경
        author.setName(name);
    }

    @Override
    public String toString() {
        return "Book name : " + name + ", " + author;
    }

    static class Author {

        private String name; // 저자 이름

        public Author(String name) {
            this.name = name;
        }

        public String getName() { // 저자 이름 반환
            return name;
        }

        public void setName(String name) { // 저자 이름 변경
            this.name = name;
        }

        @Override
        public String toString() {
            return "Author : " + name;
        }
    }

    public static void main(String[] args) {
        Author author1 = new Author("조슈아 블로크");
        Book book1 = new Book("이펙티브 자바", author1);

        // 얕은 복사 후 변경
        Book shallowCopyBook = book1.shallowCopy();
        shallowCopyBook.changeAuthor("Joshua Bloch");

        // 얕은 복사 결과 출력
        System.out.println("After shallow copy and change:");
        System.out.println("Original book1: " + book1);
        System.out.println("Shallow copied book: " + shallowCopyBook);

        Author author2 = new Author("마틴 파울러");
        Book book2 = new Book("리팩터링", author2);

        // 깊은 복사 후 변경
        Book deepCopyBook = book2.deepCopy();
        deepCopyBook.changeAuthor("Martin Fowler");

        // 깊은 복사 결과 출력
        System.out.println("After deep copy and change:");
        System.out.println("Original book2: " + book2);
        System.out.println("Deep copied book: " + deepCopyBook);
    }
}
```
### 출력 예시
```java
After shallow copy and change:
Original book1: Book name : 이펙티브 자바, Author : Joshua Bloch
Shallow copied book: Book name : 이펙티브 자바, Author : Joshua Bloch
  ㄹ
After deep copy and change:
Original book2: Book name : 리팩터링, Author : 마틴 파울러
Deep copied book: Book name : 리팩터링, Author : Martin Fowler
```

`shallowCopy()` 메서드는 새로운 Book 객체를 만들지만, 내부의 Author 객체는 원본과 동일한 객체를 참조합니다. 

즉, Book 객체는 새로 만들었지만, Author 객체는 새로 만들지 않고 기존의 것을 그대로 사용합니다. 

예를 들어, book1에서 `shallowCopyBook`을 만든 후, `shallowCopyBook`의 저자 이름을 “Joshua Bloch”로 바꾸면 book1의 저자 이름도 “Joshua Bloch”로 바뀝니다. 둘이 같은 Author 객체를 공유하고 있기 때문에 두 Book 객체의 Author가 동시에 변경됩니다.

반면 `deepCopy()` 메서드는 Book 객체와 Author 객체 모두 새로운 객체로 만들어줘요. 그래서 book2에서 `deepCopyBook`을 만들고 `deepCopyBook`의 저자 이름을 “Martin Fowler”로 바꾸어도, book2의 저자 이름은 여전히 “마틴 파울러”로 남아 있어요. `deepCopyBook`과 book2가 서로 다른 Author 객체를 참조하고 있기 때문입니다.

객체의 복사 방식에 따라 원본 객체와 복사된 객체 간의 관계가 달라지게 됩니다.


## 🤷🏻‍♂️ 그럼 보통 개발을 할 때, 원본에 영향을 줘야하기 때문에 얕은 복사만 쓰면 되는거 아닌가요?

꼭 그렇지는 않습니다. 각자 상황에 맞게 적절히 사용해야 합니다.

### 얕은 복사 사용 시기
- **불변 객체(Immutable Object)**를 다룰 때
- **원본 객체의 변경을 허용**할 때
- **성능이 중요한 경우**

### 깊은 복사 사용 시기
- **원본 객체의 변경을 방지**하고 싶을 때
- **독립적인 객체가 필요한 경우**
- **데이터 무결성이 중요한 경우**

### 복사 방식 비교

| 특징 | 얕은 복사 | 깊은 복사 |
|------|-----------|-----------|
| 메모리 사용량 | 적음 | 많음 |
| 성능 | 빠름 | 느림 |
| 독립성 | 없음 | 있음 |
| 구현 복잡도 | 간단 | 복잡 |

### 주의사항

**얕은 복사의 위험성**
- 원본 객체 변경 시 예상치 못한 부작용 발생 가능
- 멀티스레드 환경에서 동시성 문제 발생 가능

**깊은 복사의 고려사항**
- 순환 참조가 있는 객체의 경우 무한 루프 발생 가능
- 복잡한 객체 구조의 경우 구현이 어려움