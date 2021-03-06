# 13 동시성
> 안전한 동시성 코드 구현

<br><br>
## Introduction
> 객체는 처리의 추상화다. 스레드는 일정의 추상화다.<br>- James O. Coplien

동시성과 깔끔한 코드는 양립하기 아주 어렵다.       
13 장에서는 `여러 스레드를 동시에 돌리는 이유`를 논한다. `여러 스레드를 동시에 돌리는 어려움`도 논한다.      
이런 `어려움에 대처하고 깨끗한 코드를 작성하는 방법`도 몇 가지 제안한다. 마지막으로 `동시성을 테스트하는 방법과 문제점`을 논한다.

<br><br>
## 동시성이 필요한 이유?
- 동시성은 결합(coupling)을 없애는 전략이다. 즉 무엇(What)과 언제(When)을 분리하는 전략이다.       
- 무엇과 언제를 분리하면 어플리케이션 구조와 효율이 극적으로 나아진다.      
  - 구조적인 관점에서 프로그램은 거대한 루프 하나가 아니라 작은 협력 프로그램 여럿으로 보인다. <br>
    → 시스템을 이해하기가 쉽고 문제를 분리하기도 쉽다.
- 예시 : Servlet 모델
  - 이론적으로 Servlet 개발자는 모든 웹 요청을 관리할 필요가 없다.
    <br>(요청을 개별적으로 처리하는 데에만 신경을 쓰며 요청 큐를 직접 관리하는 부담을 덜 수 있다.) 
  - 물론, Servlet이 제공하는 결합분리 전략은 완벽하지 않지만 Servlet이 제공하는 구조적인 이점은 그 자체로 가치가 있다.
- 동시성(무엇과 언제를 분리)을 채택하는 이유에는 크게 2가지가 있다.
  1. `구조적 이점이 크기 때문에` 사용하는 것이다.
  2. `응답 시간과 작업 처리량 개선`
  <br>예를 들면, 웹 크롤링 시스템이 단일 쓰레드라면 하나의 사이트의 데이터만 수집 가능하다.
  <br>더군다나 사이트의 갯수가 늘어난다면, 기존보다 시간이 더 오래걸리는 문제가 발생한다.
  <br>그렇기에 이 같은 경우, 다중 스레드 알고리즘을 이용해서 시스템의 성능을 높일 수 있다.
  
#### 미신과 오해
동시성이 반드시 필요한 상황은 존재하지만, **동시성은 어렵다.**         
그렇기 때문에, **각별히 주의하지 않으면 난감한 상황에 처한다.**

###### 동시성과 관련한 일반적인 미신과 오해
  - 동시성은 항상 성능을 높여준다?      
    동시성은 `때론` 성능을 높여준다.          
    대기 시간이 아주 길어 여러 스레드가 프로세서를 공유할 수 있거나,            
    여러 프로세서가 동시에 처리할 독립적인 계산이 정말 많은 경우에만 성능을 높여진다.             
    어느 쪽도 일반적으로 발생하는 상황은 아니기에, 때론 성능을 높여준다고 말한다
  - 동시성을 구현해도 설계는 변하지 않는다?  
    단일 스레드 시스템과 다중 스레드 시스템은 `설계가 판이하게 다르다.`          
    일반적으로 `무엇`과 `언제`를 분리하면 시스템 구조가 크게 달라진다.
  - 웹 또는 EJB 컨테이너를 사용하면 동시성을 이해할 필요가 없다.<br>
    우리는 실제로는 `컨테이너가 어떻게 동작하는지, 어떻게 동시 수정, 데드락 등과 같은 문제를 피할 수 있는지를 알아야만 한다.`

###### 동시성과 관련한 타당한 생각
* 동시성은 다소 부하를 유발한다.  
  성능 측면에서 부하가 걸리며, 코드도 더 짜야한다.
* 동시성은 복잡하다.    
  간단한 문제라도 동시성은 복잡하다.
* 일반적으로 동시성 버그는 재현하기 어렵다.    
  그래서 진짜 결함으로 간주되지 않고 일회성 문제로 여겨 무시하기 쉽다.
* 동시성을 구현하려면 흔히 근본적인 설계 전략을 재고해야한다.

<br><br>
## 난관 : 동시성을 구현하기가 어려운 이유는 무엇일까?
```java
public class X {
    private int lastIdUsed;
    
    public X(int lastIdUsed){
        this.lastIdUsed = lastIdUsed;
    }
    
    public int getNextId() {
        return ++lastIdUsed;    
    }
}
```
위 클래스를 기반으로한 인스턴스 x 를 생성하면서 값을 42로 넣어준다.       
그리고 2개의 스레드가 인스턴스 x를 공유하며 getNextId()를 호출한다 가정한다.     
그렇다면 결과 값은 아래 3중 하나일 것이다.

* 한 스레드는 43을 받고 다른 스레드는 44를 받으며 lastIdUsed는 44가 된다.
* 한 스레드는 44를 받고 다른 스레드는 43을 받으며 lastIdUsed는 44가 된다.
* **한 스레드는 43을 받고 다른 스레드는 43을 받으며 lastIdUsed는 43가 된다.**

