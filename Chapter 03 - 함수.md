## Introduction
- 프로그램의 가장 기본적인 단위는 함수다.
- 함수가 읽기 쉽고 이해하기 쉬운 이유는 무엇일까? 
- 의도를 분명히 표현하는 함수를 어떻게 구현할 수 있을까? 
- 함수에 어떤 속성을 부여해야 처음 읽는 사람이 프로그램 내부를 직관적으로 파악할 수 있을까?

## 작게 만들어라!
함수를 만들 때 최대한 ‘작게!’ 만들어라.

```java
public static String renderPageWithSetupsAndTeardowns( PageData pageData, boolean isSuite) throws Exception {
	boolean isTestPage = pageData.hasAttribute("Test"); 
	if (isTestPage) {
		WikiPage testPage = pageData.getWikiPage(); 
		StringBuffer newPageContent = new StringBuffer(); 
		includeSetupPages(testPage, newPageContent, isSuite); 
		newPageContent.append(pageData.getContent()); 
		includeTeardownPages(testPage, newPageContent, isSuite); 
		pageData.setContent(newPageContent.toString());
	}
	return pageData.getHtml(); 
}
```
위 코드도 길다. 되도록 한 함수당 3~5줄 이내로 줄이는 것을 권장한다.
 ```java
public static String renderPageWithSetupsAndTeardowns( PageData pageData, boolean isSuite) throws Exception { 
	if (isTestPage(pageData)) 
		includeSetupAndTeardownPages(pageData, isSuite); 
	return pageData.getHtml();
}
```

#### 블록과 들여쓰기
- if/else, while문 등에 들어가는 블록은 한 줄이어야 한다. 거기서 보통 함수를 호출하기 때문이다. 
- 각 함수 별 들여쓰기 수준이 2단을 넘어서지 않고, 각 함수가 명백하다면 함수는 더욱 읽고 이해하기 쉬워진다.

## 한가지만 해라.
- 함수는 한가지만 해야 한다. 그 한가지를 잘해야 한다.
- 지정된 함수 이름 아래에서 추상화 수준이 하나인 단계만 수행한다면 그 함수는 한 가지 작업만 하는 것이다.
- 한가지 작업만 하는 함수는 자연스럽게 섹션으로 나누기 어렵다.

## 함수 당 추상화 수준은 하나로!
- 함수가 ‘한가지’ 작업만 하려면 함수 내 모든 문장의 추상화 수준이 동일해야 된다.  
- 한 함수 내에 추상화 수준이 섞이게 된다면 읽는 사람이 헷갈린다.

#### 위에서 아래로 코드 읽기:내려가기 규칙
- 코드는 위에서 아래로 이야기처럼 읽혀야 좋다.  
- 함수 추상화 부분이 한번에 한단계씩 낮아지는 것이 가장 이상적이다. (내려가기 규칙)
- 클린 코드 예시 : 각 함수는 다음 함수를 소개하고, 각 함수는 일정한 추상화 수준을 유지한다.

## Switch 문
- switch문은 작게 만들기 어렵다. 본질적으로 N가지를 처리한다.
```java
public Money calculatePay(Employee e) throws InvalidEmployeeType {
	switch (e.type) { 
		case COMMISSIONED:
			return calculateCommissionedPay(e); 
		case HOURLY:
			return calculateHourlyPay(e); 
		case SALARIED:
			return calculateSalariedPay(e); 
		default:
			throw new InvalidEmployeeType(e.type); 
	}
}
```
- 각 switch문을 저차원 클래스에 숨기고 절대로 반복하지 않는 방법 : 
  - 다형성을 이용하여 switch문을 abstract factory에 숨겨 다형적 객체를 생성하는 코드 안에서만 switch를 사용하도록 한다.
```java
public abstract class Employee {
	public abstract boolean isPayday();
	public abstract Money calculatePay();
	public abstract void deliverPay(Money pay);
}
-----------------
public interface EmployeeFactory {
	public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType; 
}
-----------------
public class EmployeeFactoryImpl implements EmployeeFactory {
	public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
		switch (r.type) {
			case COMMISSIONED:
				return new CommissionedEmployee(r) ;
			case HOURLY:
				return new HourlyEmployee(r);
			case SALARIED:
				return new SalariedEmploye(r);
			default:
				throw new InvalidEmployeeType(r.type);
		} 
	}
}
```

