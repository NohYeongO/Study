# 예제 코드
```java
class A {
  private static final String STR = "ABC";
  private static Long l = Long.valueOf(-1L);
  private static int i = -1;
  private static C c = new C();

  private final String a = "ABC";
  private final int ii = 1;
  private C cc = new C();

  public static void main(String[] args) {
    A a = new A();
    B b = new B();
  }
}

class B extends A {
  // ...
}
```
---

## JVM의 주요 구성 요소

JVM은 클래스로더(ClassLoader), 런타임 데이터 영역(Runtime Data Area), 실행 엔진(Execution Engine) 으로 구성된다.

### 1. 런타임 데이터 영역 (Runtime Data Area)

#### 1.1 메서드 영역 (Method Area)
- 클래스 로딩 시 저장되는 정보가 포함된다.
- 상수 풀(Constant Pool) → 기본 자료형 리터럴, 변수명, 클래스명, 메서드명 등의 심볼 정보
- 메타데이터(Metadata)
  - 접근 플래그(Access Flags) → 클래스, 메서드, 필드의 접근 제어자 정보
  - 클래스명(This Class), 부모 클래스(Super Class), 인터페이스 목록(Interfaces)
  - 필드 정보(Fields), 메서드 정보(Methods), 속성 정보(Attributes)

#### 1.2 힙(Heap)
- `new` 키워드로 생성된 객체와 배열 인스턴스를 저장
- GC(Garbage Collector) 의 주요 활동 영역

#### 1.3 JVM 스택(Stack)
- 각 스레드가 실행하는 메서드 호출을 저장하는 공간
- FILO(First-In, Last-Out) 방식
- 스택 프레임(Stack Frame) 에는 로컬 변수, 피연산자 스택, 반환 주소가 포함됨

#### 1.4 PC 레지스터 (PC Registers)
- 실행 중인 바이트코드 명령어의 메모리 주소를 저장하고 업데이트

#### 1.5 네이티브 스택 (Native Stack)
- JNI(Java Native Interface) 를 통해 C, C++ 등의 네이티브 코드를 실행하는 공간
- 네이티브 메서드(`System.gc()`, `Thread.sleep()` 등)의 실행을 지원

---

### 2. 실행 엔진 (Execution Engine)
JVM이 바이트코드를 실행하는 핵심 모듈이다.

#### 2.1 인터프리터 (Interpreter)
- 클래스 파일(`.class`)에서 바이트코드를 한 줄씩 해석하며 실행
- 빠르게 실행할 수 있지만, 반복 코드 실행 시 속도가 느려지는 단점 존재

#### 2.2 JIT(Just-In-Time) 컴파일러
- 자주 실행되는 바이트코드를 네이티브 코드로 변환하여 캐싱
- 최초 실행 시 인터프리터가 사용되지만, 반복 실행 시 JIT 컴파일러가 개입하여 속도를 향상
- 변환된 네이티브 코드 실행 시 JVM Stack을 활용

#### 2.3 GC (Garbage Collector)
- 힙 메모리 및 메서드 영역에서 참조되지 않는 객체를 제거
- 메모리 누수를 방지하고, JVM의 메모리 사용을 최적화

---

## ClassLoader란?

### 1. ClassLoader의 개념
- ClassLoader는 JVM의 런타임 메모리 영역과 상호작용하여 `.class` 파일을 찾아 읽고, 이를 JVM의 메서드 영역(Method Area)에 적재하는 역할을 수행하는 컴포넌트이다.
- 즉, 자바 바이트코드(`.class` 파일)를 메모리에 로드하여 프로그램이 실행될 수 있도록 돕는 핵심 요소이다.
- 동적 로딩(Dynamic Loading) 을 지원하여, 프로그램 실행 시점에서 필요한 클래스를 찾아 로드할 수 있다.

---

### 2. ClassLoader의 흐름
1. JVM이 ClassLoader를 사용하여 클래스가 이미 로드되어 있는지 확인
2. 클래스가 존재하면 기존 클래스를 사용
3. 클래스가 로드되지 않았다면, ClassLoader가 `.class` 파일을 찾음
4. `.class` 파일이 존재하지 않으면 `ClassNotFoundException` 발생
5. `.class` 파일이 존재하면, JVM이 바이트코드를 메서드 영역(Method Area)에 적재
6. 적재된 클래스는 이후 프로그램에서 사용 가능

