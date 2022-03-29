---
layout: post
title: "게임서버 공부_winsock"
description: "책 내용 + 공부/실습한거"
date: 2022-03-29
tags: [winsock, error handle, gameserver]
comments: true
share: true
---
## 오류 처리
네트워크 프로그래밍에서는 오류가 자주 발생한다. 그러므로 오류 발생시 정확한 정보를 전달받을 수 있게 오류처리를 할 필요가 있다. 오류 처리를 하는 방법은 아래와 같은 세가지 유형이다.
1. 오류 처리할 필요 없는 경우 : 리턴 값이 없거나 항상 성공하는 소켓 함수
2. 리턴 값만으로 오류를 알수 있는 함수 : WSAStartup() 등
3. 리턴 값으로는 오류 여부만 파악하고, 구체적 내용은 오류코드로 확인해야 하는 경우 : 대부분의 소켓 함수.

3번의 경우 에러코드를 바탕으로 어떠한 문제때문에 에러가 발생했는지 확인할 필요가 있다. 윈도우에서는 int WSAGetLastError(void)를 통해 가장 최근에 발생한 에러의 에러코드를 얻을수 있다. 이를 FormatMessage함수를 사용하여 오류코드를 해석할수 있다.

```cpp
DWORD FormatMessage(
  [in]           DWORD   dwFlags,
  [in, optional] LPCVOID lpSource,
  [in]           DWORD   dwMessageId,
  [in]           DWORD   dwLanguageId,
  [out]          LPTSTR  lpBuffer,
  [in]           DWORD   nSize,
  [in, optional] va_list *Arguments
);
```
- **dwFlags** : 서식 옵션 및 lpSource를 어떻게 해석할지 결정하는 옵션.FORMAT_MESSAGE_ALLOCATE_BUFFER | FORMAT_MESSAGE_FROM_SYSTEM 사용한다. 저장공간을 FormatMessage함수가 알아서 할당 | OS으로부터 메세지를 받아옴. 0x00001100값. 이 값을 입력하면 lpSource->NULL, nSize->0, Arguments->NULL대입.
- lpSource : 메세지가 정의된 위치 입력.
- **dwMessageId** : 메시지 식별자. WSAGetLastError()를 여기에 입력.
- **dwLanguageId** : 오류메세지를 표시할 언어. MAKELANGID(LANG_NEUTRAL, SUBLANG_DEFAULT)을 입력하여 제어판에서 설정한 기본 언어로 메세지를 받는다.
- **lpBuffer** : 오류 메세지를 담을 포인터. FormatMessage가 공간을 할당해주므로, 사용 후 LocalFree()해주어햐 한다.
- nSize : lpBuffer의 크기. 여기서는 FormatMessage함수가 알아서 할당하므로 0
- Arguments : 포멧 메세지의 형식지정자(프엪의 %d처럼)값. 여기선 사용 안함.


- DWORD : 
- LPCVOID : 
- LPTSTR : 
- va_list : 

코드
```cpp
void err_quit(char *msg)
{
	LPVOID lpMsgBuf;
	FormatMessage(
		FORMAT_MESSAGE_ALLOCATE_BUFFER | FORMAT_MESSAGE_FROM_SYSTEM,
		NULL, WSAGetLastError(),
		MAKELANGID(LANG_NEUTRAL, SUBLANG_DEFAULT),
		(LPTSTR)&lpMsgBuf, 0, NULL);
	MessageBox(NULL, (LPCUTSTR)lpMsgBuf, msg, MB_ICONERROR);
	LocalFree(lpMsgBuf);
	exit(1);
}

...
if (socket(...) == INVALID_SOCKET) err_quit("socket()");
...

```

```cpp
void err_display(char* msg)
{
	LPVOID lpMsgBuf;
	FormatMessage(
		FORMAT_MESSAGE_ALLOCATE_BUFFER | FORMAT_MESSAGE_FROM_SYSTEM,
		NULL, WSAGetLastError(),
		MAKELANGID(LANG_NEUTRAL, SUBLANG_DEFAULT),
		(LPTSTR)&lpMsgBuf, 0, NULL);
	printf("[%s] %s", msg, (LPCUTSTR)lpMsgBuf);
	LocalFree(lpMsgBuf);
}

...
if (socket(...) == INVALID_SOCKET) err_display("socket()");
...

```