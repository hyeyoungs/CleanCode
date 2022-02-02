# 09 단위 테스트
> 테스크 코드를 가독성 있게 작성하는 방법과 원칙

## Introduction
지금은 애자일과 TDD 덕택에 단위 테스트를 자동화하는 프로그래머들이 이미 많아졌으며 점점 더 늘어나는 추세다.          
하지만 우리 분야에 테스트를 추가하려고 급하게 서두르는 와중에 많은 프로그래머들이 제대로 된 테스트 케이스를 작성해야 한다는 좀 더 미묘한(그리고 더욱 중요한) 사실을 놓쳐버렸다.

## TDD 법칙 세가지
**TDD의 기본원칙은 실제 코드를 짜기 전에 단위 테스트부터 짜는 것이다.**

이외에도 세 가지의 법칙이 있다.

1. 실패하는 단위 테스트를 작성할 때까지 실제 코드를 작성하지 않는다.
2. 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 작성한다.
3. 현재 실패하는 테스트를 통과할 정도로만 실제 코드를 작성한다.

이렇게 실제 코드와 테스트 코드를 묶어서 개발해야 한다.

## 깨끗한 테스트 코드 유지하기
- 안 좋은 예시 : '지저분해도 빨리' 
  - 그저 돌아만 가고, 일회성으로 실제 코드를 테스트하는 코드는 굉장히 나쁘다.
- 테스트 코드를 막 짜면 안되는 이유
    * **실제 코드가 진화할수록 테스트 코드도 변화해야한다.**
        * 테스트 코드가 지저분할수록 변경하기 어려워지며, 실제 코드보다 테스트 코드를 작성하는데 많은 시간을 허비하게 된다.
    * 새 버전을 출시할 때마다 테스트 케이스를 유지보수하는 비용이 늘어난다.
    * 결국, 테스트 작성을 비난하게 되고 악순환이 된다.
- **테스트 코드는 실제 코드 못지 않게 중요하다.**
    * 테스트 코드는 이류 시민이 아니다.
    * 실제 코드 못지 않게 깨끗하게 짜야 한다.
    * **테스트 코드를 깨끗하게 유지하자.**           

#### 테스트는 유연성, 유지보수성, 재사용성을 제공한다
테스트 코드를 깨끗하게 유지하지 않으면 결국은 잃어버린다.            
그리고 테스트 케이스가 없으면 실제 코드를 유연하게 만드는 버팀목도 사라진다.   
<br>
맞다, 제대로 읽었다. **코드에 유연성, 유지보수성, 재사용성을 제공하는 버팀목이 바로 단위 테스트**다.            
이유는 단순하다. 테스트 케이스가 있으면 변경이 두렵지 않으니까!         
테스트 케이스가 없다면 모든 변경이 잠정적인 버그다.                   
아키텍쳐가 아무리 유연하더라도, 설계를 아무리 잘 나눴더라도, 테스트 케이스가 없으면 개발자는 변경을 주저한다. 버그가 숨어들까 두렵기 때문이다.               

**테스트 케이스가 있으면 변경이 두렵지 않다.**        
테스트 케이스가 없다면 모든 변경이 잠정적인 버그다. 실제로 테스트 커버리지가 높을수록 공포는 줄어든다.
아키텍처가 부실한 코드나 설계가 모호하고 엉망인 코드라도 별다른 우려 없이 변경할 수 있다.          
아니, 오히려 안심하고 아키텍처와 설계를 개선할 수 있다.            

그러므로 **실제 코드를 점검하는 자동화된 단위 테스트 슈트는 설계와 아키텍처를 깨끗하게 보존하는 열쇠다.**       
테스트는 유연성, 유지보수성, 재사용성을 제공한다. **테스트 케이스가 있으면 변경이 쉬워지기 때문이다.**                

따라서 테스트 코드가 지저분하면 코드를 변경하는 능력이 떨어지며 코드 구조를 개선하는 능력도 떨어진다.           
**테스트 코드가 지저분할수록 실제 코드도 지저분해진다. 결국 테스트 코드를 잃어버리고 실제 코드도 망가진다.**             

