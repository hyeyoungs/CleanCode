# 08 경계
> 외부 코드에 적당한 경계를 지어 깔끔하게 통합하는 방법

## Introduction
시스템에 들어가는 모든 소프트웨어를 직접 개발하는 경우는 드물다.        
떄로는 패키지를 사고, 때로는 오픈 소스를 이용한다.       
이 외부 코드를 우리 코드에 깔끔하게 통합해야만 한다.      

## 외부 코드 사용하기 : Encapsulation
인터페이스 제공자는 적용성을 최대한 넓히려고 애쓴다.       
반면, 사용자는 자신의 요구에 집중하는 인터페이스를 바란다.       
이런 긴장으로 인해 시스템 경계에서 문제가 생길 소지가 많다.

java.util.Map 예시
```java
// Bad
// Sensor라는 객체를 담는 Map 생성
Map sensors = new HashMap();
// Sensor 객체 가져오기
Sensor s = (Sensor) sensors.get(sensorId);
```
- Map이 반환하는 Object를 올바른 유형으로 변환할 책임은 Map을 사용하는 클라이언트에 있다.
- 의도가 분명히 드러나지 않는다.
```java
// Bad (위에보다는 낫다.)
Map<String, Sensor> sensors = new HashMap<Sensor>();
...
Sensor s = sensor.get(sensorId);
```
- 제네릭스를 사용하면, 코드 가독성이 나아진다.
- 하지만, 여전히 "Map<String, Sensor>가 사용자에게 필요하지 않은 기능까지 제공한다"는 문제를 해결하지 못한다.
- 프로그램에서 Map<String, Sensor> 인스턴스를 여기저기 넘긴다면, Map 인터페이스가 변할 경우, 수정할 코드가 상당히 많아진다. (= sensor 데이터가 손상될 수 있고, 이는 우리 의도와 벗어난다.)

```java
// Good
public class Sensors {
    private Map sensors = new HashMap();
    public Sensor getById(String id) {
        return (Sensor) sensors.get(id);
    }
    // 이하 생략
}
```
- 경계 인터페이스인 Map을 Sensors 안으로 숨긴다. 
<br> Sensors 클래스 안에서 객체 유형을 관리하고 변환한다.
<br>→ Sensors 클래스는 프로그램에 필요한 인터페이스만 제공한다. 적절한 경계로 우리 코드를 보호할 수 있다.
<br>( = 코드는 이해하기는 쉽지만 오용하기는 어렵다. 또한, Sensors 클래스는 나머지 프로그램이 설계 규칙과 비즈니스 규칙을 따르도록 강제할 수 있다.)

- 핵심은 Map 클래스 를 사용할 때마다 위와 같이 캡슐화하라는 소리가 아니라, Map(= 유사한 경계 인터페이스)을 여기저기 넘기지 말라는 말이다.
<br> 즉, Map 인스턴스를 공개 API의 인수로 넘기거나 반환값으로 사용하지 않는다.

## 경계를 살피고 익히기
외부 코드를 사용하면 적은 시간에 더 많은 기능을 출시하기 쉬워진다.            
만약 외부에서 가져온 패키지를 사용하고 싶다면?      
우리 자신을 위해 우리가 사용할 코드를 테스트하는 것이 바람직하다.

외부 코드를 익히기는 어렵고 통합하기도 어렵다.      
다르게 접근해보자.      
곧바로 우리쪽 코드를 작성해 외부 코드를 호출하는 대신 `먼저 간단한 테스트 케이스를 작성해 외부 코드를 익혀보자.` (학습 테스트)

## 학습 테스트는 공짜 이상이다
학습테스트에 드는 비용이 없다.       
학습테스트는 이해도를 높여주는 정확한 실험이다.     
패키지 새 버전이 나올 때마다 새로운 위험이 생기지만, 새 버전이 우리 코드와 호환되지 않으면 학습 테스트가 이 사실을 곧바로 밝혀낸다.

학습 테스트를 이용한 학습의 필요 여부와 상관없이, 실제 코드와 동일한 방식으로 인터페이슬를 사용하는 테스트 케이스(= 경계 테스트)가 필요하다.       
경계 테스트가 있다면 패키지의 새 버전으로 이전하기 쉬워진다.

