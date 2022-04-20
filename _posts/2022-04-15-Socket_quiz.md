---
layout: post
title: "게임서버 공부_socket_함수 퀴즈 / 과제"
description: "윈속 퀴즈 / 과제"
date: 2022-04-20
tags: [winsock, socket, gameserver]
comments: true
share: true
---

## 퀴즈
- IPv4에서 SOCKADDR_IN의 각 요소와, 어떻게 초기화 할지, 각 요소의 정렬 방식
- string을 ip주소로 변환하는 함수는?, 입력과 출력의 정렬방식 ->  에러시 어떤 함수로?
- ip 주소를 string으로 변환하는 함수는?, 입력과 출력의 정렬방식 -> 에러시 어떤 함수로?
- SOCKADDR의 각 요소를 출력하기 위해 필요한 함수
- 개본적인 소켓 서버의 흐름(반복문 포함, 여러 클라이언트를 받음)
- bind()의 역할, 인자, 반환값.
- listen()의 역할, 인자, 반환값.
- accept()의 역할, 인자, 반환값.
- recv()의 역할, 인자, 반환값.
- recv()가 언제 반환하는지, 반환값의 상황에 따른 반환값
- send()의 역할, 인자, 반환값.
- send()가 언제 반환하는지, 주의할점은 무엇인지.

## 과제 1
### 1.우선 유니코드 형태로 변경.
[FormatMessageW ](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-formatmessagew)
, [유니코드 활용](https://lunikism.com/entry/C-%EC%9C%A0%EB%8B%88%EC%BD%94%EB%93%9C%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C-%EB%8B%AC%EB%9D%BC%EC%A7%80%EB%8A%94-%EC%A3%BC%EC%9A%94-%ED%95%A8%EC%88%98), [InetPtonW ](https://docs.microsoft.com/en-us/windows/win32/api/ws2tcpip/nf-ws2tcpip-inetptonw)

### 2.과제 수행. 