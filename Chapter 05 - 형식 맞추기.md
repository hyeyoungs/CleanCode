# 05 형식 맞추기
> 가독성에 필수적인 포맷팅

## Introduction
질서정연하고 깔끔하며, 일관적인 코드를 본다면 사람들에게 전문가가 짰다는 인상을 심어줄 수 있다.
<br>반대로, 코드가 어수선해 보인다면 프로젝트 전반적으로 무성의한 태도로 작성했다고 생각할 것이다.
<br><br>프로그래머라면 형식을 깔끔하게 맞춰 코드를 짜야한다.
<br>코드 형식을 맞추기 위한 간단한 규칙을 정하고, 그 규칙을 착실히 따라야 하며,
<br>팀으로 일한다면 팀이 합의해 규칙을 정하고 모두가 그 규칙을 따라야 한다.
<br>필요하다면 규칙을 자동으로 적용하는 도구를 활용한다 (e.g. Android Studio의 Code Formatter)

## 형식을 맞추는 목적
코드 형식은 중요하다! 너무 중요해서 무시하기 어렵다.  
코드 형식은 의사소통의 일환이며, **의사소통은 전문 개발자의 일차적인 의무다.**  
오늘 구현한 기능이 다음 버전에서 바뀔 확률은 높고, 시간이 지나면 원래 코드의 흔적을 찾아볼 수 없는 경우도 많지만,  
오늘 구현한 코드의 스타일과 가독성 수준은 유지보수의 용이성과 확정성에 지속적인 영향을 미친다.

> **코드는 사라져도 스타일과 규율은 사라지지 않는다!**

## 적절한 행 길이를 유지하라 (코드의 세로 길이)
소스코드는 얼마나 길어야 적당할까?  
500줄을 넘지 않고 대부분 200줄 정도인 파일로도 커다란 시스템을 구축할 수 있다.  
실제로 전문 자바 프로젝트들(JUnit, FitNesse, Time and Money) 등이 이렇게 구현되어있다.

코드 길이를 200줄 정도로 제한하는 것은 반드시 지킬 엄격한 규칙은 아니지만,  
일반적으로 큰 파일보다는 작은 파일이 이해하기 쉽다.

### 신문 기사처럼 작성하라
좋은 신문 기사는 최상단에 표제(기사를 몇마디로 요약하는 문구)가 있다.  
첫 문단에는 전체 기사 내용을 요약하며, 기사를 읽으며 내려갈 수록 세세한 사실이 조금씩 드러나며 세부사항이 나오게 된다.  
소스파일 이름(표제)는 간단하면서도 설명이 가능하게 지어,  
이름만 보고도 올바른 모듈을 살펴보고 있는지를 판단 할 수 있도록 한다.  
소스파일의 첫 부분(요약 내용)은 고차원 개념과 알고리즘을 설명한다.  
아래로 내려갈수록 의도를 세세하게 묘사하며, 마지막에는 가장 저차원 함수(아마 Getter/Setter)와 세부 내역이 나온다.  
신문이 사실, 날짜, 이름 등을 무작위로 뒤섞은 긴 기사 하나만 싣는다면 아무도 신문을 읽지 않을 것이다.

### 개념은 빈 행으로 분리하라
코드의 각 줄은 수식이나 절을 나타내고, 여러 줄의 묶음은 완결된 생각 하나를 표현한다.  
생각 사이에는 빈 행을 넣어 분리해야한다. 그렇지 않다면 단지 줄바꿈만 다를 뿐인데도 코드 가독성이 현저히 떨어진다.

```java
// 빈 행을 넣지 않을 경우
package fitnesse.wikitext.widgets;
import java.util.regex.*;
public class BoldWidget extends ParentWidget {
    public static final String REGEXP = "'''.+?'''";
    private static final Pattern pattern = Pattern.compile("'''(.+?)'''",
        Pattern.MULTILINE + Pattern.DOTALL);
    public BoldWidget(ParentWidget parent, String text) throws Exception {
        super(parent);
        Matcher match = pattern.matcher(text); match.find(); 
        addChildWidgets(match.group(1));}
    public String render() throws Exception { 
        StringBuffer html = new StringBuffer("<b>");         
        html.append(childHtml()).append("</b>"); 
        return html.toString();
    } 
}
```

