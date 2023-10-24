PSA 제대로 이해하기
====================
![Spring 로고](https://github.com/sh8121/tech-blog/assets/20632477/6d1be92a-ea3c-436a-80bc-55abb93a9e84)

Spring 프로젝트를 관통하는 핵심 개념으로 가장 많이 꼽히는 것이 IOC, PSA, AOP 입니다. 이 세가지 개념을 두고 흔히 Spring 의 3대 요소 혹은 Spring Triangle 이라고 표현합니다.   
이번 글에서는 이 중 PSA 에 대한 원론적인 이야기를 해보겠습니다.   
PSA 는 Portable Service Abstraction 의 약자입니다. 번역하면 '휴대가 용이한 서비스 추상화' 정도가 되겠습니다. PSA 를 정확히 이해하려면 '서비스 추상화' 와 '휴대의 용이성' 이라는 두가지 측면을 나누어서 살펴봐야 합니다.   

## 서비스 추상화
'추상화' 는 사전적으로는 '복잡한 자료, 모듈, 시스템 등으로부터 핵심적인 개념 또는 기능을 간추려 내는 것' 을 의미합니다. 프로그래밍 관점에서 해석하면 '여러 객체 들이 갖는 개별 속성이나 각각의 기능을 배제하고 공통의 클래스를 뽑아내는 것' 이나 '각각의 구현체의 고유 행위는 배제한체 공통의 책임과 역할로 인터페이스를 뽑아내는 것' 으로 해석할 수 있겠습니다.   
'서비스 추상화' 는 결국 Spring 이 제공하는 여러가지 서비스(Transaction, Cache, Logging, Metric...) 들이 추상화된 형태로 제공된다는 의미입니다.   
구체적인 예시를 통해 서비스 추상화를 좀 더 자세히 들여다 보겠습니다.   
### Transaction 추상화
Spring Transaction 은 서비스 추상화가 적용된 대표적인 기능입니다. 예제를 통해 자세히 알아보겠습니다. Java Application 에서 Transaction 을 사용하는 가장 기본적인 코드는 아래와 같습니다.   
```java
Connection conn = DriverManager.getConnection("jdbc:h2:tcp://localhost/~/test", "SA", "");
conn.setAutoCommit(false);
try {
    //데이터 관련 처리 로직
    conn.commit();
} catch (SQLException ex) {
    conn.rollback();
} finally {
    conn.close();
}
```
JDBC API 를 통해 Transaction 을 시작하고(conn.setAutoCommit(false)) 작업의 성공 여부에 따라 Commit, Rollback 을 수행한 후 리소스를 회수하는 전형적인 코드입니다.(Transaction 자체를 다루는 글이 아니므로 JDBC API 나 Transaction 에 대한 자세한 설명은 생략하겠습니다.)    
보통 Transaction 의 시작과 종료는 Service 계층(비즈니스 로직 수행 계층) 에서 일어나고, 구체적인 데이터 처리는 Repository 계층(데이터 접근 계층) 에서 일어나기 때문에 위 코드가 Service 계층이고 주석처리된 "데이터 관련 처리 로직" 부분에서 Repository 계층을 호출한다고 보시면 됩니다.   
Spring Transaction 은 JDBC API 로 Transaction 을 사용하는 방식의 문제점을 해결하기 위해 나온 기능입니다. 그렇다면 JDBC API 를 사용하는게 어떤 문제점이 있는지, 특히 서비스 추상화 관점에서 살펴보겠습니다.   
#### 1. 특정 기술 의존   
사실 JDBC API 는 Java 환경에서 Transaction 을 포함한 Database 기술을 다루는 기반이 되는 기술이긴 하지만, Application 개발에서 JDBC API 를 직접 사용하는 경우는 많지 않습니다. JDBC API 를 감싸서 추가적인 편의를 제공하는 많은 기술 들이 존재하기 때문입니다. 그리고 이런 기술들 중 일부는 Transaction 을 제어하는 방법까지 별도로 제공하기도 합니다. JPA 에서는 EntityManager 기반의 Transaction 제어 기능을 제공하고 Hibernate 에서는 Session 기반의 기능을 제공합니다.   
문제는 위 코드에서 Transaction 제어 기술을 JDBC 에서 JPA 로 바꾸려고 한다면, Transaction 을 처리하는 코드 전체를 변경해야 한다는 겁니다.   
```java
var emf = Persistence.createEntityManagerFactory("test");
var em = emf.createEntityManager();
var tx = em.getTransaction();
tx.begin();
try {
    //데이터 관련 처리 로직
    tx.commit();
} catch (RuntimeException ex) {
    tx.rollback();
} finally {
    em.close();
}
```
Service 계층에서 Transaction 제어 기술로 JPA 를 선택하게 되면 위와 같은 형태로 코드를 작성하게 됩니다. 기존의 JDBC API 를 사용하는 코드와는 완전히 다른 코드가 된 것을 볼 수 있습니다.   
이렇게 현재의 Service 계층은 JDBC 나 JPA 같은 구체적인 기술과 강하게 결합되어 있어, 변경이 쉽지 않습니다.   
#### 2. 특정 맥락 의존   
위의 코드는 하나의 Transaction 단위를 다루는 가장 흔한 코드이지만, 사실 Application 에서는 Transaction 을 좀 더 복잡하게 사용하는 맥락이 존재하는데 간단히만 짚어 보겠습니다.(Transaction 에 대한 자세한 내용은 기회가 되면 다른 글을 통해 다루어 보겠습니다.)   
예를 들어 팀과 멤버라는 데이터를 다루는 비즈니스를 생각해 보겠습니다. 멤버는 단독으로 삭제될 수 있지만, 팀이 삭제될 때도 같이 삭제되어야 합니다. Transaction 관점에서 보면 멤버의 삭제는 독립적인 Transaction 으로 수행될 수도 있지만, 팀을 삭제하는 Transaction 에 묶여서 수행될 수도 있어야 합니다. 이런 모든 시나리오에 대응가능한 멤버 삭제 서비스 계층 코드는 아래와 같은 모습일 것입니다.   
```
IF transaction exists in context
    get existing transaction
ELSE 
    create new transaction

    //Do Some Logic

IF transaction exists in context
    return transaction to context
ELSE
    close transaction
```
실제 코드는 Transaction 을 직접 만들었느냐, 기존 Transaction 에 합류했느냐에 따라 commit/rollback 에 대한 처리도 다르게 가져가야 하기 때문에 훨씬 더 복잡해집니다. 여기서는 사용되는 맥락에 따라 Transaction 을 다루는 코드 흐름이 달라진다는 것에 focus 를 맞추겠습니다.   
그런데 위의 JDBC 기반 예제 코드를 보면, 항상 Connection 을 새로 만들어서 사용하고 있기 때문에 당연히 Transaction 도 새로 만들게 되고 결과적으로 기존 Transaction 에 합류하는 시나리오를 처리할 수 없습니다.(JPA 는 좀 더 복잡한 메커니즘으로 동작하긴 하지만 기본적으로는 새로운 Transaction 을 만든다고 보시면 됩니다.) 결국 기존 Transaction 에 합류하는 시나리오에 대응하기 위해 기존 코드를 바꾸거나, 전용 서비스 메서드를 별도로 만들거나 하는 등의 추가 작업이 필요합니다.   

사실 Service 계층은 Application 의 주요 비즈니스 로직을 다루는 계층입니다. 그리고 Transaction 관련 코드는 비즈니스 로직과는 상관없는 기술 관련 코드입니다. Transaction 을 다루기 위해 의존하는 기술이 바뀐다거나, Transaction 을 다루는 맥락이 바뀐다는 이유로 Service 계층에 변경이 가해진다는 것은 Application 을 유지보수 하는 관점에서 굉장히 부담스러운 일입니다. Spring Transaction 은 이런 부담을 덜어주기 위해 '서비스 추상화' 를 도입합니다.   
이번에는 위의 서로 다른 코드 들에서 공통점을 생각해보겠습니다. Transaction 을 다루는 기술이 JDBC 이든, JPA 이든 간에 어쨋든 Transaction 은 commit 되거나 rollback 되어야 합니다. Transaction 이 새로 만들어 지든, 기존 것이 얻어 지든 간에 어쨋든 Transaction 은 현재 코드에 얻어져야 합니다. 이런 추상화 과정을 통해 Spring 은 아래의 인터페이스를 제공합니다.   
```java
public interface PlatformTransactionManager extends TransactionManager {
    TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;
    void commit(TransactionStatus status) throws TransactionException;
    void rollback(TransactionStatus status) throws TransactionException;
}
```
Transaction 을 다루는 기술의 차이, Transaction 의 맥락의 차이를 배제하고 극단적으로 추상화하면, Transaction 은 '얻어져서 commit 되거나 rollback 된다' 라고 정의할 수 있습니다. Spring 의 PlatformTransactionManager 는 이런 관점에서 세가지 메서드(getTransaction, commit, rollback) 을 제공합니다. 그리고 Transaction 을 다루는 기술에 따라 서로 다른 구현체까지 제공합니다.   
```java
DataSource dataSource = new DriverManagerDataSource("jdbc:h2:tcp://localhost/~/test", "SA", "");
PlatformTransactionManager transactionManager = new DataSourceTransactionManager(dataSource);
TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionAttribute());
try {
    //데이터 관련 처리 로직
    transactionManager.commit(status);
} catch (Exception ex) {
    transactionManager.rollback(status);
}
```
Spring 이 제공하는 PlatformTransactionManager 의 구현체 중 DataSourceTransactionManager 는 JDBC API 기반으로 Transaction 을 처리하는 구현체 입니다. 구현체의 내부 동작이 JDBC API 기반의 첫번째 예제와 유사합니다. 하지만 이를 사용하는 서비스 계층에서는 구현 상세에 대해 알 필요없이, PlatformTransactionManager 의 인터페이스 스펙에 맞게 사용하면 됩니다.   
만약 기술 요구사항이 바뀌어 JDBC API 기반이 아니라 JPA API 기반으로 Transaction 을 지원해야 한다고 가정해보겠습니다.   
```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("test");
PlatformTransactionManager transactionManager = new JpaTransactionManager(emf);
TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionAttribute());
try {
    //데이터 관련 처리 로직
    transactionManager.commit(status);
} catch (Exception ex) {
    transactionManager.rollback(status);
}
```
PlatformTransactionManager 의 구현체로 JpaTransactionManager 를 선택했습니다. JpaTransactionManager 는 내부적으로 JPA Spec 으로 Transaction 을 처리합니다.   
JDBC, JPA API 를 직접 의존하는 예제와, PlatformTransactionManager 기반의 예제 사이에는 큰 차이가 존재합니다. 바로 사용 기술의 변경이 (JDBC -> JPA) 영향을 끼치는 범위입니다.   
```java
//JDBC
Connection conn = DriverManager.getConnection("jdbc:h2:tcp://localhost/~/test", "SA", "");

//JPA
var emf = Persistence.createEntityManagerFactory("test");
var em = emf.createEntityManager();
```
특정 기술에 직접 의존하는 예제는 Transaction 을 다루기 위한 기반을 만드는 코드(Connection 생성, EntityManager 생성) 도 다르고   
```java
//JDBC
conn.setAutoCommit(false);
try {
    //데이터 관련 처리 로직
    conn.commit();
} catch (SQLException ex) {
    conn.rollback();
} finally {
    conn.close();
}

//JPA
var tx = em.getTransaction();
tx.begin();
try {
    //데이터 관련 처리 로직
    tx.commit();
} catch (RuntimeException ex) {
    tx.rollback();
} finally {
    em.close();
}
```
Transaction 을 다루는 코드도 다릅니다.   
하지만 PlatformTransactionManager 기반의 예제 코드를 보면   
```java
//JDBC
DataSource dataSource = new DriverManagerDataSource("jdbc:h2:tcp://localhost/~/test", "SA", "");
PlatformTransactionManager transactionManager = new DataSourceTransactionManager(dataSource);

//JPA
EntityManagerFactory emf = Persistence.createEntityManagerFactory("test");
PlatformTransactionManager transactionManager = new JpaTransactionManager(emf);
```
Transaction 을 다루기 위한 기반 코드, 즉 PlatformTransactionManager 의 구현체를 할당하는 코드에는 차이가 있지만   
```java
TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionAttribute());
try {
    //데이터 관련 처리 로직
    transactionManager.commit(status);
} catch (Exception ex) {
    transactionManager.rollback(status);
}
```
Transaction 을 다루는 코드에는 차이가 없습니다. 단지 공통의 인터페이스인 PlatformTransactionManager 을 같은 방식으로 사용하는 것이죠. 추가로 PlatformTransactionManager 의 구현체 들은 위에서 언급했던 Transaction 의 다른 맥락들(새로운 Transaction, 기존 Transaction 에 합류...) 도 해결을 해줍니다.(내부적으로 Transaction 동기화라는 개념을 사용합니다.)     
Spring Application 에서 PlatformTransactionManager 의 구현체를 선택하고, 초기화하는 과정은 DI Container 가 담당합니다. 결국 Spring Bean 으로서 동작하는 서비스 컴포넌트는 PlatformTransactionManager 를 주입받기 때문에 구현체가 변경되더라도 아무런 영향을 받지 않습니다. 이게 서비스 추상화의 강력함입니다.      

## 휴대의 용이성
사실 PSA 에서 Portable 에 대해서는 여러가지 의견이 분분합니다. Portable 의 사전적 의미에 따라 PSA 를 '휴대가 용이한 서비스 추상화' 라고 해석하는 경우도 있지만, '서비스 추상화' 혹은 '일관성있는 서비스 추상화' 로 해석하여 서비스 추상화 자체에 집중하는 경우도 있습니다. 여기서는 '휴대가 용이한 서비스 추상화' 로 해석하는 관점에서 '휴대의 용이성' 을 살펴보겠습니다.   
'휴대의 용이성' 이란 '사용하는 기술이나 환경이 변하더라도 코드 조각을 쉽게 재사용할 수 있는 상태' 를 의미합니다. 달리 말하면 '새로운 기술이나, 새로운 환경에 이식하기 쉬운 상태' 라고도 할 수 있습니다.   
### POJO
새로운 기술이나 환경에 쉽게 이식할 수 있다는 것은 결국 코드 조각이 특정 기술, 환경에 의존하지 않는다는 것을 의미합니다. Java 에서 특정 기술, 환경, 규약 등에 의존하지 않고 JDK API 에만 의존하여 만든 객체를 POJO(Plain Old Java Object) 라고 합니다. Application 을 구성하는 코드 조각을 POJO 로 작성하게 되면, Application 코드의 이식성, 휴대성이 높아집니다.   
Spring 은 여러가지 기술 요소들을 통해 Application 코드를 POJO 방식으로 만들 수 있도록 도와줍니다.   
### 서비스 추상화와 DI
먼저 서비스 추상화 자체가 DI 와 결합하는 것 만으로도 '코드의 휴대성' 이 올라갑니다.   
```java
@RequiredArgsConstructor
static class Service {

    private final PlatformTransactionManager transactionManager;

    public void processData() {
        TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionAttribute());
        try {
            //데이터 관련 처리
            transactionManager.commit(status);
        } catch (Exception ex) {
            transactionManager.rollback(status);
        }
    }
}
```
PlatformTransactionManager 로 Transaction 을 처리하는 Service 계층의 코드는 위와 같은 포맷으로 작성할 수 있습니다. PlatformTransactionManager 구현체는 DI Container 에 의해 주입받기 때문에 Service 클래스에서는 신경 쓸 필요가 없습니다.   
POJO 관점에서 보면, 이 코드는 Spring Framework 에는 의존하고 있지만(PlatformTransactionManager, TransactionStatus...) 구체적인 기술(JDBC, JPA...) 에는 의존하지 않습니다. 기술에 직접 의존하는 클래스 보다는 POJO 에 가깝다고 할 수 있습니다.   
Service 클래스가 현재 Spring + JPA 기반의 Application 에서 동작하고 있다고 가정해보겠습니다. 이 클래스를 Spring + JDBC 기반의 다른 환경에서 사용해야 한다면, 특별한 변경없이 그대로 코드를 가져와서 이식할 수 있습니다. 특정 기술에 직접 의존하는 클래스에 비해 휴대성, 이식성이 훨씬 뛰어나다고 할 수 있습니다.   
### AOP 
AOP 역시 코드의 휴대성을 높여주는 도구로써 기능합니다.(AOP 가 이루고자 하는 주된 목표가 코드의 휴대성을 높이는 것은 아니지만, 직간접적으로 코드의 휴대성에 기여하는 것은 분명하며 여기서는 그 지점에 집중하겠습니다.)   
사실 Spring 기반의 Application 에서 Service 계층에 Transaction 을 적용할 때, 주로 사용하는 코드 포맷은 아래와 같습니다.   
```java
static class Service {

    @Transactional
    public void process() {
        //데이터 관련 처리
    }
}
```
Spring 은 Transaction 을 AOP 로 다룰 수 있도록 여러가지 기반을 제공하며, DI Container 에 이러한 기반이 되는 Bean 들이 등록되어 있으면 @Transactional 어노테이션을 보고 Service 객체를 Transaction Proxy 로 바꿔주며 이 Transaction Proxy 가 내부적으로 PlatformTransactionManager 를 사용합니다. (AOP, Proxy 등에 대한 자세한 내용은 별도의 글을 통해 다루도록 하겠습니다.)    
POJO 관점에서 보면 이번 Service 클래스는 코드 라인상에는 Spring 관련 의존성이 전혀 존재하지 않습니다. 단지 메서드의 메타데이터로서 @Transactional 어노테이션만 존재합니다. 한층 더 POJO 에 가까워졌다고 할 수 있습니다.   
PlatformTransactionManager 를 직접 사용하는 기존 Service 클래스는 쉽게 이식할 수 있는 환경이 Spring 기반의 환경으로 제한되는 반면, 이번 Service 클래스는 그런 제약에서 조차 비교적 자유롭습니다. 가령, 어노테이션 기반으로 Transaction 기능을 지원하는 다른 프레임워크가 있다고 할 때, 이번 Service 클래스는 @Transactional 을 해당 프레임워크의 어노테이션으로 바꾸는 것 만으로도 이식이 가능합니다.   
결과적으로 서비스 추상화에 DI, 그리고 AOP 가 서로 협력하며 코드를 POJO 에 가깝게 만들어 주고 코드의 휴대성을 높여준다고 할 수 있습니다.   

## 정리
정리하겠습니다. PSA 는 '서비스 추상화' 와 '휴대의 용이성' 으로 나누어서 살펴볼 수 있습니다. 서비스 추상화는 Spring 이 주요 기능 들을 제공할 때 지키는 개발 철학으로서 각종 서비스를 추상화된 인터페이스와 그 구현체 모음으로 제공하는 것을 의미하며, 휴대의 용이성은 Spring 이 서비스 추상화와 DI, AOP 등의 개발 방법론을 차용함으로써 Spring 기반의 Application 코드가 POJO 에 가까워지고 휴대가 용이해지는 것을 의미합니다.   
이 내용을 정리하면서 개인적으로 든 생각은 PSA 중 SA 는 스프링의 개발 철학, 개발 방법론에 가까운 개념이고, P(Portability) 는 그러한 개발 방법론을 통해 이루고자 하는 목표에 가깝다는 것입니다. 굳이 따지자면 목표와 수단의 관계라고 할까요. 그렇기 때문에 Portability 는 서비스 추상화 뿐만 아니라 IOC, AOP 와도 관련되는 것 같습니다. IOC 와 AOP 도 Portability, POJO 를 달성하기 위한 수단으로 기능하는 것이죠. 그럼에도 불구하고 PSA 라는 이름이 지어진 것은 Portability 라는 목표를 달성하는데 중심축이 되는 개념이 서비스 추상화이기 때문이 아닐까 싶습니다.      
긴 글 읽어주셔서 감사합니다. 다음에 더 좋은 글로 찾아뵙겠습니다.   







