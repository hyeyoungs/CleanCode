# 07 오류 처리
> 자바에서 예외를 처리하는 방식과 exception을 잘 사용하는 방법

## Introduction
- 오류 처리는 프로그램에 반드시 필요한 요소 중 하나다.
- 뭔가 잘못될 가능성은 늘 존재한다.
- 뭔가 잘못되면 바로 잡을 책임은 바로 우리 프로그래머에게 있다.

## 리턴코드 대신 Exceptions를 사용하라 
- 예전 프로그래밍 언어들은 exceptions를 제공하지 않았다.
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
- 예시: 파일이 없으면 Exception을 제대로 던지는지 확인하는 단위 테스트
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


## Unchecked Exceptions를 사용하라
- Checked exception VS Unchecked Exception
  - 예외처리에 드는 비용 대비 이득을 생각해봐야 한다.
  - 예시
    1. 특정 메소드에서 checked exception을 throw하고
    2. 3단계(메소드 콜) 위의 메소드에서 그 exception을 catch한다면
    3. 모든 중간단계 메소드에 exception을 정의해야 한다.(자바의 경우 메소드 선언에 throws 구문을 붙이는 등)
- Open/Closed Principle violation
  - 상위 레벨 메소드에서 하위 레벨 메소드의 디테일에 대해 알아야 하기 때문에 캡슐화 또한 깨진다.
  - 필요한 경우 checked exceptions를 사용해야 되지만 일반적인 경우 득보다 실이 많다.

## Exceptions로 의미를 제공하라 
- 예외를 던질 때는 전후 상황을 충분히 덧붙인다. 그러면 오류가 발생한 원인과 위치를 찾기가 쉬워진다.
- 예외가 발생한 이유와 좀 더 구체적인 Exception 타입을 제공하라.

## 호출자를 고려해 Exception 클래스를 정의하라 
- Exception class를 만드는 데에서 가장 중요한 것은 "어떤 방식으로 오류를 잡을까"이다.

- 예시: 외부 api의 클래스인 ACMEPort 클래스를 사용하는 상황을 살펴보자.
```java
  // Bad
  // catch문의 내용이 거의 같다.
  
  ACMEPort port = new ACMEPort(12);
  try {
    port.open();
  } catch (DeviceResponseException e) {
    reportPortError(e);
    logger.log("Device response exception", e);
  } catch (ATM1212UnlockedException e) {
    reportPortError(e);
    logger.log("Unlock exception", e);
  } catch (GMXError e) {
    reportPortError(e);
    logger.log("Device response exception");
  } finally {
    ...
  }
```
```java
  // Good
  // ACME 클래스를 LocalPort 클래스로 래핑해 new ACMEPort().open() 메소드에서 던질 수 있는 exception들을 간략화
  
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

## 정상적인 상황을 정의하라 (= Default값을 설정하라) 
- 일반적으로는 위에서 봤던 방식들이 유용하지만, catch문에서 예외적인 상황(special case)을 처리해야 하는 경우 코드가 더러워지는 일이 발생할 수 있다.
- 이런 경우, Martin Fowler의 Special Case Pattern을 사용하자.
- 1. 코드를 부르는 입장에서 예외적인 상황을 신경쓰지 않아도 된다.
- 2. 예외상황은 special case object 내에 캡슐화된다.
```java
  // Bad

  try {
    MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
    m_total += expenses.getTotal();
  } catch(MealExpensesNotFound e) {
    m_total += getMealPerDiem();
  }
```
```java
  // Good

  // caller logic.
  ...
  MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
  m_total += expenses.getTotal();
  ...
  
  public class PerDiemMealExpenses implements MealExpenses {
    public int getTotal() {
      // return the per diem default
    }
  }
  
  // 이해를 돕기 위해 직접 추가한 클래스
  public class ExpenseReportDAO {
    ...
    public MealExpenses getMeals(int employeeId) {
      MealExpenses expenses;
      try {
        expenses = expenseReportDAO.getMeals(employee.getID());
      } catch(MealExpensesNotFound e) {
        expenses = new PerDiemMealExpenses();
      }
      
      return expenses;
    }
    ...
  }
