---
layout: post
title: "게임서버 공부_socket_멀티 스레드"
description: "윈속 스레드 관련 함수 정리"
date: 2022-04-24
tags: [winsock, socket, gameserver, thread, synchronization]
comments: true
share: true
---

## 1. 스레드

### 1.1 스레드 생성
<details> 
<summary> CreateThread(6);
</summary>
<div markdown="1">

```cpp
# include <processthreadsapi.h>
// 성공 : 스레드 핸들 반환 -> 다른 API 스레드 정보 접근용.
// 실패 : NULL, 자세한건 GetLastError
HANDLE CreateThread(
  [in, optional]  LPSECURITY_ATTRIBUTES   lpThreadAttributes, // NULL
  [in]            SIZE_T                  dwStackSize,        // 스택 사이즈, default 사용은 0
  [in]            LPTHREAD_START_ROUTINE  lpStartAddress,     // 스레드 함수
  [in, optional]  __drv_aliasesMem LPVOID lpParameter,        // 스레드 함수 전달 인자
  [in]            DWORD                   dwCreationFlags,    // 스레드 생성 제어. 0 - 바로시작/ CREATE_SUSPENDED - ResumeThread() 대기
  [out, optional] LPDWORD                 lpThreadId          // 스레드 ID저장됨. 필요없을수도.
);
```
- lpStartAddress가 접근 불가하거나 이상해도 해당 함수는 핸들을 반환. 그리고 스레드 동작 후 예외가 발생한 다음 스레드가 종료된다.
</div>
</details>

### 1.2 스레드 함수

<details> 
<summary> ThreadProc(1) </summary> <div markdown="1">

```cpp
// 아래의 형태로 사용자가 구현.
// 선언 및 정의 ThreadProc는 예약 키워드이지만 바꿔도 됨.
DWORD WINAPI ThreadProc(LPVOID lpParameter)
{
    // 스레드 기능 구현
}
```
</div>
</details>

### 1.3 스레드 종료
<details> 
<summary> ExitThread(1) </summary> <div markdown="1">

```cpp
# include <processthreadsapi.h>
void ExitThread(
  [in] DWORD dwExitCode
);
```
스래드 내부에서 스레드를 종료.
</div> </details>
<details> 

<summary> TerminateThread(2) </summary> <div markdown="1">

```cpp
# include <processthreadsapi.h>
// 성공 : TRUE / 실패 : FALSE, 자세한건 GetLastError
BOOL TerminateThread(
  [in, out] HANDLE hThread,   // 스레드 핸들. ID아님.
  [in]      DWORD  dwExitCode
);
```
스레드 외부에서 핸들을 통해 스레드를 종료시키는 함수.
</div> </details>

### 1.4 스레드 우선순위 레벨
<details> 
<summary> SetThreadPriority(2) </summary> <div markdown="1">

```cpp
#difine THREAD_PRIORITY_IDLE            -15
#difine THREAD_PRIORITY_LOWEST          -2
#difine THREAD_PRIORITY_BELOW_NORMAL    -1
#difine THREAD_PRIORITY_NORMAL          0
#difine THREAD_PRIORITY_ABOVE_NORMAL    1
#difine THREAD_PRIORITY_HIGHEST         2
#difine THREAD_PRIORITY_TIME_CRITICAL   15

# include <processthreadsapi.h>
// 성공 : TRUE / 실패 : FALSE, GetLastError
BOOL SetThreadPriority(
  [in] HANDLE hThread,
  [in] int    nPriority
);
```
</div> </details>
<details> 
<summary> GetThreadPriority(1) </summary> <div markdown="1">

```cpp
// 스레드 우선순위 반환.
int GetThreadPriority(
  [in] HANDLE hThread
);
```
우선순위 레벨을 반환함.


</div> </details>

### 1.5 스레드 종료 대기 -> 동기화 객체 state확인.

<details> 
<summary> WaitForSingleObject(2) </summary> <div markdown="1">

```cpp
#include <synchapi.h>
// WAIT_ABANDONED : 포기된 동기화객체. 해당 객체 반환불가 상태면 OS가 강제로 신호상태로 만들고, WAIT_ABANDONED를 반환(ex - 스레드가 동기화 객체 선점 후 Terminated되는 경우).
// WAIT_OBJECT_0 : 신호상태가 되어 선점 받음
// WAIT_TIMEOUT : 시간초과
// WAIT_FAILED : 실패. 자세한건 GetLastError
DWORD WaitForSingleObject(
  [in] HANDLE hHandle,
  [in] DWORD  dwMilliseconds  // 최대 대기 시간.
);
```
특정 오브젝트(이 성우 스레드)가 종료될때까지 기다리는 함수.
</div> </details>

<details> 
<summary> WaitForSingleObjects(4) </summary> <div markdown="1">

```cpp
#include <synchapi.h>
// WAIT_ABANDONED : 포기된 동기화객체. 해당 객체 반환불가 상태면 OS가 강제로 신호상태로 만들고, WAIT_ABANDONED를 반환(ex - 스레드가 동기화 객체 선점 후 Terminated되는 경우). https://codemuri.tistory.com/160
// WAIT_OBJECT_0 : 신호상태가 되어 선점 받음
// WAIT_TIMEOUT : 시간초과
// WAIT_FAILED : 실패. 자세한건 GetLastError
DWORD WaitForMultipleObjects(
  [in] DWORD        nCount,         // 동기화 객체 개수
  [in] const HANDLE *lpHandles,     // 동기화 객체 배열
  [in] BOOL         bWaitAll,       // SIGNED 전부 : TRUE / SIGNED 하나라도: FALSE
  [in] DWORD        dwMilliseconds  // 대기 시간
);
```
여러개의 오브젝트(이 경우 스레드)들이 종료될때까지 기다리는 함수.
</div> </details>

