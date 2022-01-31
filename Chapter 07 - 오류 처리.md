# 07 오류 처리
> Exception을 잘 사용하는 방법

## Introduction
- 오류 처리는 프로그램에 반드시 필요한 요소 중 하나다.
- 뭔가 잘못될 가능성은 늘 존재한다.
- 뭔가 잘못되면 바로 잡을 책임은 바로 우리 프로그래머에게 있다.

## 리턴코드 대신 Exception을 사용하라 
- 예전 프로그래밍 언어들은 Exception을 제공하지 않았다.
- 그 경우, 개발자들은 오류 flag를 설정하거나 오류 코드를 리턴, 호출하는 측에서 오류 코드를 반환해줘야 했다.
```java
// Bad
public class DeviceController {
  ...
  public void sendShutDown() {
    DeviceHandle handle = getHandle(DEV1);
    // 디바이드 상태를 체크한다.
    if (handle != DeviceHandle.INVALID) {
      // 레코드 필드에 디바이스 상태를 저장한다.
      retrieveDeviceRecord(handle);
      // 디바이스 상태가 일시정지 상태가 아니라면 종료한다.
      if (record.getStatus() != DEVICE_SUSPENDED) {
        pauseDevice(handle);
        clearDeviceWorkQueue(handle);
        closeDevice(handle);
      } else {
        logger.log("Device suspended. Unable to shut down");
      }
    } else {
      logger.log("Invalid handle for: " + DEV1.toString());
    }
  }
  ...
}
```
- 하지만 이러한 방식은 예외처리를 잊어버리기 쉽고 로직을 헷갈리게 하기 쉽다.
- 그래서 오류가 발생하면 예외를 던지는 것이 낫다. 

```java
// Good
public class DeviceController {
  ...
  public void sendShutDown() {
    try {
      tryToShutDown();
    } catch (DeviceShutDownError e) {
      logger.log(e);
    }
  }
    
  private void tryToShutDown() throws DeviceShutDownError {
    DeviceHandle handle = getHandle(DEV1);
    DeviceRecord record = retrieveDeviceRecord(handle);
    pauseDevice(handle); 
    clearDeviceWorkQueue(handle); 
    closeDevice(handle);
  }
  
  private DeviceHandle getHandle(DeviceID id) {
    ...
    throw new DeviceShutDownError("Invalid handle for: " + id.toString());
    ...
  }
  ...
}
```
- 보기만 좋아진게 아니라 "실제 로직"과 "예외처리" 부분이 나뉘어져 필요한 부분에 집중할 수 있게 된다.

## Try-Catch-Finally문을 먼저 써라 
- try문은 transaction처럼 동작하는 실행코드로, catch문은 try문에 관계없이 프로그램 상태를 일관성 있게 유지해야 한다.
- 그러므로 예외가 발생한 코드를 짤 때는 try-catch-finally 문으로 시작하는 편이 낫다.
- 이렇게 함으로써 코드의 "Scope 정의"가 가능해진다.
- 예시 : 파일이 없으면 Exception을 제대로 던지는지 확인하는 단위 테스트
```java
  // 1. StorageException을 던지지 않으므로 이 단위 테스트는 실패한다.
  @Test(expected = StorageException.class)
  public void retrieveSectionShouldThrowOnInvalidFileName() {
    sectionStore.retrieveSection("invalid - file");
  }
  
  public List<RecordedGrip> retrieveSection(String sectionName) {
    // 실제로 구현할 때까지 비어있는 더미를 반환한다.
    return new ArrayList<RecordedGrip>();
  }
```
```java
  // 2. 코드가 예외를 던지므로 이제 테스트는 통과한다.
  public List<RecordedGrip> retrieveSection(String sectionName) {
    try {
      FileInputStream stream = new FileInputStream(sectionName)
    } catch (Exception e) {
      throw new StorageException("retrieval error", e);
    }
  return new ArrayList<RecordedGrip>();
}
```
```java
  // 3. Exception의 범위를 FileNotFoundException으로 줄여 정확히 어떤 Exception이 발생한지 체크한다.
  public List<RecordedGrip> retrieveSection(String sectionName) {
    try {
      FileInputStream stream = new FileInputStream(sectionName);
      stream.close();
    } catch (FileNotFoundException e) {
      throw new StorageException("retrieval error", e);
    }
    return new ArrayList<RecordedGrip>();
  }
```
- 먼저 강제로 예외를 일으키는 테스트 케이스를 작성한 후 테스트를 통과하게 코드를 작성하는 방법을 권장한다.
- 그러면 자연스럽게 try 블록의 트랜잭션 범위부터 구현하게 되므로 범위 내에서 트랜잭션 본질을 유지하기 쉬워진다.