```java
// 빈 행을 넣을 경우
package fitnesse.wikitext.widgets;

import java.util.regex.*;

public class BoldWidget extends ParentWidget {
    public static final String REGEXP = "'''.+?'''";
    private static final Pattern pattern = Pattern.compile("'''(.+?)'''", 
        Pattern.MULTILINE + Pattern.DOTALL
    );

    public BoldWidget(ParentWidget parent, String text) throws Exception { 
        super(parent);
        Matcher match = pattern.matcher(text);
        match.find();
        addChildWidgets(match.group(1)); 
    }

    public String render() throws Exception { 
        StringBuffer html = new StringBuffer("<b>"); 
        html.append(childHtml()).append("</b>"); 
        return html.toString();
    } 
}
```

### 세로 밀집도

줄바꿈이 개념을 분리한다면, 반대로 세로 밀집도는 연관성을 의미한다.  
즉, 서로 밀집한 코드 행은 세로로 가까이 놓여야 한다.

```java
// 의미없는 주석으로 변수를 떨어뜨려 놓아서 한눈에 파악이 잘 안된다.

public class ReporterConfig {
    /**
    * The class name of the reporter listener 
    */
    private String m_className;

    /**
    * The properties of the reporter listener 
    */
    private List<Property> m_properties = new ArrayList<Property>();
    public void addProperty(Property property) { 
        m_properties.add(property);
    }
```

```java
// 의미 없는 주석을 제거함으로써 코드가 한눈에 들어온다.
// 변수 2개에 메소드가 1개인 클래스라는 사실이 드러난다.

public class ReporterConfig {
    private String m_className;
    private List<Property> m_properties = new ArrayList<Property>();

    public void addProperty(Property property) { 
        m_properties.add(property);
    }
```

### 수직 거리

서로 밀접한 개념은 세로로 가까이 둬야 한다.  
두 개념이 서로 다른 파일에 속한다면 규칙이 통하지 않지만,  
타당한 근거가 없다면 서로 밀접한 개념은 한 파일에 속해야 마땅하다.(protected 변수를 피해야 하는 이유다.)  
같은 파일에 속할 정도로 밀접한 두 개념은 세로 거리로 연관성(= 한 개념을 이해하는 데 다른 개념이 중요한 정도)을 표현한다.

#### 변수선언

변수는 사용하는 위치에서 최대한 가까이 선언한다.  
Chapter 03 공부한 우리가 만든 함수는 매우 짧음으로 지역 변수는 각 함수 맨 처음에 선언한다.

```java
// InputStream이 함수 맨 처음에 선언 되어있다.

private static void readPreferences() {
    InputStream is = null;
    try {
        is = new FileInputStream(getPreferencesFile()); 
        setPreferences(new Properties(getPreferences())); 
        getPreferences().load(is);
    } catch (IOException e) { 
        try {
            if (is != null) 
                is.close();
        } catch (IOException e1) {
        } 
    }
}
```

```java
// 모두들 알다시피 루프 제어 변수는 Test each처럼 루프 문 내부에 선언

public int countTestCases() { 
    int count = 0;
    for (Test each : tests)
        count += each.countTestCases(); 
    return count;
}
```

```java
// 드물지만, 긴 함수에서는 블록 상단 또는 루프 직전에 변수를 선언 할 수도 있다.
...
for (XmlTest test : m_suite.getTests()) {
    TestRunner tr = m_runnerFactory.newTestRunner(this, test);
    tr.addListener(m_textReporter); 
    m_testRunners.add(tr);

    invoker = tr.getInvoker();

    for (ITestNGMethod m : tr.getBeforeSuiteMethods()) { 
        beforeSuiteMethods.put(m.getMethod(), m);
    }

    for (ITestNGMethod m : tr.getAfterSuiteMethods()) { 
        afterSuiteMethods.put(m.getMethod(), m);
    } 
}
...
```

#### 인스턴스 변수
자바의 경우 인스턴스 변수는 클래스 맨 처음에 선언한다.
<br>변수 간 세로로 거리를 두지 않는다 
<br>잘 설계한 클래스는 대다수 클래스 메서드가 인스턴스 변수를 사용하기 때문이다.  
<br>C++의 경우에는 마지막에 선언하는 것이 일반적이다. 어느 곳이든 잘 알려진 위치에 인스턴스 변수를 모으는 것이 중요하다.

```java
// 도중에 선언된 변수는 꽁꽁 숨겨놓은 보물 찾기와 같다. 십중 팔구 코드를 읽다가 우연히 발견한다. 
// 요즘은 IDE가 잘 되어있어서 찾기야 어렵지 않겠지만, 더러운건 마찬가지다.

public class TestSuite implements Test {
    static public Test createTest(Class<? extends TestCase> theClass,
                                    String name) {
        ... 
    }

    public static Constructor<? extends TestCase> 
    getTestConstructor(Class<? extends TestCase> theClass) 
    throws NoSuchMethodException {
        ... 
    }

    public static Test warning(final String message) { 
        ...
    }

    private static String exceptionToString(Throwable t) { 
        ...
    }

    private String fName;

    private Vector<Test> fTests= new Vector<Test>(10);

    public TestSuite() { }

    public TestSuite(final Class<? extends TestCase> theClass) { 
        ...
    }

    public TestSuite(Class<? extends TestCase> theClass, String name) { 
        ...
    }

    ... ... ... ... ...
}
```

