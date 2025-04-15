# HashMap Put 파해치기

## 📑 목차
- [HashMap 클래스파일 주석 번역](#hashmap-클래스파일-주석-번역)
- [HashMap 필드변수 및 생성자](#hashmap-필드변수-및-생성자)
- [Put 메서드](#put-메서드)
- [hashkey 동작](#hashkey-동작)
- [putVal 동작](#putvalint-hash-k-key-v-value-boolean-onlyifabsent-boolean-evict-동작)
- [resize 동작](#resize-동작)
- [최종 정리](#최종-정리)

---

### HashMap 클래스파일 주석 번역
>
> Map 인터페이스를 기반으로 한 해시 테이블 기반 구현체입니다.
> 이 구현은 Map의 선택적 연산을 모두 제공하며, null 값과 null 키를 허용합니다.
> (HashMap 클래스는 대체로 Hashtable과 비슷하지만, 동기화되어 있지 않고, null을 허용한다는 점이 다릅니다.)
> 이 클래스는 맵의 순서를 보장하지 않으며, 시간이 지나도 그 순서가 변하지 않는다고 보장하지 않습니다.
>
> ■ 성능  
> 기본 연산인 get과 put은, 해시 함수가 요소들을 잘 분산시킨다는 전제 하에, 상수 시간에 수행됩니다.  
> 컬렉션 뷰(keySet, values, entrySet)에 대한 반복은 HashMap 인스턴스의 버킷 수(용량) + 데이터 개수에 비례한 시간이 소요됩니다.  
> 초기 용량을 너무 크게 설정하거나, 부하 계수(load factor)를 너무 낮게 설정하는 것은 반복 성능에 부정적입니다.
>
> ■ 초기 용량과 부하 계수
> - 초기 용량(initial capacity): 해시 테이블 생성 시의 버킷 수
> - 부하 계수(load factor): 해시 테이블이 얼마나 채워질 수 있는지를 나타내는 값. 이 값을 넘으면 리해시(rehash) 발생.  
    >   예: 버킷 수가 16이고 부하 계수가 0.75라면, 12개 초과 시 리해시 발생.
> - 기본 부하 계수 0.75는 시간과 공간의 균형을 제공합니다.
> - 예상 데이터 개수 / 부하 계수보다 초기 용량이 크면, 리해시가 발생하지 않음.
> - 많은 매핑을 저장할 경우 충분히 큰 용량으로 생성하는 것이 리해시 비용을 줄여 효율적.
> - 키의 hashCode가 동일한 경우 성능 저하가 발생할 수 있으며, 키가 Comparable이면 비교 순서를 사용해 충돌 완화.
>
> ■ 동기화 관련
> - 이 구현체는 동기화되어 있지 않음.
> - 여러 스레드가 동시에 접근하면서 구조 변경이 발생할 경우, 반드시 외부에서 동기화 필요.
> - 구조 변경: put/remove처럼 키-값 매핑을 추가하거나 제거하는 것 (값만 바꾸는 건 구조 변경 아님)
> - 동기화 예: Map m = Collections.synchronizedMap(new HashMap(...));
>
> ■ 반복자와 fail-fast
> - 컬렉션 뷰 메서드(keySet, values, entrySet 등)의 반복자는 fail-fast.
> - 반복자 생성 후 구조 변경 시 ConcurrentModificationException 발생.
> - 이 예외는 버그를 잡기 위한 수단이지, 로직에 의존해서는 안 됨.
>
> ■ 타입 파라미터
> - <K> : 키 타입  
> - <V> : 값 타입
>
> ■ 구현 노트
> - 이 맵은 버킷 기반의 해시 테이블로 동작.
> - 버킷에 포함된 항목 수가 많으면 TreeNode 구조로 전환되어 이진 트리처럼 작동.
> - TreeNode는 java.util.TreeMap과 유사하며, instanceof로 타입 검사 후 관련 메서드 호출.
> - 대부분의 경우 TreeNode는 사용되지 않음.
>
> ■ TreeNode 버킷
> - 주로 hashCode 기준으로 정렬.
> - hashCode 충돌 시, Comparable이면 compareTo를 통해 정렬.
> - 이를 위해 리플렉션을 사용해 Comparable 여부 확인 (comparableClassFor).
> - TreeNode 사용 시 최악의 경우에도 O(log n) 성능 보장.
> - 성능 저하는 대부분 잘못된 hashCode 구현 때문.
>
> ■ TreeNode 전환 조건
> - TreeNode는 일반 노드보다 약 2배 공간 사용.
> - TREEIFY_THRESHOLD 이상 노드가 있을 경우에만 TreeNode로 전환.
> - 노드 수가 줄거나 리사이징되면 untreeify로 일반 노드로 복귀.
>
> ■ 해시 분포
> - 이상적인 hashCode 분포는 포아송 분포를 따름 (평균 0.5 기준).
> - 리스트 크기별 확률:  
    >   k=0: 0.60653066  
    >   k=1: 0.30326533  
    >   k=2: 0.07581633  
    >   k=3: 0.01263606  
    >   k=4: 0.00157952  
    >   k=5: 0.00015795  
    >   k=6: 0.00001316  
    >   k=7: 0.00000094  
    >   k=8: 0.00000006  
    >   k≥9: 0.00000001 이하
>
> ■ 기타 구현 세부사항
> - 트리 버킷의 루트는 보통 첫 노드이며, Iterator.remove() 후 바뀔 수 있음.
> - 내부 메서드는 hashCode를 인자로 받아 해시 재계산 방지.
> - split, treeify, untreeify 중에도 Node.next 필드를 유지해 순서 보존.
> - Comparator를 사용할 경우 클래스와 identityHashCode 비교로 동등 판단.
>
> ■ LinkedHashMap과의 관계
> - LinkedHashMap은 HashMap을 상속하며, 접근 순서를 유지.
> - 삽입/삭제/접근 시 hook 메서드를 호출해 자체 로직 실행.
>
> ■ 병렬 스타일 코딩
> - SSA(Static Single Assignment) 스타일로 구현되어 aliasing 에러 방지.
>
> 번역 >> Chat GPT

---

## HashMap 필드변수 및 생성자

비관적락 

낙관적락

분산락

```java
/**
 * 기본 초기 용량 - 반드시 2의 거듭제곱이어야 합니다.
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // 16

/**
 * 최대 용량 - 생성자에서 인수로 더 큰 값이 지정되었을 때 사용됩니다.
 * 반드시 2의 거듭제곱이어야 하며, 1<<30 이하이어야 합니다.
 */
static final int MAXIMUM_CAPACITY = 1 << 30; // 1,073,741,824

/**
 * 생성자에서 부하 계수(load factor)를 지정하지 않았을 때 사용되는 기본값입니다.
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;

/**
 * 버킷을 리스트 대신 트리(Tree)로 변환하기 위한 노드 수 임계값입니다.
 * 하나의 버킷에 이 값 이상의 노드가 있을 때, 요소가 추가되면 해당 버킷은 트리로 변환됩니다.
 * 이 값은 반드시 2보다 커야 하며, 축소 시 일반 버킷으로 되돌리는 트리 제거 로직과의 호환을 위해
 * 최소한 8 이상이어야 합니다.
 */
static final int TREEIFY_THRESHOLD = 8;

/**
 * 리사이즈 중 버킷(분할된 버킷)을 트리에서 리스트로 되돌릴 때 사용하는 임계값입니다.
 * TREEIFY_THRESHOLD보다 작아야 하며, 제거 시 축소 감지를 위해 최대 6 이하여야 합니다.
 */
static final int UNTREEIFY_THRESHOLD = 6;

/**
 * 트리화를 허용하는 최소 테이블 용량입니다.
 * (이보다 작으면, 노드가 너무 많을 경우 테이블이 리사이즈됩니다.)
 * 리사이즈 기준과 트리화 기준 간의 충돌을 방지하기 위해 최소 4 * TREEIFY_THRESHOLD 이상이어야 합니다.
 */
static final int MIN_TREEIFY_CAPACITY = 64;
```

---

## Put 메서드

```java
// Put 호출시
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

> `map.put(key, value)`를 호출하게 되면 put 메서드는 내부적으로 `putVal()`을 호출하며,  
> 인자값으로 `hash(key), key, value, false, true`를 넘긴다.

---

## hash(key) 동작

```java
/**
 * key.hashCode()를 계산하고, 해시의 상위 비트를 하위 비트에 퍼뜨립니다 (XOR 연산을 사용).
 * 해시 테이블은 2의 거듭제곱 크기이므로, 현재 마스크보다 위쪽 비트만 다른 해시들은
 * 항상 같은 버킷에 충돌하게 됩니다. (예: 작은 테이블에서 연속된 정수를 가진 Float 키 집합)
 * 따라서 상위 비트의 영향을 하위 비트로 확산시키는 변환을 적용합니다.
 * 여기에는 속도, 유용성, 비트 분산 품질 사이의 트레이드오프가 있습니다.
 * 일반적인 해시 값들은 이미 적절히 분산되어 있어서 추가적인 분산의 효과가 크지 않고,
 * 우리는 버킷 내 충돌이 많을 경우 트리를 사용하기 때문에,
 * 시스템적인 성능 저하를 줄이고, 인덱스 계산에 사용되지 않는 상위 비트의 효과도 포함시키기 위해,
 * 가장 저렴한 방식으로 (XOR + shift) 약간의 비트 이동만 수행합니다.
 */
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

> `hash()`를 호출하면 key가 null일 경우 0을 반환하지만, null이 아니라면 객체의 `hashCode()`를 구해 `h`에 담고,  
> `h >>> 16`을 통해 상위 비트를 오른쪽으로 16칸 이동시킨 뒤 XOR 연산을 한다.`>>>`는 부호를 무시하고 모두 0으로 채운다.  
> 이렇게 하는 이유는 큰 해시값을 가공해서 인덱스로 만들기 위함으로 보이고, 충돌을 줄이기 위한 것으로 보인다.

---

## putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) 동작
``` java
    /**
     * 이 HashMap이 구조적으로 변경된 횟수를 나타냅니다.
     * 구조적인 변경이란, HashMap의 매핑 수를 변경하거나 내부 구조를 수정하는 작업
     * (예: 리해시(rehash))를 말합니다.
     * 이 필드는 HashMap의 컬렉션 뷰(예: keySet, entrySet 등)에 대한 이터레이터가
     * 변경을 감지하고 빠르게 실패(fail-fast)하도록 하기 위해 사용됩니다.
     * (자세한 내용은 ConcurrentModificationException을 참고하세요.)
     */
     transient int modCount;
    
    /**
     * 테이블을 초기화하거나 크기를 두 배로 늘립니다.
     * 테이블이 null인 경우, threshold 필드에 설정된 초기 용량 목표에 따라 새로 할당합니다.
     * 그렇지 않은 경우(이미 테이블이 존재하는 경우), 2의 거듭제곱 단위로 확장하기 때문에
     * 각 버킷의 요소는 기존 인덱스에 그대로 남거나, 새 테이블에서 2의 거듭제곱만큼 떨어진 위치로 이동해야 합니다.
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
        // Node<K,V>[], Node<K,V>, int 변수를 선언
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        
        // table이 아직 초기화되지 않았거나 크기가 0이라면 resize()를 호출해 table을 생성하고, 그 길이를 n에(16) 저장
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length; // 16
            
        // 해시 값을 기반으로 인덱스를 계산후 해당 공간이 비어있는지 확인
        if ((p = tab[i = (n - 1) & hash]) == null)
            /// 비어있다면 새 노드를 생성해 해당 위치에 저장
            tab[i] = newNode(hash, key, value, null);
            
        // 충돌 발생 = 이미 해당 인덱스에 노드가 존재하는 경우
        else {
            // 변수 선언
            Node<K,V> e; K k;
            
            // 첫 노드의 키와 해시값이 현재 삽입하려는 키와 같은지 비교
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                // 키가 같으면 덮어쓰기 위한 기존 노드를 e에 저장
                e = p;
            // 노드가 트리 구조라면, 트리 방식으로 값을 삽입 
            else if (p instanceof TreeNode)
                // 트리노드에 데이터를 추가 이미 존재한다면 해당 노드에 현재 데이터를 반환
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
                
            // 일반 연결 리스트 방식 처리   
            else {
                for (int binCount = 0; ; ++binCount) { // binCount
                    // 현재 노드의 다음 노드가 비어 있다면
                    if ((e = p.next) == null) {
                        // 새로운 노드를 끝에 추가
                        p.next = newNode(hash, key, value, null);
                        
                        // 현재 위치의 노드 수가 TREEIFY_THRESHOLD 이상이면 트리로 변경 시도
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    // 중간에 동일 키를 가진 노드를 찾으면 e에 저장 후 break
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    // 다음 노드로 이동    
                    p = e;
                }
            }
            // 동일한 키를 가진 노드를 찾은 경우
            if (e != null) { // existing mapping for key
                // 값을 새로 삽입한 데이터로 변경 = 덮어씌워짐
                V oldValue = e.value;
                
                // onlyIfAbsent가 false이거나 기존 값이 null이면 값 덮어쓰기
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                
                // LinkedHashMap에 정의
                // 해당 노드를 제일 뒤에 넣는 작업 + ++modCount(구조변경횟수 증가)
                afterNodeAccess(e);
                return oldValue; // 새로 변경된 value 반환
            }
        }
        // 구조 변경된 횟수 증가
        ++modCount;
        // 현재 크기가 임계값을 초과하면 resize() 수행
        if (++size > threshold)
            resize();
        // 가장 오래된 노드 제거 함수(?)  
        afterNodeInsertion(evict);
        return null;
    }
```
### 왜 (n - 1) & hash를 사용하는지?  
- HashMap의 테이블 크기(n)는 항상 2의 거듭제곱입니다.  
- 그렇기 때문에 index = hash % n 대신 index = hash & (n - 1) 연산으로 인덱스를 계산할 수 있습니다.  
- 이 방식은 나눗셈보다 훨씬 빠른 비트 연산으로 동작하며, 해시값의 하위 비트만을 추출하는 효과가 있습니다.  
- 예를 들어 n = 16이면 n - 1 = 15 = 0b1111, hash의 하위 4비트만 인덱스로 사용됩니다.  
`(n - 1) & hash는 테이블의 크기 안에서 해시값을 효율적으로 인덱스로 변환하기 위한 최적화된 공식입니다.`

---

## resize() 동작
``` java
    /**
     * 리사이즈가 발생하는 다음 크기 값 (capacity * load factor).
     *
     * @serial
     */
    // (이 Javadoc 설명은 직렬화 시점에 해당됩니다.
    // 추가로, 테이블 배열이 아직 할당되지 않은 경우,
    // 이 필드는 초기 배열 용량을 나타내며, 0이면 DEFAULT_INITIAL_CAPACITY를 의미합니다.)
    int threshold;

    final Node<K,V>[] resize() {
        // 필드 table을 참조 = 최초 호출시 table은 null일 수 있음
        Node<K,V>[] oldTab = table;
        // 현재 테이블의 용량, 없으면 0
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        // 현재 threshold (0일 수도 있음)
        int oldThr = threshold;
        int newCap, newThr = 0;
        
        // oldCap > 0: 기존 table이 이미 존재하는 경우
        if (oldCap > 0) {
            // 기존 용량이 MAXIMUM_CAPACITY(1,073,741,824) 이상인 경우
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE; // 더 이상 확장 불가, 임계값을 int 최대값(2,147,483,647)으로 설정
                return oldTab; // 기존 테이블 그대로 반환
            }
            // 그 외의 경우: 용량을 2배로 늘리기 위한 작업
            // newCap은 기존 용량을 왼쪽으로 1비트 이동시켜 2배로 설정
            // 만약 newCap이 MAXIMUM_CAPACITY보다 작고, 기존 용량이 DEFAULT_INITIAL_CAPACITY 이상이면
            // 임계값(threshold)도 두 배로 증가시킴
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // threshold도 2배로 설정
        }
        // oldCap == 0 && oldThr > 0 이면, threshold가 초기용량 역할을 했던 상황
        else if (oldThr > 0) // initial capacity was placed in threshold
            // 이걸 새 용량으로 사용
            newCap = oldThr; 
        // 처음 table이 생성될 경우, 용량도 threshold도 지정 안 된 경우 → 기본값 사용    
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY; // 16
            newThr = (int)(DEFAULT_LOAD_FACTOR/*0.75*/ * DEFAULT_INITIAL_CAPACITY/*16*/); // 12
        }
        // else if (oldThr > 0) 여기 로직이 실행됐을경우 실행
        if (newThr == 0) {
            // loadFactor는 기본생성자로 DEFAULT_LOAD_FACTOR값이 할당되어있음
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        // 새 threshold 저장
        threshold = newThr;
        
        // 새로운 테이블 배열 생성 (빈 Node 배열 16칸짜리)
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        
        // 기존 테이블이 null이(최초생성) 아니면 데이터를 옮기는 작업
        if (oldTab != null) {
            // 기존 버킷들을 하나씩 순회
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                // 현재 버킷에 데이터를 e에 담고 null인지 확인
                if ((e = oldTab[j]) != null) {
                    // 기존 table에 버킷을 null로 비워줌
                    oldTab[j] = null;
                    // 다음 노드가 있는지 확인 하나의 인덱스에 연결리스트로 데이터가 있을 수 있음
                    if (e.next == null)
                        // 노드가 없다면 새로운 table에 삽입
                        newTab[e.hash & (newCap - 1)] = e;
                    // e 가 TreeNode 타입인지 확인    
                    else if (e instanceof TreeNode)
                        // split을 통해 트리구조로 변경후 데이터를 옮김
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    // 다음 노드가 있지만 TreeNode 형태는 아닐경우 연결리스트로 데이터를 옮기는 작업
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            // 다음 노드 next에 삽입
                            next = e.next;
                            // 해시값의 비트와 oldCap을 AND했을 때 0이면 lo
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            // 해시값 & oldCap != 0 → hi
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null); // 다음 노드로 이동해서 반복
                        if (loTail != null) {
                            loTail.next = null; // 노드의 next null처리
                            newTab[j] = loHead; // 기존 인덱스에 lo head 삽입
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead; // 기존 인덱스 + oldCap 위치에 hi head 삽입
                        }
                    }
                }
            }
        }
        // 새로 resize 작업후 새로 만든 리스트 형태 테이블 반환
        return newTab;
    }
```

### 왜 e.hash & oldCap을 확인하는지?
- 리사이즈 시 기존 용량은 oldCap, 새로운 용량은 newCap = oldCap * 2입니다.
- 이때 oldCap은 항상 2의 거듭제곱이기 때문에, e.hash & oldCap은 추가된 한 비트를 기준으로 lo/hi를 나눌 수 있습니다.

```
(e.hash & oldCap) == 0 → lo 그룹 → newTab[j] 위치로 이동
(e.hash & oldCap) != 0 → hi 그룹 → newTab[j + oldCap] 위치로 이동
해시값의 특정 비트를 기준으로 기존 위치에 남을지(old 인덱스), 새로운 위치로 갈지(2배 떨어진 인덱스)를 빠르게 판단할 수 있습니다.
```

> resize() 작업은 새로운 해시 테이블(Map 내부의 배열)을 생성하는 과정으로,  
> 기본적으로 16칸짜리 버킷 배열을 만들며 시작합니다.  
> 이후 데이터가 증가할수록 용량을 2배씩 늘려가며 리사이징이 일어납니다. 


---

### ✅ 최종 정리

> `HashMap`은 처음 생성될 때 16개의 버킷을 가진 Node 배열로 시작합니다.  
> 이후 데이터가 들어오면 크기를 확인해 threshold(기본적으로 12)를 초과하는 경우 `resize()`를 통해 용량을 두 배로 늘립니다.  
> 하나의 버킷에는 여러 노드가 LinkedList처럼 연결되어 저장되며, 동일한 키가 들어오면 기존 값을 덮어씌웁니다.  
> 같은 해시 인덱스에 노드가 너무 많이 쌓이면 성능 저하를 막기 위해 Tree 구조로 전환됩니다 (기본 임계값: 8개 이상).  
> 이후 노드 수가 감소하거나, 테이블 크기가 줄어들면 다시 LinkedList 구조로 전환됩니다.
>
> `HashMap`이 **스레드 안전하지 않은 이유**는 내부적으로 `put`, `resize`, `remove` 등의 메서드가 **동기화되어 있지 않기 때문**입니다.  
> 특히 `resize()` 과정에서는 기존의 데이터를 새 배열로 재배치하는데, 이때 다른 스레드가 동시에 `put()`을 호출하면  
> 하나는 옛날 배열을 보고 있고, 하나는 새 배열을 보고 있을 수 있어 **경합과 꼬임, 데이터 손실**이 발생할 수 있습니다.  
> 이런 이유로 **멀티스레드 환경에서는 반드시 외부 동기화(Map m = Collections.synchronizedMap(...))나 ConcurrentHashMap을 사용**해야 합니다.
>
> `put()` 또는 `resize()` 과정에서 인덱스를 계산할 때 `&` 연산을 사용하는데, HashMap이 항상 2의 거듭제곱 크기를 유지하기 때문에 가능한 최적화 방식입니다.  
> 예를 들어 `(n - 1) & hash`는 `hash % n`보다 빠르게 인덱스를 계산할 수 있고, `e.hash & oldCap`은 리사이즈 도중 노드가 기존 인덱스에 남을지, 두 배 오른쪽 인덱스로 갈지를 결정하는 데 사용됩니다.