다중 스레드의 실행 순서는 우리가 알기 힘들다.         
그렇기에 어떤 경우에는 3번과 같이 잘못된 결과를 내놓는다.                         
물론, 대다수의 경우는 올바른 결과를 내놓긴 하지만         
문제는 문제가 발생할 수 있다는 가능성이 늘 있다는 것이다.              
참고로 위 같은 상황을 우리는 **RaceCondition이라 부른다.**

<br><br>
## 동시성 방어 원칙
지금부터 동시성 코드가 일으키는 문제로부터 `시스템을 방어하는 원칙과 기술을 소개한다.`

#### 단일 책임 원칙 (SRP)
SRP는 주어진 메서드/클래스/컴포넌트를 **변경할 이유가 하나여야 한다는 원칙이다.**   
동시성은 복잡성 하나만으로도 따로 분리할 이유가 충분하다.       
즉, `동시성과 관련된 코드는 따로 분리해야한다.`      

그런데 불행히도 동시성과 관련이 없는 코드에 동시성을 곧바로 구현하는 사례가 너무도 흔하다.   
동시성을 구현할 때는 다음 몇 가지를 고려해야한다.
* 동시성 코드는 독자적인 '개발', '변경', '조율주기가 있다.
* 동시성 코드에는 독자적인 난관이 있으며 다른 코드에서 겪는 난관에 비해 훨씬 어렵다.
* 잘못 구현한 동시성 코드는 별의별 방식으로 실패한다.           
  주변에 있는 다른 코드가 발목을 잡지 않더라도 동시성 하나만으로도 충분히 어렵다.

> 권장사항 : 동시성 코드는 다른 코드와 분리해라. → POJO

#### corollary : 자료 범위를 제한하라
다중 스레드에서는 **RaceCondition** 이라는 문제로 예상치 못한 결과를 내놓는다.

이를 해결하는 방안으로 `임계영역을 synchronized 키워드로 보호`하라고 권장한다.       
그리고 또한, 이런 `임계영역의 수를 줄이도록 엄청나게 노력해야 한다.`                         
공유 자료를 수정하는 위치가 많을수록 아래와 같은 가능성도 커지기 때문이다.

* 보호할 임계영역을 깜빡하고 빼먹을 수 있다.        
  그래서 자료를 수정하는 모든 코드를 망가뜨릴 수 있다.
* 모든 임계영역을 올바로 보호했는지 확인하느라 똑같은 노력과 수고를 반복한다.
* 그렇지 않아도 찾아내기 어려운 버그가 더욱 찾기 어려워진다.

> 권장사항 : 자료를 캡슐화하라, 공유한 자원을 최대한 줄여라.

#### corollary : 자료 사본을 사용하라
공유 자료를 줄이려면 처음부터 공유하지 않는 방법이 제일 좋다.                      
어떤 경우에는 `객체를 복사해 읽기 전용으로 사용하는 방법이 가능하다.`                     
어떤 경우에는 각 스레드가 객체를 복사해 사용한 후                
`한 스레드가 해당 사본에서 결과를 가져오는 경우도 있다.`

공유 객체를 피하는 방법이 있다면 코드가 문제를 일으킬 가능성도 아주 낮아진다.            
물론, 객체를 복사하는 시간과 부하가 걱정스러울 수도 있다.       
그렇지만, 대부분의 경우 공유 객체를 피하는 편이 복사 비용이 드는것 보다 낫다.

그리고, 사본으로 동기화를 피할 수 있는 방법이 있다면
내부 잠금을 없애 절약한 수행 시간이         
`사본 생성과 가비지 콜렉션에 드는 부하를 상쇄할 가능성이 크다.`

#### corollary : 스레드는 가능한 독립적으로 구현하라
다른 스레드와의 의존성이 없는 스레드를 구현하도록 해야한다.

각 스레드는 클라이언트 요청 하나를 처리한다.                      
모든 정보는 공유 출처에서 가져오며 `로컬 변수에 저장한다.`                    
그러면 각 스레드는 `동기화가 필요 없으므로 자신의 영역에서만 돌아갈 수 있다.`

HttpServlet 클래스에 파생된 클래스들은              
모든 정보를 doGet(), doPost()와 같은 `메서드의 매개변수로 받는다.`         
그래서 각 Servlet은 각자의 영역에서만 있는 것처럼 동작한다.          
또한, `지역 변수를 사용하는 한 동기화 문제는 발생하지 않게 된다.`        
물론 대부분의 Servlet들은 데이터베이스 연결과 같은 공유 자원이 필요하긴 하다.

> 권장사항 : 독립적인 스레드로 만들고자 노력해야하며 가능하다면 다른 프로세서에서 돌려도 괜찮을 정도로 `자료를 독립적인 단위로 분할하라.`

<br><br>
## 라이브러리를 이해하라
자바 5이상에서 스레드 코드를 구현한다면 다음을 고려해야한다.

* 스레드 환경에 안전한 컬렉션(concurrent)을 사용한다.
* 서로 무관한 작업을 수행할 때는 `executor` 프레임워크를 사용한다.
* 가능하다면 스레드가 차단되지 않는 방법을 사용한다.
* 일부 클래스 라이브러리는 스레드에 안전하지 못하다.

#### 스레드 환경에 안전한 컬렉션
`java.util.concurrent` 패키지가 제공하는 클래스는      
다중 스레드 환경에서 사용해도 안전하며 성능도 좋다.

