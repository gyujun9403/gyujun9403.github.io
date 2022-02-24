---
layout: post
title: "Philosophers bonus 동작방식"
description: "42서울 Philosophers 보너스 과제 동작방식 정리."
date: 2022-02-24
tags: [42Seoul, Philosophers, 철학자]
comments: true
share: true
---

/* 
일단...idx of philo는 fork후 탈출한 idx. 가장 큰 값은 최상위 부모.
forks, full, die, sem_open(철학자 수) / die_check는 하나.
thread는 각각 하나씩인데, 최상위 부모는 fullcheck / 자식들은 die_check용도.
	최상위 부모에서 메인 thread는 die_check, 다른 thread는 조건에 따라 full_check
	자식들은 메인 thread는 일상, 다른 thread는 die_check돈다. 이때 usleep(500)정도 간격으로 

fork 전에 full과 die의 모든 세마포어를 점유 시켜놓고 시작. 이후 full되거나 die될때마다 하나씩 post
[부모 프로세스]
	메인 : sem_wait(die). 하나라도 되면 바로 kill(pids)
	서브 : sem_wait(full). 철학자 개수만큼 full 되면 kill(pids)
[자식프로세스]
	메인 : 루틴 실행. 밥 다먹고 획수 다 채웠으면 full post정도만 **주의! 단 1회만 하게.**
	서브 : sem_wait(die_check)를 usleep(500)간격으로?. 바쁜대기를 시키지 않고 세마포어를 통해 자동으로 큐대기로순차적 실행하게 한다.
		만약 죽었으면(현재시간 > 죽은시간)이면 죽음 출력 및 die를 post한다.
*/