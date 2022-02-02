# 10 클래스
> 클래스를 잘 설계하는 원칙

## Introduction
- 코드의 표현력과 코드로 이루어진 함수에 아무리 신경 쓸지라도 좀 더 차원 높은 단계까지 신경 쓰지 않으면 깨끗한 코드를 얻기는 어렵다.
- `깨끗한 클래스`를 알아보자.

## 클래스 체계
- 클래스에서 변수 목록이 나온다.<br>static public 상수, static private 변수, private instance 변수 순으로 나온다. (공개 변수가 필요한 경우는 거의 없다.)
- 변수 목록 다음에는 공개함수가 나온다. 비공개 함수는 자신을 호출하는 공개 함수 직후에 넣는다.
- 즉, 추상화 단계가 순차적으로 내려가고, 프로그램은 신문 기사처럼 읽힌다.

#### 캡슐화
- 변수와 유틸리티 함수는 가능한 공개하지 않는 편이 낫다.
- 하지만, 때로는 변수나 유틸리티 함수를 protected로 선언해 테스트 코드에 접근을 허용하기도 한다. 테스트는 아주 중요하기 때문이다.
- 하지만, 그전에 비공개 상태를 유지할 온갖 방법을 강구한다.
- `캡슐화를 풀어주는 결정은 언제나 최후의 수단`이다.

## 클래스는 작아야 한다!
- 클래스를 만들 때, 가장 중요한 것은 `크기`다. `클래스는 작아야 한다.`
- 클래스를 설계할 때도 함수와 마찬가지로 작게가 기본 규칙이다.
- 얼마나 작아야 하는가? → 기준 : `맡은 책임의 갯수`

#### 단일 책임 원칙(Single Responsibility Principle)
- 클래스나 모듈을 변경할 이유가 하나, 단 하나뿐이어야 하는 원칙이다.
- `SRP`는 '책임'이라는 개념을 정의하며 적절한 크기를 제시한다.
- `클래스는 책임 (=변경할 이유)가 하나여야 한다`는 의미다.

```java
// Bad
public class SuperDashboard extends JFrame implements MetaDataUser {
    public Component getLastFocusedComponent()
    public void setLastFocused(Component lastFocused)
    public int getMajorVersionNumber()
    public int getMinorVersionNumber()
    public int getBuildNumber()    
}
```

위의 클래스는 책임, 즉 변경할 이유가 두가지다.
- 첫째, SuperDashboard는 소프트웨어 버전 정보를 추적한다. 그런데 버전 정보는 소프트웨어를 출시할 때마다 달라진다.
- 둘째, SuperDashboard는 자바 스윙 컴포넌트를 관리한다. (SuperDashboard는 최상위 GUI 윈도우의 스윙 표현인 JFrame에서 파생한 클래스다.)
- 즉, 스윙 코드를 변경할 때마다 버전 번호가 달라진다.

```java
// Good : 단일 책임 클래스
// SuperDashboard에서 버전 정보를 다루는 메서드를 세개를 따로 빼내 Version이라는 독자적인 클래스를 만든다.
public class Version {
    public int getMajorVersionNumber()
    public int getMinorVersionNumber()
    public int getBuildNumber()    
}
```

- Version 클래스는 다른 애플리케이션에서 재사용하기 아주 쉬운 구조다!
<br><br>
- SRP는 객체 지향 설계에서 중요한 개념이다.
  - 개발자가 무엇이 어디에 있는지 쉽게 찾는다.
  - 그래야 변경을 가할 때 직접 영향이 미치는 컴포넌트만 이해해도 충분하다.
  - 큰 클래스 몇개가 아니라 작은 클래스 여럿으로 이뤄진 시스템이 더 바람직하다.
  - 작은 클래스는 각자 맡은 책임이 하나며, 변경할 이유가 하나며, 다른 작은 클래스와 협력해 시스템에 필요한 동작을 수행한다.

#### 응집도(Cohesion)
- 클래스 응집도가 높다는 의미
  - 클래스는 인스턴스 변수 수도 작아야 한다.
  - 그리고 각 클래스 메서드는 클래스 인스턴스 변수를 하나 이상 사용해야 한다.
  - 일반적으로 메서드가 변수를 더 많이 사용할수록 메서드와 클래스는 응집도가 더 높다.
  - 그리고 모든 인스턴스 변수를 메서드마다 사용하는 클래스는 응집도가 가장 높다.
- 일반적으로 응집도는 높아야한다. 하지만 응집도가 가장 높은 클래스는 가능하지도 바람직하지도 않다.
  - 그렇지만 우리는 응집도가 높은 클래스를 선호한다.
  
