---
title: 자바 연결 리스트와 배열
date: 2026-02-20 20:41:00 +0900
categories: [Java, Data Structure]
tags: [java, study, 자료구조, 큐, 스택]
---

오늘은 큐와 스택에 대해서 간단하게 알아봅시다.

![스택, 큐 그림](../assets/img/posts/stack-queue.jpg)

스택은 그냥 프링글스 통이라고 생각하면 됩니다.<br>
Last in First Out줄여서 LIFO인데,
위에서 하나씩 꺼내먹죠.(pop)<br>
넣을때도 밑에서 못넣죠.
위에서 넣어야 합니다.(push)<br>
구멍 내서 넣는다는 말을 하지 맙시다.
참고로, 우리가 많이쓰는 ctrl + z 같은거도 스택입니다.<br>

큐는 그냥 줄서기에요.<br>
놀이 공원 놀러가면 줄서는데 나보다 뒤에 선 사람이 먼저 들어가면 화나잖아요?<br>
그럴때 쓰는게 큐입니다.<br>
이 친구는 First in First Out이라고, 줄여서 FIFO라고 합니다.<br>
뒤에서 줄서고(offer) 앞에서 나가요(poll)
주로 이제 게임에서 큐 돌린다고 많이 말하잖아요. 그럴때 씁니다.

자바에서는 스택과 큐 안에 정수를 넣는다고 가정하면
```java
//스택
Stack<Integer> stack = new Stack<>();
stack.push(1); //stack에 1 넣음
stack.push(2); //stack에 2 넣음
stack.pop(); // 2꺼냄
stack.peek(); // 맨위에 뭐가 있는지 확인함. 지금은 1이 있음.


//큐
Queue<Integer> q = new LinkedList<>();
q.offer(1); //1넣음
q.offer(2); //2넣음
q.poll(); // 1 튀어 나옴
```
이렇게 가능합니다.

근데 요즘은
```java
ArrayDeque<Integer> arrayDeque = new ArrayDeque<>();
```
이걸 많이 씁니다.<br>
Array는 배열이고, Deque은 큐인데, 양방향이 뚫려있는 거라고 생각하면 됩니다.

```java
ArrayDeque<Integer> ad = new ArrayDeque<>();
        
ad.offerFirst(1);// 맨앞에 1넣음
ad.offerLast(2); // 맨뒤에 2 넣음
ad.poll();
ad.pop();
```
양방향이 뚫려있으니깐, 앞뒤로 넣고 빼는게 가능합니다.<br>
왜 이걸 더 많이 쓰냐면, 얘는 큐도 되고 스택도 됩니다. 편하죠?<br>
그리고 Vector를 상속받지 않아서 더 빨라요. <br>
자바 초창기에 만들어진 Vector라는 클래스가 있는데 이 친구는 데이터가 꼬일까봐 락을 거는데 이거 때문에 느려요.

엥? 그러면 큐도 되고 스택도 되면 나중에 여러명이서 코드짜다가 문제 생기는거 아니에요? 라고 물으실 수 있는데, 맞습니다.<br>

특히나
```java 
ad.poll();
ad.pop();
```
이런식으로 짜버리면 큐야 스택이야? 하면서 엄청 헷갈리겠죠.

그래서 명시적으로
```java
Queue<Integer> queue = new ArrayDeque<>();
queue.offer(1);
queue.poll();
//또는
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1);
stack.pop();
```
이렇게 인터페이스로 타입을 묶어 누구나 알 수 있게 스택인지 큐인지 써놓고, 그에 맞게 사용하면 될것같습니다.<br>
주석 안써도 알아보기 쉽겠죠?<br>
근데 stack이 아니라 왜 deque이냐고요?<br>
stack은 오래되서 ArrayDeque이랑 일단 호환이 안되고, 자바 doc에서 Deque을 LIFO 스택으로 사용할 것을 권장했습니다.<br>

(참고 : https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Deque.html)<br>

isEmpty()같은 문법도 있는데, 이 글은 문법 보다는 그냥 큐랑 스택이 어떻게 굴러가는지에 대해서 알아보는 글이니깐, ai에게 물어보거나 구글 검색해봅시다.<br>
(사실 저도 문법 맨날 까먹어서 물어봅니다.)