## 깨끗한 테스트 코드
깨끗한 테스트 코드를 만들려면? 세 가지가 필요하다. **가독성, 가독성, 가독성.**            
테스트 코드도 실제 코드와 똑같이 최소의 표현으로 많은 것을 나타내야한다. (명료성, 단순성, 풍부한 표현력)

###### 목록 9-1 SerializedPageResponderTest.java
```java
public void testGetPageHieratchyAsXml() throws Exception {
  crawler.addPage(root, PathParser.parse("PageOne"));
  crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
  crawler.addPage(root, PathParser.parse("PageTwo"));

  request.setResource("root");
  request.addInput("type", "pages");
  Responder responder = new SerializedPageResponder();
  SimpleResponse response =
    (SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
  String xml = response.getContent();

  assertEquals("text/xml", response.getContentType());
  assertSubString("<name>PageOne</name>", xml);
  assertSubString("<name>PageTwo</name>", xml);
  assertSubString("<name>ChildOne</name>", xml);
}

public void testGetPageHieratchyAsXmlDoesntContainSymbolicLinks() throws Exception {
  WikiPage pageOne = crawler.addPage(root, PathParser.parse("PageOne"));
  crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
  crawler.addPage(root, PathParser.parse("PageTwo"));

  PageData data = pageOne.getData();
  WikiPageProperties properties = data.getProperties();
  WikiPageProperty symLinks = properties.set(SymbolicPage.PROPERTY_NAME);
  symLinks.set("SymPage", "PageTwo");
  pageOne.commit(data);

  request.setResource("root");
  request.addInput("type", "pages");
  Responder responder = new SerializedPageResponder();
  SimpleResponse response =
    (SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
  String xml = response.getContent();

  assertEquals("text/xml", response.getContentType());
  assertSubString("<name>PageOne</name>", xml);
  assertSubString("<name>PageTwo</name>", xml);
  assertSubString("<name>ChildOne</name>", xml);
  assertNotSubString("SymPage", xml);
}

public void testGetDataAsHtml() throws Exception {
  crawler.addPage(root, PathParser.parse("TestPageOne"), "test page");

  request.setResource("TestPageOne"); request.addInput("type", "data");
  Responder responder = new SerializedPageResponder();
  SimpleResponse response =
    (SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
  String xml = response.getContent();

  assertEquals("text/xml", response.getContentType());
  assertSubString("test page", xml);
  assertSubString("<Test", xml);
}
```
- 목록 9-1은 FitNess에서 가져온 코드다. 아래 테스트 케이스 세 개는 이해하기 어렵기에 개선할 여지가 충분하다.                
- 첫째, addPage와 assertSubString을 부르느라 중복되는 코드가 매우 많다.                  
- 좀 더 중요하게는 자질구레한 사항이 너무 많아 테스트 코드의 표현력이 떨어진다.
- 예를 들어, PathParser 호출을 살펴보자. PathParser는 문자열을 pagePath 인스턴스로 변환한다.           
- 이 코드는 테스트와 무관하며 테스트 코드의 의도만 흐린다.            
- responder 객체를 생성하는 코드와 response를 수집해 변환하는 코드 역시 잡음에 불과하다.            
- 게다가 resource와 인수에서 요청 URL을 만드는 어설픈 코드도 보인다.
- **마지막으로 위 코드는 읽는 사람을 고려하지 않는다.**            
불쌍한 독자들은 온갖 잡다하고 무관한 코드를 이해한 후라야 간신히 테스트 케이스를 이해한다.         

