---
layout: post
title: "Minishell pipe and redirections"
description: "42서울 과제 Minishell과제에서 파이프/리다이렉션을 구현 방식을 정리한 글 입니다."
date: 2022-01-16
tags: [42Seoul, minishell]
comments: true
share: true
---

## 파이프
![img](/images/pipe001.jpeg)
우선 부모자식 프로세스는 fork를 통해 pipe(fds)를 같이하게 된다. pipe(fds)를 통해 초기화된 fds는 프로세스간 공유하는 저장공간(파일)의 read only, write onlye 디스크립터들이다. 부모던 자식이던 fds[0]은 read, fds[1]은 write이다.


따라서 하나의 프로세스가 조작해야하는 파이프는 총 2개이다. 이전 프로세스와 연결된 파이프, 이후 프로세스와 연결된 파이프이다. 
### fork() 및 pipe() 방식
~~우선 빌트인함수는 fork 하지 않는다. 따라서 built인 함수는 minishell 프로세스에서 처리한다. 하지만, pipe는 fork한 부모자식간~~
우선 파싱한 리스트를 바탕으로 cmd 사이는 fork 하여 연결한다. 이때 리다이렉션의 파일은 fork하지 않는다. <<, < 같은경우 뒤에 있지만 먼저 연결 되어야 하기 때문에... 앞뒤 리스트가 리다이렉션인지 파악해서 stdin/out을 dup2로 잘연결시키는 방법을 사용하는게 좋을거같다.

근데 일단 파이프부터 구현하자.
fork는 리스트 순서대로 cmd만 fork하
우선 pipe()하기 전에 동적할당하고, 그 후 pipe하여 소를 lst에 별도로 저장한 후 fork한다.


### out -> out	연결
각각은 fork된 별도의 프로세스이며 
- 우선 이전의 프로세스를 기다려야하고? ->  어짜피 exec하면 입력이 올때까지 무한정 대기
- 필요 없어진 이전 프로세스의 write close
- 파이프로 연결될 exec는 표준 입/출력을 dup2한다. -> 이전 프로세스의 표준 출력
이때, 빌트인 함수의 경우 fork하고 exec없이 해당 함수를 수행 후 exit한다.


1. while을 돌면서 빌트인함수/외부함수/리다이렉션 구분하여 표시/카운터한다.
	빌트인의 경우 i, 외부함수의 경우 o, 리다이렉션은 rR