---
layout: post
title: "게임서버 공부_winsock_서버 기본 및 함수"
description: "책 내용 + 공부/실습한거"
date: 2022-04-08
tags: [winsock, socket, gameserver]
comments: true
share: true
---
## 서버 기본 흐름
![](/images\network\socket_server.jpg)


## 각 함수 실사용
```cpp
#pragma comment(lib, "ws2_32")
#define _WINSOCK_DEPRECATED_NO_WARNINGS
#include <WinSock2.h>
#include <stdlib.h>
#include <stdio.h>
#include <iostream>

#define SERVERPORT 9000
#define BUFSIZE 512
#include "error_handle.h"

int main(int argc, char* argv[])
{
	int retval;             // 반환값 중간 저장
    int addrlen;            // client 통신시 SOCKADDR_IN길이를 저장할 변수
	char buf[BUFSIZE + 1];  // 데이터 담을 버퍼
	SOCKADDR_IN serveraddr; // 서버
    SOCKET client_sock;
	SOCKADDR_IN clientaddr;

	ZeroMemory(&serveraddr, sizeof(serveraddr));    // 설정 안할거 전부 0
	serveraddr.sin_family = AF_INET;                // IPv4 TCP/IP
	serveraddr.sin_addr.s_addr = htonl(INADDR_ANY); // 네트워크 데이터정렬 long
	serveraddr.sin_port = htons(SERVERPORT);        // 네트워크 데이터 정렬 short

	WSADATA wsa;
	if (WSAStartup(MAKEWORD(2, 2), &wsa) != 0)      // 윈속 ddl 메모리 올림
		return 1;
	SOCKET listen_sock = socket(AF_INET, SOCK_STREAM, 0);                   // listen소켓 생성
	if (listen_sock == INVALID_SOCKET) err_quit("socket()");    
	retval = bind(listen_sock, (SOCKADDR*)&serveraddr, sizeof(serveraddr)); // listen소켓 port에 바인딩
	if (retval == SOCKET_ERROR) err_quit("bind()");
	retval = listen(listen_sock, SOMAXCONN);                                // listen소켓의 port listen형태로 변환
	if (retval == SOCKET_ERROR) err_quit("listen()");
	while (1)
	{
		addrlen = sizeof(clientaddr);                                           // 이거 왜 여기다가?
		client_sock = accept(listen_sock, (SOCKADDR*)&clientaddr, &addrlen);    // 블락. 클라이언트와 통신 수락 후 기술자 반환
		if (client_sock == INVALID_SOCKET)
		{
			err_display("accept()");
			break;
		}
		std::cout << "\n[TCP 서버] 클라이언트 접속 : IP주소=" << inet_ntoa(clientaddr.sin_addr) << "포트번호=" << ntohs(clientaddr.sin_port) << std::endl;
		while (1)
		{
			retval = recv(client_sock, buf, BUFSIZE, 0);    // 블락. 버퍼에 길이 만큼 데이터 받음
			if (retval == SOCKET_ERROR) {
				err_display("send()");
				break;
			}
			else if (retval == 0)   // recv시 받은 길이가 0이면 클라이언트가 통신 끊은거임 -> 반복문 탈출
				break;
			else
			{
				buf[retval] = '\0';
				std::cout << "[TCP / " << inet_ntoa(clientaddr.sin_addr) << ":" << ntohs(clientaddr.sin_port) << "] " << buf << std::endl;
				retval = send(client_sock, buf, retval, 0); // 블락. 버퍼에 길이만큼의 데이터 보냄
				if (retval == SOCKET_ERROR) {
					err_display("send()");
					break;
				}
				buf[0] = '\0';
			}
		}
		closesocket(client_sock);   // 클라이언트에서 통신 종료 확인시 클라소켓 닫음
		std::cout << "\n[TCP 서버] 클라이언트 종료 : IP주소=" << inet_ntoa(clientaddr.sin_addr) << "포트번호=" << ntohs(clientaddr.sin_port) << std::endl;
	}
	closesocket(listen_sock);       // 서버의 대기가 끝나면 listen소켓을 닫음 
	WSACleanup();   // 윈속 메모리에서 내림
	return 0;
}
```
## 함수 정리
```cpp
int WSAAPI bind(
  [in] SOCKET         s,
  [in] const sockaddr *name,
  [in] int            namelen
);
```
포트(local address)를 소켓에 연결(bind) 하는 함수.
- 인자
	- s : 소켓 디스크립터
	- name : 해당 소켓의 주소 정보를 담은 구조체. (SOCKADDR *)로 형변환
	- namelen : name의 길이 <-SOCKADDR_IN / SOCKADDR_IN6 등 확인용
- 반환
	- 성공시 0, 에러시 SOCKET_ERROR(WASGetLastError로 구체적인 정보)

