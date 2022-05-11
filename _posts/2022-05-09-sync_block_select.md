---
layout: post
title: "게임서버 공부_socket_동기, 블럭 그리고 socket"
description: "socket 모델 이해를 위한 동기와 블럭 공부"
date: 2022-05-24
tags: [winsock, socket, gameserver, synchronization, blocking]
comments: true
share: true
---

동기/비동기, 블로킹/논블로킹은 관점의 차이

동기(**Sync**hronous) / 비동기(**Async**hronous) : **순서**, **상황**, **필요성**
- 순서가 필요하면 동기 / 함수의 반환값이 필요하여 블럭/논블럭이던 대기하고 있으면 동기.
- 순서 상관없으면(독립적) 비동기 / 함수를 부르고 하던 일 하다가(병렬적), 콜백함수를 통해 결과값을 전달받으면 비동기.

동기화 함수 : 독립된 스레드/프로세스간에 동기성(순서)를 가질수 있게 하는 함수들 -> 

블로킹(**Blocking**)/논블로킹(**Nonblocking**) : **제어권**, **동작**, **I/O**
- 블로킹 :제어권이 넘어가서 호출한 스레드는 대기(BLOCK)하는 것.
- 논블로킹 :제어권을 그대로 가지고 있음. 함수를 호출한 프로세스/스레드는 다음 코드로 진행하고, 호출된 함수는 병렬적으로 동작.

동기/비동기, 블로킹/넌블로킹 실제

read()함수로 데이터를 읽어서 파싱하는 코드
- I/O로부터 데이터를 읽어올때까지 기다리므로 블로킹
- 데이터를 받아와야 이후 파싱이 가능하므로 동기적인 코드

write()함수로 처리한 데이터를 파일에 입력.
- 통상적인 write()는 쓰기 작업이 완료될때까지 block된다.
- 그러나 쓰고자하는 fd가 non-blocking모드라면 write()또한 non-blocking으로 동작한다.
- 또한 block/non-block과는 별개로 쓰기 완료한 코드는 이후 동작에 영향을 미치지 않으므로 비동기적이다.

[동기/비동기식 처리](https://seohs.tistory.com/432)

[각각의 관점 참조한 블로그](https://velog.io/@nittre/%EB%B8%94%EB%A1%9C%ED%82%B9-Vs.-%EB%85%BC%EB%B8%94%EB%A1%9C%ED%82%B9-%EB%8F%99%EA%B8%B0-Vs.-%EB%B9%84%EB%8F%99%EA%B8%B0)

[리눅스에서의 block/non-block](https://www.linuxtoday.com/blog/blocking-and-non-blocking-i-0/)

non-blocking과 멀티플랙싱