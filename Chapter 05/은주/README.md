# Chapter 05. 레코드

## 5.1. 데이터 집계 유형
- 데이터 집계 : 다양한 소스로부터 데이터를 수집하고 의도한 목적에 부합하도록 표현하는 과정

### 5.1.1. 튜플
- 튜플 : 요소의 유한하며 순서가 있는 집합
  - 프로그래밍 언어에서는 여러 값 또는 객체를 모은 자료 구조를 의미한다
- 구조적 튜플 : 요소들의 순서에만 의존하므로 `인덱스`를 통해 접근 가능
- 명목상 튜플 : 인덱스를 사용하지 않고 `컴포넌트명` 사용하여 접근 가능

### 5.1.2. 간단한 POJO
```java
public final class UserPOJO {

    private String        username;
    private boolean       active;
    private LocalDateTime lastLogin;

    public UserPOJO() { }

    public UserPOJO(String username,
                    boolean active,
                    LocalDateTime lastLogin) {
        this.username = username;
        this.active = active;
        this.lastLogin = lastLogin;
    }

    public String getUsername() {
        return this.username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public boolean isActive() {
        return this.active;
    }

    public void setActive(boolean active) {
        this.active = active;
    }

    public LocalDateTime getLastLogin() {
        return this.lastLogin;
    }

    public void setLastLogin(LocalDateTime lastLogin) {
        this.lastLogin = lastLogin;
    }

    @Override
    public int hashCode() {
        return Objects.hash(this.username, this.active, this.lastLogin);
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }

        if (obj == null || getClass() != obj.getClass()) {
            return false;
        }

        UserPOJO other = (UserPOJO) obj;
        return Objects.equals(this.username, other.username)
               && this.active == other.active
               && Objects.equals(this.lastLogin, other.lastLogin);
    }

    @Override
    public String toString() {
        return new StringBuilder().append("UserPOJO [username=")
                                  .append(this.username)
                                  .append(", active=")
                                  .append(this.active)
                                  .append(", lastLogin=")
                                  .append(this.lastLogin)
                                  .append("]")
                                  .toString();
    }

    public static void main(String... args) {

        var user = new UserPOJO();
        user.setUsername("ben");
        user.setActive(true);

        System.out.println("user = " + user);
    }
}
```
- 인수를 가진 생성자가 있을 경우 빈 생성자도 정의해야 한다
- getter / setter 메서드를 가지고 있다
- **hashCode 와 equals 메서드는 변경 사항이 생길 경우 모두 수정 필요하다**
- toString 은 반드시 필요하진 않지만 다른 메서드와 마찬가지로 유형 변경시마다 수정 필요하다
- 이처럼 기본적인 작업을 수행하려면 **형식적인 코드가 과도하게 많이 필요하다**

### 5.1.3. 불변 POJO 만들기
```java
public final class UserImmutable {

    private final String        username;
    private final boolean       active;
    private final LocalDateTime lastLogin;

    public UserImmutable(String username,
                         boolean active,
                         LocalDateTime lastLogin) {
        this.username = username;
        this.active = active;
        this.lastLogin = lastLogin;
    }

    public String getUsername() {
        return this.username;
    }

    public boolean isActive() {
        return this.active;
    }

    public LocalDateTime getLastLogin() {
        return this.lastLogin;
    }

    @Override
    public int hashCode() {
        return Objects.hash(this.username,
                            this.active,
                            this.lastLogin);
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }

        if (obj == null || getClass() != obj.getClass()) {
            return false;
        }

        UserImmutable other = (UserImmutable) obj;
        return Objects.equals(this.username, other.username)
               && this.active == other.active
               && Objects.equals(this.lastLogin, other.lastLogin);
    }

    @Override
    public String toString() {
        return new StringBuilder().append("UserImmutable [username=")
                                  .append(this.username)
                                  .append(", active=")
                                  .append(this.active)
                                  .append(", lastLogin=")
                                  .append(this.lastLogin)
                                  .append("]")
                                  .toString();
    }

    public static void main(String... args) {

        var user = new UserImmutable("ben", true, null);
        System.out.println("user = " + user);
    }
}
```
- final 로 필드가 선언되어 데이터를 변형할 수 없게 하면, setter 메서드가 불필요하다
- 필드는 반드시 객체 생성 시 선언되어야 하므로 완전한 '통과' 생성자만 사용 가능하다
- getter 를 가지고 있다
- **부가적으로 생성된 메서드는 변경되지 않는다**
  