###### 목록 9-2 SerializedPageResponderTest.java (refactored)
```java
public void testGetPageHierarchyAsXml() throws Exception {
  makePages("PageOne", "PageOne.ChildOne", "PageTwo");

  submitRequest("root", "type:pages");

  assertResponseIsXML();
  assertResponseContains(
    "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>");
}

public void testSymbolicLinksAreNotInXmlPageHierarchy() throws Exception {
  WikiPage page = makePage("PageOne");
  makePages("PageOne.ChildOne", "PageTwo");

  addLinkTo(page, "PageTwo", "SymPage");

  submitRequest("root", "type:pages");

  assertResponseIsXML();
  assertResponseContains(
    "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>");
  assertResponseDoesNotContain("SymPage");
}

public void testGetDataAsXml() throws Exception {
  makePageWithContent("TestPageOne", "test page");

  submitRequest("TestPageOne", "type:data");

  assertResponseIsXML();
  assertResponseContains("test page", "<Test");
}
```
- 이제 목록 9-2를 살펴보자. 목록 9-1을 개선한 코드로, 목록 9-1과 정확히 동일한 테스트를 수행한다.        
하지만 목록 9-2는 좀 더 깨끗하고 좀 더 이해하기 쉽다.
- **BUILD-OPERATE-CHECK 패턴이 위와 같은 테스트 구조에 적합하다.**             
  - 각 테스트는 명확히 세 부분으로 나눠진다.             
    - 첫 부분은 테스트 자료를 만든다. 
    - 두 번째 부분은 테스트 자료를 조작한다. 
    - 세 번째 부분은 조작한 결과가 올바른지 확인한다.          

- 잡다하고 세세한 코드를 거의 다 없앴다는 사실에 주목한다.
- 테스트 코드는 본론에 돌입해 진짜 필요한 자료 유형과 함수만 사용한다.             
- 그러므로 코드를 읽는 사람은 온갖 잡다하고 세세한 코드에 주눅들고 헷갈릴 필요 없이 코드가 수행하는 기능을 재빨리 이해한다.           

#### 도메인에 특화된 테스트 언어
* DSL은 특정한 도메인(산업, 분야 등 특정 영역)에서 발생하는 문제점을 해결하는 것에 중점을 두고 도메인을 기준으로 모든 것을 풀어나가기 위해서 제공되는 언어다.
* 특정 영역의 해결에는 그 영역에 맞는 특화된 도구를 사용하자는 의미이다.
* 테스트 API를 구현해 도메인 특화 언어를 만들면 테스트 코드를 짜기가 쉬워진다.
    * **흔히 쓰는 시스템 조작 API를 사용하는 대신 API 위에다 함수와 유틸리티를 구현하여 테스트하라는 의미이다.**
    * **이렇게 구현한 함수와 유틸리티는 테스트 코드에서 사용하는 특수 API가 된다. (테스트 언어)**


#### 이중 표준
- 테스트 코드는 표현력이 풍부해야 하지만, 실제 코드 만큼 효율적일 필요는 없다
- 예를 들어, `StringBuffer`를 사용하지 않고, String 연산은 해도 된다.
- 테스트 환경에선 리소스 문제가 발생할 확률이 낮기 때문이다.
        

## 테스트 당 assert 하나
JUnit으로 테스트 코드를 짤 때 함수마다 assert를 단 하나만 사용해야 한다고 주장하는 학파가 있다.                
가혹하다 여길지 모르지만 확실히 장점이 있다. assert가 하나라면 결론이 하나기 때문에 코드를 이해하기 빠르고 쉽다.             

목록 9-5를 보자.         
하지만 목록 9-2는 어떨까? "출력이 XML이다"라는 assert문과 "특정 문자열을 포함한다"는 assert문을 하나로 병합하는 방식이 불합리해 보인다.                                 
하지만 목록 9-7에서 보듯이 테스트를 쪼개 각자가 assert를 수행하면 된다.           

###### 목록 9-7 SerializedPageResponderTest.java (단일 Assert)
```java 
public void testGetPageHierarchyAsXml() throws Exception { 
  givenPages("PageOne", "PageOne.ChildOne", "PageTwo");
  
  whenRequestIsIssued("root", "type:pages");
  
  thenResponseShouldBeXML(); 
}

public void testGetPageHierarchyHasRightTags() throws Exception { 
  givenPages("PageOne", "PageOne.ChildOne", "PageTwo");
  
  whenRequestIsIssued("root", "type:pages");
  
  thenResponseShouldContain(
    "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
  ); 
}
```

위에서 함수 이름을 바꿔 given-when-then 이라는 관례를 사용했다는 사실에 주목한다. 그러면 테스트 코드를 읽기가 쉬워진다.             
**하지만 불행하게도 위에서 보듯이 테스트를 분리하면 중복되는 코드가 많아진다.**          

