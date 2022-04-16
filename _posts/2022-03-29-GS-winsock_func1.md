---
layout: post
title: "게임서버 공부_winsock_구조체/변환func"
description: "책 내용 + 공부/실습한거"
date: 2022-03-29
tags: [winsock, socket, gameserver]
comments: true
share: true
---

## 자주 사용하는 구조체/초기화 모아보기
SOCKADDR_IN가 사용되는 곳 -> 포트 바인딩 및 클라이언트 accept



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
    - `wVersionRequested` : 프로그램이 요구하는 최상위 윈속 버전. 상위 8bit에 부버전, 하위 8bit에 주버전을 전달한다(MAKEWORD(주, 부)).
    - `lpWSAData` : `lpWSAData구조체`를 전달하면 윈속 구현 정보를 담아준다.
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
  - `af` : 지정된 주소 체계(TCP/UDP IPv4 : 2)
  - `type` : 지정된 소켓 타입(TCP : SOCK_STREAM / UDP : SOCK_DGRAM)
  - `protocol` : 사용될 소켓타입(TCP : IPPROTO_TCP/ UDP : IPPROTO_UDP)

이때 `af` : `2`, `type` : `SOCK_STREAM`이거나 `SOCK_DGRAM`인 경우 `protocol`은 `0` 가능.
- 반환
  - 성공 : 소켓 자원 할당 및 디스크립터 반환
  - 실패 : INVALID_SOCKET error

### 소켓 닫기
```cpp
#include "winsock2.h"
int WSAAPI closesocket(
  [in] SOCKET s
);

SOCKET WSAAPI WSASocketA(
  [in] int                 af,
  [in] int                 type,
  [in] int                 protocol,
  [in] LPWSAPROTOCOL_INFOA lpProtocolInfo,
  [in] GROUP               g,
  [in] DWORD               dwFlags
);
```
인자로 들어온 소켓 디스크립터에 해당하는 소켓을 닫고 리소스를 반환하는 함수.
- 인자
  - s : 소켓 디스크립터
- 반환
  - 성공 : 0 / 실패 : SOCKET_ERROR

`socket()`과 `WSAsocket()`의 차이점.
<blockquote>
----------------------------------------------

안녕하세요 TCP/IP 저자 조경민입니다.

먼저 `socket()` 대신 `WSASocket()`같이 BSD타입의 socket 함수를 사용하기 보다 Winsock API를 사용하는 이유는 윈도우즈에서만 동작 가능한 확장 기능을 사용하기 위해서 사용하는 것입니다.