## 서술적인 이름을 사용하라!
> “코드를 읽으면서 짐작했던 기능을 각 루틴이 그대로 수행한다면 깨끗한 코드라 불러도 되겠다” - 워드
- 작은 함수는 그 기능이 명확하므로 서술형 이름을 붙이기가 더 쉽다. 
- 일관성 있는 서술형 이름을 사용한다면 코드를 순차적으로 이해하기도 쉬워진다.
  - 일관성 : 모듈 내에서 함수 이름은 같은 문구, 명사, 동사를 사용

## 함수 인수
- 함수에서 이상적인 인수 개수는 0개(무항)다.
- 인수는 코드 이해에 방해가 되는 요소이므로 최선은 0개이고, 차선은 1개뿐인 경우이다.
- 출력 인수(함수의 반환 값이 아닌 입력 인수로 결과를 받는 경우)는 이해하기 어려우므로 왠만하면 쓰지 않는 것이 좋겠다.

#### 많이 쓰는 단항 형식
- 인수에 질문을 던지는 경우  
  `boolean fileExists(“MyFile”);`
- 인수를 뭔가로 변환해 결과를 변환하는 경우  
  `InputStream fileOpen(“MyFile”);`
- 이벤트 함수일 경우 (이 경우에는 이벤트라는 사실이 코드에 명확하게 드러나야 한다.)

위의 3가지가 아니라면 단항 함수는 가급적 피하는 것이 좋다.

#### 플래그 인수
- 플래그 인수는 추하다. 쓰지마라. bool 값을 넘기는 것 자체가 그 함수는 한꺼번에 여러가지 일을 처리한다고 공표하는 것과 마찬가지다.

#### 이항 함수
- 단항 함수보다 이해하기가 어렵다.
- Point 클래스의 경우에는 이항 함수가 적절하다. 2개의 인수간의 자연적인 순서가 있어야 한다.
  - `Point p = new Point(x,y);`
- 무조건 나쁜 것은 아니지만, 인수가 2개이니 만큼 이해가 어렵고 위험이 따르므로 가능하면 단항으로 바꾸도록 한다.

#### 삼항 함수
- 이항 함수보다 이해하기가 훨씬 어려우므로, 위험도 2배 이상 늘어난다.
- 삼항 함수를 만들 때는 신중히 고려하라.

#### 인수 객체
- 인수가 많이 필요할 경우, 일부 인수를 독자적인 클래스 변수로 선언할 가능성을 살펴보자.
- x,y를 인자로 넘기는 것보다 Point를 넘기는 것이 더 낫다.

#### 인수 목록
- 때로는 String.format같은 함수들처럼 인수 개수가 가변적인 함수도 필요하다.
- String.format의 인수는 List형 인수이기 때문에 이항함수라고 할 수 있다.

#### 동사와 키워드
- 단항 함수는 함수와 인수가 동사/명사 쌍을 이뤄야한다.  
  - `writeField(name);`  
- 함수 이름에 키워드(인수 이름)을 추가하면 인수 순서를 기억할 필요가 없어진다.  
  - `assertExpectedEqualsActual(expected, actual);`  

## 부수 효과를 일으키지 마라!
- 부수효과는 거짓말이다. 함수에서 한가지를 하겠다고 약속하고는 남몰래 다른 짓을 하는 것이므로, 한 함수에서는 딱 한가지만 수행할 것!
- 아래 코드에서 `Session.initialize();`는 함수명과는 맞지 않는 부수효과이다.
```java
public class UserValidator {
	private Cryptographer cryptographer;
	public boolean checkPassword(String userName, String password) { 
		User user = UserGateway.findByName(userName);
		if (user != User.NULL) {
			String codedPhrase = user.getPhraseEncodedByPassword(); 
			String phrase = cryptographer.decrypt(codedPhrase, password); 
			if ("Valid Password".equals(phrase)) {
				Session.initialize();
				return true; 
			}
		}
		return false; 
	}
}
```

#### 출력인수
- 일반적으로 출력 인수는 피해야 한다.   
- 함수에서 상태를 변경해야 한다면 함수가 속한 객체 상태를 변경하는 방식을 택하라.
