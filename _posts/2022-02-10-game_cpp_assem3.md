---
layout: post
title: "C++ 게임개발 인강03_서브루틴 및 스택"
description: "리마인드용 이미지, 코드위주로"
date: 2022-02-09
tags: [assembly]
comments: true
share: true
---
## 서브루틴 및 스택
```
%include "io64.inc"

section .text
global CMAIN
CMAIN:
    ;write your code here
    
    ; rbp(Base Pointer) : 스택의 특정 위치를 저장할때 사용하는 포인터 레지스터
    ; rsp : 스택의 top이 저장되는 레지스터

    ; 12줄에서 23줄 까지 전부 5와 2를 대소비교 하는 코드...
    push rax
    push rbx; 서브루틴MAX 호출 이전에, 
            ; 서브루틴에 rax, rbx를 사용할것 이므로, 이 값들을 저장한다.
    push 5  ; stack에 5 push
    push 2  ; stack에 2 push
    call MAX; stack에 돌아올(ret) 주소값을 push
    PRINT_DEC 8, rax
    NEWLINE
    add rsp, 16 ; stack에 쌓아둔 5와 2를 무시하기 위해 rsp를 2칸위로 이동.
                ; 물론 pop해도 됨.
    pop rbx;rbx의 값이 나중에 들어갔으므로 먼저 pop하여 rbx에 저장한다. 
    pop rax
    
    xor rax, rax
    ret
    
; 큰 값을 rax에 저장하는 서브루틴.
MAX:
    push rbp    ;이전의 rbp를 임시 저장하기 위해 stack에 push
                ; 서브루틴 내부에 서브루틴이 있는 경우 그 값을 임시저장해야한다.
                ; rsp는 stack이 누적될수록 그 위치가 변하므로, 
                ; 변하지 않는 위치인 rbx를 기반으로 호출한 함수의 매개변수 등을 찾아온다.
    mov rbp, rsp;현재 서브루틴의 stack 위치를 rbp 저장
    

    mov rax, [rbp + 16] ; MAX호출 직전에 stack에 넣은 5를 rax에 mov
                        ; stack 한칸당 8bit이고 stack은 높은주소->낮은주소로 가므로
                        ; 먼저 쌓인 stack의 값을 가져오기 위해서는 
                        ; 2칸(1칸은 bps, 그 위는 ret할 주소) +한다.
    mov rbx, [rbp + 24]    ;MAX호출 직전에 stack에 넣은 2 rbx에 mov
    cmp rax, rbx
    jg L1   ; Jump if Greater. rax가 rbx보다 크면 L1라벨로 점프
    mov rax, rbx    ; jump안했으므로 rbx가 rax보다 크므로 큰 값 rax에 저장
L1:
    pop rbx ; rax가 크므로 바꿀필요 없음
    ret ;stack을 참고하여 call한 위치로 이동한다. 
    
```
![코드 이미지](\images\game_cpp\assembly08.png)
당장은 stack 이미지를 만들만큼 중요한 내용은 아닌거 같으니... 스택 메모리 강의 19분쯤 참고.

참고 블로그 : [TwoRivers](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=krquddnr37&logNo=20191417881), [JiR4Vvit의 블로그](https://jiravvit.tistory.com/87), [parksj.log](https://velog.io/@parksj3205/Assembly-%EB%A0%88%EC%A7%80%EC%8A%A4%ED%84%B0)