```cpp
int WSAAPI listen(
  [in] SOCKET s,
  [in] int    backlog
);
```
연결된 포트를 listen상태로 변환시키는 함수.
- 인자
	- s : 소켓 디스크립터
	- backlog : 대기 가능한 클라이언트의 최대 수(처리가능 한). max : SOMAXCONN
- 반환
	- 성공시 0, 에러시 SOCKET_ERROR(WASGetLastError로 구체적인 정보)

```cpp
SOCKET WSAAPI accept(
  [in]      SOCKET   s,
  [out]     sockaddr *addr,
  [in, out] int      *addrlen
);
```
서버는 여러 클라이언트들과 통신하고 소켓 통신은 1:1방식의 통신이다. 따라서 listen은 계속 클라이언트를 대기하고, 따로 통신을 소켓이 accept를 통해 생성된 client소켓이다.

이때 클라이언트가 접속할때까지 서버는 wait상태, suspended상태로 전환된다(CPU사용 거의 없음). 클라이언트가 접속하면 wake up.
- 인자
	- s : 소켓 디스크립터 -> listen까지 진행된 소켓.
	- addr : 해당 소켓의 주소 정보를 담은 구조체. (SOCKADDR *)로 형변환
	- addrlen : addr의 길이 <-SOCKADDR_IN / SOCKADDR_IN6 등 확인용
- 반환
	- 성공시 0, 에러시 SOCKET_ERROR(WASGetLastError로 구체적인 정보)

### recv와 send
recv와 send함수는 [소켓버퍼](https://cjwoov.tistory.com/30)에 연결에 접근할수 있게 만든 함수(API)이다. 

```cpp
int recv(
  [in]  SOCKET s,
  [out] char   *buf,
  [in]  int    len,
  [in]  int    flags
);
```
입력된 소켓으로부터 데이터를 받아오는 함수. 자세한 사항은 아래에.
- 인자
	- s : 소켓 디스크립터
	- buf : 받은 데이터를 저장할 주소.
	- len : buf 길이. 수신버퍼로부터 복사할 최대 데이터 크기이다.
	- flags : 플래그. 주로 0
- 반환
	- 정상적으로 통신이 된 경우
		- [1, len] : 받아온 데이터의 길이.
		- 0 : 클라이언트의 접속 종료(정상 종료).
	- 에러시 SOCKET_ERROR(WASGetLastError로 구체적인 정보)

소켓버퍼의 특성상 받은 데이터의 길이가 len보다 짧더라도, 그 데이터가 다 들어오지 않을 수 있다. 즉, 한번에 데이터를 다 못받을 수 있다. 따라서 자신이 받을 데이터의 크기를 알고 있다면 그 길이가 될때까지 반환값 길이를 누적해서 여러번 받아야한다. -> vista이후 운영체제의 경우 MSG_WAITALL옵션을 flags로 사용해서 len만큼 데이터를 받을수 있다.

```cpp
int WSAAPI send(
  [in] SOCKET     s,
  [in] const char *buf,
  [in] int        len,
  [in] int        flags
);
```
입력된 소켓으로 데이터를 보내는 함수. 데이터는 buf에 저장되며, 송신 버퍼에 복사된다. 데이터가 송신버퍼에 복사되면 리턴되므로, 이것이 상대방이 데이터를 잘 받았는지와는 상관 없다. 송신버퍼의 남은 공간이 len보다 커질때까지 대기(블럭)된다.
- 인자
	- s : 소켓 디스크립터
	- buf : 보낼 데이터를 저장할 주소.
	- len : buf 길이. 
	- flags : 플래그. 주로 0
- 반환
	- 정상적으로 통신이 된 경우
		- 블로킹 소켓 입력 : 송신버퍼에 넣은 데이터의 길이(len).
		- 논 블로킹 소켓 입력 : 송신버퍼에 넣은 데이터의 길이([1,len]).
	- 에러시 SOCKET_ERROR(WASGetLastError로 구체적인 정보)

추가
netstat -an로 확인.
통신 성립 전
![](/images\network\socket_listen.png)
- server
	- listen서버 : 0.0.0.0:9000 에서 listen중

통신 설립 후
![](/images\network\socekt_netstat.png)
- server
	- listen서버 : 0.0.0.0:9000 에서 listen중
	- client서버 : 127.0.0.1:9000 에서 client(127.0.0.1:10919)와 통신 성립
- client
	- client : 127.0.0.1:10919에서 clinet서버(127.0.0.1:9000)과 통신 성립

이때 ctrl c 를 하면 책에서는 time_wait없이 그냥 끝난다고 되어있지만, 현재는 개선되었는지 ctrl c를 해도 time_wait상태이다.
client강제 종료
![](/images\network\socket_ctrl_c.png)
- server
	- listen서버 : 0.0.0.0:9000 에서 listen중
	- ~client서버 : 127.0.0.1:9000 에서 client(127.0.0.1:10919)와 통신 성립~ 사라짐
- client
	- client : 127.0.0.1:10919에서 time_wait상태.