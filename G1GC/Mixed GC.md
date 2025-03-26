## 🧩 Mixed GC 세부 단계

---

### 🔸 Initial Mark (STW, Young GC와 함께 수행됨)
- **GC Root에서 직접 참조하고 있는 객체들 중, Old Region 객체를 참조하는 Survivor Region의 객체들을 마킹**.
- 마킹된 **Survivor Region**은 이후 **Root Region Scan**의 시작점으로 사용됨.
- 이 단계는 **Young GC 직후, 동일한 STW 구간**에서 수행됨.

### 🔸 Root Region Scan (Concurrent)
- Initial Mark에서 마킹된 **Survivor Region**을 시작점으로 사용.
- 해당 Region 안의 객체들이 참조하고 있는 **다른 Old 객체**를 따라가며 **마킹**.

### 🔸 Concurrent Mark (Concurrent)
- Root Region Scan에서 마킹한 Old객체부터 **멀티스레드로 깊이 탐색**하며 **Old 영역을 마킹**.
- STW가 아니므로 애플리케이션과 병행.
## SATB (Snapshot At The Beginning)

### 문제 상황
GC의 Concurrent Mark 단계에서 참조가 변경되면, 마킹 누락이 발생할 수 있음.

```java
// 초기 상태
A.ref = B;
// GC가 Concurrent Mark 중인 상황에서
A.ref = C; // 참조 변경!
```
### SATB
- 참조 변경이 발생하면, 참조를 가진 객체(A) 를 SATB 버퍼에 기록함.
- 이 동작은 Write Barrier를 통해 자동으로 삽입됨.

  
### 🔸 Remark (STW)
- STW를 걸고 SATB Buffer에 남은 객체들을 마킹.
- 누락된 참조 보정 → 마킹 작업을 최종적으로 완성.
- SATB 버퍼를 비움.

### 🔸 Cleanup
- 죽은 객체만 존재하는 Region은 즉시 회수.
- 가비지 비율이 높은 Region들을 추려서 **old_cset_candidates (CSet 후보)**에 저장.
- 이후 Mixed GC가 이 후보 Region을 대상으로 실행됨.

### 🔸 Mixed GC
- JVM이 Old Region의 사용량을 백그라운드에서 추적하다가 **IHOP (Initiating Heap Occupancy Percent)** 기준을 넘으면 Mixed GC 트리거.
- CSet에 등록된 Region 중 일부를 선택하여, 살아있는 객체는 복사, 죽은 객체는 버림. 
- 이렇게 해서 Old 영역까지 점진적으로 정리됨.
- Mixed GC는 **Young GC + CSet으로 선정된 일부 Old Region을 함께 청소**하는 과정.
- Mixed GC에서 CSet(Collected Set)에 포함된 Region은 **해당 GC 사이클 안에서 반드시 처리되어야** 함.
- STW 시간이 초과될 경우 해당 사이클에서 수집한 CSet은 사용하지 않고, 다음 사이클에 다시 수집.
- CSet은 마킹 정보를 기반으로 선정되기 때문에, **다음 GC 사이클로 넘기는 것은 안전하지 않음**.


---
## ✨ 주요 개념

- **Region**: 힙을 고정 크기로 나눈 블록 단위 (Young, Survivor, Old 등으로 역할 구분)
- **Remembered Set (RSet)**: 다른 Region에서 이 Region을 참조하고 있는 포인터 정보를 모아둔 데이터 구조
- **SATB (Snapshot At The Beginning)**: 마킹 시작 시점의 참조 상태를 기준으로 객체 생존 여부를 판단하는 기법
- **CSet (Collection Set)**: GC 대상 Region들로 구성된 집합

---
```
✅ 요약 흐름도

[Young GC]
     ↓
[Initial Mark] (STW)
     ↓
[Root Region Scan] (Concurrent)
     ↓
[Concurrent Mark] (Concurrent + 멀티스레드)
     ↓
[Remark] (STW, SATB Buffer 처리)
     ↓
[Cleanup] (회수 + CSet 후보 선정)
     ↓
[Mixed GC] (Young GC + 일부 Old Region)
```