#### 종속 함수

한 함수가 다른 함수를 호출한다면(=종속 함수) 두 함수는 세로로 가까이 배치한다. 
<br>가능하면 호출되는 함수를 호출하는 함수보다 뒤에 배치한다. (프로그램이 자연스럽게 읽힐 수 있도록) 
<br>이러한 규칙을 일관되게 적용한다면 독자는 방금 함수에서 호출한 함수가 잠시 후에 정의될 것이라고 자연스레 예측한다.

```java
/*
   첫째 함수에서 가장 먼저 호출하는 함수가 바로 아래 정의된다.
   다음으로 호출하는 함수는 그 아래에 정의된다. 그러므로 호출되는 함수를 찾기가 쉬워지며 전체 가독성도 높아진다.
 */

/*
   makeResponse 함수에서 호출하는 getPageNameOrDefault함수 안에서 "FrontPage" 상수를 사용하지 않고,
   상수를 알아야 의미 전달이 쉬워지는 함수 위치에서 실제 사용하는 함수로 상수를 넘겨주는 방법이
   가독성 관점에서 훨씬 더 좋다
 */

public class WikiPageResponder implements SecureResponder { 
    protected WikiPage page;
    protected PageData pageData;
    protected String pageTitle;
    protected Request request; 
    protected PageCrawler crawler;

    public Response makeResponse(FitNesseContext context, Request request) throws Exception {
        String pageName = getPageNameOrDefault(request, "FrontPage");
        loadPage(pageName, context); 
        if (page == null)
            return notFoundResponse(context, request); 
        else
            return makePageResponse(context); 
        }

    private String getPageNameOrDefault(Request request, String defaultPageName) {
        String pageName = request.getResource(); 
        if (StringUtil.isBlank(pageName))
            pageName = defaultPageName;

        return pageName; 
    }

    protected void loadPage(String resource, FitNesseContext context)
        throws Exception {
        WikiPagePath path = PathParser.parse(resource);
        crawler = context.root.getPageCrawler();
        crawler.setDeadEndStrategy(new VirtualEnabledPageCrawler()); 
        page = crawler.getPage(context.root, path);
        if (page != null)
            pageData = page.getData();
    }

    private Response notFoundResponse(FitNesseContext context, Request request)
        throws Exception {
        return new NotFoundResponder().makeResponse(context, request);
    }

    private SimpleResponse makePageResponse(FitNesseContext context)
        throws Exception {
        pageTitle = PathParser.render(crawler.getFullPath(page)); 
        String html = makeHtml(context);
        SimpleResponse response = new SimpleResponse(); 
        response.setMaxAge(0); 
        response.setContent(html);
        return response;
    } 
...
```

#### 개념의 유사성

개념적인 친화도가 높을 수록 코드를 서로 가까이 배치한다.  
앞서 살펴보았듯이 한 함수가 다른 함수를 호출하는 종속성, 변수와 그 변수를 사용하는 함수가 그 예다.  
그 외에도 비슷한 동작을 수행하는 함수 무리 또한 개념의 친화도가 높다.

```java
// 같은 assert 관련된 동작들을 수행하며, 명명법이 똑같고 기본 기능이 유사한 함수들로써 개념적 친화도가 높다.
// 이런 경우에는 종속성은 오히려 부차적 요인이므로, 종속적인 관계가 없더라도 가까이 배치하면 좋다.

public class Assert {
    static public void assertTrue(String message, boolean condition) {
        if (!condition) 
            fail(message);
    }

    static public void assertTrue(boolean condition) { 
        assertTrue(null, condition);
    }

    static public void assertFalse(String message, boolean condition) { 
        assertTrue(message, !condition);
    }

    static public void assertFalse(boolean condition) { 
        assertFalse(null, condition);
    } 
...
```

### 세로 순서

일반적으로 함수 호출 종속성은 아래방향으로 유지하므로, 호출되는 함수를 호출하는 함수보다 뒤에 배치한다.  
그러면 소스코드가 자연스럽게 고차원 → 저차원으로 내려간다.  
가장 중요한 개념을 가장 먼저 표현하고, 세세한 사항은 마지막에 표현한다.  
그렇게 하면 첫 함수 몇개만 읽어도 개념을 파악하기 쉬워질 것이다.

