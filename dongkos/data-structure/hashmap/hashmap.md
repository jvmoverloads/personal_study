# HashMap

## 특징

- key - value 쌍으로 이루어져 있다.
- get으로 검색 시, O(1)의 상수시간을 갖는다.
  - 정말일까?
- 데이터가 정렬되어 있지 않다.
  - 왜 그럴까?

## 구현

- hashMap은 내부에 Node라는 class의 리스트를 갖고 있다.
- 내부에 Node들의 리스트인 table을 갖고 있다.
  - Node는 단방향 연결리스트이다.

```java
    class HashMap {
        // transient keyword를 붙이면 serialization시 무시한다.
        transient Node<K,V>[] table;
    }

    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        // 다음 Node를 가르키고 있음.
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;

            return o instanceof Map.Entry<?, ?> e
                    && Objects.equals(key, e.getKey())
                    && Objects.equals(value, e.getValue());
        }
    }
```

### 삽입
```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
    
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```
- key 값을 해시해서 넣는다
  - 왜 hashing 할까?
  - 내부 자료 구조를 생각해보면 table은 Node의 배열이다. 또한 hashMap은 get으로 가져올때 O(1)의 시간 복잡도를 갖는다.
  여기서 연결되는 것은 배열에서 특정 인덱스로 값을 가져오는 것은 O(1)의 시간복잡도를 갖는다. 그렇다면 특정 key로 table에서 
    하나의 Node를 가져오는 연산을 한다는 것인데, hashMap의 key는 어떤 타입이든 될 수 있다.
  ```java
    Map<String, String> hashMap = new HashMap<>();
    hashMap.put("dongko", "hyper");
  
    Map<Integer, String> hashMap = new HashMap<>();
    hashMap.put(1, "hyper");
  ```
  배열의 인덱스는 정수여야 하는데 키값은 정수가 아니니, **키값을 어떤 알고리즘을 이용해서 정수로 만들어야 한다.** 
    이때 사용하는 것이 hash이다. 그러면 자연스럽게 hash Collision이 발생할 수 있음을 알 수 있다. 왜냐하면 hash function에
인자로 받는 값은 무한대(숫자, 문자, 자바 클래스, 거시기 등) 이지만 결과 값은 특정 자리수의 정수이기 때문이다.
    ```java
    Long l = 1023L;
    Integer i = 1245;
    Double d = 10.2;
    Float f = 12.4F;
    String str = "dongko";
    Dongko dongko = new Dongko("dongko");
    
    System.out.println(l.hashCode());
    System.out.println(i.hashCode());
    System.out.println(d.hashCode());
    System.out.println(f.hashCode());
    System.out.println(str.hashCode());
    System.out.println(dongko.hashCode());
    ===================================
    결과 : 
    1023
    1245
    641859584
    1095132774
    -1326161944
    1595212853
    ```

    자 그렇다면 이렇게 얻은 hash 값, key, value를 가지고 어떻게 insert를 할까?
    ```java
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
   ```
  굉장히 복잡한 듯 보이지만 하나씩 나눠서 보자
    ```java
    // Node를 담는 배열 table이 비어 있으면 초기화 시킨다.
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
  
    // hash 값을 기반으로 배열의 인덱스를 만든다.
    // 여기서 (n - 1) & hash 이라는 식을 사용한다.
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    ```
    table을 초기화 하면 DEFAULT_INITIAL_CAPACITY라는 변수를 사용하는데 기본 값은 16이다. 
    & -> 이건 비트연산인데 두 값을 이진수로 만들고 각 자리수를 비교하여 둘 다 1이면 1, 그 외는 0이다.
    초기 사이즈 16과 dongko class의 hashcode **1595212853**를 & 연산하면 결과가 5가 나온다.
    이를 배열의 인덱스로해서 Node를 새로 만들고 넣어준다! 그러면 table의 5번째 인덱스에는 
    key 값이 dongko class이고 value가 hyper인 노드가 하나 들어가게 된다!
    
  - 그렇다면 만약 어떤 키값이 들어왔을때 **(n - 1) & hash** 연산의 값이 5라면 어떻게 될까?
      ```java
      for (int binCount = 0; ; ++binCount) {
        // table의 5번인덱스로 와서 Node의 next가 있는지 계속 탐색
        // LinkedList니깐 next가 있는지 계속 보다가 없으면 
          if ((e = p.next) == null) {
            // 빈자리에 새로운 노드를 끼워넣는다! 이는 자바에서 hash collision을 해결하는 방식이고 체이닝이라고 한다.
              p.next = newNode(hash, key, value, null);
              if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                  treeifyBin(tab, hash);
              break;
          }
      }
      ```
  - 이렇기 때문에 hashMap의 get이 O(1)이라는 말은 틀렸다! (특정 상황에서는) 밑에 get에서 한번 더 보자
  - 심화 버전으로 계속되는 충돌로 인해 LinkedList의 길이가 길어진다면 hashMap은 LinkedList를 Red-Black-Tree로 전환한다.
  ```java
    else if (p instanceof TreeNode)
        // 이미 트리로 전환 된 경우 트리에 값을 삽입
        e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
    else {
        for (int binCount = 0; ; ++binCount) {
            if ((e = p.next) == null) {
                p.next = newNode(hash, key, value, null);
                
                // TREEIFY_THRESHOLD 값인 8을 넘어가면 
                // LinkedList를 Red-Black-Tree로 전환한다.
                if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                    treeifyBin(tab, hash);
                break;
            }
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k))))
                break;
            p = e;
        }
    }
  ```
  - Red Black Tree는 이진 탐색 트리의 일종으로 이진 탐색 트리의 최악의 경우를 대비하여 스스로 균형을 맞추는 자료구조이다.
    ```mermaid
        graph TD
            A[6] --> B[4]
            A --> C[9]
            B --> D[2]
            B --> E[5]
            D --> F[1]
            D --> G[3]
            C --> H[8]
            C --> I[10]
            H --> J[7]
    
    ```
    - 최악의 경우
    
    ```mermaid
        graph TD
            A[10]
            A --> B[9]
            B --> C[8]
            C --> D[7]
            D --> E[6]
            E --> F[5]
            F --> G[4]
            G --> H[3]
            H --> I[2]
            I --> J[1]
    
    ```
    
### 조회
- 복잡해 보이지만 이것도 하나씩 짚어보자.
```java
   final Node<K,V> getNode(Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n, hash; K k;
        // table이 비어 있지 않은지 체크
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & (hash = hash(key))]) != null) {
            if (first.hash == hash && ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```
- 운좋게도 충돌이 발생하지 않아서 배열에 하나의 Node만 있는경우는 그 첫번째 Node를 가져온다.
```java
    if (first.hash == hash && ((k = first.key) == key || (key != null && key.equals(k))))
        return first;
```
- 충돌이 발생해서 LinkedList에 여러개가 있는 경우 일치하는 값을 찾을때까지 반복해서 찾는다
```java
    do {
        if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    } while ((e = e.next) != null);
```
- 이미 트리로 바꼈다면 트리에서 일치하는 값을 탐색한다.
```java
    if (first instanceof TreeNode)
        return ((TreeNode<K,V>)first).getTreeNode(hash, key);
```
    













