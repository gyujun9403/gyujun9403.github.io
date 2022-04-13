---
layout: post
title: "게임서버 공부_winsock_데이터 처리"
description: "책 내용 + 공부/실습한거"
date: 2022-04-13
tags: [winsock, socket, gameserver]
comments: true
share: true
---

## 1. 데이터 처리의 필요성
소켓 통신을 구현하면 기본적인 골격은 비슷하여, 대부분의 응용프로그램이 비슷한 흐름을 따라간다.하지만 소켓 통신 전달받은 데이터의 형태는 각 응용프로그래밍 별로 상이하며, 이것이 TCP/IP계층의 4단계 응용프로그래밍 수준의 프로토콜이다. [각 계층별 프로토콜](https://ko.wikipedia.org/wiki/%EC%9D%B8%ED%84%B0%EB%84%B7_%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C_%EC%8A%A4%EC%9C%84%ED%8A%B8)

각 프로그램마다

## 2. 데이터 처리 필요한 정보
### 2.1. 어떤 데이터인지에 대한 정보
받은 데이터가 어떤 데이터인지 식별하고, 이에 맞게 파싱할 정보가 필요하다.
### 2.2. 데이터 경계(크기)를 구분할 정보
1. 송신측이 정해진 길이의 데이터만을 보내는것.
    - 장 : 구현이 쉽고, 파싱등이 빠르다.
    - 단 : 길이변경이 어려우며, 가장 긴길이를 감안하여 고정길이를 정하면 낭비가 생긴다.
2. 송신측이 가변길이 데이터를 보내지만, 그 끝에 특별한 표시(EOR)를 하는 것.
    - 장 : 유연하다
    - 단 : 데이터 내부에 EOR과 같은 문자가 있는 경우 예외처리가 필요하다. 또한 문자열 전체를 확인해야 하므로, 이를 고려하지 않고 설계하면 성능이 급격하게 떨어진다.
3. 데이터의 길이(고정길이)를 먼저 보내고 그 다음 데이터를 보내는 방식.
    - 장 : 구현 및 처리 효율성이 쉽다. 
    - 단 : 데이터를 2번 보내고 2번 다 받아야 된다.
4. 송신측에서 일방적으로 데이터를 보내고 접속을 끊어버리는 경우.

데이터 정렬은 빅 엔디안 방식을 사용하는게 좋다.

참고 : 구조체 패딩문제(구조체 멤버 맞춤?)가 소켓 통신에서도 발생한다. 따라서 #pragma pack(1)을 구조체 앞에 명시하여 패딩의 경계를 1byte로 설정한다.

### 3. 데이터 전송 방식 상세
#### 3.1 고정 데이터 전송 방법

서버 수신 관련 사용자 정의 함수
<details>
<summary> recvn </summary>
<div markdown="1">

```cpp
int recvn(SOCKET s, char* buf, int len, int flags)
{
	int received;
	char* ptr = buf;
	int left = len;

	while (left > 0)
	{
		received = recv(s, ptr, left, flags);
		if (received == SOCKET_ERROR)
			return SOCKET_ERROR;
		else if (received == 0)
			break;
		left -= received;
		ptr += received;
	}
	return (len - left);
}
```
</div>
</details>

서버측 수신 부분
```cpp
#define BUFSIZE 50  //서로 합의된 데이터 크기

...
    char buf[BUFSIZE];
    // 수신 버퍼서 BUFSIZE만큼 데이터 받아옴.
    recvn(sock_client, buf, BUFSIZE, 0);    
```
클라이언트 측 송신 부분
```cpp
#define BUFSIZE 50  //서로 합의된 데이터 크기

...
    char buf[BUFSIZE];
    // 송신 버퍼에 BUFSIZE만큼 데이터를 입력.
    send(sock_server, buf, BUFSIZE, 0);    
```
#### 3.2 가변 데이터 전송 방법
데이터 끝에 올 EOR를 우선 송수신측에서 약속해야한다. 보통은 \n이나 \r\n을 사용한다.
다만, 한 글자식 소켓 수신 버퍼에서 읽어오는건 메모리 밖의 영역에 접근하는 것이므로, 큰 성능저하가 발생한다. 따라서 수신버퍼에서 한번에 많은 데이터를 읽어서 메모리로 가져 오고, 이 문자에서 EOR을 찾는 방식이 좋다.

리눅스, 유닉스 같은 경우 소켓 데이터를 파일의 한 종류처럼 사용 -> 한글자식 접근할때 마다 외부 저장장치에 접근해야 하므로 엄청난 성능 저하 발생


서버 수신 관련 사용자 정의 함수
<details>
<summary> _recv_ahead </summary>
<div markdown="1">

```cpp
// 소켓 수신 버퍼에서 데이터를 한번에 가져오고, 
// 글자 하나씩 p로 넘겨줌
int _recv_ahead(SOCKET socket, char* p)
{
	static int nbytes = 0;
	static char buf[1024];
	static char* ptr;

    // nbytes : 읽은 데이터 중 남은 데이터.
    // recv를 통해 buf크기(1024)만틈 읽고 한글자(p)식 반환
    // buf크기만큼 p를 반환 했으면 다시 buf를 채움.
	if (nbytes == 0 || nbytes == SOCKET_ERROR)
	{
		nbytes = recv(socket, buf, sizeof(buf), 0);

		if (nbytes == SOCKET_ERROR || nbytes == 0)
			return nbytes;

		ptr = buf;
	}

	--nbytes;
	*p = *ptr++;
	return 1;
}
```
</div>
</details>

<details>
<summary> recvline </summary>
<div markdown="1">

```cpp
// _recv_ahead()에서 한 글자식 받아서 확인하고
// 개행까지 buf에 저장하고 함수를 끝냄.
// 개행을 못 만나면 최대길이만큼 반환.
int recvline(SOCKET socket, char* buf, int maxLen)
{
	int n, nbytes;
	char c, * ptr = buf;

	for (n = 1; n < maxLen; n++)
	{
		nbytes = _recv_ahead(socket, &c);

		if (nbytes == 1)
		{
			*ptr++ = c;
			if (c == '\n')
				break;
		}
		else if (nbytes == 0)
		{
			*ptr = 0;
			return n - 1;
		}
		else
			return SOCKET_ERROR;
	}
	*ptr = 0;
	return n;
}
```
</div>
</details>

서버측 수신 코드
```cpp
#define BUFSIZE 512

...
    char buf[BUFSIZE];
    // 한번에 받아온거 중 개행 만큼 or BUFSIZE 만큼 데이터 받아옴.
    recvline(sock_client, buf, BUFSIZE + 1);    
```
클라이언트 측 송신 코드
```cpp
#define BUFSIZE 50

...
    char buf[BUFSIZE];
    len = strlen(data);
    strncpy(buf, data, len);
    // 데이터 뒤에 개행 붙임.
    buf[len + 1] = '\n';
    // 송신 버퍼에 개행을 포함한 데이터를 입력.
    send(sock_server, buf, len + 1, 0);    
```

#### 3.3 고정 + 가변 데이터 전송 방법

쉽게 생각하면 고정길이만큼 2번 받으면 된다.

서버측 수신 코드
```cpp
...
    char buf[BUFSIZE];
    char len_data; // 다음 데이터 크기가 들어감

    // 크기를 받아옴
    recvn(sock_client, &len_data, sizeof(int), 0);
    // 크기만큼 데이터를 받음
    recvn(sock_clinet, buf, len_data, 0);
```

클라이언트 측 송신 코드
```cpp
...
    char buf[BUFSIZE];
    char len = strlen(data);
    strncpy(buf, data, len);

    // 크기를 보냄
    send(sock_server, &len, sizeof(char), 0);    
    // 데이터를 입력.
    send(sock_server, buf, len, 0);    
```

#### 3.4 





##질문

근데 윈도우는???


inet_ntop()와 InetNtop()둘중 하나로 바꾸라고 하는데... 그 둘의 차이?
InetNtop()는 윈도우 특화, inet_ntop().는 ANSI C에서 라고 하는데...
-> [참조](https://www.winsocketdotnetworkprogramming.com/winsock2programming/winsock2advancedInternet3c.html)

어떤 책에서 데이터를 json이나 http등으로 보낸다고 들었는데, 그렇다면 특정 단어나 패턴이 나올때까지 파싱하는 구조인가?