## Unchecked Exception을 사용하라
- Checked Exception
  - Checked Exception에 드는 비용 대비 이득을 생각해봐야 한다.
  - 🥲 Checked Exception은 OCP를 위반한다.
    - Open/Closed Principle violation
      - 상위 레벨 메소드에서 하위 레벨 메소드의 디테일에 대해 알아야 하기 때문에 캡슐화 또한 깨진다.
  - 😮 때로는 유용하다. 아주 중요한 라이브러리(ex. IO Exception, SQL Exception)를 작성한다면 모든 예외를 잡아야 한다.
  - → 필요한 경우 Checked Exception를 사용해야 되지만 일반적인 경우 득보다 실(의존성)이 많다.
  
- Checked Exception이 OCP를 위반하는 예시
```java
public class DeviceController {
  ...
  public void sendShutDown() { // 3
    try {
        tryToShutDown();
    } catch (DeviceShutDownError e) {
        logger.log(e);
    }
  } // 3
  
  private void tryToShutDown() throws DeviceShutDownError { // 2
    DeviceHandle handle = getHandle(DEV1);
    DeviceRecord record = retrieveDeviceRecord(handle);
    pauseDevice(handle);
    clearDeviceWorkQueue(handle);
    closeDevice(handle);
  }
  
  private DeviceHandle getHandle(DeviceId id) {
      ...
          throw new DeviceShutDownError("Invalid handle for: " + id.toString()); // 1
      ...
  }
  ...
}
```

1. 특정 메소드에서 checked exception을 throw하고 상위 메소드에서 그 exception을 catch한다면 모든 중간단계 메소드에 exception을 throws해야 한다.
2. OCP(계방 폐쇄 원칙) 위배
<br>상위 레벨 메소드에서 하위 레벨 메소드의 디테일에 대해 알아야 하기 때문에 OCP 원칙에 위배된다.

## Exception에 의미를 제공하라 
- 예외를 던질 때는 전후 상황을 충분히 덧붙인다. 그러면 오류가 발생한 원인과 위치를 찾기가 쉬워진다.
- 오류메세지에 정보를 담아 예외와 함께 던진다. (= 예외가 발생한 이유와 좀 더 구체적인 Exception 타입을 제공하라.)
  - ex. 실패한 연산 이름과 실패 유형도 언급한다.

## 호출자를 고려해 Exception 클래스를 정의하라 
- Exception class를 만드는 데에서 가장 중요한 것은 `어떤 방식으로 오류를 잡을까`이다.
- 예시: 외부 api의 클래스인 ACMEPort 클래스를 사용하는 상황을 살펴보자. <br> → 외부 API를 사용할 때는 감싸기 기법이 최선이다.
```java
  // Bad 
  // 오류를 형편없이 분류했다. 
  // catch문의 내용이 거의 같다.
  
  ACMEPort port = new ACMEPort(12);
  try {
    port.open();
  } catch (DeviceResponseException e) {
    reportPortError(e); // 1) 오류를 기록한다.
    logger.log("Device response exception", e); // 2) 프로그램을 계속 수행해도 좋은지 확인한다.
  } catch (ATM1212UnlockedException e) {
    reportPortError(e); // 1) 오류를 기록한다.
    logger.log("Unlock exception", e); // 2) 프로그램을 계속 수행해도 좋은지 확인한다.
  } catch (GMXError e) {
    reportPortError(e); // 1) 오류를 기록한다.
    logger.log("Device response exception"); // 2) 프로그램을 계속 수행해도 좋은지 확인한다.
  } finally {
    ...
  }
```
```java
  // Good
  // 호출하는 라이브러리 API를 감싸서 예외 유형 하나를 반환한다.
  // ACMEPort 클래스를 LocalPort 클래스로 감쌌다. 
  // LocalPort 클래스는 ACMEPort 클래스가 던지는 예외를 잡아 변환하는 Wrapper 클래스다.
  // LocalPort클래스로 감싸주니, new ACMEPort().open() 메소드에서 던질 수 있는 Exception 부분이 실제 로직과 나뉘어졌다. 
  // 외부 API를 사용할 때는 감싸기 기법이 최선이다.
  // 외부 API를 감싸면 외부 라이브러리와 프로그램 사이에서 의존성이 크게 줄어든다.
  
  
  LocalPort port = new LocalPort(12);
  try {
    port.open();
  } catch (PortDeviceFailure e) {
    reportError(e);
    logger.log(e.getMessage(), e);
  } finally {
    ...
  }
  
  public class LocalPort {
    private ACMEPort innerPort;
    public LocalPort(int portNumber) {
      innerPort = new ACMEPort(portNumber);
    }
    
    public void open() {
      try {
        innerPort.open();
      } catch (DeviceResponseException e) {
        throw new PortDeviceFailure(e);
      } catch (ATM1212UnlockedException e) {
        throw new PortDeviceFailure(e);
      } catch (GMXError e) {
        throw new PortDeviceFailure(e);
      }
    }
    ...
  }
```

