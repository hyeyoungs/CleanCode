# 02 의미 있는 이름
> 의미있게 이름 짓는 규칙

## 의도를 분명히 밝혀라
변수와 함수와 클래스 이름은 존재 이유, 기능, 사용 방법이 드러나야 한다. 따로 주석이 필요 없을 정도로.
#### Example1
- Bad
```java
int d; // 경과 시간(단위 날짜)
```
- Good
  <br>의도가 들어나는 이름을 사용하면 코드 이해와 변경이 쉬워진다.
```java
int elapsedTimeInDays;
int daysSinceCreation;
int daysSinceModification;
int fileAgeInDays;
```
#### Example2
- Bad
  <br>코드 맥락이 코드 자체에 명시적으로 드러나지 않는다.
```java
public List<int[]> getThem(){
    List<int[]> list1 = new ArrayList<int[]>();
    for (int[] x : theList)
        if (x[0] == 4)
                list1. add(x);
    return list1;
}
```
- Good
  <br>각 개념에 이름만 붙여도 코드가 상당히 나아진다.
```java
public List<int[]> getFlaggedCells() {
    List<int[]> flaggedCells = new ArrayList<int[]>;
    for (int[] cell : gameBoard)
        if (cell[Status_VALUE] == FLAGGED)
            flaggedCells.add(cell);
    return flaggedCells;
}
```
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;명시적인 간단한 클래스를 만들고, 명시적인 함수를 사용해주니 이해하기 쉬워졌다.
```java
public List<int[]> getFlaggedCells() {
    List<Cell> flaggedCells = new ArrayList<Cell>();
    for (Cell cell : gameBoard)
        if (cell.isFlagged())
            flaggedCells.add(cell);
    return flaggedCells;
}
```

## 그릇된 정보를 피하라
### 중의적으로 해석될 수 있는 이름을 사용해서는 안 된다.
#### Example1
- Bad
  hp, aix, sco는 변수 이름으로 적합하지 않다. 유닉스 플랫폼을 가리키는 이름이기 때문이다.
```java
int hp;
int aix;
int sco;
```
- Good
```java
int hypotenuse;
```
#### Example2
- Bad
  <br>개발자에게는 특수한 의미를 가지는 단어(List 등)는 실제 컨테이너가 List가 아닌 이상 accountList와 같이 변수명에 붙이지 말자.
```java
int accountList;
```
- Good
  <br>차라리 accountGroup, bunchOfAccounts, accounts등으로 네이밍 하자.
```java
int accoutGroup;
int bunchOfAccounts;
```
### 서로 흡사한 이름을 사용하면 안된다.
#### Example
- Bad
```java
int XYZControllerForEfficientHandlingOfStrings;
int XYZControllerForEfficientStorageOfStrings;
```

## 의미 있게 구분하라
### 연속적인 숫자를 덧붙인 이름(a1, a2, …, aN)은 저자의 의도가 드러나지 않는다.
- Bad
```java
public static void copyChars(char a1[], char a2[]) {
    for(int i = 0; i < a1.length; i++) {
        a2[i] = a1[i];    
    }
}
```
- Good
```java
public static void copyChars(char source[], char destination[]) {
    for(int i = 0; i < source.length; i++) {
        destination[i] = source[i];
    }
}
```
### noise word(불용어) 역시 아무런 정보도 제공하지 못한다.
- 클래스 이름에 Info, Data와 같은 불용어를 붙이지 말자. 정확한 개념 구분이 되지 않는다.
<br>이들이 혼재할 경우 서로의 역할을 정확히 구분하기 어렵다.
  - Name VS NameString
  - getActiveAccount() VS getActiveAccounts() VS getActiveAccountInfo()
  - money VS moneyAmount
  - message VS theMessage
  <br>읽는 사람이 차이를 알도록 이름을 지어라.

