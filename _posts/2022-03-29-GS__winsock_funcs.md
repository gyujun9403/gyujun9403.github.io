---
layout: post
title: "게임서버 공부_winsock_func"
description: "책 내용 + 공부/실습한거"
date: 2022-03-29
tags: [winsock, socket, gameserver]
comments: true
share: true
---

## 윈속 함수
```cpp
#include "winsock2.h"
int WSAAPI WSAStartup(
  [in]  WORD      wVersionRequested,
  [out] LPWSADATA lpWSAData
);
```
프로그램에서 사용할 윈속 버전을 요청하여, 윈속 라이브러리(WS2_32.DLL)을 초기화하는 함수. 실패시WS2_32.DLL는 메모리에 로드되지 않는다.
- 인자
    - wVersionRequested : 프로그램이 요구하는 최상위 윈속 버전. 상위 8bit에 부버전, 하위 8bit에 주버전을 전달한다(MAKEWORD(주, 부)).
    - lpWSAData : lpWSAData구조체를 전달하면 윈속 구현 정보를 담아준다.
- 반환
    - 성공 : 0 / 실패 : errno 직접 반환

```cpp
#include "winsock2.h"
int WSAAPI WSACleanup();
```
윈속 사용을 중지하고, 관련 리소스를 반환하는 함수.
- 반환
    - 성공 : 0 / 실패 : errno 직접 반환

## 소켓 생성과 닫기

### 소켓 생성
```cpp
#include "winsock2.h"
SOCKET WSAAPI socket(
  [in] int af,
  [in] int type,
  [in] int protocol
);
```
SOCKET : 소켓 디스크립터, WSAAPI : Microsoft's socket API

사용자가 요청한 프로토콜을 이용하여 내부적 리소스가 할당되고, 이에 접근할수 있는 핸들러 값. 소켓 디스크립터라고 부르며, 소켓 함수에 인자로 넘겨준다.
- 인자
  - af : 지정된 주소 체계(TCP/UDP IPv4 : 2)
  - type : 지정된 소켓 타입(TCP : SOCK_STREAM / UDP : SOCK_DGRAM)
  - protocol : 사용될 소켓타입(TCP : IPPROTO_TCP/ UDP : IPPROTO_UDP)

이때 af : 2, type : SOCK_STREAM이거나 SOCK_DGRAM인 경우 protocol은 0 가능.
- 반환
  - 성공 : 소켓 자원 할당 및 디스크립터 반환
  - 실패 : INVALID_SOCKET error

### 소켓 닫기
```cpp
#include "winsock2.h"
int WSAAPI closesocket(
  [in] SOCKET s
);
```
인자로 들어온 소켓 디스크립터에 해당하는 소켓을 닫고 리소스를 반환하는 함수.
- 인자
  - s : 소켓 디스크립터
- 반환
  - 성공 : 0 / 실패 : SOCKET_ERROR


윈속 초기화 -> 소켓 생성 -> 통싱 -> 소켓 닫음 -> 윈속 종료