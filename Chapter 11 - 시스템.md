# 11 시스템
> 관심사 분리 


## 도시를 세운다면?
- 도시가 돌아가는 이유
  - 한사람이 도시를 관리하는 것이 아니라, 각 분야를 관리하는 팀이 있기 때문이다.
  - 적절한 추상화와 모듈화 때문이다.
- 소프트웨어 팀도 도시처럼 구성한다.
  - 깨끗한 코드를 구현하면 낮은 추상화 수준에서 관심사를 분리하기 쉬워진다.
- 이 장에서는 높은 추상화 수준(= `시스템 수준`)에서도 `깨끗함을 유지 하는 방법`을 살펴보자.


## 시스템 제작과 시스템 사용을 분리하라
- `제작` 과 `사용` 은 **서로 다른 관심사이다.**
- 소프트웨어 시스템은 `준비 과정`과 `런타임 로직`을 분리해야 한다.
  - 준비 과정 : 어플리케이션 객체를 제작하고 의존성을 서로’연결’하는 과정
  - 런타임 로직 : 준비 과정 이후에 이어지는 작업
- 애플리케이션은 시작 단계라는 관심사를 분리해야 한다.
- 불행히도 대다수 어프릴케이션은 시작 단계라는 관심사를 분리하지 않는다.


###### 관심사가 분리되지 않은 코드
```java
public class MyServiceImpl extends Service {
    private int a;
    private int b;
    
    public MyServiceImpl(int a, int b) {
        this.a = a;
        this.b = b;
    }
}
```
```java
public class SampleClass {

    private Service service; 
 
    public Service getService() { // 인수가 있을 수도 있고 없을 수도 있다.    
        if (service == null) {
            // MyServiceImpl 객체를 생성하기 전에 인수를 만들 수도 있고 안 만들 수도 있다.     
            service = new MyServiceImpl(...); // 객체를 생성하는 것은 모든 상황에 적합한 기본값일까?       
        } 
        return service;      
    }
}
```    
- 위 코드는 `늦은 초기화 방식`기법을 사용하고 있다.
- 실제로 필요할 때까지 객체를 생성하지 않으므로 불필요한 부하가 걸리지 않는다. → 애플리케이션을 시작하는 시간이 그만큼 빨라진다. 
- `nullPointer`를 반환하지 않는다.


- 하지만, 이 같은 코드로 인해 발생하는 문제가 여러개 있다.
- `getService()` 메스드가 MyServiceImpl와 의존적인 메서드가 되는 문제가 발생한다.
<br>→ `MyServiceImpl` 객체를 직접 생성하므로 해당 클래스에 의존적이다.
- `getService()` 메스드가 객체를 생성함에 있어 인수도 사용하므로 생성자 인수에 명시적으로 의존한다.
- 런타임 로직에서 `MyServiceImpl` 객체를 사용하지 않더라도        
  코드상에서 의존성 문제를 해결하지 않으면 컴파일이 안된다.
- 결국, `getService()` 메서드 외에도 전체 문맥을 알아야 되는 문제가 생긴다.


- 테스트에서 문제
  * `MyServiceImpl`이 무거운 객체이며 단위 테스트를 한다고 가정한다.
  * 메서드를 테스트하기 전에 적절한 `Service` 객체를 `service` 필드에 할당해야 한다.
  * 즉, `MyServiceImpl`에 속한 `serive` 필드의 값을 넣어주는 작업을 매번 해야 한다.
  * 더불어 일반 런타임 로직에다 객체 생성 로직을 섞어놓은 탓에 모든 실행 경로(service가 null인 경로와 null이 아닌 경로 등)도 테스트해야 한다.


- SRP 원칙을 깨는 문제
  * 객체를 의존하면서 관리해야 하는 책임
  * 객체를 직접 생성하는 책임
  * 2가지 책임이 따르기에 작게나마 SRP 단일 책임 원칙을 깨진다.

- 모든 상황에 적합한 객체인지 모른다는 문제
  * 어떤 요구 사항, 기능에서는 `MyServiceImpl` 객체가 아닌 다른 객체가 필요하다.
  * 즉, 다른 Service를 구현하는 객체로 바꿔야 할 경우가 있다.
  * 하지만 직접적으로 `new MyServiceImpl()`를 생성해서 사용한다.
  * 이같은 코드가 100개의 파일에 있을 경우 100개의 파일 전부를 수정해야 하는 문제가 있다.