### 5.1.4. POJO 를 레코드로 만들기
```java
public record User(
    String username, 
    boolean active, 
    LocalDateTime lastLogin
) {
    ...
}
```
- User 레코드는 불변 POJO 와 동일한 기능을 갖추고 있다

## 5.2. 도움을 주기 위한 레코드
- 레코드에서는 다른 데이터 클래스들이 갖고 있는 **전형적인 보일러 플레이트 코드가 대폭 줄어든다**
  - 일반적인 접근자와 데이터를 다루는 equals, hashCode 메서드를 자동 생성해주므로 !
- 제네릭 타입은 자바에서의 다른 타입과 마찬가지로 지원된다
- 데이터 컴포넌트는 아래와 같이 변환된다
  - `private final` 필드
  - `public 접근자` 메소드
- 이 클래스는 java.lang.Object 가 아닌 **java.lang.Record 를 명시적으로 상속**한다

### 5.2.2. 레코드의 특징
- 반복적으로 보일러플레이트를 작성할 필요 없이 효율적인 기능을 제공한다

#### 컴포넌트 접근자
- 모든 레코드 컴포넌트는 private 필드로 저장되며, 외부에서는 public 접근자 메서드를 통해서만 접근 가능하다

#### 표준, 간결, 사용자 정의 생성자
- 표준 생성자 : 레코드의 각 컴포넌트에 따라 자동으로 생성되는 생성자
  - 기본 생성자는 입력값의 유효성을 검사하거나 필요에 따라 데이터를 약간 조정하기 위해 재정의 될 수 있음
```java
record User(String username,
            boolean active,
            LocalDateTime lastLogin) {

    public User(String username,
                boolean active,
                LocalDateTime lastLogin) {
        
        Objects.requireNonNull(username);
        Objects.requireNonNull(lastLogin);

        this.username = username.toLowerCase();
        this.active = active;
        this.lastLogin = lastLogin;
    }
}
```
- 간결하게 디자인된 아래와 같은 형태도 제공된다
  - 생성자에서는 괄호를 포함하여 모든 인수를 생략한다
  - 컴팩트 생성자는 유효성 검사를 수행하기에 완벽한 지점이다
```java
record User(String username,
            boolean active,
            LocalDateTime lastLogin) {

    public User {
        Objects.requireNonNull(username);
        Objects.requireNonNull(lastLogin);
        username = username.toLowerCase();
    }
}
```

#### 객체 식별과 설명
- 레코드 타입의 두 인스턴스는 컴포넌트의 데이터가 동일하면 동일하다고 간주된다
- toString() 역시 컴포넌트에서 자동 생성된다

#### 어노테이션
- 어노테이션을 지원하기 위해서 대상 필드, 파라미터, 또는 메서드가 있는 어노테이션은 컴포넌트에 적용되는 경우 해당 위치로 전파된다

### 5.2.3. 누락된 기능
#### 추가적인 상태
- 레코드는 바디에 필드가 추가되는 경우 컴파일러 오류가 발생한다
- 메소드는 필드와는 달리 상태관리를 하지 않기에 추가 가능하다

#### 상속
- **레코드는 이미 내부적으로 java.lang.Record 를 상속한 final 타입**이다. 
  - 이미 하나의 타입을 상속하고 있으므로, 레코드는 상속을 사용할 수 없다
- 모든 구현체가 인터페이스 계약을 공유하는 한, 인터페이스와 default 메서드를 통해 동작 공유하는 것은 간단한 접근 방식일 수 있다.

#### 컴포넌트의 기본값과 컴팩트 생성자
- 정적 팩토리 메서드는 사용자 정의 생성자보다 더 직관적이며, 중복되는 시그니처에 대한 최선의 해결책이 될 수 있다

#### 단계별 생성
- 불변 자료 구조에서는 하프초기화 (객체나 시스템이 완벽하게 초기화되지 않은 상태. 즉, 변수 중 일부가 아직 할당되지 않은 경우) 상태의 객체가 없다
- **빌더패턴을 활용해서 변경 가능한 중간 단계 변수 사용하고, 최종적으로는 불변 결과를 생성할 수 있다**
- 빌더 디자인 패턴
  - 필요한 데이터가 준비될 때까지 처리를 연기하여, 단계적으로 자료 구조를 설계할 수 있다
  - 빌더 클래스는 복잡한 자료 구조를 생성하는 것만 담당하며, 구조 자체는 데이터의 표현하는 것에만 집중한다
