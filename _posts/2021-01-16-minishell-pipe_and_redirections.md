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
하나의 프로세스가 조작해야하는 파이프는 총 2개이다. 이전 프로세스와 연결된 파이프, 이후 프로세스와 연결된 파이프이다. 
out -> out	연결
각각은 fork된 별도의 프로세스이며
우선 이전의 프로세스를 기다려야하고? ->  어짜피 exec하면 입력이 올때까지 무한정 대기
필요 없어진 이전 프로세스의 write close
파이프로 연결될 exec는 표준 입/출력을 dup2한다. -> 이전 프로세스의 표준 출력
dup2()를 통해 
out -> in	연결
이때, 파이프 및 리다이렉션은 한방향이고, 프로세스는 동시에 fork되지만 그 순서는 순차적이므이다. 따라서 여러 프로세스의 out들이 하나의 in(부모 프로세스)에 연결되어도 괜찮다.
in	-> out	연결
in	-> in	연결


1. while을 돌면서 빌트인함수/외부함수/리다이렉션 구분하여 표시/카운터한다.
	빌트인의 경우 i, 외부함수의 경우 o, 리다이렉션은 rR