실제로 `ConcurrentHashMap` 같은 경우는 거의 모든 `HashMap`보다 빠르다.        
동시 읽기/쓰기를 지원하며,    
자주 사용하는 복합 연산을 다중 스레드상에서 안전하게 만든 메서드로 제공한다.

|클래스|설명|
|-----|----|   
|ReentrantLock|한 메서드에서 잠그고 다른 메서드에서 푸는 락(lock)이다.|         
|Semaphore|전형적인 세마포다. 개수(count)가 있는 락이다.
|CountDownLatch|지정한 수만큼 이벤트가 발생하고나서야 대기중인 스레드를 모두 해제하는 락이다.<br>모든 스레드에게 동시에 공평하게 시작할 기회를 준다.  |  

> 권장사항 : 언어가 제공하는 클래스를 검토하라, <br>자바에서는 `java.util.concurrent`, `java.util.concurrent.atomic`, `java.util.concurrent.locks`를 익혀라.

<br><br>
## 실행 모델을 이해하라
|용어|설명|
|---|----|
|BoundResource(한정된자원)|다중 스레드 환경에서 사용하는 자원으로, 크기나 숫자가 제한적이다.<br>데이터베이스 연결, 길이가 일정한 읽기/쓰기 버퍼등이 있다.|   
|MutualExclusion(상호 배제)|한 번에 한 스레드만 공유 자료나 공유 자원을 사용할 수 있는 경우를 가리킨다.|   
|Starvation(기아)|한 스레드나 여러 스레드가 굉장히 오랫동안 혹은 영원히 자원을 기다린다.<br>예를 들어, 실행 시간이 짧은 스레드에게 높은 우선순위를 준다면<br>그리고 실행 시간이 짧은 스레드가 지속적으로 이어질 경우,<br>실행 시간이 긴 스레드는 기아 상태에 빠진다.|     
|Deadlock(교착상태)|여러 스레드가 서로가 가지고 있는 자원을 사용하기 위해 무한정 기다리는 상태|       
|Livelock(라이브락)|락을 거는 단계에서 각 스레드가 서로를 방해한다<br>스레드는 계속 진행하려 하지만, 공명(resonance)으로 인해,<br>굉장히 오랫동안 혹은 영원히 진행하지 못한다.|   

#### Producer-Consumer
하나 이상의 `생산자 스레드`가 `정보를 생성해` 버퍼나 `대기열에 넣는다.`                
하나 이상 `소비자 스레드`가 `대기열에서 정보를 가져와 사용`한다.                  
생산자 스레드와 소비자 스레드가 사용하는 `대기열은 한정된 자원`이다.

생산자 스레드는 대기열의 빈공간이 있지 않으면 기다린다.           
소비자 스레드는 대기열이 비어있으면 채워질 때 까지 기다린다.

그렇기에 대기열을 올바르게 사용하고자         
`생산자 스레드와 대기열 스레드는 서로에게 시그널을 보낸다.`               
하지만 잘못하면 생산자 스레드와 소비자 스레드가 둘 다 진행 가능함에도       
동시에 서로의 시그널을 기다리는 현상이 발생할 수 있다.


#### Readers-Writers
읽기 스레드는 공유 자원을 사용한다.              
쓰기 스레드는 공유 자원을 갱신한다.                        
`이런 경우 처리량이 문제의 핵심이 된다.`

여기서 말하는 '처리량'은 읽고 갱신하는 모든 작업을 끝낸 시간이라 봐도 된다.         
처리율이 가장 높게 나오려면 읽기 스레드와 쓰기 스레드는 막힘없이 진행되야 한다.        
하지만, 한쪽 스레드가 자원을 점유하면 다른 쪽 스레드는 자원에 접근하지 못한다.

따라서 읽기 스레드의 요구와 쓰기 스레드의 요구를 적절히 만족시켜    
처리량도 적당히 높이고 기아도 방지하는 해법이 필요하다.

대개는 쓰기 스레드가 버퍼를 오랫동안 점유하는 바람에      
여러 읽기 스레드가 버퍼를 기다리느라 처리량이 떨어진다.

처리량을 높이고자 한다면            
읽기 스레드가 버퍼 접근에 대한 우선권을 가지면 된다.       
하지만, 쓰기 스레드가 기아 상태에 빠지게 되면서      
공유 자원은 갱신되지 않은 정체된 정보로 가득차게 된다.

그렇기에 우리는 읽기-쓰기 이 둘 사이의 균형을 맞추어      
동시 갱신 문제를 방지하는 것을 주안점으로 두어야 한다.


#### Dining Philosophers
```
원탁을 둘러싼 여러 명의 철학자들이 있다.     
각 철학자의 왼쪽에 포크가 놓여 있으며 테이블의 중앙에 큰 스파게티 한 그릇이 놓여 있다.    
그들은 배가 고파지기 전까지 각자 생각을 하며 시간을 보낸다.     
배가 고파지면 그들은 자신의 양쪽에 놓여 있는 포크 2개를 잡고 스파게티를 먹는다.    
철학자는 포크 2개가 있어야만 스파게티를 먹을 수 있다.      
그렇지 않다면 옆 사람이 포크를 다 사용하기 전까지 기다려야 한다.     
스파게티를 먹은 철학자는 다시 배가 고파질 때까지 포크를 놓고 있게 된다.       
```     

