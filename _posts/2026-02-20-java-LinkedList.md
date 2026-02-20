---
title: 자바 연결 리스트와 배열
date: 2026-02-20 20:41:00 +0900
categories: [Java, Data Structure]
tags: [java, study, 자료구조, 연결리스트]
---

![배열과 연결리스트 그림](../assets/img/posts/list-linkedlist.png)


배열은 영화관 좌석 같이 따닥따닥 붙어 있고, 번호가 매겨져 있음.<br>
메모리상 연속적이라 누군가를 끼워 앉게 하려면 그 뒤 모든 사람은 한칸 씩 뒤로 움직여야함.<br>
이런 과정에서 불필요한 연산을 피하기 위해서 연결 리스트를 사용.

연결 리스트란?<br>
첫번째 장소에 데이터와 다음 장소로 가는 약도가 있음.
떨어져 있어도 상관이 없음. 즉, 메모리상 비연속적임.<br>
따라서 메모리상 데이터를 추가하거나 삭제할때 더 효율적임.

```
class Node {
	int data;
	Node next; // 다음번 주소를 가리키는 참조 변수
	
	public Node(int d) {
		this.data = d; //이 노드가 가지고 있는 데이터.
		this.next = null; //현재 노드는 다음으로 가는 노드를 가리키고 있지 않음.
	}
}
```

이런 식으로 작성

```	
public class Main {
    public static void main(String[] args) {
        Node node1 = new Node(10);
        Node node2 = new Node(20);

		node1.next = node2; //노드1의 다음이 node2를 가리킴.
    }
}
```
만일 node1.next.data이라면 node2.data와 같다.



예시 코드
```
class Node {
    int data;
    Node next;
    
    public Node(int d) {
        this.data = d;
        this.next = null;
    }
}

public class Main {
    public static void main(String[] args) {
        // 노드 생성
        Node node1 = new Node(10);
        Node node2 = new Node(20);
        Node node3 = new Node(30);

        // 연결
        node1.next = node2;
        node2.next = node3;
        // node3.next는 자동으로 null. 왜냐하면 기본 생성자에서 this.next = null이니깐.

        // 확인
        Node current = node1;
        while (current != null) {
            System.out.println(current.data);
            current = current.next; // 다음 칸으로
        }
    }
}
```

다만 위 코드는 그냥 이해를 위한 예시일뿐 코드 짤때는 끙끙거리지 말고, 그냥 ArrayList와 LinkedList쓰면 된다.

그럼 배열은 언제 쓰냐?<br>
데이터가 자주 바뀌거나 삭제 되지 않고, 몇번 데이터가 필요한 조회가 많을때.<br>
사실상 대부분의 상황에서 쓴다는 뜻.<br>
ex)페이지네이션<br>

50번째 데이터 조회하고 싶으면 50번째 데이터를 바로 찾을 수 있어서 O(1) 만큼 걸림.
근데 연결리스트에서 50번째 데이터 찾아가려면? O(n) 만큼 걸릴 수 있음.<br>
참고로 O()는 빅오표기법으로, 작을수록 빠르고 좋다.<br>

연결 리스트는 앞서 언급한대로, 중간 삽입이나 삭제가 많은 경우.<br>
ex)플레이리스트