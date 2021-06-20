# Chapter 11 컬렉션 프레임워크

## 1.6 Arrays
1. 배열의 복사 : `copyOf()`
    - 참조형의 경우 **얕은 복사** : 참조형의 필드(element)는 주소값만 복사되기 때문에 원본과 같은 객체를 참조하게 됨<br>
    <img src="https://raw.githubusercontent.com/jjjlyn/java-study/main/shallow_copy.png" width="640" height="360">

2. 배열 채우기 : `fill()`, `setAll()`
   ```java
   public static void fill(Object[] a, Object val) {
        int i = 0;

        for(int len = a.length; i < len; ++i) {
            a[i] = val;
        }
    }

    public static <T> void setAll(T[] array, IntFunction<? extends T> generator) {
        Objects.requireNonNull(generator);

        for(int i = 0; i < array.length; ++i) {
            array[i] = generator.apply(i);
        }
    }
    ```

3. `sort()`, `binarySearch()`
- Arrays에서 채택한 sort 알고리즘
    - `compareTo()`메소드를 기준으로 정렬한다.

- Arrays에서 채택한 search 알고리즘
    - binarySearch는 반드시 오름차순으로 정렬되어야 정확한 값이 나온다.

4. `equals()`, `toString()`

- 배열 출력
    - `toString()` : 배열의 요소가 참조형인 경우 요소의 주소값이 출력됨
    - `deepToString()` : 배열의 모든 요소를 재귀적으로 호출
    ```java
    public static String deepToString(Object[] a) {
        if (a == null) {
            return "null";
        } else {
            int bufLen = 20 * a.length;
            if (a.length != 0 && bufLen <= 0) {
                bufLen = 2147483647;
            }

            StringBuilder buf = new StringBuilder(bufLen);
            deepToString(a, buf, new HashSet());
            return buf.toString();
        }
    }

    private static void deepToString(Object[] a, StringBuilder buf, Set<Object[]> dejaVu) {
        if (a == null) {
            buf.append("null");
        } else {
            int iMax = a.length - 1;
            if (iMax == -1) {
                buf.append("[]");
            } else {
                dejaVu.add(a);
                buf.append('[');
                int i = 0;

                while(true) {
                    Object element = a[i];
                    if (element == null) {
                        buf.append("null");
                    } else {
                        Class<?> eClass = element.getClass();
                        if (eClass.isArray()) {
                            if (eClass == byte[].class) {
                                buf.append(toString((byte[])element));
                            } else if (eClass == short[].class) {
                                buf.append(toString((short[])element));
                            } else if (eClass == int[].class) {
                                buf.append(toString((int[])element));
                            } else if (eClass == long[].class) {
                                buf.append(toString((long[])element));
                            } else if (eClass == char[].class) {
                                buf.append(toString((char[])element));
                            } else if (eClass == float[].class) {
                                buf.append(toString((float[])element));
                            } else if (eClass == double[].class) {
                                buf.append(toString((double[])element));
                            } else if (eClass == boolean[].class) {
                                buf.append(toString((boolean[])element));
                            } else if (dejaVu.contains(element)) {
                                buf.append("[...]");
                            } else {
                                deepToString((Object[])element, buf, dejaVu);
                            }
                        } else {
                            buf.append(element.toString());
                        }
                    }

                    if (i == iMax) {
                        buf.append(']');
                        dejaVu.remove(a);
                        return;
                    }

                    buf.append(", ");
                    ++i;
                }
            }
        }
    }
    ```

5. `asList()`
- 반환된 List의 크기 변경 불가
- 내부 element의 내용은 변경 가능 : Array.ArrayList (static class) 내부에서 배열을 들고 있음

## 1.7 Comparator와 Comparable
- interface Comparator 혹은 Comparable을 implements한 클래스는 **정렬**을 지원한다고 간주하면 된다.
- 오름차순 정렬이 기본이 된다 : 공백 < 숫자 < 대문자 < 소문자 (Character 클래스의 경우 유니코드로 비교한다.)
- `a.compareTo(b)`
    - a > b  : return 1
    - a == b : return 0
    - a < b  : return -1

## 1.8 HashSet
- 내부 구조
    - Separate Chaining 방식 (Java 1.8 ~ )<br>
    <img src="https://raw.githubusercontent.com/jjjlyn/java-study/main/hash_collision_2.png" width="640" height="360"><br>
    <img src="https://raw.githubusercontent.com/jjjlyn/java-study/main/hash_collision.png" width="640" height="360"><br>
    <img src="https://raw.githubusercontent.com/jjjlyn/java-study/main/hash_structure.png" width="360" height="640">