초기화 기법을 한 번 정도 사용한다면 별로 심각한 문제가 아니다.     
하지만 많은 애플리케이션이 이처럼 좀스러운 설정 기법을 수시로 사용한다.       
그래서 전반적인 설정 방식이 애플리케이션 곳곳에 흩어져 있다.       
모듈성은 저조하며 대개 중복이 심하다.

체계적이고 탄탄한 시스템을 만들고 싶다면          
흔히 쓰는 좀스럽고 손쉬운 기법으로 모듈성을 깨서는 절대 안 된다.          
객체를 생성하거나 의존성을 연결할 때도 마찬가지이다.        
설정 논리는 일반 실행 논리와 분리해야 모듈성이 높아진다.          
또한 주요 의존성을 해소하기 위한 방식, 즉 전반적이며 일관적인 방식도 필요하다.


#### Main 분리
> `main`에서 객체를 생성하고 그 외에서는 객체를 생성하지 않고 사용만 하자

<img width="500" src="https://user-images.githubusercontent.com/50267433/102687887-d2119680-4235-11eb-8c7a-579223885a0b.png">

- main 함수에서 시스템에 필요한 객체를 생성한 후 어플리케이션에 넘긴다.
- 어플리케이션은 그저 만들어진 객체를 사용한다.
- 모든 객체가 잘 생성되었다고 가정하고, 객체를 이용한 개발에 집중할 수 있다.

- 즉, `main`에서 객체를 생성하고 그 외에서는 객체를 생성하지 않고 사용만 하자


#### 팩토리
> 때로는 객체가 생성되는 시점을 애플리케이션이 결정할 필요도 생긴다.

<img width="500" src="https://user-images.githubusercontent.com/50267433/102687881-c7ef9800-4235-11eb-8b24-b05d7dc41eeb.png">

- 객체의 생성 시기를 직접 결정하려면, Abstract Factory 패턴을 사용한다.
- 어플리케이션은 그저 만들어진 객체를 사용한다. 객체가 생성되는 구체적인 방법을 모른지만, 인스턴스가 생성되는 시점을 완벽하게 통제한다.

* 모든 의존성이 `main`에서 `OrderProcessing`으로 향한다.
* 즉, `OrderProcessing`은 `LineItem`이 생성되는 구체적인 방법을 모른다.
* 구체적인 생성 방법은 `main`의 `LineItemFactoryImplementation`가 안다.
* 그럼에도 `OrderProcessing`은 `LineItem` 인스턴스가 생성 시점을 완벽하게 통제한다.
* 필요하다면 `OrderProcessing`에서만 사용하는 생성자 인수도 넘길 수 있다.
* 이유로는 `ABSTRACT FACTORY 패턴`을 사용하기 때문이다.
  * `LineItem`을 생성하는 시점은 `OrderProcessing`이 결정하지만            
    `LineItem`을 생성하는 코드는 `OrderProcessing`이 모른다.


#### 의존성 주입
> `제어의 역전 기법(IoC)`을 `의존성 관리`에 적용한 매커니즘

* 한 객체가 맡은 보조 책임을 새로운 객체에게 전적으로 떠넘긴다.
* 새로운 객체는 넘겨받은 책임만 맡으므로 단일 책임 원칙을 가지게 된다.**(관리 책임)**
* 의존성 관리 맥락에서 객체는 의존성 자체를 **인스턴스로 만드는 책임은 지지 않는다.**
* 대신에 `인스턴스 생성 책임`을 다른 '전담 매커니즘'에 넘겨야 하므로 제어를 역전한다.
* `스프링 프레임워크`에서는 `ApplicationContext`나 `BeanFactory`에 책임을 넘긴다.
* 초기 설정은 시스템 전체에서 필요하므로 대개 `main 루틴`이나 `특수 컨테이너`를 사용한다.

* 예시- JNDI 검색    
  - JNDI 검색은 의존성 주입을 '부분적으로' 구현한 기능이다.        
  - 객체는 디렉터리 서버에 이름을 제공하고 그 이름에 일치하는 서비스를 요청한다.