위 상황에서 철학자를 스레드로, 포크를 공유 자원으로 바꾸게 되면      
이는 자원을 놓고 경쟁하는 프로세스와 비슷한 상황이 된다.

잘 설계되지 않은 시스템은       
`deadlock`, `livelock`, `처리량 문제`, `효율성 저하` 문제에 맞닥뜨리기 쉽다.       
일상에서 접하는 대다수 다중 스레드 문제는 위 세범주 중에 하나에 속한다.

> 권장 사항 : 위에서 설명한 기본 알고리즘과 각 해법을 이해하라.

<br><br>
## 동기화하는 메서드 사이에 존재하는 의존성을 이해하라
동기화하는 `메서드 사이에 의존성이 존재`하면        
동시성 코드에 찾아내기 어려운 `버그가 생긴다.`

자바 언어는 개별 메서드를 보호하는 `synchronized`라는 개념을 지원한다.      
하지만, `공유 클래스 하나에 동기화된 메서드가 여럿이라면 구현이 올바른지 다시 한 번 확인해보도록 하자`

> 권장사항 : 공유 객체 하나에는 메서드 하나만 사용한다.

공유 객체 하나에 여러 메서드가 필요한 상황도 생긴다.   
그럴 때는 다음 3가지 방법을 고려해야한다.

1. `클라이언트에서 잠금` :     
    클라이언트에서 첫 번째 메서드를 호출하기 전에 서버를 잠근다.       
    마지막 메서드를 호출할 때까지 잠금을 유지한다.
2. `서버에서 잠금` :    
    서버에서는 하나의 트랜젝션을 구현하고             
    클라이언트는 이 메서드를 호출한다.
3. `연결 서버` :      
    잠금을 수행하는 중간 단계를 생성한다.      
    `서버에서 잠금 방식`과 유사하지만 원래 서버는 변경하지 않는다.


<br><br>

## 동기화하는 부분을 작게 만들어라
자바에서 `synchronized` 키워드를 사용하면 락을 설정한다.           
같은 락으로 감싼 임계영역은 한 번에 한 스레드만 실행이 가능하다.            
`다만, 락은 스레드를 지연시키고 부하를 가중시킨다.`           
그러므로 여기저기서 `synchronized`문을 남발하는 코드는 바람직하지 않다.

하지만, `성능적 이슈가 있더라도 임계영역은 반드시 보호해야 한다.`               
양쪽의 의견을 맞추기 위해 코드를 짤 때는 `임계영역 수를 최대한 줄여야한다.`

임계영역 개수를 줄인답시고 거대한 임계영역 하나로 구현하는 프로그래머도 있다.     
하지만, 필요 이상으로 임계영역 크기를 키우면        
스레드간에 경쟁이 늘어나고 프로그램 성능이 떨어진다는점을 알아두자

> 권장 사항 : 동기화하는 부분을 최대한 작게 만들어라.     

<br><br>
## 올바른 종료 코드는 구현하기 어렵다
영구적으로 돌아가는 시스템을 구현하는 방법과        
잠시 돌다 깔끔하게 종료하는 시스템을 구현하는 방법은 다르다.

깔끔하게 종료하는 코드는 올바르게 구현하기 어렵다.           
가장 흔히 발생하는 문제가 데드락이기 때문이다.             
스레드가 절대 오지 않을 시그널을 기다리면서 깔끔하게 종료하지 못한다.

이는, 부모/자식 스레드간의 관계에서도 나타난다.

**Case one**   
모든 자식 스레드가 완료되면 종료되는 부모 스레드가 있다고 가정한다.                     
자식 스레드에서 데드락이 걸린다면, 부모 스레드와 시스템은 영원히 종료하지 못한다.

**Case two**           
사용자에게서 종료하라는 지시를 받았다고 가정한다.            
부모 스레드는 모든 자식 스레드에게 작업을 멈추고 종료하라는 시그널을 전달한다.             
- 그런데, 자식 스레드 중 2개가 생산자/소비자 관계라면?
- 생산자 스레드는 재빨리 종료되었는데, `소비자 스레드가 생산자 스레드에서 오는 메시지를 기다린다면?`

생산자에게서 메시지를 기다리는 소비자 스레드는       
`blocked 상태`에 있으므로 종료하라는 시그널을 못받는다.

그러면 `소비자 스레드는 생산자 스레드를 영원히 기다리고, 부모 스레드는 자식 스레드를 영원히 기다린다.`

실제로 이와 같은 상황은 종종 발생한다.

깔끔하게 종료하는 다중 스레드 코드를 짜야 한다면           
시간을 투자해 올바르게 구현하기를 바란다.

> 권장사항 : 
<br>종료 코드를 개발 초기부터 동작하도록 고민하고 구현하라. <br>생각보다 오래 걸리고 어려우므로 이미 나온 알고리즘을 검토하자.          

<br><br>
## 스레드 코드 테스트 하기
많은 테스트를 진행하더라도 100%의 정확성을 보장하지는 못한다.              
그렇기 때문에 올바른 코드라고 증명하기는 현실적으로 불가능하다.                              
그럼에도 테스트는 위험을 최대한 낮추어주는 고마운 녀석이다.