## 발음하기 쉬운 이름을 사용하라
- Bad
```java
class DtaRcrd102 {
    private Date genymdhms;
    private Date modymdhms;
    private final String pszqint = "102";
    /* ... */
};
```
- Good
```java
// Good
class Customer {
    private Date generationTimestamp;
    private Date modificationTimestamp;
    private final String recordId = "102";
    /* ... */
};
```

## 검색하기 쉬운 이름을 사용하라
- 문자 하나를 사용하는 이름과 상수는 텍스트 코드에서 쉽게 눈에 띄지 않는다.
<br>긴 이름이 짧은 이름보다 좋다. 검색하기 쉬운 이름이 상수보다 좋다.
- 이름 길이는 변수의 범위에 비례해야 한다.

## 인코딩을 피하라
### 헝가리식 표기법
변수명에 해당 변수의 타입(String, Int 등)을 적지 말자.
- Bad
```java
PhoneNumber phoneString; 
// 타입이 바뀌어도 이름은 바뀌지 않는다!
```

### 멤버 변수 접두어
맴버 변수에 접두어를 붙이지 말자.
- Bad
```java
public Class Part {
    private String m_dsc; // 설명 문자열
    void setName(String name) {
        m_dsc = name;    
    }
}
```
- Good
```java
public Class Part {
    private String description;
    void setDescription(String description) {
        this.description = description;    
    }
}
```

### 인터페이스 클래스와 구현 클래스
인터페이스 클래스와 구현 클래스를 나눠야 한다면 구현 클래스의 이름에 정보를 인코딩하자.

|            | Interface class | Concrete(Implementation) class |
| ---------- | --------------- | ------------------------------ |
| Bad        | IShapeFactory   | ShapeFactory                   |
| Good       | ShapeFactory    | ShapeFactoryImp                |
| Good       | ShapeFactory    | CShapeFactory                  |


## 자신의 기억력을 자랑하지 마라
- 독자가 머리속으로 한번 더 생각해 변환해야 할만한 변수명을 쓰지 말라.
<br>(ex. URL에서 호스트와 프로토콜을 제외한 소문자 주소를 r이라는 변수로 네이밍하는 등)
- 똑똑한 프로그래머와 전문가 프로그래머를 나누는 기준 한가지는 Clarity(`명료함`)이다.
<br>전문가 프로그래머는 남들이 이해하는 코드를 내놓는다.

## 클래스 이름
- 명사 혹은 명사구를 사용하라. (`Customer`, `WikiPage`, `Account`, `AddressParser`)
<br>Manager, Processor, Data, Info와 같은 단어는 피하자.
- 동사는 사용하지 않는다.

## 메서드 이름
- 동사 혹은 동사구를 사용하라. (`postPayment`, `deletePayment`, `deletePage`, `save`)
- 접근자, 변경자, 조건자는 get, set, is로 시작하자.
```java 
string name = employee.getName();
customer.setName("mike");
if (paycheck.isPosted())...
```
- 생성자를 오버로드할 경우 정적 팩토리 메서드를 사용한다. 메서드는 인수를 설명하는 이름을 사용한다.
```java 
// 첫번째 보다 두 번째 방법이 더 좋다.  
Complex fulcrumPoint = new Complex(23.0);  
Complex fulcrumPoint = Complex.FromRealNumber(23.0);  
// 생성자 사용을 제한하려면 해당 생성자를 private로 선언한다.
```

## 기발한 이름을 피하라
특정 문화에서만 사용되는 재미있는 이름보다 의도를 분명히 표현하는 이름을 사용하라.
```java
HolyHandGrenade → DeleteItems 
whack() → kill()
```

## 한 개념에 한 단어를 사용하라
추상적인 개념 하나에 단어 하나를 사용하자.
```java
fetch, retrieve, get → 하나를 일관성 있게 사용하라.
controller, manager, driver → 하나를 일관성 있게 사용하라.
```

## 말장난을 하지 마라
한 단어를 두 가지 목적으로 사용하지 말자.
<br>한 개념에 한 단어를 사용하라는 규칙은 `매개변수와 반환값이 의미적으로 똑같다`면 문제가 없다.
<br>아래와 같은 경우에는 add를 append나 insert로 바꿔야 한다.