```java
MyService myService = (MyService)(jndiContext.lookup(“NameOfMyService”));
```

* 호출하는 객체는 실제로 반환되는 객체의 유형을 제어하지 않는다.              
  = 호출하는 객체는 객체를 생성하는 코드에 관여하지 않는다.         
* 호출하는 객체는 의존성을 능동적으로 해결한다.

* 진정한 의존성 주입은 여기서 한 걸음 더 나간다.
  * 클래스는 수동적이기 때문에 클래스가 의존성을 해결하려 시도하지 않게끔 만들어야 한다.
  * 한 걸음 나아간 방법 :
    * **객체 사용 클래스 :** `setter 함수`나 `생성자`에 인수를 제공
    * **DI 컨테이너 :** 객체를 생성하고 `setter 함수`나 `생성자`를 사용해 의존성을 설정
    * 실제로 생성되는 객체 유형은 실제 파일에서 지정하거나 특수 생성 모듈에서 코드로 명시한다.
  * 특히, 스프링 프레임워크는 가장 널리 알려진 자바 `DI 컨테이너`를 제공한다.
    * 객체 사이의 의존성은 `XML 파일`에 정의한다.
    * 자바 코드에서는 이름으로 특정한 객체를 요청한다.

* 그렇다면, 의존성 주입을 사용하면 초기화 지연의 장점은 포기해야 하는 걸까?
  * 초기화 지연 기법은 의존성 주입을 사용하더라도 때론 여전히 유용하다.
  * 대대수 `DI 컨테이너`는 필요할 때 까지 객체를 생성하지 않는다.
  * `계산 지연`이나 `최적화`에 쓸 수 있도록 팩토리를 호출하거나 프록시를 생성하는 방법을 제공한다.
  * 즉, `계산 지연 기법`이나 이와 유사한` 최적화 기법`에서 의존성 주입 메커니즘을 사용할 수 있다.

## 확장
`처음부터 올바르게` 시스템을 만들 수 있다는 믿음은 **미신이다.**

오늘은 주어진 사용자 스토리에 맞춰 시스템을 구현해야 한다.          
내일은 새로운 스토리에 맞춰 시스템을 조정하고 확장하면 된다.            
이것이 반복적이고 점진적인 애자일 방식의 핵심이다.                
TDD, 리팩토링, 깨끗한 코드 수준에서 시스템을 조정하고 확장하기 쉽게 만든다.

하지만, '소프트웨어 시스템 아키텍처'는 **사전 계획이 필요하다.**    
단순한 아키텍처를 복잡한 아키텍처로 조금씩 키울 수 없다는 현실은 정확하다.   
소프트웨어 시스템은 수명이 짧다는 본질로 인해 아키텍처의 점진적인 발전이 가능하다.     
**관심사를 적절히 분리해 관리한다면 소프트웨어 아키텍처는 점진적으로 발전할 수 있다.**


예시 : 관심사를 적절히 분리하지 못하는 아키텍처   
EJB1과 EJB2 아키텍처는 관심사를 적절히 분리하지 못했기에 유기적인 성장이 어려웠다.    
영속적으로 저장될 Bank 클래스에 필요한 엔티티 빈을 살펴보자.    
엔티티 빈은 관계형 자료, 즉 테이블 행을 표현하는 객체로, 메모리에 상주한다.

먼저, 클라이언트가 사용할 지역 인터페이스나 원격 인터페이스를 정의해야 한다.       
아래는 가능한 지역 인터페이스다.

###### Bank EJB용 EJB2 지역 인터페이스
```java
package com.example.banking;
import java.util.Collections;
import javax.ejb.*;

public interface BankLocal extends java.ejb.EJBLocalObject {
    String getStreetAddr1() throws EJBException;
    String getStreetAddr2() throws EJBException;
    String getCity() throws EJBException;
    String getState() throws EJBException;
    String getZipCode() throws EJBException;
    void setStreetAddr1(String street1) throws EJBException;
    void setStreetAddr2(String street2) throws EJBException;
    void setCity(String city) throws EJBException;
    void setState(String state) throws EJBException;
    void setZipCode(String zip) throws EJBException;
    Collection getAccounts() throws EJBException;
    void setAccounts(Collection accounts) throws EJBException;
    void addAccount(AccountDTO accountDTO) throws EJBException;
}
```
* 열거하는 속성은 Bank 주소, 은행이 소유하는 계좌다.
* 각 계좌 정보는 Account EJB로 처리한다.

