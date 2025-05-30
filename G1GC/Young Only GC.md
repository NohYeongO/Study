## 🧩 Young-Only GC 실행 과정

## 1. 사용하지 않는 객체 찾기

### ✔ GC 루트(GC Root)에서 탐색 시작
**GC 루트란?**  
GC가 탐색을 시작하는 출발점이 되는 객체

### **GC 루트 종류:**
1. `static` 변수에 저장된 객체
2. 실행 중인 `Thread` 객체
3. 스택 프레임의 지역 변수 및 매개변수

### 🔸 Reachability Analysis(도달 가능성 분석) 수행
- **GC 루트에서 도달할 수 있는 객체는 살아있는 객체로 판단 → Mark**
- **도달할 수 없는 객체는 가비지(Garbage)로 판단 → Free List 대상**


## 2. 가비지 객체 제거

- **마킹되지 않은 객체(사용되지 않는 객체)를 힙에서 제거**
- **해당 메모리 주소를 Free List에 추가 → JVM이 새 객체를 할당 가능하도록 처리**

## 3. 살아남은 객체 이동

### 🔸 Young Generation 내부에서 객체 이동
- 살아남은 객체들은 **Eden → Survivor 영역(S0 또는 S1)으로 이동**
- **Survivor 영역에서 여러 번 살아남으면 Old Generation으로 승격(Promotion)**
    - 객체가 Survivor 영역에서 **설정된 임계값** 만큼 살아남으면 Old로 이동

### 🔸 참조 주소 업데이트
- **객체들이 새로운 위치에서 정상적으로 작동하도록 참조된 주소를 업데이트**

### 🔸 이전 주소들을 Free List에 다시 등록
- **GC가 이동한 객체들의 이전 주소를 Free List에 추가하여 JVM이 재사용 가능하도록 함**

## 4. 카드 테이블 (Card Table)
- **Young GC 실행 중에 Old Generation의 객체가 Young 영역의 객체를 참조할 수 있음 → 이때 GC 실행 시 오류 발생 가능**
- **G1GC에서는 '카드 테이블(Card Table)'이 존재하여 이러한 문제를 해결**

### 🔸 카드 테이블이란?
- **Old Generation에서 Young Generation의 객체를 참조할 경우, 해당 정보를 카드 테이블에 업데이트**
- **GC 실행 중에 Old Generation 전체를 검사할 필요 없이 카드 테이블만 확인하여 빠르게 참조 여부를 확인 가능**