- 타입 내부에 빌더 클래스를 **정적 중첩 클래스로 직접 배치하면 타입과 빌더간의 결합성을 강화할 수 있다**

## 5.3. 사용 사례와 일반적인 관행
### 5.3.1. 레코드 유효성 검사 및 데이터 정제
- 레코드는 컴팩트 생성자를 제공하는데, 이는 표준 생성자의 모든 컴포넌트에 접근 가능하며 별도의 인수가 필요하지 않다.
  - **유효성 검사 및 데이터 정제 로직** 넣기에 이상적!
  - 특정 로직을 레코드 내부로 이동시키면, 원본 데이터에 상관 없이 일관된 데이터 표현 확보 가능!

### 5.3.4. 명목상 튜플로써의 로컬 레코드
- 로컬 레코드를 사용해서 메서드 내부에 레코드를 선언하여 스코프를 제한할 수 있다
- 로컬 레코드를 활용하면 후속 연산을 할 때 명시적인 람다 표현식 대신 **로컬 레코드의 직관적인 메서드 또는 메서드 참조를 사용할 수 있으므로**, 선언적 스트림 파이프 라인의 사용성과 명료성을 향상시킬 수 있다

### 5.3.5. Optional 데이터 처리
- Optional 을 통해서 값이 null 일지라도 NullPointerException을 발생시키지 않고 컨테이너와 안전하게 상호작용 가능하다
  
#### null 이 아닌 컨테이너 확보
- Optional 을 사용할 때 컴팩트 생성자에서 Objects.requireNonNull 로 유효성을 검사한다

#### 컴팩트 생성자 추가
- Non-Optional<T> 기반의 인수들로 추가 생성자를 제공해서 Optional.ofNullable 로 감싸서 생성자를 만든다

### 5.3.6. 레코드의 진화된 직렬화
- 레코드는 클래스와 마찬가지로 java.io.Serializable 마커 인터페이스 상속 시 자동으로 직렬화되며 추가적인 코드가 필요하지 않다
- 비레코드 객체의 직렬화는 private 상태에 접근하기 위해 고비용의 리플렉션을 사용한다
  - 리플렉션은 성능 오버헤드와 보안문제가 있다
    - 동적으로 결정된 타입 정보를 활용하기에 JVM 의 모든 최적화 기법을 적용하기 어려움 -> 따라서 다소 성능이 느림
- 레코드는 **불변 상태로만 정의**되므로 생성 후에 상태에 영향을 주는 어떤 코드도 없어 직렬화 과정이 간단하다
  - 직렬화 : 레코드의 `컴포넌트` 만을 기반으로 함
  - 역직렬화 : 리플렉션 대신 `기본 생성자`만을 필요로 함
- 역직렬화 과정에서 값이 없거나, 알려지지 않은 컴포넌트 발생 시 기본값이 자동 적용된다
- 레코드는 자신의 컴포넌트들에 의해서만 정의되므로, **두 레코드가 동일한 컴포넌트를 가져도 서로 상호 교환해서 사용할 수 없다.** (직렬화된 대상의 타입이 다르기 때문)

### 5.3.7. 레코드 패턴 매칭 (자바 19+)
- 자바 16 에서는 instanceof 연산자에 대한 패턴 매칭이 등장하며 연산자 사용 후 별도 형변환을 해야 할 필요성이 사라졌다
- 자바 17, 18도 switch 표현식에 대한 패턴 매칭을 도입했다
- 자바 19 버전 이후에는 구조 분해 기능을 포함하여 **레코드의 컴포넌트들이 해당 범위 내에서 변수로 직접 사용**할 수 있게 되었다

## 5.4. 레코드를 마무리하며
- 레코드는 가능한 한 적은 코드로 **극도의 간결성**을 제공한다
- **안전하고 일관된 사용성**을 제공하며, POJO 나 사용자 정의 타입처럼 유연하지 않을 수는 있지만 그만큼 버그 위험을 감소시키며 기능을 제공한다
  
## 핵심 요약
- 레코드는 `컴포넌트` 에 의해서만 정의되는 투명한 데이터 애그리게이터 타입이다
- 레코드 타입은 전형적인 보일러플레이트 코드 없이 모든 레코드 타입에서 사용 가능하다