```

## null을 반환하지 마라 
- null을 리턴하고 싶은 생각이 들면 위의 Special Case object를 리턴하라.
- 써드파티 라이브러리에서 null을 리턴할 가능성이 있는 메서드가 있다면 Exception을 던지거나 Special Case object를 리턴하는 매서드로 래핑하라.
```java
  // BAD!!!!

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
  
  
  // 위 peristentStore가 null인 경우에 대한 예외처리가 안된 것을 눈치챘는가?
  // 만약 여기서 NullPointerException이 발생했다면 수십단계 위의 메소드에서 처리해줘야 하나?
  // 이 메소드의 문제점은 null 체크가 부족한게 아니라 null체크가 너무 많다는 것이다.
```
```java
  // Bad
  List<Employee> employees = getEmployees();
  if (employees != null) {
    for(Employee e : employees) {
      totalPay += e.getPay();
    }
  }
```

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

## null을 전달하지 마라 ##
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
// NullPointerException은 안나지만 윗단계에서 InvalidArgumentException이 발생할 경우 처리해줘야 함.
public class MetricsCalculator {
  public double xProjection(Point p1, Point p2) {
    if(p1 == null || p2 == null){
      throw InvalidArgumentException("Invalid argument for MetricsCalculator.xProjection");
    }
    return (p2.x – p1.x) * 1.5;
  }
}

// Bad
// 좋은 명세이지만 첫번째 예시와 같이 NullPointerException 문제를 해결하지 못한다.
public class MetricsCalculator {
  public double xProjection(Point p1, Point p2) {
    assert p1 != null : "p1 should not be null";
    assert p2 != null : "p2 should not be null";
    
    return (p2.x – p1.x) * 1.5;
  }
}
```

## 결론
깨끗한 코드와 안정성이 높ㅍ은 코드는 상충하는 목표가 아니다.      
예외처리와 로직을 분리하면 튼튼하고 깨끗한 코드를 작성할 수 있다.       
또한, 독립적인 추론이 가능해지며 코드 유지보수성도 크게 높아진다.

## Issue
### 1. Checked exception VS Unchecked Exception(what?, 특징, 예시, when?)
- Checked Exception과 Unchecked Exception의 가장 명확한 구분 기준은 ‘반드시 예외 처리를 해야 하느냐’이다. 
  - Checked Exception이 발생할 가능성이 있는 메소드라면 반드시 로직을 try/catch로 감싸거나 throw로 던져서 처리해야 한다. 
  - 반면에 Unchecked Exception은 명시적인 예외처리를 하지 않아도 된다. 
    <br>이 예외는 피할 수 있지만 개발자가 부주의해서 발생하는 경우가 대부분이고, 
    <br>미리 예측하지 못했던 상황에서 발생하는 예외가 아니기 때문에 굳이 로직으로 처리를 할 필요가 없도록 만들어져 있다.  
  
- when?
    - Exception을 상속하면 Checked Exeption 명시적인 예외처리가 필요하다
      - (예) IOException, SQLException
    - RuntimeException을 상속하면 Unchecked Exeption 명시적인 예외처리가 필요하지 않다.
      - (예) NullPointException, IllegalArgumentException, IndexOutOfBoundException
      
- Effective Java - Exception에 관한 규약

> 자바 언어 명세가 요구하는 것은 아니지만, 업계에 널리 퍼진 규약으로
Error클래스를 상속해 하위 클래스를 만드는 일은 자제하자.        
즉, 사용자가 직접 구현하는 unchecked throwable은 모두 RuntimeException의 하위 클래스여야 한다.          
Exception, RuntimeException, Error를 상속하지 않는 throwable을 만들 수도 있지만,       
이러한 throwable은 정상적인 사항보다 나을 게 하나도 없으면서 API 사용자를 헷갈리게 할 뿐이므로 절대로 사용하지 말자.