---

### 3. ClassLoader의 종류 (JVM 클래스 로더 계층 구조)
#### 3.1 Bootstrap ClassLoader (부트스트랩 클래스 로더)
- JVM이 실행될 때 가장 먼저 실행되는 최상위 클래스 로더.
- `java.lang.Object`, `java.lang.String`, `java.util.*` 같은 표준 라이브러리를 로드한다.
- JDK 9 이전에는 `rt.jar` 파일에서 클래스를 로드했으며, JDK 9 이후에는 `lib/modules` 구조를 사용한다.

#### 3.2 Platform ClassLoader (플랫폼 클래스 로더, JDK 9 이후)
- `java.sql`, `java.logging` 등의 플랫폼 관련 API 클래스를 로드하는 역할을 한다.
- JDK 8까지는 `Extension ClassLoader` 가 있었으나, JDK 9 이후 `Platform ClassLoader` 로 대체되었다.

#### 3.3 Application ClassLoader (애플리케이션 클래스 로더)
- 사용자 정의 클래스(`A.class`, `B.class` 등)를 로드하는 역할을 한다.
- 기본적으로 `classpath` (`-cp` 또는 `-classpath` 옵션)에서 클래스를 찾는다.
- 애플리케이션 실행 시 직접 개발한 클래스들은 이 ClassLoader가 로드한다.

---

### 4. ClassLoader의 주요 동작 과정
#### 4.1 로딩 (Loading)
- `.class` 파일을 읽어 들여 JVM의 메서드 영역(Method Area)에 적재한다.
- 클래스가 이미 로드되었는지 먼저 확인하고, 로드되지 않았다면 ClassLoader를 통해 찾는다.
- `.class` 파일이 존재하지 않으면 `ClassNotFoundException` 이 발생한다.

#### 4.2 링크 (Linking)
- 클래스의 바이트코드를 검증하고, 정적인 변수들을 초기화하는 과정.
- 링크 과정은 "검증(Verify) → 준비(Prepare) → 해석(Resolve)" 3단계로 나뉜다.
  - 검증(Verify): `.class` 파일의 바이트코드가 유효한지 확인.
  - 준비(Prepare): `static` 변수를 생성하고 기본값(`0`, `null`)으로 초기화.
  - 해석(Resolve): 심벌릭 참조(Symbolic Reference)를 실제 메모리 주소로 변환.

#### 4.3 초기화 (Initialization)
- `static` 변수의 실제 값을 할당하고, `static` 블록이 실행됨.
- 이 과정이 끝나면 클래스는 완전히 로드된 상태가 된다.

---

### 5. ClassLoader의 위임 방식
1. Application ClassLoader 가 클래스를 찾으려고 하면, 먼저 부모인 Platform ClassLoader 에게 요청한다.
2. Platform ClassLoader 가 클래스를 찾으려고 하면, 먼저 Bootstrap ClassLoader 에게 요청한다.
3. Bootstrap ClassLoader 가 해당 클래스를 찾을 수 있으면 로드하고, 없으면 Platform ClassLoader 에게 요청한다.
4. Platform ClassLoader 가 못 찾으면, Application ClassLoader 가 최종적으로 클래스를 로드한다.

✅ 즉, ClassLoader는 항상 "부모가 먼저 클래스를 로드하도록 위임"하는 구조다.

---

### 6. ClassLoader가 .class 파일에서 CAFEBABE를 확인하는 과정

- .class 파일의 첫 4바이트는 JVM이 올바른 바이트코드 파일인지 확인하는 매직 넘버(Magic Number) 를 포함하고 있다.

- Java 클래스 파일의 매직 넘버는 0xCAFEBABE 이며, JVM은 이를 통해 유효한 Java 클래스 파일인지 확인한다.

- .class 파일을 로드할 때, ClassLoader는 먼저 파일의 첫 4바이트를 읽어 CAFEBABE 값이 맞는지 검사한다.

- 만약 다른 값이면 ClassFormatError 예외가 발생하여 JVM이 클래스를 로드하지 않는다.

---
### 내가 생각한 JVM 메모리 구조 및 흐름
![](/image/JVM.jpeg)