```java
// Stack을 구현한 코드
// 아래 클래스는 응집도가 아주 높다.
// size()를 제외한 두 메서드는 두 변수를 모두 사용한다.
public class Stack {
    private int topOfStack = 0;
    List<Integer> elements = new LinkedList<Integer>();

    public int size() {
        return topOfStack;
    }

    public void push(int element) {
        topOfStack++;
        elements.add(element);
    }

    public int pop() throws PoppedWhenEmpty {
        if (topOfStack == 0)
            throw new PoppedWhenEmpty();
        int element = elements.get(--topOfStack);
        elements.remove(topOfStack);
        return element;
    } 
}
```

- "함수는 작게, 매개변수 목록을 짧게"
  - 때때로 "함수는 작게, 매개변수 목록을 짧게"라는 전략을 따르다 보면 몇몇 메서드만이 사용하는 인스턴스 변수가 아주 많아진다.
      * **이는 새로운 클래스로 쪼개야 한다는 신호다.**
      * `응집도가 높아지도록 변수와 메서드를 적절히 분리해 새로운 클래스 두세 개로 분리해야한다.`

#### 응집도를 유지하면 작은 클래스 여럿이 나온다
- **큰 함수 일부를 작은 함수 하나로 빼내고 싶은데, 빼내려는 코드가 큰 함수에 정의된 변수 넷을 사용한다. 새 함수의 인수로 넘겨야 할까?**
    * 전혀 아니다!
    * 만약 **네 변수를 클래스 인스턴스 변수로 승격한다면 새 함수는 인수가 필요없다.** 그만큼 함수를 쪼개기 쉬워진다.
    * **불행히도 이렇게 하면 클래스가 응집력을 잃는다.**
        * **몇몇 함수만 사용하는 인스턴스 변수가 점점 더 늘어나기 때문이다.**
- 그런데 잠깐!
  - **몇몇 함수가 몇몇 변수만 사용한다면 독자적인 클래스로 분리해도 되지 않는가!**
  - 맞다! 클래스가 응집력을 잃는다면 쪼개면 된다.

## 변경하기 쉬운 클래스
- 깨끗한 시스템은 클래스를 체계적으로 정리해 변경에 수반하는 위험을 낮춘다.

###### 10-9 변경이 어려운 예시 
```java
public class Sql {
    public Sql(String table, Column[] columns)
    public String create()
    public String insert(Object[] fiedls)
    public String selectAll()
    public String findByKeys(String keyColumn, String keyValue)
    public String select(Column column, String pattern)
    public String select(Criteria criteria)
    public String preparedInsert()
    private String columnList(Column[] columns)
    private String valuesList(Object[] fields, final Column[] columns)
    private String selectWithCriteria(String criteria)
    private String placeholderList(Column[] columns)
}
```

* 위 코드는 변경이 어렵다.
    * 새로운 SQL 문을 지원하려면 반드시 클래스를 손대야 한다.
    * 기존 SQL문 하나를 수정할 때도 반드시 클래스를 손대야 한다.
* **즉, 클래스가 책임이 너무 많다. SRP를 위반한 것.**

###### 10-10 변경이 쉬운 예시
```java
// 닫힌 클래스 집합 (SRP + OCP)
abstract public class Sql {
    public Sql(String table, Column[] columns)
    abstract public String generate();
}

public class CreateSql extends Sql {
    public CreateSql(String table, column[] columns)
    @Override public String generate()
}

public class SelectSql extends Sql {
    public SelectSql(String table, Column[] columns)
    @Override public String generate()
}

public class InsertSql extends Sql {
    ...
}

public class SelectWithCriteriaSql extends Sql {
    ...
}
...
```

* 10-9 변경이 어려운 예시에서 공개 인터페이스를 각각 Sql 클래스에서 파생하는 클래스로 만들었다.
* 위 코드를 쉽게 말하면 `SRP`와 `OCP`를 지원하는 코드다.
  * `SRP` : 각 클래스는 하나의 책임만을 수행한다.
    * 함수 하나를 수정했다고 다른 함수가 망가질 위험도 없다.
    * 테스트 관점에서 모든 논리를 구석구석 증명하기도 쉽다.
    * **추가, 변경에 유연해진 것**이다.
  * `OCP` : 위의 클래스(10-10)는 기존 클래스(10-9)에서 파생클래스를 생성하는 방식으로, 새 기능에 개방적인 동시에 다른 클래스를 닫아놓는 방식이다.
    * 새기능을 추가할 때, 시스템을 확장할 뿐 기존 코드를 변경하지 않는다.

#### 변경으로부터 격리
- `DIP` : 클래스가 상세한 구현이 아니라 추상화에 의존해야 한다는 원칙
<br><br>
- 상세한 구현(concrete class)에 의존하는 클라이언트 클래스는 구현이 바뀌면 위험에 빠진다.
  - 상세한 구현에 의존하는 코드는 테스트가 어렵고 변경에 취약하다.