- Checked Exeption이 나쁜 이유
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
   - 상위 레벨 메소드에서 하위 레벨 메소드의 디테일에 대해 알아야 하기 때문에 OCP 원칙에 위배된다.
3. 필요한 경우 checked exception을 사용해야 되지만 일반적인 경우 득보다 실이 많다.

### 2. Open/Closed Principle이란?

### 3. SpringBoot의 다양한 예외 처리 방법 및 내부의 구현
#### SpringBoot의 다양한 예외 처리 방법
스프링에서 예외처리는 크게 3가지로 나눌 수 있다.

1. 컨트롤러단에서 처리 Controller Level - @ExceptionHandler
2. 전역 처리 Global Level - @ControllerAdvice
3. 메서드단위 처리 Method Level - try/catch

> 정확히는 DispatcherServlet에서 발생하는 예외를 `HandlerExceptionResolver`가 처리하는 처리 방법들.
##### 3-1. Controller레벨에서 처리 - @ExceptionHandler

스프링에서 Controller에서 발생하는 예외를 공통적으로 처리할 수 있는 기능을 제공한다.

**`@ExceptionHandler`애노테이션을 통해 Controller의 메서드에서 throw된 Exception에 대한 공통적인 처리를 할 수 있다.**

```java
@RestController
public class TestController {

    private final Logger logger = LoggerFactory.getLogger(UserController.class);

    // 예외 핸들러
    @ExceptionHandler(value = TestException.class)
    public String controllerExceptionHandler(Exception e) {
        logger.error(e.getMessage());
        return "/error/404";
    }

    @GetMapping("hello1")
    public String hello1() {
        throw new TestException("hello1 에러 "); // 강제로 예외 발생
    }

    @GetMapping("hello2")
    public String hello2() {
        throw new TestException("hello2 에러 "); // 강제로 예외 발생
    }
}
```

TestController내에서 발생하는 TestException에 대해서 예외가 발생하면 `controllerExceptionHandler`메서드에서 모두 처리해준다.

* Controller 메서드 내의 하위 서비스 (Service, Repository등등)에서 예외가 발생하더라도, 중간에 처리하지 않는 이상 Controller단까지 예외가 던져지게 되고 `@ExceptionHandler`가 예외를 처리하게 된다.
    * Checked Exception, Runtime Exception 상관 없이 Controller까지 예외를 throw하면 처리가 가능하다.

##### 3-2. Global 레벨에서의 처리 - @ControllerAdvice

만약 하나의 Controller말고 여러 Controller에서 발생하는 예외를 처리하려면 `@ControllerAdvice`를 사용해야 한다.

* @ControllerAdvice
    * 모든 Controller에서 발생하는 예외를 처리할 수 있게 해주는 애노테이션
    * DispatcherServlet에서 발생하는 예외를 전역적으로 처리해준다.
* @RestControllerAdvice
    * @ControllerAdvice + @ResponseBody

> @ControllerAdvice는 DispatcherServlet에서 발생하는 예외만 처리할 수 있다.
>
> 필터에서 발생하는 예외는 따로 처리해주지 않으면 처리가 불가하다.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    private final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    @ExceptionHandler(value = TestException.class)
    public String testExceptionHandler(Exception e) {
        logger.error(e.getMessage());
        return "/error/404";
    }
}
```

* Controller에서 발생하는 예외를 전역적으로 처리해준다.

> Controller의 `@ExceptionHandler`와 ControllerAdvice의 `@ExceptionHandler`중 높은 우선순위는?
>
> * **Controller의 `@ExceptionHandler`가 먼저다.**

#### 내부의 구현

### 4.Martin Fowler의 Special Case Pattern이란?
