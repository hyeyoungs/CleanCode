# 01 클린 코드
> 클린 코드가 중요한 이유

## 코드가 존재하리라
코드를 자동으로 생성해주는 시스템들이 나타나는 현상 때문에, 코드는 더 이상 문제가 아니며, 모델이나 요구사항에 집중해야 한다는 주장이 생겨나고 있다.<br>

🤔 그렇다면, 코드는 이제 필요없는 것인가?<br>
❌ 코드가 사라지는 것은 불가능하다.<br>

코드는 요구사항을 상세히 표현하는 수단이다.<br>
코드는 기계가 실행할 정도로 요구사항을 상세히 표현하는 언어이며,<br>
기계가 이해하고 실행할 정도로 코드가 엄밀하고 정확하고 상세하게 정형화되게 작성되어야 한다.<br>
어느 순간이든 기계에게 정밀한 표현이 필요하다. 그러므로 코드는 항상 존재할 것이다.

## 나쁜 코드
킬러 앱으로 인기를 커다랗게 끈 회사는 머지않아 망했다. 망한 원인은 나쁜 코드였다.<br>
일정을 맞추기 위해, 나쁜 코드를 작성하고 나중에 손보겠다고 하지만, `나중은 결코 오지 않는다.`

## 나쁜 코드로 치르는 대가
나쁜 코드는 개발 속도를 크게 떨어뜨린다.<br>
나쁜 코드가 쌓일수록 팀 생산성은 떨어진다.

### 원대한 재설계의 꿈
- 나쁜 코드가 쌓여 결국 팀이 관리층에게 재설계를 요구한다.
- 새 타이거 팀이 기존 시스템 기능을 모두 제공하고 새 시스템도 제공해야 한다.
- 기존 팀원들은 기존의 코드를 유지보수하게 된다.
- 두 팀은 오랫동안 경쟁을 한다.
- 새 시스템이 기존 시스템을 따라잡으면 초창기 타이거 팀원들은 새 팀원들로 교체 되어있다.
- 그리고 그들은 다시 재 설계를 요구한다.

→ `클린코드는 비용 절감 뿐만 아니라 생존을 위한 길이다.` 🌟

### 태도
코드가 엉망이여서, 한줄만 고치면 될줄 알았던 문제가 모듈을 수백 개 건드리는 증상은 흔하다.<br>
좋은 코드가 나쁜 코드로 전략하는 이유는 전적으로 프로그래머에게 있다.<br>

대다수 관리자는 좋은 코드를 원한다.<br>
`좋은 코드를 사수하는 일은 프로그래머의 책임이다.`<br>
나쁜 코드의 위험을 이해하지 못하는 관리자 말을 그대로 따르는 행동은 전문가 답지 못하다.

### 원초적 난제
- 🤔  난제 : 기한을 맞추려면 나쁜 코드를 양산할 수 밖에 없다. <br>
- 나쁜 코드를 양산하면 기한을 맞추지 못한다. 엉망진창인 상태로 인해 속도가 곧바로 늦어지고, 결국 기한을 놓친다.<br>
- 🌟 기한을 맞추는 유일한 방법 : `코드를 최대한 깨끗하게 유지하는 습관이다.`

### 클린 코드라는 예술?
클린 코드를 구현하는 것은 예술 행위와 비슷하다.<br>
클린 코드를 작성하려면, 힘들게 습득한 `클린 코드에 대한 감각`을 활용해 `작은 기법들을 적용`하는 것이 필요하다.<br>

`깨끗한 코드를 어떻게 작성할까?`<br>
- 🔑 핵심은 `코드 감각`이다. <br>
  `코드 감각`이 있으면,
  - `좋은 코드와 나쁜 코드를 구분`한다.
  - 또한, `나쁜 코드를 좋읕 코드로 바꾸는 전략도 파악`한다.

### 클린 코드란?
프로그래머마다 정의가 다양하다.

#### 비야네 스트롭
- 즐겁고 읽히는 코드
- `효율`적인 코드<br>
  (성능을 최적으로 유지하도록 코드를 구현해야 한다는 뜻도 있고, 나쁜 코드는 더 나쁜 코드로 만든다는 의미가 있다.)