아래는 위의 인터페이스를 구현한 Bank 빈에 대한 구현 클래스다.
###### 상응하는 EJB2 엔티티 빈 구현
```java
package com.example.banking;
import java.util.Collections;
import javax.ejb.*;

public abstract class Bank implements javax.ejb.EntityBean {
    // Business logic...
    public abstract String getStreetAddr1();
    public abstract String getStreetAddr2();
    public abstract String getCity();
    public abstract String getState();
    public abstract String getZipCode();
    public abstract void setStreetAddr1(String street1);
    public abstract void setStreetAddr2(String street2);
    public abstract void setCity(String city);
    public abstract void setState(String state);
    public abstract void setZipCode(String zip);
    public abstract Collection getAccounts();
    public abstract void setAccounts(Collection accounts);
    
    public void addAccount(AccountDTO accountDTO) {
        InitialContext context = new InitialContext();
        AccountHomeLocal accountHome = context.lookup("AccountHomeLocal");
        AccountLocal account = accountHome.create(accountDTO);
        Collection accounts = getAccounts();
        accounts.add(account);
    }
    
    // EJB container logic
    public abstract void setId(Integer id);
    public abstract Integer getId();
    public Integer ejbCreate(Integer id) { ... }
    public void ejbPostCreate(Integer id) { ... }
    
    // The rest had to be implemented but were usually empty:
    public void setEntityContext(EntityContext ctx) {}
    public void unsetEntityContext() {}
    public void ejbActivate() {}
    public void ejbPassivate() {}
    public void ejbLoad() {}
    public void ejbStore() {}
    public void ejbRemove() {}
}
```
* 객체를 생성하는 팩토리인 LocalHome 인터페이스는 생략했다.
* 기타 Bank 탐색(질의) 메서드도 생략했다.

- 문제점
  - 비즈니스 논리는 EJB2 애플리케이션 '컨테이너'에 강하게 결합된다.
  - 이렇듯 비즈니스 논리가 덩치 큰 컨테이너와 밀접하게 결합된 탓에 독자적인 단위 테스트가 어렵다. 
  - 컨테이너를 흉내 내거나(어렵다!) 아니면 많은 시간을 낭비하며 EJB와 테스트를 실제 서버에 배치해야 한다. 
  - 그래서 EJB2 코드는 프레임워크 밖에서 재사용하기란 사실상 불가능하다. 
  - 결국 객체 지향 프로그래밍이라는 개념조차 뿌리가 흔들린다. 빈은 다른 빈을 상속 받지 못한다.
  
#### 횡단(cross-cutting) 관심사
- EJB2 아키텍처는 일부 영역에서 관심사를 거의 완벽하게 분리한다.
  - 예) 트랜잭션, 보안, 일부 영속적인 동작은 소스 코드가 아니라 배치 기술자에서 정의한다.
- 영속성과 같은 관심사는 모든 객체가 전반적으로 동일한 방식을 이용하게 만들어야 한다.
  - 예) 특정 DBMS나 독자적인 파일을 사용하고, 테이블과 열은 같은 명명 관례를 따르며, 트랜잭션 의미가 일관적이면 더욱 바람직하다.
- 사실 EJB 아키텍처가 영속성, 보안, 트랜잭션을 처리하는 방식은 관점 지향 프로그래민(AOP)을 예견했다고 보인다.
  - AOP에서 관점이라는 모듈 구성 개념은 "특정 관심사를 지원하려면 시스템에서 특정 지점들이 동작하는 방식을 일관성 있게 바꿔야 한다"라고 명시한다.
- 자바에서 사용하는 관점 혹은 관점과 유사한 메커니즘 세 개를 살펴보자.

