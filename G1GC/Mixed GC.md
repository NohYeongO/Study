## 🧩 Mixed GC 세부 단계

---

### 🔸 Initial Mark (STW, Young GC와 함께 수행됨)
- **GC Root에서 직접 참조하고 있는 객체들 중, Old Region 객체를 참조하는 Survivor Region의 객체들을 마킹**.
- 마킹된 **Survivor Region**은 이후 **Root Region Scan**의 시작점으로 사용됨.
- 이 단계는 **Young GC 직후, 동일한 STW 구간**에서 수행됨.

### 🔸 Root Region Scan (Concurrent)
- Initial Mark에서 마킹된 **Survivor Region**을 시작점으로 사용.
- 해당 Region 안의 객체들이 참조하고 있는 **다른 Old 객체**를 따라가며 **마킹 큐에 추가**.

### 🔸 Concurrent Mark (Concurrent)
- 마킹 큐에서 객체를 꺼내어, **객체를 멀티스레드로 깊이 탐색**하며 **Old 영역을 마킹**.
- STW가 아니므로 애플리케이션과 병행.
- **참조 변경 시 마킹 누락 가능성 존재 → SATB 사용 필요**
```java
ex )
A.ref = B; // 원래 참조
// Concurrent Mark 도중
A.ref = C; // 참조 변경!
// 이 경우 GC는 B를 아직 마킹하지 않았을 수도 있는데
// 참조가 끊겼기 때문에 B를 살아있는 객체에서 누락시킬 위험이 있음!
```
#### SATB (Snapshot At The Beginning)
- 마킹이 시작된 시점을 기준으로 **기존 참조 상태**를 기준으로 생존 여부 판단.
- 참조가 변경되기 전의 객체를 **SATB Buffer**에 저장.
- **Write Barrier**를 통해 버퍼 기록 처리.

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

