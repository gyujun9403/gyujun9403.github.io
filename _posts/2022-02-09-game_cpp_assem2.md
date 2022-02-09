---
layout: post
title: "C++과 언리얼로 만드는 MMORPG 게임 개발 시리즈 cpp 강의"
description: "리마인드용 이미지, 코드위주로"
date: 2022-02-09
tags: [assembly]
comments: true
share: true
---
## 조건문
```
%include "io64.inc"

section .bss
    a resb 1

section .text
global CMAIN
CMAIN:
    mov rbp, rsp; for correct debugging
    ;write your code here
    GET_DEC 1, ax   ;div를 하기 위해서는 그 값이 ax에 저장되어야한다.
    mov bl, byte 2
    div bl          ;ax의 값이 bl로 나눠어져서 ah에는 나머지 al에는 몫
    cmp ah, byte 1
    je  EQUALS
    PRINT_DEC 1, 0  ;위에서 점프를 하지 않았으면 짝수라는 뜻
    jmp END         ;순차적으로 진행하므로 jmp하지 않으면 EQUALS라벨 코드 실행됨.
EQUALS: 
    PRINT_DEC 1, 1
END:
    xor rax, rax
    ret
```
![EFlags Register](https://sites.google.com/site/processorv2/_/rsrc/1311342363402/home/12-5-the-pentium-processor/eflags-register/eflags.png?height=518&width=655)

조건문에서 cmp 후 그 결과에 따라 EFLAGS 레지스터의 ZF플래그가 변화한다. je, jnp등의 조건명령어들은 플래그를 확인 후 분기한다.

[JUMP명령어 들이 분기시 확인하는 플래그 들](http://unixwiz.net/techtips/x86-jumps.html)

위의 코드에서도 조건을 만족하게 되면 레지스터 윈도우의 eflags에서 ZF가 활성화된다.

## 반복문
반복문은 조건문과 JUMP를 혼합하여 구현한다.
```
%include "io64.inc"

section .data
    msg db 'Hello', 0x0

section .text
global CMAIN
CMAIN:
    mov rbp, rsp; for correct debugging
    ;write your code here
    mov eax, 3
    
LOOP_HELLO:
    PRINT_STRING msg
    NEWLINE
    dec eax         ;카운트 감소
    cmp eax, byte 0 ;카운트가 0인지 확인
    jne  LOOP_HELLO ;~플래그가 활성화 되었으면(0이면) 위의 LOOP_HELLO로 점프
    xor rax, rax
    ret
```   

## 주소와 배열
```
%include "io64.inc"

section .data
    arr db 0x01, 0x02, 0x03, 0x04, 0x05 
    arr2 times 5 db 0x01    ;0x01로 1byte씩 5개 할당 

section .text
global CMAIN
CMAIN:
    mov rbp, rsp; for correct debugging
    ;write your code here

    mov rax, arr ;db 배열의 시작 주소를 rax에 저장
    PRINT_HEX 1, [rax]
    NEWLINE
    PRINT_HEX 1, [rax + 1]
    NEWLINE
    PRINT_HEX 1, [rax + 2]
    NEWLINE
    PRINT_HEX 1, [rax + 3]
    NEWLINE
    xor rax, rax
    ret
```