## 자바 프록시
- 자바 프록시는 단순한 상황에 적합하다.
- 개별 객체나 클래스에서 메서드 호출을 감싸는 경우가 좋은 예다.
```java
// Bank.java (패키지 이름을 감춘다)
import java.utils.*;

// 은행 추상화
public interface Bank {
    Collection<Account> getAccounts();
    void setAccounts(Collection<Account> accounts);
}

// BankImpl.java
import java.utils.*;

// 추상화를 위한 POJO("Plain Old Java Object") 구현
public class BankImpl implements Bank {
    private List<Account> accounts;

    public Collection<Account> getAccounts() {
        return accounts;
    }

    public void setAccounts(Collection<Account> accounts) {
        this.accounts = new ArrayList<Account>();
        for (Account account: accounts) {
            this.accounts.add(account);
        }
    }
}
// BankProxyHandler.java
import java.lang.reflect.*;
import java.util.*;

// 프록시 API가 필요한 "InvocationHandler"
public class BankProxyHandler implements InvocationHandler {
    private Bank bank;

    public BankHandler (Bank bank) {
        this.bank = bank;
    }

    // InvocationHandler에 정의된 메서드
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String methodName = method.getName();
        if (methodName.equals("getAccounts")) {
            bank.setAccounts(getAccountsFromDatabase());
            return bank.getAccounts();
        } else if (methodName.equals("setAccounts")) {
            bank.setAccounts((Collection<Account>) args[0]);
            setAccountsToDatabase(bank.getAccounts());
            return null;
        } else {
            ...
        }
    }

    // 세부사항은 여기에 이어진다.
    protected Collection<Account> getAccountsFromDatabase() { ... }
    protected void setAccountsToDatabase(Collection<Account> accounts) { ... }
}

// 다른 곳에 위치하는 코드
Bank bank = (Bank) Proxy.newProxyInstance(
    Bank.class.getClassLoader(),
    new Class[] { Bank.class },
    new BankProxyHandler(new BankImpl())
);
```
- 단순한 예제지만 코드가 상당히 많으며 제법 복잡하다. 바이트 조작 라이브러리를 사용하더라도 만만찮게 어렵다.
- 코드 '양'과 크기는 프록시의 두 가지 단점이다. 다시 말해서, 프록시를 사용하면 깨끗한 코드를 작성하기 어렵다!
- 또한 프록시는 (진정한 AOP 해법에 필요한) 시스템 단위로 실행 '지점'을 명시하는 메커니즘도 제공하지 않는다.

## 순수 자바 AOP 프레임워크
다행스럽게도 대부분의 프록시 코드는 판박이라 도구로 자동화할 수 있다.   
순수 자바 관점을 구현하는 스프링 AOP, JBoss AOP 등과 같은 여러 자바 프레임워크는 내부적으로 프록시를 사용한다.   

스프링은 비즈니스 논리를 POJO로 구현한다.
- POJO는 순수하게 도메인에 초점을 맞춘다.
- POJO는 엔터프라이즈 프레임워크에 (그리고 다른 도메인에도) 의존하지 않는다. 따라서 테스트가 개념적으로 더 쉽고 간단하다.
- 상대적으로 단순하기 때문에 사용자 스토리를 올바로 구현하기 쉬우며 미래 스토리에 맞춰 코드를 보수하고 개선하기 편하다.

프로그래머는 설정 파일이나 API를 사용해 필수적인 애플리케이션 기반 구조를 구현한다.
- 여기에는 영속성, 트랜잭션, 보안, 캐시, 장애조치 등과 같은 횡당 관심사도 포함된다.
- 많은 경우 실제로는 스프링이나 JBoss 라이브러리의 관점을 명시한다.
- 이때 프레임워크는 사용자가 모르게 프록시나 바이트코드 라이브러리를 사용해 이를 구현한다.
- 이런 선언들이 요청에 따라 주요 객체를 생성하고 서로 연결하는 등 DI 컨테이너의 구체적인 동작을 제어한다.

아래는 스프링 V2.5 설정 파일 app.xml 일부로, 아주 전형적인 모습이다.
```java
<beans>
    ...
    <bean id="appDataSource"
        class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="me"/>

    <bean id="bankDataAccessObject"
        class="com.example.banking.persistence.BankDataAccessObject"
        p:dataSource-ref="appDataSource"/>

    <bean id="bank"
        class="com.example.banking.model.Bank"
        p:dataAccessObject-ref="bankDataAccessObject"/>
    ...
</beans>
```
애플리케이션에서 DI 컨테이너에게 (XML 파일에 명시된) 시스템 내 최상위 객체를 요청하려면 다음 코드가 필요하다.

