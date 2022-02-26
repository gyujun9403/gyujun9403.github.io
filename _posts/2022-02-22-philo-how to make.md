---
layout: post
title: "Philosophers bonus 동작방식"
description: "42서울 Philosophers 보너스 과제 동작방식 정리."
date: 2022-02-25
tags: [42Seoul, Philosophers, 철학자]
comments: true
share: true
---

## 멘데토리 동작방식
![멘데토리](/images/42seoul/philo/philo(heap).png)
철학자 과제는 philo프로세스 내부에 여러개의 철학자 스레드들을 생성하여 생존시키는 과제이다. 해당과제에서 메인스레드의 역할과 나머지 철학자 스레드들의 역할은 다음과 같다.
- 메인스레드는 die여부, full(음식을 다 먹음)상태를 바쁜대기를 통해 확인하는 역할을 한다. `is_die`데이터를 `write`한다.
- 철학자 스레드들은 루틴(밥생각자기...)을 실행한다. 밥을 먹고 `philo배열`에서 자기 인덱스의 `die예정 시간`, `full여부`를 갱신(`write`)한다.

기본적으로 프로세스내 스레드들은 메모리에서 stack정도만 각자의 공간을 가지고 있고, heap, code공간 등은 공유한다. 이러한 heap의 특성을 이용해 뮤텍스를 최소화하기 위해 아래와 같이 heap영역의 데이터를 조작하였다.
- 메인 스레드만 `is_die`를 수정하고, 철학자 스레드들은 읽기만한다.
- 철학자 스레드들은 `philo배열`에서 각자의 인덱스의 데이터만 수정하고, 메인스레드는 읽기만한다.

추가적으로, `printf`하여 출력할때, 출력 도중 다른 스레드로 넘어가서 텍스트가 섞이는 현상이 간혹 발생했다. 따라서 `announce`라는 별도의 뮤텍스를 만들어 `mutex_lock(announce)`를 통과한 스레드만 출력할수 있게(한번에 하나씩) 하였다. (그림엔 표시 안됨)

## 보너스 동작방식
보너스에서 철학자는 별도의 프로세스들이므로 어떠한 메모리도 공유되지 않는다. 또한 해당 과제에서 허용되는 함수중 `kill`을 제외한 어떠한 signal관련 함수도 사용 불가하다. 따라서 세마포어의 점유(`wait`) 및 해제(`post`)를 통해서만 다른 프로세스에게 상태를 전달할수 있다.

동작방식을 구현하면서 가장 중요하게 작용한 개념은 다음과 같다. 세마포어의 수량이 부족하여 대기하는 프로세스/스레드의 경우 대기 큐에 들어가서 대기상태로 전환되기 때문에, CPU 점유에 영향을 미치치 않는다. 또한, 뮤텍스와 달리 세마포어를 선점(wait)하지 않은 프로세스에서 풀수 있다.  

이러한 개념들을 최대한 활용하여 바쁜 대기를 최소화 해보는 방식으로 코드를 작성했다. 
1. 우선 상태를 확인하기위한 세마포어를 할당 및 점유시킨다. 철학자 수만큼 `die`와 `full` 세마포어를 할당하고, fork 이전에 이를 전부 점유시킨다.
2. 이후 메인 프로세스(philo_bonus)의 [die], [full]스레드는 각각 `sem_wait(die)`, `sem_wait(full)`코드에 의해 대기큐 상태로 빠진다. -> **바쁜대기 회피**

이후는 아래 이미지에서 설명.

![보너스 포크](/images/42seoul/philo/philo_bonus(fork).png)
보너스에서 포트는 중앙에 놓여있다. 사용 가능한 포크가 생기면, `sem_wait`에 의해 block되었던 프로세스 중 하나가 (OS에 의해) run state로 집입하게 되에 먹고자고생각하고... 를 수행하게 된다. 당연히 먹고 나서는 포크를 반환한다.

![보너스 배부름](/images/42seoul/philo/philo_bonus(full).png)

1. 각 철학자들은 지정된 회수만큼 식사하면 `sem_post(full)`하여 점유를 하나씩 푼다.
2. 메인 프로세스(philo_bonus)의 [full]스레드는 철학자 개수만큼 `sem_wait(full)`한다. 
3. 철학자 수를 만족시키면 그 이후엔 `sem_post(die)`된다(이때 die출력은 하지 않음.). 모든 철학자 프로세스들을 `kill`하고 할당된 값들을 반환하며 프로그램을 끝낸다.

![보너스 주금](/images/42seoul/philo/philo_bonus(die).png)

1. 철학자 프로세스의 [die_check]스레드는 하나의 `die_check세마포어`에 의해 한번에 하나씩 돌아가면서 상태를 확인한다. -> 바쁜 대기를 통한 CPU부하 최소화
2. [die_chekc]스레드에서 죽음이 확인되면 die출력을 하고, `sem_post(die)`를 한다.
3. 메인 프로세스(philo_bonus)의 [die]스레드가 깨어나고 모든 철학자 프로세스들을 `kill`하고 할당되었던 자원들을 반환후 프로세스를 종료한다.


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
		만약 죽었으면(현재시간 > 죽은시간)이면 죽음 출력 및 die를 post한다