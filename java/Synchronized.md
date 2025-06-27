# Synchronized

`Synchronized`는 Java에서 멀티스레드 환경에서 동시성 문제를 해결하기 위한 동기화 메커니즘입니다. 여러 스레드가 동시에 공유 자원에 접근할 때 발생할 수 있는 데이터 불일치 문제를 방지합니다.

## 🤷🏻‍♂️ 동기화가 왜 필요한가요?
멀티스레드 환경에서는 여러 스레드가 동시에 같은 데이터를 수정할 때 **Race Condition**이 발생할 수 있습니다. 이는 예상치 못한 결과나 데이터 손실을 야기하므로 Thread Safety(스레드 안전성)를 보장해야 합니다.

## 🤷🏻‍♂️ Synchronized를 어떻게 사용하나요?

### 1. 메서드 레벨 동기화
```java
public class Counter {
    private int count = 0;
    
    // 인스턴스 메서드 동기화
    public synchronized void increment() {
        count++;
    }
    
    // 정적 메서드 동기화
    public static synchronized void staticMethod() {
        // 클래스 레벨 락 사용
    }
}
```

### 2. 블록 레벨 동기화
```java
public class Counter {
    private int count = 0;
    private Object lock = new Object();
    
    public void increment() {
        synchronized(lock) {
            count++;
        }
    }
}
```

## 🤷🏻‍♂️ 동기화의 원리는 어떻게 되나요?

### 모니터 락 (Monitor Lock)
- 각 객체는 하나의 모니터 락을 가집니다.
- synchronized 블록에 진입할 때 락을 획득합니다.
- 블록을 벗어날 때 락을 해제합니다.

### 락의 종류
- **인스턴스 락**: synchronized 인스턴스 메서드나 synchronized(this)
- **클래스 락**: synchronized 정적 메서드나 synchronized(Class.class)

## 🤷🏻‍♂️ Synchronized의 장단점은 무엇인가요?

### 장점
- **Thread Safety**: 스레드 안전성 보장
- **단순성**: 사용하기 쉬운 동기화 메커니즘
- **자동 락 해제**: 예외 발생 시에도 자동으로 락 해제

### 단점
- **성능 오버헤드**: 락 획득/해제 비용
- **블로킹**: 다른 스레드가 대기해야 함
- **데드락 위험**: 잘못된 사용 시 데드락 발생 가능

## 🤷🏻‍♂️ 실제 사용 예제를 보여주세요.

```java
public class BankAccount {
    private double balance;
    private final Object lock = new Object();
    
    public void deposit(double amount) {
        synchronized(lock) {
            if (amount > 0) {
                balance += amount;
            }
        }
    }
    
    public boolean withdraw(double amount) {
        synchronized(lock) {
            if (amount > 0 && balance >= amount) {
                balance -= amount;
                return true;
            }
            return false;
        }
    }
}
```

## 🤷🏻‍♂️ 주의사항은 무엇인가요?

### 데드락 방지
```java
// 잘못된 예 - 데드락 위험
public void method1() {
    synchronized(lock1) {
        synchronized(lock2) { /* 작업 */ }
    }
}

public void method2() {
    synchronized(lock2) {
        synchronized(lock1) { /* 작업 */ }
    }
}

// 올바른 예 - 일관된 락 순서
public void method1() {
    synchronized(lock1) {
        synchronized(lock2) { /* 작업 */ }
    }
}

public void method2() {
    synchronized(lock1) {
        synchronized(lock2) { /* 작업 */ }
    }
}
```

## 🤷🏻‍♂️ 성능 최적화 방법은 무엇인가요?

### 락의 범위 최소화
```java
// 비효율적
public synchronized void processData() {
    heavyComputation();  // 긴 작업
    updateSharedData();  // 짧은 동기화 필요 부분
}

// 효율적
public void processData() {
    heavyComputation();
    synchronized(this) {
        updateSharedData();
    }
}
```

## 🤷🏻‍♂️ Synchronized의 대안은 무엇인가요?

### 1. ReentrantLock
```java
private final ReentrantLock lock = new ReentrantLock();

public void increment() {
    lock.lock();
    try {
        count++;
    } finally {
        lock.unlock();
    }
}
```

### 2. Atomic 클래스
```java
private AtomicInteger count = new AtomicInteger(0);

public void increment() {
    count.incrementAndGet();
}
```

## 🤷🏻‍♂️ 결론
`Synchronized`는 Java에서 가장 기본적이고 널리 사용되는 동기화 메커니즘입니다. 적절히 사용하면 스레드 안전성을 보장할 수 있지만, 성능과 데드락 위험을 고려하여 신중하게 사용해야 합니다. 상황에 따라 `ReentrantLock`, `Atomic` 클래스 등의 대안을 고려하는 것이 좋습니다.