```java
XmlBeanFactory bf = new XmlBeanFactory(new ClassPathResource("app.xml", getClass()));
Bank bank = (Bank) bf.getBean("bank");
```
XML은 장황하고 읽기 어렵다는 문제가 있음에도 불구하고, 이런 설정 파일에 명시된 '정책'이 겉으로 보이지 않지만 자동으로 생성되는 프록시나 관점 논리보다는 단순하다.    
이런 아키텍처가 너무 매력적이라 스프링 프레임워크는 EJB 버전 3를 완전히 뜯어고치는 계기를 제공했다.    
EJB3는 XML 설정 파일과 자바 5 에너테이션 기능을 사용해 횡단 관심사를 선언적으로 지원하는 스프링 모델을 따른다.     
아래는 EJB3로 Bank 객체를 다시 작성한 코드다.

```java
package com.example.banking.model;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.Collection;

@Entity
@Table(name = "BANKS")
public class Bank implements java.io.Serializable {
    @Id @GeneratedValue(strategy=GenerationType.AUTO)
    private int id;

    @Embeddable // An object “inlined” in Bank’s DB row
    public class Address {
        protected String streetAddr1;
        protected String streetAddr2;
        protected String city;
        protected String state;
        protected String zipCode;
    }

    @Embedded
    private Address address;
    @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER, mappedBy="bank")
    private Collection<Account> accounts = new ArrayList<Account>();
    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public void addAccount(Account account) {
        account.setBank(this);
        accounts.add(account);
    }

    public Collection<Account> getAccounts() {
        return accounts;
    }

    public void setAccounts(Collection<Account> accounts) {
        this.accounts = accounts;
    }
}
```
EJB2 코드보다 위 코드가 훨씬 더 깨끗하다.    
일부 상세한 엔티티 정보는 애너테이션에 포함되어 그대로 남아있지만, 모든 정보가 애너테이션 속에 있으므로 코드 자체는 깔끔하고 깨끗하다.

## AspectJ 관점
- AspectJ는 언어 차원에서 관점을 모듈화 구성으로 지원하는 자바 언어 확장이다.
- 스프링 AOP와 JBoss AOP가 제공하는 순수 자바 방식은 관점이 필요한 상황 중 80~90%에 충분하다.
- AspectJ는 관점을 분리하는 강력하고 풍부한 도구 집합을 제공하긴 하지만, 새 도구를 사용하고 새 언어 문법과 사용법을 익혀야 한다는 단점이 있다.
- 스프링 프레임워크는 AspectJ에 미숙한 팀들이 애너테이션 기반 관점을 쉽게 사용하도록 다양한 기능을 제공한다.

## 테스트 주도 시스템 아키텍처 구축
- 관점으로 관심사를 분리하는 방식은 그 위력이 막강하다.
- 애플리케이션 도메인 논리를 POJO로 작성할 수 있다면, 즉 코드 수준에서 아키텍처 관심사를 분리할 수 있다면, 진정한 테스트 주도 아키텍처 구축이 가능해진다.
- 그때그때 새로운 기술을 채택해 단순한 아키텍처를 복잡한 아키텍처로 키워갈 수도 있다.
- 설계가 아주 멋진 API조차도 정말 필요하지 않으면 과유불급이다.

> 최선의 시스템 구조는 각기 POJO (또는 다른) 객체로 구현되는 모듈화된 관심사 영역(도메인)으로 구성된다. 이렇게 서로 다른 영역은 해당 영역 코드에 최소한의 영향을 미치는 관점이나 유사한 도구를 사용해 통합한다. 이런 구조 역시 코드와 마찬가지로 테스트 주도 기법을 적용할 수 있다.

## 의사 결정을 최적화하라
- 모듈을 나누고 관심사를 분리하면 지엽적인 관리와 결정이 가능해진다.
- 가장 적합한 사람에게 책임을 맡기면 가장 좋다.
- 최대한 정보를 모아 조금 늦더라도 최선의 결정을 내리자.

> 관심사를 모듈로 분리한 POJO 시스템은 기민함을 제공한다. 이런 기민함 덕택에 최신 정보에 기반해 최선의 시점에 최적의 결정을 내리기가 쉬워진다. 또한 결정의 복잡성도 줄어든다.

