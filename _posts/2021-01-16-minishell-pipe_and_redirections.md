---
layout: post
title: "Minishell pipe and redirections"
description: "42서울 과제 Minishell과제에서 파이프/리다이렉션을 구현 방식을 정리한 글 입니다."
date: 2022-01-16
tags: [42Seoul, minishell]
comments: true
share: true
---

# 파이프
![img](/images/pipe001.jpeg)
우선 부모자식 프로세스는 `fork`를 통해 `pipe(fds)`를 같이하게 된다. `pipe(fds)`를 통해 초기화된 `fds`는 프로세스간 공유하는 저장공간(파일)의 read only, write onlye 디스크립터들이다. 부모던 자식이던 `fds[0]`은 read, `fds[1]`은 write이다.


따라서 하나의 프로세스가 조작해야하는 파이프는 총 2개이다. 이전 프로세스와 연결된 파이프, 이후 프로세스와 연결된 파이프이다. 
## 1. fork() 및 pipe() 방식
~~우선 빌트인함수는 fork 하지 않는다. 따라서 built인 함수는 minishell 프로세스에서 처리한다. 하지만, pipe는 fork한 부모자식간~~

우선 파싱한 리스트를 바탕으로 `cmd` 사이는 `fork` 하여 연결한다. ~~이때 리다이렉션의 파일은 fork하지 않는다. <<, < 같은경우 뒤에 있지만 먼저 연결 되어야 하기 때문에... 앞뒤 리스트가 리다이렉션인지 파악해서 stdin/out을 dup2로 잘연결시키는 방법을 사용하는게 좋을거같다.~~

![img2](/images/[minishell]data_structure_after_parsing001.jpeg)
과제를 진행하면서 `cmd`끼리만 리스트로 연결되고, 해당 리스트에 명령어/리다이렉션이 서로 다른 `char **` 변수에 저장되는 형식으로 변경됨.

## 2. 세부 fork 및 dup 동작 방식
각각은 fork된 별도의 프로세스이며 
- 우선 이전의 프로세스를 기다려야하고? ->  어짜피 exec하면 입력이 올때까지 무한정 대기
- 필요 없어진 이전 프로세스의 write를 close(**안하면 EOF가 없어서 계속 읽게된다**).
- 파이프로 연결될 exec는 표준 입/출력을 dup2한다. -> 이전 프로세스의 표준 출력
이때, 빌트인 함수의 경우 fork하고 exec없이 해당 함수를 수행 후 exit한다.
각 프로세스별로 이러한 동작을 하면서 fork 및 dup하게 되면 이후는 반복문에 모든걸 맡기면 된다.

- 자식당 하나의 프로세스만을 fork하는 방식이며, fork된 자식도 하나만의 프로세스만을 fork한다.
	- 이유 : pipe는 부모자식간에 IPC통신이므로.
- 이때 명령어당 하나의 프로세스에 할당되며, 가장 나중에 fork된 명령어가 가장 처음의 명령어를 실행한다. 따라서 상향식으로 명령어들이 실행되고 종료상태를 반환하게 한다.
- 모든 명령은 자식프로세스에서 동작한다. 부모프로세스, 즉 셸은 자식프로세스가 끝나기만을 기다린다.