하지만, 단일 스레드가 아닌 멀티 스레드로 넘어오면      
테스트를 진행하는 방법도 기존보다 많이 복잡해진다.

> 권장사항 :         
문제를 일부러 발생시키는 테스트 케이스를 작성하라.                       
프로그램 설정과 시스템 설정과 `부하를 바꿔가며 자주 돌려라.`              
테스트가 실패하면 이제 원인을 추적하라.           
다시 돌렸더니 통과하더라는 이유로 그냥 넘어가면 절대로 안된다.

`테스트를 진행하는 방법도 기존보다 많이 복잡해진다.`              
라고 말했던 것은 `고려할 사항이 아주 많다는 뜻이다.`          
아래에 몇가지 구체적인 지침을제시한다.

* 말이 안 되는 실패는 `잠정적인 스레드 문제로 취급`한다.
* 다중 스레드를 고려하지 않은 `순차 코드부터 제대로 돌게 만들자.`
* 다양한 환경에 쉽게 끼워 넣을 수 있도록 코드를 구현해라.
* 상황에 맞춰 조정할 수 있게 코드를 구현해라.
* 프로세서 수보다 많은 스레드를 돌려보라.
* 기존과는 다른 플랫폼에서 돌려보라.
* 코드에 보조 코드를 넣어 돌려라, `강제로 실패를 일으키게 해라.`

#### 말이 안 되는 실패는 잠정적인 스레드 문제로 취급하라
이런 경우, 스레드가 다른 코드와 교류하는 방식을 직관적으로 이해하기 힘들다.         
스레드 코드에 잠입한 버그는 수천, 아니 수백만 번에 한 번 씩 드러나기도 한다.   
즉, 실패를 재현하기가 아주 어렵다.

그래서 많은 개발자가 단순한 일회성 문제로 치부하고 무시한다.               
하지만, 일회성 문제를 계속 무시한다면 잘못된 코드 위에 코드가 계속 쌓인다.

> 권장사항 :         
시스템 실패를 '일회성'이라 치부하지 마라                
이후에는 발생하지 않도록 최대한 해결하고자 노력해야 한다.

#### 다중 스레드를 고려하지 않은 순차 코드부터 제대로 돌게 만들자
`스레드와 연관되지 않은 멤버들로 구성된 POJO를 만들라는 이야기`이다.            
스레드와 연관된 코드가 존재하지 않기에 다양한 상황에서 유용하게 사용할 수 있으며        
클래스는 본연의 목적과 동작만 수행하는 `잘 캡슐화된` 클래스가 될 것이다.

`POJO는 독립적인 클래스로, 다중 스레드 환경이 아닌 곳에서도 테스트할 수 있다.`       
시스템은 가능한 한 스레드와 관련이 없는 POJO로 작성하는 것이 좋다.

> 권장사항 :       
스레드 관련 버그와 그렇지 않은 버그를 동시에 잡으려 하지 마라.       
작성한 코드가 스레드 밖에서 잘 작동하는지 먼저 체크하라.

#### 다중 스레드를 쓰는 코드 부분을 다양한 환경에 쉽게 기워 넣을 수 있게 스레드 코드를 구현하라

* 단일 스레드로 실행하거나, 다중 스레로 실행하거나, 실행 중 스레드 수 바꿔본다.
* 스레드 코드를 실제 환경과 테스트 환경에서 돌려본다.
* 테스트 코드를 빨리, 천천히, 다양한 속도로 돌려본다.
* 반복 테스트가 가능하도록 테스트 케이스를 작성한다.

> 권장사항 : 다양한 환경에서 사용 및 실행될 수 있는 스레드 코드를 구현해라.

#### 다중 스레드를 쓰는 코드 부분을 상황에 맞게 조율할 수 있게 작성하라
적절한 스레드 개수를 파악하려면 상당한 시행착오가 필요하다.        
처음부터 다양한 설정으로 프로그램의 성능 측정 방법을 찾는 것을 권장한다.

스레드 개수를 조율하기 쉽게 코드를 구현한다.         
`프로그램이 돌아가는 도중에 스레드 개수를 변경하는 방법도 고려한다.`         
프로그램 처리율과 효율에 따라 스스로 스레드 개수를 조율하는 코드도 고민하자.

#### 프로세서 수보다 많은 스레드를 돌려보라
시스템이 스레드를 스와핑할 때도 문제가 발생한다.         
스와핑을 일으키려면 프로세서 수보다 많은 스레드를 돌려야 한다.                  
스와핑이 잦을수록 임계영역을 빼먹은 코드나 데드락을 일으키는 코드를 찾기 쉬워진다.

#### 다른 플랫폼에서 돌려보라
실패코드를 작성했지만, OS에 따라서 실패율이 다를 수 있다.                
그 이유는 단지, OS마다 스레드를 처리하는 정책이 달랐기 때문이다.

다중 스레드 코드는 플랫폼에 따라 다르게 돌아간다.          
따라서 일단, 코드가 돌아갈 수 있는 플랫폼 전부에서 테스트를 수행하자.

> 권장사항 : 처음부터 그리고 자주 타겟이 되는 모든 플랫폼에서 코드를 돌려라.

