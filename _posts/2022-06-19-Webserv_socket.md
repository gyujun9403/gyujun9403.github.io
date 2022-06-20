---
layout: post
title: "Webserv_socket"
description: "42서울 과제 Webserv과제 수행을 위한 소켓 공부"
date: 2022-6-19
tags: [42Seoul, webserv, socket]
comments: true
share: true
---

- select는 I/O multiplexing함수로, block/non-block과는 상관 없다.
    - I/O multiplexing함수는 소켓 관련 함수를 호출할 때, 성공할 시점으로 동기화?블럭? 시켜주는 함수이다.
    - block소켓, 관련 함수 : blocking되는걸 막아줌
    - non-block소켓, 관련 함수 : WOULDBLOCK같은 재차 확인해야하는 에러 발생을 막는다.
- read set에 fd들을 등록하고, select가 반환을 하면 read set에는 읽기 가능한 fd들만 남고, 나머지는 사라진다.
    - fd가 사라졌는지, 남아있는지 확인하는게 FD_ISSET메크로이다.
    - read가능했던 소켓은 처리가 끝나도 fd_set에 남아 있으니, fd_set을 0으로 초기화(FD_ZERO) 시켜줘야하한다.
    - fd_set을 매번 새로 등록 해주어야 한다(FD_SET).
- listen소켓은 read set에 등록한다.
    - listen소켓이 read set에 있으면 accept하여 클라 정보를 받아오고 세션에 추가.
- 사용자 정보를 따로 관리해야 한다. 
    - 보통 fd와 buffer, recvbyte, sendbyte가 있다...
- 비동기에서 recv할 때, 상대방 수신 버퍼의 문재 때문에 상대가 보낸 데이터의 일부만 도착 할 수 있다
    - 커널단에서 되도록이면 한번에 보내려고 해서 자주 발생하는 문제는 아님.
- TCP소켓통신 시, stream 특성상 recv할 때, 데이터가 연달아서 올 수 있다.
    - 10byte, 5byte각각 한번에 와서 recv시 15btye로 읽힐수 있다.
    - 이 경우, 일단 버퍼에 다 옮기고 헤더를 읽어서 전체 헤더와 바디의 크기를 파악하여 자른다.
- 단점 : fd_set의 크기가 제한적임(배열 형태로 비트마스킹을 사용하고 있어서...). fd_set을 여러개 만들어서 돌려도 되지만...

세션 : 

참고 : 특정 소켓만 제거하는건 FD_CLR