- 저장 순서가 유지되지 않음 (cf. 순서 보장 : LinkedHashSet을 사용) : HashMap 내부 Buckets라는 `Node<K, V>[]`이 K의 hash를 기준으로 배열 인덱싱을 하기 때문에 add, remove등의 연산에 의해 매번 순서가 달라질 수 있다.
- 중복 허용하지 않음 : `add()`와 `remove()`를 뜯어보자
    - 어떤 식으로 element를 찾아서 추가하는 걸까? : HashSet의 `add()`를 살펴보자
    ```java
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
        HashMap.Node[] tab;
        int n;
        if ((tab = this.table) == null || (n = tab.length) == 0) {
            n = (tab = this.resize()).length;
        }

        Object p;
        int i;
        // 버킷(initial size : 16)에 Node를 집어넣을 때 n - 1 & hash 연산으로 indexing을 한다.
        if ((p = tab[i = n - 1 & hash]) == null) {
            tab[i] = this.newNode(hash, key, value, (HashMap.Node)null);
        } else {
            Object e;
            Object k;
            if (((HashMap.Node)p).hash == hash && ((k = ((HashMap.Node)p).key) == key || key != null && key.equals(k))) {
                e = p;
            } else if (p instanceof HashMap.TreeNode) {
                e = ((HashMap.TreeNode)p).putTreeVal(this, tab, hash, key, value);
            } else {
                int binCount = 0;

                while(true) {
                    if ((e = ((HashMap.Node)p).next) == null) {
                        ((HashMap.Node)p).next = this.newNode(hash, key, value, (HashMap.Node)null);
                        if (binCount >= 7) {
                            this.treeifyBin(tab, hash);
                        }
                        break;
                    }

                    if (((HashMap.Node)e).hash == hash && ((k = ((HashMap.Node)e).key) == key || key != null && key.equals(k))) {
                        break;
                    }

                    p = e;
                    ++binCount;
                }
            }

            if (e != null) {
                V oldValue = ((HashMap.Node)e).value;
                if (!onlyIfAbsent || oldValue == null) {
                    ((HashMap.Node)e).value = value;
                }

                this.afterNodeAccess((HashMap.Node)e);
                return oldValue;
            }
        }

        ++this.modCount;
        if (++this.size > this.threshold) {
            this.resize();
        }

        this.afterNodeInsertion(evict);
        return null;
    }
    ```
    - 어떤 식으로 element를 찾아서 삭제하는 걸까? : HashSet의 `remove()`를 살펴보자
    ```java
    final HashMap.Node<K, V> removeNode(int hash, Object key, Object value, boolean matchValue, boolean movable) {
        HashMap.Node[] tab;
        HashMap.Node p;
        int n;
        int index;
        // 버킷 인덱싱 방식
        if ((tab = this.table) != null && (n = tab.length) > 0 && (p = tab[index = n - 1 & hash]) != null) {
            HashMap.Node<K, V> node = null;
            Object k;
            if (p.hash == hash && ((k = p.key) == key || key != null && key.equals(k))) {
                node = p;
            } else {
                HashMap.Node e;
                if ((e = p.next) != null) {
                    if (p instanceof HashMap.TreeNode) {
                        node = ((HashMap.TreeNode)p).getTreeNode(hash, key);
                    } else {
                        label88: {
                            while(e.hash != hash || (k = e.key) != key && (key == null || !key.equals(k))) {
                                p = e;
                                if ((e = e.next) == null) {
                                    break label88;
                                }
                            }

                            node = e;
                        }
                    }
                }
            }

            Object v;
            if (node != null && (!matchValue || (v = ((HashMap.Node)node).value) == value || value != null && value.equals(v))) {
                if (node instanceof HashMap.TreeNode) {
                    ((HashMap.TreeNode)node).removeTreeNode(this, tab, movable);
                } else if (node == p) {
                    tab[index] = ((HashMap.Node)node).next;
                } else {
                    p.next = ((HashMap.Node)node).next;
                }

                ++this.modCount;
                --this.size;
                this.afterNodeRemoval((HashMap.Node)node);
                return (HashMap.Node)node;
            }
        }

        return null;
    }
    ```

    - 중복 여부를 판단하는 기준? : `equals()` && `hashCode()`를 사용
    ```java
    // 보조 해시 함수 (key의 해시값을 변형하여 해시 충돌을 줄임)
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
    ```