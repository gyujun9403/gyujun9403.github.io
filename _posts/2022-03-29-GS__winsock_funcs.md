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

socket()과 WSAsocket()의 차이점.
<blockquote>
----------------------------------------------

안녕하세요 TCP/IP 저자 조경민입니다.

먼저 `socket()` 대신 `WSASocket()`같이 BSD타입의 socket 함수를 사용하기 보다 Winsock API를 사용하는 이유는 윈도우즈에서만 동작 가능한 확장 기능을 사용하기 위해서 사용하는 것입니다.

리눅스와 윈도우 호환되는 범용 BSD 타입의 socket()의 함수 원형은 다음 처럼
```
SOCKET socket (
int af,
int type,
int protocol
);
```
밖에 지원 안되지만, Winsock의 `WSASocket()`의 함수 원형의 경우
```
SOCKET WSASocket (
int af,
int type,
int protocol,
LPWSAPROTOCOL_INFO lpProtocolInfo,
GROUP g,
DWORD dwFlags
);
```
더 많은 인자를 지원하는 것만 보셔도 윈속 확장 기능을 더 사용하기 위해서 BSD의 `socket()` 함수는 호환성을 위해서 남겨두고 Winsock API를 따로 만든 이유를 알 수 있습니다.
참고로, Winsock의 `socket()`함수는 기본적으로 소켓 속성을 오버랩드 I/O로 주긴 합니다. 그러나 Asynchronous I/O를 구현 하기 위해서 BSD의 select() 뿐 아니라 Winsock에서는 윈도우즈의 메시지 메커니즘이나 Win32 Event 메커니즘을 이용하는 WSAAsyncSelect 또는 WSAEventSelect를 추가로 지원하며 윈도우즈 플랫폼에서 `select()`보다 더 효율적인 프로그래밍을 가능하게 합니다.

**즉, Winsock API는 BSD 함수보다 윈도우즈의 특성에 맞도록 보다 효율적인 성능을 내도록 하는 함수인 것이죠.**

다음 세번째 인자의 protocol의 경우 BSD나 Winsock이나 af가 INET(인터넷)이며, type이 SOCK_STREAM(연결지향)이면protocol이 0이여도 IPPROTO_TCP로 동작됩니다. 디폴트 프로토콜을 선택한다는 의미이지요. 하지만, 명시해주는 것이 좋을 것 같습니다.

예로, AF_INET에서 RAW 타입으로 ICMP 프로토콜을 사용한다면
socket(AF_INET,SOCK_RAW,IPPROTO_ICMP) 를 해주어야 합니다.
프로토콜 값 및 af, type은 IPX(AF_IPX)나 적외선통신(AF_IRDA), 블루투스(AF_BTH) 등 마다
각각 프로토콜 및 type이 정해져 있습니다.
</blockquote>

### 소켓 주소 구조체
```cpp
#include <ws2def.h> //winsock2.h에 포함됨.
typedef struct sockaddr {
        ushort  sa_family;
        char    sa_data[14];
} SOCKADDR;

typedef struct sockaddr_in {  // 16byte
        short   sin_family;
        u_short sin_port;
        struct  in_addr sin_addr;
        char    sin_zero[8];
} SOCKADDR_IN;

typedef struct sockaddr_in6 { // 28byte
        short   sin6_family;
        u_short sin6_port;
        u_long  sin6_flowinfo;
        struct  in6_addr sin6_addr;
        u_long  sin6_scope_id;
} SOCKADDR_IN6;

typedef struct sockaddr_in6 SOCKADDR_IN6;
typedef struct sockaddr_in6 *PSOCKADDR_IN6;
typedef struct sockaddr_in6 FAR *LPSOCKADDR_IN6;
```
- sockaddr : 기본적인 소켓 구조체. sockaddr_in과 sockaddr_in6의 기본이지만, 실제로는 잘 사용하지 않는다.
  - sa_family : 주소체계를 나타냄(TCP/IP : AF_INET or AF_INET6).
- sockaddr_in : IPv4에서 사용하는 소켓 구조체
  - sin_family : 주소체계 의미. AF_INET값을 가짐.
  - sin_port : 포트번호. u_short(16bit)값.
  - sin_addr : IP주소. 32bit구조체.
  - sin_zero : Reserved for system use. A WSK application should set the contents of this array to zero.
- sockaddr_in6 : IPv6에서 사용하는 소켓 구조체

소켓 주소 구조체를 사용하는 경우 sockaddr_in이던 sockaddr_in6이던 SOCKADDR포인터형(SOCKADDR *)으로 cast후 사용해야 한다.

```cpp
SOCKADDR_IN addr;
...
SocketFunc(..., (SOCKADDR *)addr, sizeof(addr), ...);
...
```

윈속 초기화 -> 소켓 생성 -> 통싱 -> 소켓 닫음 -> 윈속 종료