- 논리가 간단해야 한다. 그래야 버그가 숨어들지 못한다.
- 의존성을 최대한 줄여야 유지보수가 쉬워진다.
- 메모리 누수, 경쟁 상태, 일관되지 않는 네이밍 등 `디테일`에 신경쓰자.
- 깨끗한 코드는 `한 가지를 제대로` 한다.

#### 그래디 부치
- 단순하고 직접적이다. 잘 쓴 문장처럼 읽힌다. 절대 설계자의 의도를 숨기지 않는다.<br>
  → 코드는 `문제와 해답을 명확히 제시`해야 한다.
- 명쾌한 추상화와 단순한 제어문으로 가득하다.<br>
  → 코드는 추측이 아니라 `사실`에 기반해야 하며, 반드시 `필요한 내용만` 담아야 한다.

#### 큰 데이브 토마스
- 클린 코드는 작성자가 아닌 사람도 읽기 쉽고 `고치기 쉽다.`
- `테스트 케이스`가 존재한다.
- 의미 있는 이름이 붙는다.
- 특정 목적을 달성하는 방법은 하나만 제공한다.
- 의존성과 API는 `최소`이며, 명확히 정의한다.

#### 마이클 페더스
- `주의` 깊게 짠 코드다. 고치려고 살펴봐도 딱히 손 댈 곳이 없다.<br>
  → 작성자가 이미 `모든 사항을 고려`하여 구현

#### 론 제프리스
- 모든 테스트를 통과한다.
- `중복` 줄이기 → 클래스, 메서드, 함수 등을 최대한 줄인다.
- `한 기능`만 수행하기 → 클래스, 메서드는 한가지 일만 한다.
- `표현력` 높이기 → 네이밍으로 코드가 하는 일 명시, 하나의 객체/메서드가 여러 기능을 수행할 경우 여러 개로 나누기
- `일찍`부터 작게 `추상화`하기 → 프로젝트 빨리 진행할 수 있게 한다.

#### 워드 커닝햄
- 읽고, 끄덕이고, 다음으로 넘어갈 수 있는 코드를 작성하라.<br>
  → 읽으면서 짐작한 대로 돌아가는 코드
- 언어를 단순하게 보이도록 만드는 코드<br>
  → 언어를 단순하게 보이도록 만드는 열쇠는 언어가 아닌 프로그래머다.

## 우리는 작가다
우리는 작가다. 작가에게는 독자와 잘 소통할 책임도 있다.<br>
코드를 짤 때, 당신의 노력을 평가할 독자를 위해 글을 쓰는 작가임을 명심하라.<br>

🤔 사람들은 코드를 읽는 것보다 짜는데 시간이 많이 필요하다고 생각할지도 모른다.<br>
실제로 그렇지 않다.<br>
우리는 새 코드를 짜면서 끊임없이 기존 코드를 읽는다.<br> '코드 읽는 시간 : 짜는 시간' 비율이 '10 : 1'을 넘는다.<br>
→ 🌟 `읽기 쉬운 코드`가 매우 중요하다. 읽기 쉽게 만들면 짜기도 쉬워진다.

## 보이스카우트 규칙
잘 짠 코드가 전부가 아니다. `시간이 지나도 언제나 깨끗하게 유지`해야 한다.<br>
우리는 적극적으로 `코드의 퇴보`를 막아야 한다.

🌟 이를 위한 원칙 : `체크 아웃할 때보다 좀 더 깨끗한 코드를 체크인 하라.`

한꺼번에 많은 시간과 노력을 투자해 코드를 정리할 필요가 없다.<br>
변수 이름 하나를 개선하고, 조금 긴 함수 하나를 분할하고, 약간의 중복을 제거하고, 복잡한 if문 하나를 정리하면 충분하다.

## 결론
이 책을 읽는다고 뛰어난 프로그래머가 된다는 보장은 없다.<br>
단지 뛰어난 프로그래머가 생각하는 방식과 기술과 도구를 소개할 뿐이다.<br>
좋은 코드와 나쁜 코드를 소개하고, 나쁜 코드를 좋은 코드로 바꾸는 방법도 소개한다.<br>
나머지는 당신의 몫이다. `연습해라.`