## 정상적인 상황을 정의하라
- 일반적으로는 외부 API를 감싸 독자적인 예외를 던지고, 코드 위에 처리기를 정의해 중단된 계산을 처리한다.
- 대게는 멋진 처리 방식이지만, 때로는 중단이 적합하지 않은 때도 있다.
- catch문에서 예외적인 상황(special case)을 처리해야 하는 경우 코드가 더러워지는 일이 발생할 수 있다.
- 이런 경우, Special Case Pattern을 사용하자.
  - 클래스를 만들거나 객체를 조작해 특수 사례를 처리하는 방식이다.
   
```java
  // Bad
  try {
    MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
    m_total += expenses.getTotal();
  } catch(MealExpensesNotFound e) {
    m_total += getMealPerDiem();
  }
```
- 위에서 식비를 비용으로 청구했다면 직원이 청구한 식비를 총계에 더한다.
- 식비를 비용으로 청구하지 않았다면 일일 기본 식비를 총계에 더한다.
- 그런데 예외가 논리를 따라가기가 어렵게 만든다.
- 그러므로, 특수 상황을 처리할 필요가 없게 만들어보자.
```java
  // Good
  // Special Case Pattern
  ...
  MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
  m_total += expenses.getTotal();
  ...
  
  public class PerDiemMealExpenses implements MealExpenses {
    public int getTotal() {
      // 기본값으로 일일 기본 식비를 반환한다.
    }
  }
```
- ExpenseReportDAO를 고쳐 언제나 MealExpense 객체를 반환한다.
- 청구한 식비가 없다면 일일 기본 식비를 반환하는 MealExpense 객체를 반환한다.

→ 클래스를 만들거나 객체를 조작해 특수 사례를 처리하면, 
<br>클라이언트 코드(코드를 부르는 입장)에서 예외적인 상황을 신경쓰지 않아도 된다. 
<br>클래스나 객체가 예외적인 상황을 캡슐화해서 처리하기 때문이다.

## null을 반환하지 마라 
오류를 유발하는 행위가 있다.    
그 중 첫째가 `null을 반환하는 습관`이다.
```java
  // BAD
  public void registerItem(Item item) {
    if (item != null) {
      ItemRegistry registry = peristentStore.getItemRegistry();
      if (registry != null) {
        Item existing = registry.getItem(item.getID());
        if (existing.getBillingPeriod().hasRetailOwner()) {
          existing.register(item);
        }
      }
    }
  }
```
- 위코드는 나쁜 코드다.
- null을 반환하는 코드는 일거리를 늘릴 뿐만 아니라 호출자에게 문제를 떠넘긴다. <br>null 확인을 빼먹으면 애플리케이션이 통제 불능 사오항에 빠질지도 모른다. 
- peristentStore의 값은 null인지 확인이 빠졌다는 사실을 눈치챘는가? 
- 만약 여기서 NullPointerException이 발생했다면 수십단계 위의 메소드에서 처리해줘야 하는 문제가 발생한다. 
- 이 메소드의 문제점은 null 체크가 부족한게 아니라 null 체크가 너무 많다는 것이다.
- 메서드에서 null을 반환하고픈 유혹이 든다면 그 대신 예외를 던지거나 특수 사례 객체를 반환한다. 

```java
  // Bad
  List<Employee> employees = getEmployees();
  if (employees != null) {
    for(Employee e : employees) {
      totalPay += e.getPay();
    }
  }
```
- getEmployees()는 null도 반환한다.
- 반드시 null을 반환할 필요가 없다. getEmployees를 변경해 빈 리스트를 반환하도록 처리하자.