#### 코드에 보조 코드(instrument)를 넣어 돌려라. 강제로 실패를 일으키게 해보라
흔히 스레드 코드는 오류를 찾기가 쉽지 않다.                  
간단한 테스트로는 버그가 드러나지 않기 때문이다.               
더 정확히 말하면, 처음에는 아무 문제 없다가       
몇 시간, 며칠 혹은 몇 주가 지나서야 한 번씩 모습을 드러낸다.

스레드 버그가 재현이 어려운 이유는 아주 소수의 경우에만 실패하기 때문이다.   
그래서 버그를 발견하고 찾아내기가 아주 어렵다.

🤔 발생 가능성이 낮은 버그를 자주 발생시킬 방법은 없을까?         
💡 `보조 코드를 추가해 코드가 실행되는 흐름을 바꿔준다! `    
아래와 같은 메서드를 이용해서 코드를 다양한 순서로 실행하자

* Object.wait()
* Object.sleep()
* Object.yield()
* Object.priority()

각 메서드는 스레드가 실행되는 순서에 영향을 미친다.         
따라서 버그가 드러날 가능성도 높아진다.        
잘못된 코드라면 가능한 초반에 그리고 가능한 자주 실패하는 편이 좋다.

코드에 보조 코드를 추가하는 방법은 2가지이다.

1. 직접 구현하기
2. 자동화

###### 직접 구현하기
코드에다 직접 `wait()`, `sleep()`, `yeild()`, `priority()` 함수를 추가한다.          
특별히 까다로운 코드를 테스트할 때 적합하다.

```java
public synchronized String nextUrlOrNull() {
    if(hasNext()) {
        String url = urlGenerator.next();
        Thread.yield(); // 테스트를 위해 추가되었다.    
        updateHasNext();
        return url;
    }
    return null;
}
```
`yield()`는 `yield()`를 호출한 스레드를 실행 대기 상태로 만들고               
동일한 우선순위 또는 높은 우선순위를 갖는 다른 스레드에게 실행 기회를 준다.           
즉, 다른 스레드에게 흐름을 양보하는 메서드이다.

만약 위 코드에서 문제가 발생한다면 이는 `yield()`를 추가해 생긴 문제가 아니다.      
`yield()`는 **이미 존재하던 문제를 명백히 만든것 뿐이다.**

하지만 이같이 보조 코드를 넣는 방식은 몇가지 문제점이 있다.

* 보조 코드를 삽입할 적정 위치를 직접 찾아야 한다.
* 어떤 함수를 어디서 호출해야 적당한지 어떻게 알까?
* 배포 환경에 보조 코드를 그대로 남겨두면 프로그램 성능이 떨어진다.
* 보조 코드 또한 무작위적이다.

보조 코드를 삽입해도 오류가 드러날지도, 드러나지 않을지도 모른다.      
오히려, 드러나지 않을 확률이 더 높다.

이를 해결하기 위해서는

* 배포 환경이 아니라 테스트 환경에서 보조 코드를 실행할 방법이 필요하다.
* 실행할 때마다 설정을 바꿔줄 방법도 필요하다.

그리고 위 2가지를 실현하기 위해서는 POJO 클래스를 작성해야한다.    
그래야 전적으로 오류가 드러날 확률이 높아진다.

또한, POJO 클래스와 스레드를 제어하는 클래스로 프로그램을 분할하면                 
앞서 문제로 제기되었던, 보조 코드를 추가할 위치를 찾기가 쉬워진다.                    
더불어 여러 상황에서 `sleep()`, `yield()` 등으로 POJO를 호출하게 하여     
다양한 테스트 지그(Jig)를 구현할 수 있다.

###### 자동화
보조 코드를 자동으로 추가하려면 `AOF`, `CGLIB`, `ASM`등과 같은 도구를 사용하면 된다.

```java
public class ThreadJigglePoint {
    public static void jiggle() { }
}
```      
```java
public synchronized String nextUrlOrNull() {
    if(hasNext()) {
        ThreadJiglePoint.jiggle();
        String url = urlGenerator.next();
        ThreadJiglePoint.jiggle();
        updateHasNext();
        ThreadJiglePoint.jiggle();
        return url;
    }
    return null;
} 
```    
`nextUrlOrNull()`의 다양한 위치에 `ThreadJiglePoint.jiggle();`를 추가시켰다.

`ThreadJiglePoint.jiggle();` 호출은      
무작위로 `sleep()`이나 `yield()`를 호출하거나      
때로는 아무것도 호출하지 않는다.     
그리고 이는 스레드의 흐름을 여러 방향으로 흔들기 위함이다.

`ThreadJigglePoint` 클래스를 2가지 방법으로 구현하면 편리하다          
하나는 `jinggle()` 메서드를 비워두고 배포 환경에서 사용한다.        
다른 하나는 무작위로 `nop`, `sleep`이나 `yield` 등을 테스트 환경에서 수행한다.

두번째로 말한 방법으로 테스트를 수천 번 실행하면 스레드 오류가 드러날지 모른다.        
코드가 수천 번에 이르는 테스트를 통과한다면 나름대로 할 만큼 했다고 말해도 된다.

코드를 흔드는(Jiggle) 이유는 스레드를 매번 다른 순서로 실행하기 위해서다.             
좋은 테스트 케이스와 흔들기 기법은 오류가 드러날 확률을 크게 높여준다.