## 가로 형식 맞추기

대다수의 프로그래머들은 명백히 짧은 행을 선호하므로 짧은 행이 바람직하다.  
Hollerith가 제안한 80자 제한은 다소 인위적이므로 조금 더 늘여도 좋다. 하지만 120자 이상을 넘어간다면 주의 부족이다.  
필자 개인적으로는 120자 정도로 길이를 제한한다.

### 가로 공백과 밀집도

가로로는 공백을 사용해 밀접한 개념과 느슨한 개념을 표현한다

```java
private void measureLine(String line) { 
    lineCount++;

    // 흔히 볼 수 있는 코드인데, 할당 연산자 좌우로 공백을 주어 왼쪽,오른쪽 요소가 확실하게 구분된다.
    int lineSize = line.length();
    totalChars += lineSize; 

    // 반면 함수이름과 괄호 사이에는 공백을 없앰으로써 함수와 인수의 밀접함을 보여준다
    // 괄호 안의 인수끼리는 쉼표 뒤의 공백을 통해 인수가 별개라는 사실을 보여준다.
    lineWidthHistogram.addLine(lineSize, lineCount);
    recordWidestLine(lineSize);
}
```

추가로 연산자의 우선순위를 강조하기 위해서도 공백을 사용한다 `return b*b - 4*a*c;`  
하지만 Code Formatter등의 도구가 연산자 우선순위까지 고려하지 못하므로 공백을 임의로 넣어주더라도 사라지는 경우가 대부분이다.

### 가로 정렬

```java
public class FitNesseExpediter implements ResponseSender {
    private        Socket          socket;
    private     InputStream       input;
    private     OutputStream       output;
    private     Reques          request;         
    private     Response       response;    
    private     FitNesseContex      context; 
    protected     long          requestParsingTimeLimit;
    private     long          requestProgress;
    private     long          requestParsingDeadline;
    private     boolean          hasError;

    ...
```

보기엔 깔끔해 보일지 모르나, 코드가 엉뚱한 부분을 강조해 변수 유형을 자연스레 무시하고 이름부터 읽게 된다.  
게다가 Code Formatter 대부분들은 이렇게 해놔봤자 무시하고 원래대로 돌려놓는다.
그러므로 선언문과 할당문을 별도로 정렬할 필요가 없다.  
정렬이 필요할 정도로 목록이 길다면 목록의 길이가 문제이지 정렬이 부족해서가 아니다. 
선언부가 길다는 것은 클래스를 쪼개야 한다는 것을 의미한다.

### 들여쓰기
들여쓰기를 잘 해놓으면 구조가 한 눈에 들어온다.

#### 들여쓰기 무시하기

간단한 if문, while문, 짧은 함수에서 들여쓰기를 무시하고픈 유혹이 생긴다.  
하지만 들여쓰기로 제대로 범위를 표현한 코드가 가독성이 더 높다. 유혹을 뿌리치자.

```java
// 이렇게 한행에 다 넣을 수 있다고 한 행에 범위를 뭉뚱그린 코드는 피한다.

public class CommentWidget extends TextWidget {
    public static final String REGEXP = "^#[^\r\n]*(?:(?:\r\n)|\n|\r)?";

    public CommentWidget(ParentWidget parent, String text){super(parent, text);}
    public String render() throws Exception {return ""; } 
}
```

```java
// 한줄이라도 정성스럽게 들여쓰기로 감싸주자. 가독성을 위해서다.

public class CommentWidget extends TextWidget {
    public static final String REGEXP = "^#[^\r\n]*(?:(?:\r\n)|\n|\r)?";

    public CommentWidget(ParentWidget parent, String text){
        super(parent, text);
    }

    public String render() throws Exception {
        return ""; 
    } 
}
```

### 가짜 범위

빈 while문이나 for문을 접할 때가 있다. 가능한 피해야 되지만, 피하지 못 할 경우엔  
빈 블록을 올바로 들여쓰고 괄호로 감싸라. 세미클론은 새 행에다 제대로 들여써서 넣어준다.

## 팀 규칙

팀에 속해있다면, 가장 우선시 되어야 하고 선호해야 할 규칙은 팀 규칙이다.  
코딩 스타일을 의논하여(괄호를 어디에 넣을지, 네이밍은 어떻게 할지 등) IDE Formatter로 지정하여 구현하는 것이 옳은 방식이다.  
**좋은 소프트웨어 시스템은 읽기 쉬운 문서로 이뤄지고, 읽기 쉬운 문서는 스타일이 일관적이고 매끄러워야 한다.**