TEMPLATE METHOD 패턴을 사용하면 중복을 제거할 수 있다.              
given/when 부분을 부모 클래스에 두고 then 부분을 자식 클래스에 두면 된다.           
아니면 완전히 독자적인 테스트 클래스를 만들어 @Before 함수에 given/when 부분을 넣고 @Test 함수에 then 부분을 넣어도 된다.          
하지만 모두가 배보다 배꼽이 더 크다. 이것저것 감안해 보면 결국 목록 9-2처럼 assert 문을 여럿 사용하는 편이 좋다고 생각한다.

나는 **단일 assert 문**이라는 규칙이 훌륭한 지침이라 생각한다.            
목록 9-5에서 봤듯이, 대체로 나는 단일 assert를 지원하는 해당 분야 테스트 언어를 만들려 노력한다.            
하지만 때로는 주저 없이 함수 하나에 여러 assert 문을 넣기도 한다. 단지 assert 문 개수는 최대한 줄여야 좋다는 생각이다.

#### 테스트당 개념 하나
어쩌면 "테스트 함수마다 한 개념만 테스트하라"는 규칙이 더 낫겠다.          
이것저것 잡다한 개념을 연속으로 테스트하는 긴 함수는 피한다.              
목록 9-8은 바람직하지 못한 테스트 함수다. 독자적인 개념 세 개를 테스트하므로 독자적인 테스트 세 개로 쪼개야 마땅하다.           
이를 한 함수로 몰아넣으면 **독자가 각 절이 거기에 존재하는 이유와 각 절이 테스트하는 개념을 모두 이해해야 한다.**

###### 목록 9-8
```java
/**
 * addMonth() 메서드를 테스트하는 장황한 코드
 */
public void testAddMonths() {
  SerialDate d1 = SerialDate.createInstance(31, 5, 2004);

  SerialDate d2 = SerialDate.addMonths(1, d1); 
  assertEquals(30, d2.getDayOfMonth()); 
  assertEquals(6, d2.getMonth()); 
  assertEquals(2004, d2.getYYYY());
  
  SerialDate d3 = SerialDate.addMonths(2, d1); 
  assertEquals(31, d3.getDayOfMonth()); 
  assertEquals(7, d3.getMonth()); 
  assertEquals(2004, d3.getYYYY());
  
  SerialDate d4 = SerialDate.addMonths(1, SerialDate.addMonths(1, d1)); 
  assertEquals(30, d4.getDayOfMonth());
  assertEquals(7, d4.getMonth());
  assertEquals(2004, d4.getYYYY());
}
```

셋으로 분리한 테스트 함수는 각각 다음 기능을 수행한다.     

* (5월처럼) 31일로 끝나는 달의 마지막 날짜가 주어지는 경우
  - (6월처럼) 30일로 끝나는 한 달을 더하면 날짜는 30일이 되어야지 31일이 되어서는 안 된다.
  - 두 달을 더하면 그리고 두 번째 달이 31일로 끝나면 날짜는 31일이 되어야 한다.
* (6월처럼) 30일로 끝나는 달의 마지막 날짜가 주어지는 경우
  - 31일로 끝나는 한 달을 더하면 날짜는 30일이 되어야지 31일이 되면 안 된다.

개념들을 이렇게 정리해 표현하면 장황한 코드 속에 여러 개념을 테스트하고 있음을 알 수 있다.            
이 경우 assert 문이 여럿이라는 사실이 문제가 아니라, 한 테스트 함수에서 여러 개념을 테스트한다는 사실이 문제다.             
**그러므로 가장 좋은 규칙은 "개념 당 assert 문 수를 최소로 줄여라"와 "테스트 함수 하나는 개념 하나만 테스트하라"라 하겠다.**

## F.I.R.S.T.
깨끗한 테스트는 다음 다섯 가지 규칙을 따르는데, 각 규칙에서 첫 글자를 따오면 FIRST가 된다.

**빠르게Fast:**  
테스트는 빨라야 한다. 테스트는 빨리 돌아야 한다는 말이다.           
테스트가 느리면 자주 돌릴 엄두를 못 낸다.            
자주 돌리지 않으면 초반에 문제를 찾아내 고치지 못한다. 코드를 마음껏 정리하지도 못한다.          
결국 코드 품질이 망가지기 시작한다.