## 명백한 가치가 있을 때 표준을 현명하게 사용하라
- EJB2는 단지 표준이라는 이유만으로 많은 팀이 사용했다.
- 가볍고 간단한 설계로 충분했을 프로젝트에서도 EJB2를 채택했다.
- 아주 과장되게 포장된 표준에 집착하게 되면 고객 가치가 뒷전으로 밀려날 수 있다.

> 표준을 사용하면 아이디어와 컴포넌트를 재사용하기 쉽고, 적절한 경험을 가진 사람을 구하기 쉬우며, 좋은 아이디어를 갭슐화하기 쉽고, 컴포넌트를 엮기 쉽다. 
  <br>하지만 때로는 표준을 만드는 시간이 너무 오래 걸려 업계가 기다리지 못한다. 어떤 표준은 원래 표준을 제정한 목적을 잊어버리기도 한다.

## 시스템은 도메인 특화 언어가 필요하다
- DSL(Domain-Specific Language, 도메인 특화 언어)은 간단한 스크립트 언어나 표준 언어로 구현한 API를 가리킨다.
- DSL로 짠 코드는 도메인 전문가가 작성한 구조적인 산문처럼 읽힌다.
- 좋은 DSL은 도메인 개념과 그 개념을 구현한 코드 사이에 존재하는 '의사소통 간극'을 줄여준다.
- 도메인 전문가가 사용하는 언어로 도메인 논리를 구현하면 도메인을 잘못 구현할 가능성이 줄어든다.
- 효과적으로 사용한다면 DSL은 추상화 수준을 코드 관용구나 디자인 패턴 이상으로 끌어올린다. 그래서 개발자가 적절한 추상화 수준에서 코드 의도를 표현할 수 있다.

> 도메인 특화 언어(DSL)를 사용하면 고차원 정책에서 저차원 세부사항에 이르기까지 모든 추상화 수준과 모든 도메인을 POJO로 표현할 수 있다.

## 결론
- 시스템 역시 깨끗해야 한다.
- 모든 추상화 단계에서 의도는 명확히 표현해야 한다.
<br>→ 그러려면, POJO를 작성하고 관점 혹은 관점과 유사한 메커니즘을 사용해 각 구현 관심사를 분리해야 한다.
- 시스템을 설계하든 개별 모듈을 설계하든, 실제로 돌아가는 가장 단순한 수단을 사용해야 한다. 

## Findings
- 관심사 분리
  - 시작에 대한 관심사 분리
    - 객체의 생성과 객체를 사용하는 부분을 분리한다.
    - 객체의 생성은 시작 단계에서, 비즈니스 로직은 객체를 사용하는데 집중한다.
  - 요청에 대한 관심사 분리
    - AOP(관점 지향 프로그래밍)
      - 애플리케이션의 부가 기능을 모듈화하고, 핵심기능과 상호작용 할 수 있게 한다.
      - 스프링은 IoC/DI를 이용해 프락시 기반 관점 지향 프로그래밍을 지원한다.
- 테스트 주도 시스템 아키텍처 구축 
  - 최선의 시스템 구조는 각기 POJO (또는 다른) 객체로 구현되는 모듈화된 관심사 영역(도메인)으로 구성된다. 
  - 어플리케이션 도메인 논리를 POJO로 작성할 수 있다면, 즉 코드 수준에서 아키텍처 관심사를 분리할 수 있다면 진정한 테스트 주도 아키텍처 구축이 가능해진다. 
  - 설계가 최대한 분리되어 각 추상화 수준과 범위에서 코드가 적당히 단순해질 수 있다면 소프트웨어 프로젝트를 진행해 결과물을 재빨리 출시한 후, 기반 구조를 추가하며 조금씩 확장해 나가도 괜찮다.

profile

## Question 
- abstract factory란?
- AOP란?
- proxy란?
- POJO란?
- 애플리케이션 도메인 논리를 POJO로 작성할 수 있다면, (= 코드 수준에서 아키텍처 관심사를 분리할 수 있다면) 진정한 테스트 주도 아키텍처 구축이 가능해진다는 말이 무슨 뜻일까?