### 1.6 스레드 중지 및 재실행

<details> 
<summary> SuspendThread(1) </summary> <div markdown="1">

```cpp
#include <processthreadsapi.h>
// 성공 : suspend(중지) count
// 실패 : (DWORD) -1 -> DWORD형의 -1...
DWORD SuspendThread(
  [in] HANDLE hThread
);
```
특정 스레드 일시 정지(block? sleep?)
</div> </details>

<details> 
<summary> ResumeThread(1) </summary> <div markdown="1">

```cpp
#include <processthreadsapi.h>
// 성공 : suspend(중지) count
// 실패 : (DWORD) -1 -> DWORD형의 -1...
DWORD ResumeThread(
  [in] HANDLE hThread
);
```
일시 정지된 특정 스레드를 재시작.
</div> </details>

<details> 
<summary> sleep(1) </summary> <div markdown="1">

```cpp
#include <synchapi.h>
void Sleep(
  [in] DWORD dwMilliseconds
);DLE hThread
```
해당 스레드를 입력된 시간(msec 단위)정지. 그 후 재시작.
</div> </details>


## 2. 스레드 동기화
### 2.1 임계 영역
<details> 
<summary> 초기화 - InitializeCriticalSection(1) </summary> <div markdown="1">

```cpp
#include <synchapi.h>
CRITICAL_SECTION CS
// 윈도우 비스타 이후부터는 STATUS_NO_MEMORY 예외 없이 무조건 성공. 
void InitializeCriticalSection(
  [out] LPCRITICAL_SECTION lpCriticalSection
);
```
반환값이 없으므로, 끝나고 errno확인할 것.
</div>
</details>

<details> 
<summary> 접근 - EnterCriticalSection(1) </summary> <div markdown="1">

```cpp
#include <synchapi.h>
CRITICAL_SECTION CS
// https://docs.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-entercriticalsection#return-value
void EnterCriticalSection(
  [in, out] LPCRITICAL_SECTION lpCriticalSection
);
```
</div>
</details>

<details> 
<summary> 탈출 - LeaveCriticalSection(1) </summary> <div markdown="1">

```cpp
#include <synchapi.h>
CRITICAL_SECTION CS

void LeaveCriticalSection(
  [in, out] LPCRITICAL_SECTION lpCriticalSection
);
```
</div>
</details>

<details> 
<summary> 삭제 - DeleteCriticalSection(1) </summary> <div markdown="1">

```cpp
#include <synchapi.h>
CRITICAL_SECTION CS

void DeleteCriticalSection(
  [in, out] LPCRITICAL_SECTION lpCriticalSection
);
```
</div>
</details>

### 2.2 이벤트

<details> 
<summary> 생성 - CreateEventW(4) </summary> <div markdown="1">

```cpp
#include <synchapi.h>
// 성공 : 핸들
// 이미 있음 : ERROR_ALREADY_EXISTS, 자세한 정보는 GetLastError
// 실패 : NULL, 자세한 정보는 GetLastError
HANDLE CreateEventW(
  [in, optional] LPSECURITY_ATTRIBUTES lpEventAttributes, // null
  [in]           BOOL                  bManualReset,      // 수동 TRUE / 자동 FALSE
  [in]           BOOL                  bInitialState,     // 생성시 신호 상태
  [in, optional] LPCWSTR               lpName             // 이벤트 명.
);
```
- bManualReset
  - 수동리셋 : TRUE - 대기하는 스레드 모두 깨우고 신호상태 유지.
  - 자동리셋 : FALSE - 대기하는 스레드 하나만 깨우고 비신호 상태 변환.
- lpName : 이름이 안곂치면 create, 곂치면 open(다른 프로세스에서 쓰고 있는걸 공유하게 됨).
</div>
</details>

<br>

이벤트 대기 : [동기화 객체 state확인 함수들](#15-스레드-종료-대기---동기화-객체-state확인)

<br>

이벤트 제거시 핸들 제거CloseHandle() 사용. [CloseHandle()로 닫는 객체들](https://docs.microsoft.com/en-us/windows/win32/api/handleapi/nf-handleapi-closehandle#remarks)
<details> 
<summary> 닫기 - CloseHandle(1) </summary> <div markdown="1">

```cpp
#include <handleapi.h>
// 성공 : TRUE / 실패 : FALSE, 자세한 정보는 GetLastError
BOOL CloseHandle(
  [in] HANDLE hObject
);
```
원격(연결된 상대) 주소 정보(IP, PORT)정보 반환.
</div> </details>

## 추가
### 소켓 구조체에서 address정보를 얻는 함수.

<details> 
<summary> 원격 - getpeername(3) </summary> <div markdown="1">

```cpp
#include <winsock.h>
int getpeername(
  [in]      SOCKET   s,
  [out]     sockaddr *name,
  [in, out] int      *namelen
);
```
원격(연결된 상대) 주소 정보(IP, PORT)정보 반환.
</div> </details>

<details> 
<summary> 지역 - getsockname(3) </summary> <div markdown="1">

```cpp
#include <winsock.h>
int getsockname(
  [in]      SOCKET   s,
  [out]     sockaddr *name,
  [in, out] int      *namelen
);
```
지역(호스트 = 나) 주소 정보(IP, PORT)정보 반환.

</div> </details>