> 권장사항 : 흔들기 기법을 사용해 오류를 찾아내라.     

<br><br>
## 결론
- 다중 스레드 코드는 올바로 구현하기 어렵다.
- `다중 스레드 코드를 작성한다면` 각별히 깨끗하게 코드를 짜야 한다.
  - 가장 먼저, `SRP 준수한다.`
    - POJO를 사용해 스레드를 아는 코드와 스레드를 모르는 코드를 분리한다.
    - 스레드 코드를 테스트 할 때는 전적으로 스레드만 테스트 한다. 
    <br>= 스레드 코드는 최대한 집약되고 작아야 한다는 의미다.
- `동시성 오류를 일으키는 잠정적인 원인을 철저히 이해한다.`
  - ex
    - 여러 스레드가 공유 자료를 조작하거나 자원 풀을 공유할 때 동시성 오류가 발생한다.
    - 루프 반복을 끝내거나 프로그램을 깔끔하게 종료하는 등 경계 조건의 경우가 까다로우므로 특히 주의한다.
- `사용하는 라이브러리와 기본 알고리즘을 이해한다.`
  - 특정 라이브러리 기능이 기본 알고리즘과 유사한 어떤 문제를 어떻게 해결하는지 파악한다.
- `보호할 코드 영역을 찾아내는 방법과 특정 코드 영역을 잠그는 방법을 이해한다.`
  - 쓸데 없는 구간을 잠그지 마라. 
  - 잠긴 구간에서 또 다른 잠긴 구간을 부르는 것을 기피하라. <br>이는 "공유하는 정보와 공유하지 않는 정보"에 대한 깊은 이해를 요구한다. 
  - 공유 객체의 갯수와 공유 영역을 최소한으로 줄여라. 
  - 클라이언트가 공유 객체의 상태(잠금 등)를 관리하는 대신 공유 객체의 디자인을 변경하라.
- `스레드 관련 코드는 여러 설정, 환경에서 반복적이고 지속적으로 수행해 보라.`
  - 문제는 돌연 발생할 것이다. 초반에 드러나지 않는 문제들은 보통 "한번만 발생하는" 문제로 치부된다. 
  - 이러한 일회성 문제들은 보통 시스템에 부하가 걸린 때나 뜬금없이 발생한다. 
  - 그러므로 스레드 관련 코드는 여러 설정, 많은 플랫폼에서 반복적이고 지속적으로 테스트를 수행해 보라.
- 당신의 코드를 `시간을 들여 보조 코드를 추가`하게 되면 오류를 찾을 가능성이 크게 높아질 것이다. 
  - 직접 코드를 작성할 수도 있고 자동화 툴을 사용할 수도 있다. 
  - 스레드 코드는 출시하기 전까지 최대한 오래 테스트 해야할 것이다.
- 깔끔한 접근 방식을 사용한다면, 제대로 된 코드를 만들어낼 가능성은 급격히 올라갈 것이다.

<br><br>

## Findings
#### 여러 스레드를 동시에 돌리는 이유
1. `구조적 이점이 크기 때문에` 
   - 동시성은 무엇(What)과 언제(When)을 분리하는 전략이다.
   - 구조적인 관점에서 프로그램은 거대한 루프 하나가 아니라 작은 협력 프로그램 여럿으로 보이기에, 시스템을 이해하기가 쉽고 문제를 분리하기도 쉽다.
2. `응답 시간과 작업 처리량 개선`

#### 동시성 프로그래밍의 미신과 오해
- 동시성은 때로 성능을 높여준다 (O)
  - 대기시간이 아주 길어 여러 스레드가 프로세서를 공유할 수 있거나, 여러 프로세서가 동시에 처리할 독립적인 계산이 충분히 많은 경우에만 성능이 높아진다.
- 동시성을 구현하면 설계를 바꿔야 한다(O)
  - 단일 스레스 시스템과 다중 스레드 시스템은 설계가 판이하게 다르다.
  - ‘무엇’과 ‘언제’를 분리하면 시스템의 구조가 크게 달라진다.
- 컨테이너를 사용해도 동시성을 이해해야 한다(O)
  - 어플리케이션이 컨테이너를 통해 멀티 쓰레드를 사용하는 것이기 때문에 컨테이너의 동작을 이해해야 한다.
  - 동시 수정, 데드락 같은 문제를 피할 수 있는지를 알아야 한다.

#### 어려움에 대처하고 깨끗한 코드를 작성하는 방법
- 단일 책임 원칙(SRP) 설계
  - 동시성 관련 코드는 다른 코드와 분리하라.
    - 동시성 코드는 독자적인 개발, 변경, 조율 주기가 있다.
    - 동시성 코드에는 독자적인 난관이 있다. 다른 코드에서 겪는 난관과 다르며 훨씬 어렵다.
    - 잘못 구현한 동시성 코드는 별의별 방식으로 실패한다. 주변에 있는 다른 코드가 발복을 잡지 않더라도 동시성 하나만으로도 충분히 어렵다.
- 자료 범위를 제한하라
  - 공유 자료를 최대한 줄여라
    - 동시 수정 문제를 피하기 위해 객체를 사용하는 코드 내 임계영역을 synchronized 키워드로 보호하라. 
    - 보호할 임계영역을 빼먹거나, 모든 임계영역을 보호했는지 확인해야하므로, 임계 영역의 수를 최소화 해야 한다.
