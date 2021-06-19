# Chapter 11 컬렉션 프레임워크

## 1.6 Arrays
1. 배열의 복사 `copyOf()`
    - **얕은 복사** : 참조형의 필드(element)는 주소값만 복사되기 때문에 원본과 같은 객체를 참조하게 됨
2. `fill()`, `setAll()`
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
    - 후에 나오는 `compareTo()`메소드를 기준으로 정렬한다.

- Arrays에서 채택한 search 알고리즘
    - binarySearch는 반드시 오름차순으로 정렬되어야 정확한 값이 나온다.

4. `equals()`, `toString()`


5. `asList()`

## 1.7 Comparator와 Comparable
- interface Comparator 혹은 Comparable을 implements한 클래스는 **정렬**을 지원한다고 간주하면 된다.
- 오름차순 정렬이 기본이 된다 : 공백 < 숫자 < 대문자 < 소문자 (Character의 경우 유니코드로 비교한다.)
- `a.compareTo(b)`
    - a > b  : return 1
    - a == b : return 0
    - a < b  : return -1

## 1.8 HashSet
- 중복 허용
- 순서는 보장하지 않음 (cf. 순서 보장 : LinkedHashSet을 사용한다.)