```java
public static String add(String message, String messageToAppend)  
public List<Element> add(Element element)  
```

## 해법 영역에서 가져온 이름을 사용하라
개발자라면 당연히 알고 있을 `JobQueue`, `AccountVisitor(Visitor pattern)`등을 사용하지 않을 이유는 없다. 
<br>전산용어, 알고리즘 이름, 패턴 이름, 수학 용어 등은 사용하자.

## 문제 영역에서 가져온 이름을 사용하라
적절한 프로그래머 용어가 없거나 문제영역과 관련이 깊은 용어의 경우 문제 영역 용어를 사용하자.

## 의미 있는 맥락을 추가하라
클래스, 함수, namespace등으로 감싸서 맥락(Context)을 표현하라.
그래도 불분명하다면 접두어를 사용하자.

- Bad
<br> 맥락이 불분명한 변수
<br> number, verb, pluralModifier 변수 세개는 함수를 끝까지 읽어보고 나서야 통계 추측 메세지에 사용됨을 알 수 있다.
```java
private void printGuessStatistics(char candidate, int count) {
    String number;
    String verb;
    String pluralModifier;
    if (count == 0) {  
        number = "no";  
        verb = "are";  
        pluralModifier = "s";  
    }  else if (count == 1) {
        number = "1";  
        verb = "is";  
        pluralModifier = "";  
    }  else {
        number = Integer.toString(count);  
        verb = "are";  
        pluralModifier = "s";  
    }
    String guessMessage = String.format("There %s %s %s%s", verb, number, candidate, pluralModifier);
    print(guessMessage);
}
```
- Good
<br> 맥락이 분명한 변수
<br> 함수가 길어, 함수를 작은 조각으로 쪼개기 위해, GuessStatisticsMessage라는 클래스를 만든 후 세 변수를 클래스에 넣었다.
<br> 그러면, 세 변수는 맥락이 분명해진다. 
<br> → 이렇게 맥락을 개선하면 함수를 쪼개기가 쉬워지므로 알고리즘도 좀 더 명확해진다.
```java
public class GuessStatisticsMessage {
    private String number;
    private String verb;
    private String pluralModifier;

    public String make(char candidate, int count) {
        createPluralDependentMessageParts(count);
        return String.format("There %s %s %s%s", verb, number, candidate, pluralModifier );
    }

    private void createPluralDependentMessageParts(int count) {
        if (count == 0) {
            thereAreNoLetters();
        } else if (count == 1) {
            thereIsOneLetter();
        } else {
            thereAreManyLetters(count);
        }
    }

    private void thereAreManyLetters(int count) {
        number = Integer.toString(count);
        verb = "are";
        pluralModifier = "s";
    }

    private void thereIsOneLetter() {
        number = "1";
        verb = "is";
        pluralModifier = "";
    }

    private void thereAreNoLetters() {
        number = "no";
        verb = "are";
        pluralModifier = "s";
    }
}
```

## 불필요한 맥락을 없애라
`Gas Station Delux` 이라는 어플리케이션을 작성한다고 해서 클래스 이름의 앞에 GSD를 붙이지는 말자. 
<br> G를 입력하고 자동완성을 누를 경우 모든 클래스가 나타나는 등 효율적이지 못하다.
<br>`GSDAccountAddress` 대신 `Address`라고만 해도 충분하다.
- Address는 클래스 이름으로 좋다.
- accountAddress와 customerAddress는 클래스 인스턴스로 좋은 이름이지만, 클래스 이름으로는 적합하지 못하다.

## 마치면서
이름을 바꾸는 것에 두려워 하지 말고 좋은 이름으로 바꾸자.
<br>클래스 이름과 메서드 이름을 암기하는데 시간을 뺏기지 않고 자연스럽게 읽히는 코드 짜는데 집중할 수 있을 것이다. 