- 자료 사본을 사용하라
  - 공유 자료를 줄이려면, 최대한 공유하지 않는 방법이 제일 좋다. 
    - 객체를 복사해 읽기 전용으로 사용한다. 
    - 각 스레드가 객체를 복사해 사용한 후 한 스레드가 해당 사본에서 결과를 가져온다. 
    - 사본을 사용하는 방식으로 내부 잠금을 없애 수행 시간을 절약하는 것이 사본 생성과 가비지 컬렉션에 드는 부하를 상쇄할 가능성이 크다.
- Thread는 가능한 독립적으로 구현하라 
  - 다른 스레드와 자료를 공유하지 않는다. 
    - 서블릿처럼 각 Thread는 클라이언트 요청 하나를 처리한다. 
    - 모든 정보는 비공유 출처(client의 request)에서 가져오며 로컬 변수에 저장한다. 
    - 각 서블릿은 마치 자신이 독자적인 시스템에서 동작하는 양 요청을 처리한다.
- 라이브러리를 이해하라
  - java.util.concurrent 패키지를 익혀라
    - Thread Safe 한 컬렉션을 사용한다.
      - ConcurrentHashMap, AtomicLong
    - 서로 무관한 작업을 수행할 때는 executor 프레임워크를 사용한다.
    - 가능하다면 Thread가 Blocking되지 않는 방법을 사용한다.
- 동기화하는 메서드 사이에 존재하는 의존성을 이해하라 
  - 공유 객체 하나에는 메서드 하나만 사용하라 
    - 클라이언트에서 잠금 - 클라이언트에서 첫 번째 메서드를 호출하기 전에 서버를 잠근다. 마지막 메서드를 호출할 때까지 잠금을 유지한다. 
    - 서버에서 잠금 - 서버에다 <서버를 잠그고 모든 메서드를 호출한 후 잠금을 해제하는> 메서드를 구현한다. 클라이언트는 이 메서드를 호출하기만 하면 된다. 
    - 연결(Adapter)서버 - 잠금을 수행하는 중간 단계를 생성한다. ‘서버에서 잠금’방식과 유사하지만 원래 서버는 변경하지 않는다.

#### 동시성을 테스트하는 방법과 문제점
- 동시성 코드를 테스트 해야 한다.
  - 테스트를 했다고 동시성 코드가 100% 올바르다고 증명하기는 불가능하다.
  - 하지만 충분한 테스트는 위험을 낮춘다.
    - 문제를 노출하는 테스트 케이스를 작성하라.
    - 프로그램의 설정과 시스템 설정과 부하를 바꿔가며 자주 돌려라.
    - 테스트가 실패하면 원인을 추적하라.
    - 다시 돌렸더니 통과한다는 이유로 그냥 넘어가면 절대로 안된다.
- 코드에 보조 코드를 넣어 돌려라
  - 드물게 발생하는 오류를 자주 발생시키도록 보조 코드를 추가한다
    - 코드에 wait(), sleep(), yield(), priority() 함수를 추가해 직접 구현한다.
    - 보조코드를 넣어주는 도구를 사용해 테스트한다.
      - 다양한 위치에 ThreadJigglePoint.jiggle()을 추가해 무작위로 sleep(), yield()가 호출되도록 한다.
    - 테스트 환경에서 보조 코드를 돌려본다.
- 동시성 코드를 실제 환경이나 테스트 환경에서 돌려본다
  - 다양한 요청과 상황에 동시성 코드가 정상적으로 동작하는지 확인한다
    - 배포하기 전에 테스트 환경에서 충분히 오랜시간 검증한다.
    - 동시성 코드를 배포한 후에 모니터링을 통해 문제가 발생하는지 지켜본다.

<br><br>
## Feelings
- 여러 스레드를 동시에 돌리는 이유는 작업 처리량을 개선하기 위함만 있는 줄 알았었는데, 구조적인 이점도 있다는 사실을 알게 되었다. 
- 동기화는 모든 상황에서 성능을 높여주는 것이 아니다.<br> 대기시간이 아주 길어 여러 스레드가 프로세서를 공유할 수 있거나, 여러 프로세서가 동시에 처리할 독립적인 계산이 충분히 많은 경우에 성능이 높아지는 것을 알게 되었다.
- 내부 잠금을 없애 절약한 수행 시간이 사본 생성과 가비지 콜렉션에 드는 부하를 상쇄할 가능성이 크다고 한다. 내부 잠금이 생각보다 많은 부하를 일으킨다는 것을 알게 되었다.
- Java의 경우, 멀티 스레드 관련 라이브러리가 많다. 관련 라이브러리를 찾아보고, Java의 동시성 관련 이슈는 없는지도 찾아보자.
- 동시성 코드는 관리하기가 어렵고 버그가 찾기 어렵다는 것을 실감하게 되었다.
- 동시성 코드는 올바르게 구현하기가 어려움으로, 충분한 테스트를 걸쳐보자. 다양한 환경에서나 테스트 환경에서 돌려보며 충분히 오랜시간 검증해봐야겠다.
- 동시성 코드는 테스트를 어떻게 하는지 궁금했었는데, 동시성을 테스트 하는 방법을 알게 되었다.
