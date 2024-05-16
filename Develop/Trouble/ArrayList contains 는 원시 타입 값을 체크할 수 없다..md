
ArrayList 의 contains 함수는 해당 값이 현재 List 에 있는지 확인하는 함수이다. 
그런데 알고리즘 문제를 푸는 중 int[] 타입을 Arrays.asList를 이용하여 변환 후 ArrayList의 contains 함수를 이용하여 int 타입의 값이 확인이 안되는 것이었다. 확인해본 결과 contains 내부에서 값을 체크할 시 equals 함수를 사용하고 있었다. 


```java
public boolean contains(Object o) {  
    return indexOf(o) >= 0;  
}

public int indexOf(Object o) {  
    return indexOfRange(o, 0, size);  
}  
  
int indexOfRange(Object o, int start, int end) {  
    Object[] es = elementData;  
    if (o == null) {  
        for (int i = start; i < end; i++) {  
            if (es[i] == null) {  
                return i;  
            }  
        }    } else {  
        for (int i = start; i < end; i++) {  
            if (o.equals(es[i])) {   // 바로 이곳
                return i;  
            }  
        }    }    return -1;  
}
```

LinkedList 의 경우 Node 의 next 노드와 equals 함수를 이용해서 contains 상태를 확인하고 있었다. Arrays.asList 함수의 경우 int[] 타입을 변환 시킨 것이다 보니 값들이 원시타입이어서 equals 함수를 이용할 수 없어 false 가 리턴된 것으로 예상된다. 

```java 
// int[] arr 일 경우
List<Integer> list = Arrays.stream(arr).boxed().collect(Collectors.toList());
```

원시타입이 아닌 Integer 타입으로 위와 같이 변환하여 사용하는 것으로 해결하였다.