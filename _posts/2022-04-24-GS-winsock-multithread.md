---
layout: post
title: "게임서버 공부_socket_멀티 스레드"
description: "윈속 스레드 관련 함수 정리"
date: 2022-04-24
tags: [winsock, socket, gameserver, thread, synchronization]
comments: true
share: true
---
<details> 
<summary> 제목 </summary> <div markdown="1">
```cpp
```
</div> </details>

스레드 생성
<details> 
<summary> `HANDLE CreateThread(lpThreadAttributes, dwStackSize, lpStartAddress, lpParameter, dwCreationFlags, lpThreadId
);` </summary> <div markdown="1">

```cpp
# include <processthreadsapi.h>
HANDLE CreateThread(
  [in, optional]  LPSECURITY_ATTRIBUTES   lpThreadAttributes,
  [in]            SIZE_T                  dwStackSize,
  [in]            LPTHREAD_START_ROUTINE  lpStartAddress,
  [in, optional]  __drv_aliasesMem LPVOID lpParameter,
  [in]            DWORD                   dwCreationFlags,
  [out, optional] LPDWORD                 lpThreadId
);
```
</div> </details>
<details> 
<summary> 제목 </summary> <div markdown="1">
```cpp
```
</div> </details>
<details> 
<summary> 제목 </summary> <div markdown="1">
```cpp
```
</div> </details>
<details> 
<summary> 제목 </summary> <div markdown="1">
```cpp
```
</div> </details>
<details> 
<summary> 제목 </summary> <div markdown="1">
```cpp
```
</div> </details>