```java
  // Good
  List<Employee> employees = getEmployees();
  for(Employee e : employees) {
    totalPay += e.getPay();
  }
  
  public List<Employee> getEmployees() {
    if( .. there are no employees .. )
      return Collections.emptyList();
    }
}
```

## null을 전달하지 마라
- null을 리턴하는 것도 나쁘지만 null을 메서드로 넘기는 것은 더 나쁘다.
- null을 메서드의 파라미터로 넣어야 하는 API를 사용하는 경우가 아니면 null을 메서드로 넘기지 마라.
- 일반적으로 대다수의 프로그래밍 언어들은 파라미터로 들어온 null에 대해 적절한 방법을 제공하지 못한다.
- 가장 이성적인 해법은 null을 파라미터로 받지 못하게 하는 것이다.

```java
// Bad
// calculator.xProjection(null, new Point(12, 13));
// 위와 같이 부를 경우 NullPointerException 발생
public class MetricsCalculator {
  public double xProjection(Point p1, Point p2) {
    return (p2.x – p1.x) * 1.5;
  }
  ...
}

// Bad
// NullPointerException은 안 나지만 윗 단계에서 InvalidArgumentException이 발생할 경우 처리해줘야 한다.
public class MetricsCalculator {
  public double xProjection(Point p1, Point p2) {
    if(p1 == null || p2 == null){
      throw InvalidArgumentException("Invalid argument for MetricsCalculator.xProjection");
    }
    return (p2.x – p1.x) * 1.5;
  }
}

// Bad
// assert문을 사용하는 방법
// 문서화가 잘되어 코드를 읽기는 편하지만,
// 첫번째 예시와 같이 NullPointerException 문제를 해결하지 못한다.
public class MetricsCalculator {
  public double xProjection(Point p1, Point p2) {
    assert p1 != null : "p1 should not be null";
    assert p2 != null : "p2 should not be null";
    
    return (p2.x – p1.x) * 1.5;
  }
}
```
- 대다수 프로그래밍 언어는 호출자가 실수로 넘기는 null을 적절히 처리하는 방법이 없다.
- 애초에 null을 넘기지 못하도록 금지하는 정책이 합리적이다.

## 결론
깨끗한 코드와 안정성이 높은 코드는 상충하는 목표가 아니다.      
예외처리와 로직을 분리하면 튼튼하고 깨끗한 코드를 작성할 수 있다.       
또한, 독립적인 추론이 가능해지며 코드 유지보수성도 크게 높아진다.

## Findings
- 오류 flag를 설정하거나 오류 코드를 리턴해주지 말고, Exception처리를 해주자.
- Checked Exception은 상위 레벨 메소드에서 하위 레벨 메소드의 디테일에 대해 알아야 하기 때문에 캡슐화가 깨진다.
<br>이 말은 예외가 발생하는 함수를 호출한 상위 함수들에 계속해서 thorws 를 붙여줘야 한다는 것이다.
<br>필요한 경우 Checked Exception를 사용해야 되지만 일반적인 경우 득보다 실(의존성)이 많다.
- 외부 API를 사용할 때는 감싸기 기법이 최선이다.
<br>외부 API를 감싸면 외부 라이브러리와 프로그램 사이에서 의존성이 크게 줄어든다.
- Null은 반환하지 말고, 전달하지 말자.
  - NullPointerException이 발생했다면 수십단계 위의 메소드에서 처리해줘야 한다. 예외찾기가 어렵다.
  - 지금까지 Null을 반환하고 체크해주는 코드를 많이 작성했다.
    - 이를 해결하기 위한 방법
      - 메서드에서 null을 반환하고픈 유혹이 든다면 그 대신 예외를 던지거나 특수 사례 객체를 반환하자.
      - 리스트의 경우, Collections.emptyList(); 를 반환하자.

## Feelings
- checked exception(= 컴파일 단계에서 exception 확인)을 사용하는 것보다 unchecked exception(= 런타임 단계에서 exception 확인)을 사용하는게 낫다는 것을 처음 알게 되었다.
- checked exception의 경우 의존성이라는 비용이 생긴다. 득보다 실이 크다.
  <br>예외가 발생하는 함수를 호출한 상위 함수들에 계속해서 thorws 를 붙여줘야 하기 때문이다.
  <br>그렇다면, checked exception이 필요한 경우도 있는데 그 경우가 언제인지 궁금했다.
- NullPointerException의 에러의 원인을 찾기는 어렵다는 것을 다시 실감했다.