**독립적으로 Independent:**  
각 테스트는 독립적이어야 한다.    
한 테스트가 다음 테스트가 실행될 환경을 준비해서는 안 된다.       
테스트가 서로에게 의존하면 하나가 실패할 때 나머지도 잇달아 실패하므로 원인을 진단하기 어려워지며 후반 테스트가 찾아내야 할 결함이 숨겨진다.

**반복가능하게 Repeatable:**  
테스트는 어떤 환경에서도 반복 가능해야 한다.       
실제 환경, QA 환경, 버스를 타고 집으로 가는 길에 사용하는 노트북 환경(네트워크가 연결되지 않은)에서도 실행할 수 있어야 한다.      
테스트가 돌아가지 않는 환경이 하나라도 있다면 테스트가 실패한 이유를 둘러댈 변명이 생긴다.         
게다가 환경이 지원되지 않기에 테스트를 수행하지 못하는 상황에 직면한다.

**자가검증하는 Self-Validating:**  
테스트는 bool값으로 결과를 내야 한다. 성공 아니면 실패다.             
통과 여부를 알리고 로그 파일을 읽게 만들어서는 안 된다.        
통과 여부를 보려고 텍스트 파일 두 개를 수작업으로 비교하게 만들어서도 안 된다.           
테스트가 스스로 성공과 실패를 가늠하지 않는다면 판단은 주관적이 되며 지루한 수작업 평가가 필요하게 된다.

**적시에 Timely:**  
테스트는 적시에 작성해야 한다.       
단위 테스트는 테스트하려는 실제 코드를 구현하기 직전에 구현한다.            
실제 코드를 구현한 다음에 테스트 코드를 만들면 실제 코드가 테스트하기 어렵다는 사실을 발견할지도 모른다.         
어떤 실제 코드는 테스트하기 너무 어렵다고 판명날지 모른다. 테스트가 불가능하도록 실제 코드를 설계할지도 모른다.     

## 결론
- 테스트 코드는 실제 코드보다 더 중요할지도 모르겠다.
- 테스트 코드는 실제 코드의 유연성, 유지모수성, 재사용성을 보존하고 강화하기 때문이다.
- 테스트 코드는 지속적으로 깨끗하게 관리하고, 표현력을 높이고 간결하게 정리하자.
- 테스트 API를 구현해 도메인 특화 언어를 만들자. 그러면 그만큼 테스트 코드를 짜기가 쉬워진다.
- 테스트 코드가 방치되어 망가지면 실제 코드도 망가진다. 테스트 코드를 깨끗하게 유지하자.

## Findings
- 테스트 코드는 실수를 바로잡아준다.
- 테스트 코드는 반드시 존재해야하며, 실제 코드 못지않게 중요하다.
- 테스트 케이스는 변경이 쉽도록 한다. 코드에 유연성, 유지보수성, 재사용성을 제공하는 버팀목이 바로 단위테스트다. 테스트 케이스가 있으면 변경이 두렵지않다.
- 지저분한 테스트 코드는 테스트를 안하니만 못하다.
- assert 문 개수는 최대한 줄여야 좋다.
- 테스트당 assert 하나는 힘들다. 그보다 '테스트 함수마다 한 개념만 테스트'가 더 합리적이다.

## Feelings
- Build-Operator-Check는 단위 테스트를 작성하는데 사용되는 패턴이다. give-when-then 패턴과 비슷하다고 느꼈다.
- TDD는 실제 코드를 짜기 전에 단위 테스트부터 짜라고 요구한다. 이렇게 하면, 실제 코드와 맞먹을 정도의 방대한 테스트 코드는 심각한 관리 문제를 유발하기도 한다고 한다.
<br>단위 테스트를 먼저 짠다고 무조건 좋은 것이 아니다. 
<br>단위 테스트 코드를 가독성있게 구현하고 함수마다 한 개념에 대해서만 테스트하도록 구현해야, 이후 실제 코드를 짜는 것이 수월해진다는 사실을 알게 되었다.