리눅스와 윈도우 호환되는 범용 BSD 타입의 socket()의 함수 원형은 다음 처럼
```cpp
SOCKET socket (
int af,
int type,
int protocol
);
```
밖에 지원 안되지만, Winsock의 `WSASocket()`의 함수 원형의 경우
```cpp
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
참고로, Winsock의 `socket()`함수는 기본적으로 소켓 속성을 오버랩드 I/O로 주긴 합니다. 그러나 Asynchronous I/O를 구현 하기 위해서 BSD의 `select()` 뿐 아니라 Winsock에서는 윈도우즈의 메시지 메커니즘이나 Win32 Event 메커니즘을 이용하는 WSAAsyncSelect 또는 WSAEventSelect를 추가로 지원하며 윈도우즈 플랫폼에서 `select()`보다 더 효율적인 프로그래밍을 가능하게 합니다.

**즉, Winsock API는 BSD 함수보다 윈도우즈의 특성에 맞도록 보다 효율적인 성능을 내도록 하는 함수인 것이죠.**

다음 세번째 인자의 protocol의 경우 BSD나 Winsock이나 `af`가 INET(인터넷)이며, `type`이 SOCK_STREAM(연결지향)이면 `protocol`이 0이여도 IPPROTO_TCP로 동작됩니다. 디폴트 프로토콜을 선택한다는 의미이지요. 하지만, 명시해주는 것이 좋을 것 같습니다.

예로, AF_INET에서 RAW 타입으로 ICMP 프로토콜을 사용한다면
`socket(AF_INET,SOCK_RAW,IPPROTO_ICMP)` 를 해주어야 합니다.
프로토콜 값 및 `af`, `type`은 IPX(AF_IPX)나 적외선통신(AF_IRDA), 블루투스(AF_BTH) 등 마다
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
        short   sin_family;       // 호스트바이트 정렬
        u_short sin_port;         // 네트워크 바이트 정렬
        struct  in_addr sin_addr; // 네트워크 바이트 정렬
        char    sin_zero[8];      // 호스트바이트 정렬
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
- `sockaddr` : 기본적인 소켓 구조체. `sockaddr_in`과 `sockaddr_in6`의 기본형이지만, 실제로는 잘 사용하지 않는다.
  - `sa_family` : 주소체계를 나타냄(TCP/IP : AF_INET or AF_INET6).
- `sockaddr_in` : IPv4에서 사용하는 소켓 구조체
  - `sin_family` : 주소체계 의미. AF_INET값을 가짐.
  - `sin_port` : 포트번호. u_short(16bit)값.
  - `sin_addr` : IP주소. 32bit구조체.
  - `sin_zero` : Reserved for system use. A WSK application should set the contents of this array to zero.
- `sockaddr_in6` : IPv6에서 사용하는 소켓 구조체

소켓 주소 구조체를 사용하는 경우 sockaddr_in이던 sockaddr_in6이던 SOCKADDR포인터형(`SOCKADDR *`)으로 cast후 사용해야 한다.

```cpp
SOCKADDR_IN addr;
...
SocketFunc(..., (SOCKADDR *)addr, sizeof(addr), ...);
...
```

### 바이트 정렬 함수
#### 호스트->네트워크(송신) 데이터 정렬
```cpp
#include <winsock2.h>
u_short WSAAPI htons(
  [in] u_short hostshort
);
u_long WSAAPI htonl(
  [in] u_long hostlong
);
int WSAAPI WSAHtons(
  [in]  SOCKET  s,
  [in]  u_short hostshort,
  [out] u_short *lpnetshort
);
int WSAAPI WSAHtonl(
  [in]  SOCKET s,
  [in]  u_long hostlong,
  [out] u_long *lpnetlong
);
```
hton : host to network. 입력된 데이터를 네트워크 바이트 정렬로 변환. 네트워크 바이트 정렬()로 변환되면 네트워크 환경에 내보내기 위해 소켓함수 등에 사용.
- 인자
  - `hostshort` / `hostlong` : 변환할(보낼) 데이터.
  - `s` : 소켓 디스크립터
  - `lpnetshort` / `lpnetlong` : 변환된 데이터가 저장되는 포인터
- 반환
  - 일반 : 네트워크 바이트 정렬로 변환된 데이터
  - `WSA` : 성공시 0, 에러시 SOCKET_ERROR(WASGetLastError로 구체적인 정보)

#### 네트워크->호스트(수신) 데이터 정렬
```cpp
#include <winsock2.h>
u_short WSAAPI ntohs(
  [in] u_short netshort
);
u_long WSAAPI ntohl(
  [in] u_long netlong
);
int WSAAPI WSANtohs(
  [in]  SOCKET  s,
  [in]  u_short netshort,
  [out] u_short *lphostshort
);
int WSAAPI WSANtohl(
  [in]  SOCKET s,
  [in]  u_long netlong,
  [out] u_long *lphostlong
);
```
ntoh : network to host. 입력된 데이터를 호스트(사용자)의 바이트 정렬로 변환. 네트워크로 받은 바이트정렬을 사용자의 환경에 맞는 바이트 정렬로 변경할때 사용 사용.
- 인자
  - `netshort` / `netlong` : 변환할(받은) 데이터.
  - `s` : 소켓 디스크립터
  - `lphostshort` / `lphostlong` : 변환된 데이터가 저장되는 포인터
- 반환
  - 호스트 환경 바이트 정렬로 변환된 데이터
  - `WSA` : 성공시 0, 실패시 SOCKET_ERROR(WASGetLastError로 구체적인 정보)

참고 : 네트워크 바이트 정렬된걸 다시 네트워크 바이트 정렬하면(hton) 안된다

### IP주소 변환
#### String to Address
```cpp
unsigned long WSAAPI inet_addr(
  const char *cp
);
INT WSAAPI WSAStringToAddressA(
  [in]           LPSTR               AddressString,
  [in]           INT                 AddressFamily,
  [in, optional] LPWSAPROTOCOL_INFOA lpProtocolInfo,
  [out]          LPSOCKADDR          lpAddress,
  [in, out]      LPINT               lpAddressLength
);
```
문자열 형태의 IPv4주소를 받으면 이를 실제 사용할수 있는 IP Adress로 변환하는 함수. WSAStringToAddressA는 IPv6도 가능.
- 인자
  - `cp` / `AddressString` : 변환할 IP String
  - `AddressFamily` : `AF_INET` 또는 `AF_INET6`
  - `lpProtocolInfo` : `NULL`
  - `lpAddress` : `SOCKADDR_IN` 또는 `SOCKADDR_IN6`형의 구조체. 변환됨
  - `lpAddressLength` : `sizeof(SOCKADDR_IN)` 또는 `sizeof(SOCKADDR_IN6)`
- 반환
  - 변환된 IP(네트워크 바이트 정렬)
  - 성공시 0 / 실패시 SOCKET_ERROR(`WASGetLastError`로 구체적인 정보)
#### Address to String
```cpp
char *WSAAPI inet_ntoa(
  in_addr in
);
INT WSAAPI WSAAddressToStringA(
  [in]           LPSOCKADDR          lpsaAddress,
  [in]           DWORD               dwAddressLength,
  [in, optional] LPWSAPROTOCOL_INFOA lpProtocolInfo,
  [in, out]      LPSTR               lpszAddressString,
  [in, out]      LPDWORD             lpdwAddressStringLength
);
```
IPv4주소를 받으면 이를 보기 쉬운 IP String으로 변환하는 함수. WSAAddressToStringA는 IPv6도 가능.
- 인자
  - `in` : 변환할 IP Adress. 
  - `lpsaAddress` : `SOCKADDR_IN` 또는 `SOCKADDR_IN6`형의 숫자형식의 IP주소
  - `dwAddressLength` : `sizeof(SOCKADDR_IN)` 또는 `sizeof(SOCKADDR_IN6)`
  - `lpProtocolInfo` : `NULL`
  - `lpszAddressString` : 변환한 String을 저장할 버퍼
  - `lpdwAddressStringLength` : 버퍼의 크기
- 반환
  - 변환된 IP
  - 성공시 0 / 실패시 SOCKET_ERROR(`WASGetLastError`로 구체적인 정보)


윈속 초기화 -> 소켓 생성 -> 통싱 -> 소켓 닫음 -> 윈속 종료