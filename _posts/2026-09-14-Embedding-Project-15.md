---
title: 락은 최소한으로
date: 2026-09-14 13:30:00 +0900
categories: [Project, newsFinder]
tags: [java, spring, study]
---


## 지난 시간

[저번 시간](/posts/Embedding-Project-14/)에 다른 동시 입력시에 다른 스레드도 L1 캐시값을 그대로 받아서 더 빠르게 돌려주는 걸 해본다 했으니, 오늘 해보도록 합시다. 오늘은 길지 않습니다. 길어보이려고 코드도 안접을거에요.

## 진짜 별거 없음

`queryVectorCacheRepository.save(cache);`를 따로 빼줘야 합니다.<br>
근데 따로 빼주면 코드 어딘가에 빨간줄이 생기는데, 어디일까요? 

`cache`가 빨게집니다.<br> 
간단하게 그냥 락 밖에서 선언해주면 끝입니다.

그리고 저번에 만들었던 try/catch 구분이랑 시간 측정 하는 부분도 락 밑으로 따로 빼주면 됩니다.

```java

		    QueryVectorCache cache; // 따로 lock 밖에서 선언

            synchronized (lock) {
                // 중략
                }
            long saveStart = System.nanoTime();
            try {
                queryVectorCacheRepository.save(cache);
            } catch (DataIntegrityViolationException e) {
                path = "API_miss_dup";
            }
            saveTime = (System.nanoTime() - saveStart) / 1_000_000.0;
```

저는 이렇게 수정을 해 봤습니다.

저번 코드는 else로 들어가면, else 안에서 전부 처리하기 전까지 뒤에 후발주자가 대기했어야 하는데, 지금은 else에서 딱 필요한 만큼만 처리하고, 자리를 비워주기 때문에 더 빠르게 처리가 가능하겠죠.

```text
2026-09-14T12:05:26.505+09:00 DEBUG 3304 --- [newsFinder] [nio-8080-exec-9] o.j.n.queryVector.QueryVectorService     : path=L1_hit_by_lock_wait term=마늘치키 save_ms=0.0 api_ms=0.0 total_ms=183.9091
2026-09-14T12:05:26.529+09:00 DEBUG 3304 --- [newsFinder] [nio-8080-exec-8] o.j.n.queryVector.QueryVectorService     : path=API_miss term=마늘치키 save_ms=24.1514 api_ms=179.6591 total_ms=217.7174
```

끝난 시각을 보면, `05:26.505 ` `05:26.529`인데, 24ms로 갈라졌죠.
전편에서는 끝난 시간이 `34:24.704` 으로 완전히 똑같았거든요.

나중에 들어간걸 보니 `save_ms=0.0 api_ms=0.0`이죠.<br> 
나머지는 먼저 앞서간 스레드 기다리는 비용이에요.

하지만, 체감이 그렇게 크진 않을 수도 있습니다. 일단 API 호출에 걸리는 시간은 바뀌지 않습니다.<br>
애초에 먼저 들어간 친구가 그 값을 받아야 뒤에 친구도 그 값을 그대로 쓰니깐요.<br>
결국은, `2026-09-14T12:05:26.505`,`2026-09-14T12:05:26.529` 로그를 보시면 알겠지만 지금 차이가 DB에 저장하는 24ms의 차이인데, 24ms 정도면 찰나의 불과하니깐요.<br>
그래도, 제 희망 사항이지만 DB가 느리거나, 이런 자잘한 시간이 쌓이면 느리게 느껴질 수도 있겠죠.

### 커밋

[0433298](https://github.com/JinsanPark/NewsFinder/commit/04332983eba9bf88667aaa7d5b1cbc70fded69f5)


## 다음 시간

이제는 캐시에서 좀 벗어나려고 합니다.<br>
아직 다듬어야 할 부분도 많지만, 그렇다고 이 부분만 파면 진짜 한 30일은 이것만 가능할거 같네요.

지금 상황에서, API 서버가 뻑이 나버리면, 캐시에 없는 값들은 평생 대기를 해야겠죠?<br>
다음 시간에는 그러면 Voyage 서버가 맛이 갔을때 대처를 한번 해 봅시다.