- 그래서 우리는 `인터페이스`와 `추상 클래스`를 사용해 구현이 미치는 영향을 격리한다.
<br><br>
- `추상화`를 통해 테스트가 가능할 정도로 시스템의 결합도를 낮춤으로써 유연성과 재사용성도 더욱 높아진다.
- 결함도가 낮다는 말은 각 시스템 요소가 다른 요소로부터 그리고 변경으로부터 잘 격리되어있다는 뜻이다.

```java
public interface StockExchange { 
	Money currentPrice(String symbol);
}
```

- Portfolio 클래스를 구현하자. 그런데 이 클래스는 외부 TokyoStockExchange API를 사용해 포트폴리오 값을 계산한다. 
- 따라서 API 특성 상 시세 변화에 영향을 많이 받아 5분마다 값이 달라지는데, 이때문에 테스트 코드를 짜기 쉽지 않다.
- 그러므로 Portfolio에서 외부 API를 직접 호출하는 대신 StockExchange라는 인터페이스를 생성한 후 메서드를 선언하다.

<br>

```java
public Portfolio {
	private StockExchange exchange;
	public Portfolio(StockExchange exchange) {
		this.exchange = exchange; 
	}
	// ... 
}
```
- 이후 StockExchange 인터페이스를 구현하는 TokyoStockExchange 클래스를 구현한다.
- 그리고 Portfolio 생성자를 수정해 StockExchange 참조자를 인수로 받는다.

<br>

```java

public class PortfolioTest {
	private FixedStockExchangeStub exchange;
	private Portfolio portfolio;
	
	@Before
	protected void setUp() throws Exception {
		exchange = new FixedStockExchangeStub(); 
		exchange.fix("MSFT", 100);
		portfolio = new Portfolio(exchange);
	}

	@Test
	public void GivenFiveMSFTTotalShouldBe500() throws Exception {
		portfolio.add(5, "MSFT");
		Assert.assertEquals(500, portfolio.value()); 
	}
}

```

- 이제 TokyoStockExchange 클래스를 흉내내는 테스트용 클래스를 만들 수 있다.(FixedStockExchangeStub)
- 테스트용 클래스는 StockExchange 인터페이스를 구현하며 고정된 주가를 반환한다.
- 그럼으로써 무난히 테스트 코드를 작성 할 수 있다.

<br>

- 개선한 Portfolio 클래스는 상세 구현 클래스가 아닌 StockExchange라는 인터페이스에 의존하므로, 
<br>실제로 주가를 얻어오는 출처나 얻어오는 방식 등과 같은 구체적인 사실을 모두 숨길 수 있다.

## Findings
- 클래스를 잘 설계하는 원칙
  - `캡슐화` (객체의 실제 구현을 외부로부터 감추는 방식)
    - 클래스를 개발할 때 기본적으로 구현을 감추고, 외부 객체와 상호작용하는 부분만 노출한다.
  - `클래스는 작아야 한다.` → SRP
    - 클래스의 크기는 책임을 기준으로 한다.
    - 클래스 설명은 만일(if), 그리고(and), 하며(or), 하지만(but)을 사용하지 않고 25단어 내외로 가능해야 한다. <br>→ 책임이 한 가지 여야 한다.
  - `응집도가 높아지도록 클래스를 분리해야한다.` → SRP
    - "함수는 작게, 매개변수 목록을 짧게"라는 전략을 따르다 보면 몇몇 메서드만이 사용하는 인스턴스 변수가 아주 많아진다.
      <br>이는 새로운 클래스로 쪼개야 한다는 신호다.
      <br>응집도가 높아지도록 변수와 메서드를 적절히 분리해 새로운 클래스 두세 개로 분리해야한다.
  - `변경하기 쉬워야 한다.` → SRP, OCP
    - 각 클래스가 하나의 책임만을 수행하도록 하고, 새 기능을 추가할 때 시스템을 확장할 뿐 기존 코드를 변경하지 않도록 설계하면, 추가와 변경에 유연해진다. 
  - `변경으로부터 격리해야 한다.` → DIP
    - 클래스가 상세한 구현이 아니라 추상화에 의존하게 하여(=DIP), 변경이 미치는 영향을 격리한다.
    
## Feelings
- 이번 장은 객체지향의 유용함을 많이 느꼈다.
  - 나는 객체지향을 단순히 "<메소드와 변수>라는 기본 단위로 나누어 구조화 함으로, 이해하기 쉽고 수정할 부분을 빠르게 찾을 수 있으니 변경이 쉬워지겠구나." 생각을 했었다.
  - 클래스를 잘 설계하는 원칙 중 DIP를 보면서, <br>객체지향은 의존성을 관리함으로 "독립적으로 수정하고 배포할 수 있게 만들어주는" 코드 구조화 방법이라고 생각이 들었다.