## 밥 아저씨의 형식 규칙
저자가 사용하는 형식 규칙은 아래 코드에 잘 드러난다.
```java
public class CodeAnalyzer implements JavaFileAnalysis { 
    private int lineCount;
    private int maxLineWidth;
    private int widestLineNumber;
    private LineWidthHistogram lineWidthHistogram; 
    private int totalChars;

    public CodeAnalyzer() {
        lineWidthHistogram = new LineWidthHistogram();
    }

    public static List<File> findJavaFiles(File parentDirectory) { 
        List<File> files = new ArrayList<File>(); 
        findJavaFiles(parentDirectory, files);
        return files;
    }

    private static void findJavaFiles(File parentDirectory, List<File> files) {
        for (File file : parentDirectory.listFiles()) {
            if (file.getName().endsWith(".java")) 
                files.add(file);
            else if (file.isDirectory()) 
                findJavaFiles(file, files);
        } 
    }

    public void analyzeFile(File javaFile) throws Exception { 
        BufferedReader br = new BufferedReader(new FileReader(javaFile)); 
        String line;
        while ((line = br.readLine()) != null)
            measureLine(line); 
    }

    private void measureLine(String line) { 
        lineCount++;
        int lineSize = line.length();
        totalChars += lineSize; 
        lineWidthHistogram.addLine(lineSize, lineCount);
        recordWidestLine(lineSize);
    }

    private void recordWidestLine(int lineSize) { 
        if (lineSize > maxLineWidth) {
            maxLineWidth = lineSize;
            widestLineNumber = lineCount; 
        }
    }

    public int getLineCount() { 
        return lineCount;
    }

    public int getMaxLineWidth() { 
        return maxLineWidth;
    }

    public int getWidestLineNumber() { 
        return widestLineNumber;
    }

    public LineWidthHistogram getLineWidthHistogram() {
        return lineWidthHistogram;
    }

    public double getMeanLineWidth() { 
        return (double)totalChars/lineCount;
    }

    public int getMedianLineWidth() {
        Integer[] sortedWidths = getSortedWidths(); 
        int cumulativeLineCount = 0;
        for (int width : sortedWidths) {
            cumulativeLineCount += lineCountForWidth(width); 
            if (cumulativeLineCount > lineCount/2)
                return width;
        }
        throw new Error("Cannot get here"); 
    }

    private int lineCountForWidth(int width) {
        return lineWidthHistogram.getLinesforWidth(width).size();
    }

    private Integer[] getSortedWidths() {
        Set<Integer> widths = lineWidthHistogram.getWidths(); 
        Integer[] sortedWidths = (widths.toArray(new Integer[0])); 
        Arrays.sort(sortedWidths);
        return sortedWidths;
    } 
}
```

## Findings
- 적절한 길이 유지하는게 좋다.
  <br>200라인에서 500라인 이하로 유지하는게 좋다.
  - 코드 길이를 200줄 정도로 제한하는 것은 반드시 지킬 엄격한 규칙은 아니지만, 일반적으로 큰 파일 보다는 작은 파일이 이해하기 쉽다.
    - 현업에서는 대부분의 코드들도 200라인 정도를 유지한다고 한다.
  - 코드 길이가 200라인을 넘어간다면, 클래스가 여러 개의 일을 하고 있을 수 있다.
    - SRP(Single Responbility Principle - 단일 책임 원칙)위배 했을 가능성이 크다.
- 밀접한 개념은 서로 가까이 둔다. 
  - 행 묶음은 완결된 생각 하나를 표현하기 때문에 개념은 빈 행으로 분리한다.
  - 변수는 사용되는 위치에서 최대한 가까이 선언한다.

## Feelings
- 저자가 말하는 형식 맞추기란 가독성 높은 코드를 작성하기 위한 컨벤션 맞추기로 느껴졌다.
- 적절한 행의 길이를 맞추고, 변수들끼리 모아두고, 빈 행으로 개념을 분리 하는 등의 방법을 보면서 코드를 가독성 높게 만드는 방법을 알게 되었다.

## Issue
- 두 개념이 서로 다른 파일에 속한다면 규칙이 통하지 않지만,  
  타당한 근거가 없다면 서로 밀접한 개념은 한 파일에 속해야 마땅하다. 
  <br>(protected 변수를 피해야 하는 이유다.)
- 하지만 Code Formatter등의 도구가 연산자 우선순위까지 고려하지 못하므로 공백을 임의로 넣어주더라도 사라지는 경우가 대부분이다.
  <br>🤔 그렇다면 괄호로 묶어주는 것이 더 바람직하지 않을까?