## 아직 존재하지 않는 코드를 사용하기 : Adapter Pattern
- 경계에는 아는 코드와 모르는 코드를 분리하는 경계가 있다.
- 아직 개발되지 않은 모듈이 필요한데, 기능은 커녕 인터페이스조차 구현되지 않은 경우가 있을 수 있다.
- 하지만 우리는 이러한 제약때문에 우리의 구현이 늦어지는걸 탐탁치 않게 여긴다.
- 예시
    - 저자는 무선통신 시스템에 들어갈 소프트웨어 개발에 참여했다.
    - 그 소프트웨어에는 "송신기"라는 하위 시스템이 있었는데 저자의 팀원들은 송신기에 대한 지식이 거의 없었다.
    - "송신기"팀은 인터페이스를 정의하지 못한 상태였다. 
    - 하지만 저자는 "송신기"팀을 기다리는 대신 "원하는" 기능을 정의하고 자체적으로 인터페이스를 정의했다.
      - 저장한 주파수를 이용해 이 스트림에서 들어오는 자료를 아날로그 신호로 전송하라.
    - 이렇게 필요한 인터페이스를 정의함으로써 메인 로직을 더 깔끔하게 짤 수 있었고 목표를 명확하게 나타낼 수 있었다.
    - Adapter Pattern으로 API 사용을 캡슐화해 API가 바뀔 때 수정한 코드를 한 곳으로 모았다.

## 깨끗한 경계
- 통제하지 못하는 코드를 사용할 때는 너무 많은 투자를 하거나 향후 변경 비용이 지나치게 커지지 않도록 각별히 주의해야 한다.
- 경계에 위치하는 코드는 깔끔히 분리한다. 
- 또한 기대치를 정의하는 테스트 케이스도 작성한다.
- 우리가 컨트롤할 수 있는 것에 의지하는게 그렇지 않은 것에 의지하는 것보다 낫다. 그렇지 않으면 외부 코드에 휘둘리고 만다.
- Map 객체를 `새로운 클래스로 경계를 감싸거나` `Adapter 패턴을 사용해 우리가 원하는 인터페이스를 패키지가 제공하는 인터페이스로 변환`하자.
<br>→코드는 보기 편해지고 경계 인터페이스를 일관적으로 사용할 수 있게 해주며 외부 패키지 변경에도 유연하게 대응할 수 있게 해준다.

## Findings
- **우리 코드와 외부 코드를 깔끔하게 통합시키기 위해 경계를 잘 지어야한다.**
<br>오픈소스, 라이브러리를 안쓰는 프로젝트는 없다. 우리가 만든 코드에 외부에서 들어온 코드를 병합해야 한다.
<br>경계는 우리코드와 외부코드 사이를 말한다.
- **경계 짓기**
  - `Encapsulation` : 새로운 클래스로 경계를 감싸어 객체의 실제 구현을 외부로부터 감추는 방식
  <br>외부 코드를 감춘다. 외부 코드를 공개 API의 인수로 넘기거나 반환값으로 사용하지 않게 하여 우리 코드를 보호시킨다.
  - `Adapter Pattern` : 인터페이스를 이용한 디자인 패턴
  <br>외부 코드를 호출할 때 우리가 원하는 방식으로 사용하고 싶다면 Adapter Pattern을 사용한다.
- **외부 코드를 사용하고 싶다면?** 경계를 살피고 익히기
  <br> → 🌟 가장 먼저, `Learning Test`를 작성해 라이브러리를 테스트 한다.
    - 외부 코드를 배우고, 안정성도 미리 검증 할 수 있다. 
      - 학습 테스트는 이해도를 높인다. 
      - 외부 코드의 버전이 변경됐을 때, 우리 코드와 호환되는 지 확인할 수 있다.
  
## Feelings
- 나쁜 코드는 생산성을 저하한다는 말과 통제하지 않는 것 보다 통제할 수 있는 것에 의지하는게 낫다는 말이 실감이 되었다.
- 함께 프로젝트 하는 사람이 통제가 안되는 외부 코드를 가져왔을 때, 해당 코드와 관련된 파일에서 에러가 발생하면, 그때마다 디버깅하는데 3시간 넘게 걸렸고 팀 전체가 생산성 저하가 발생했다.
- 외부 코드를 가져올 경우, 무작정 다 가져오는 것이 아니라, 해당 프로젝트에 필요한 것만 가져오고 프로젝트에 맞게 수정하는 태도를 가져야겠다는 생각이 들었고 팀원이 그렇게 할 수 있도록 이끌어주는 사람이 되자.
- 외부 코드를 사용하고 싶다면, 먼저 Learning Test를 작성해 라이브러리를 테스트 해보면서, 외부 코드의 이해도를 높이고 우리 코드와 호환되는지 확인을 해야겠다.

## Question
- Adapter Pattern이란?
