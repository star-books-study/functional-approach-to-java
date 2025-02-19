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

