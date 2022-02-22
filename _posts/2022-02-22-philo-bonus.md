---
layout: post
title: "Philosophers bonus functions"
description: "42서울 Philosophers과제 방향성 정리."
date: 2022-02-22
tags: [42Seoul, Philosophers, 철학자]
comments: true
share: true
---

# Philosophers과제 함수

`memset`, `printf`, `malloc`, `free`, `write`, `fork`, `kill`,
`exit`, `pthread_create`, `pthread_detach`, `pthread_join`,
`usleep`, `gettimeofday`, `waitpid`, `sem_open`, `sem_close`,
`sem_post`, `sem_wait`, `sem_unlink`

## 1. semaphore 함수
세마포어란
프로세스/스레드간 공유하는 공통 자원/영역을 정해준 수의 프로세스/스레드만이 사용하는 기법. 세마포어는 시스템 자체에 걸쳐있는 시스템상의 파일로서 존재하며, 시스템이 해제하지 않으면 시스템이 남아있는한 사라지지 않는다.

뮤텍스와의 차이점은 다음과 같다.
1. 세마포어는 서로 다른 프로세스/스레드간 공유하는 자원을 보호한다. 뮤텍스는 한 프로세스 내의 스레드간에 공유하는 자원/영역을 보호한다.
2. 세마포어는 임계구역/공유자원에 접근하는 프로세스/스레드의 수(1~n)를 제한한수 있다. 뮤텍스는 임계구역/공유자원에 동시 접근할수 있는 스레드 수는 오직 하나이다.
3. 위와 같은 이유로 세마포어를 뮤텍스처럼 사용할수 있다(포함개념은 아니라고 한다...).

### 1.1 sem_open
```c
#include <semaphore.h>
sem_t *sem_open(const char *name, int oflag, ...);
```
이름있는 세마포어를 생성하고 초기화하여 여는 함수.
1. 인자
  - `const char *name` : 세마포어 객체(? c인데?)에 지정할 이름. 
  	- name이 `/`로 시작하는 경우(`/name`) 세마포어는 `/dev/sem/`에 생성된다(`/dev/sem/name`).
	- name이 `/`로 시작하지 않는 경우(`name`) 세마포어는 `/dev/sem/작업폴더명/`에 생성된다(`/dev/sem/작업폴더명/name`).
  - `int oflag` : 세마포어 생성시 적용할 플래그.
  	- `O_CREAT`플래그 단독: 해당 이름의 세마포어가 없으면 생성한다.
	- `O_CREAT | O_EXCL`플래그 : 해당 이름의 세마포어가 있는경우 에러 발생.
	- `O_EXCL`플래그 단독: undefind behavior
  - `...` : `O_CREAT`플래그를 사용한경우 `mode_t mode`, `const unsigned int value`를 기입한다(`sem_t *sem_open(const char *name, O_CREAT |... , mode_t mode, unsigned int value)`).
2. 반환값
  - 세마포어 디스크립터(`sem_t *`) 반환. 이 디스크립터값을 이용해서 `set_함수`들을 이용할수 있다.
  실패시 ~를 반환하고 errno설정.
3. 실제 사용 : `sem_t *sem_open("name", O_CREAT | O_EXCL, 0644, 초기값);`
3번째 인자는 mode값이며, 해당 세마포어의 접근권한을 설정한다. `0644`옵션을 주료 사용. 
- `6`은 `0b110`(0brwx)으로, 같은 유저에게 read & write권한을 부여를 의미한다.
- `4`은 `0b100`으로, 그룹에게 read 권한을 부여를 의미한다.
- `4`은 `0b100`으로, 그 외에게 read 권한을 부여를 의미한다.
- 참고 : c에서 `0`으로 시작하는 숫자는 표기법(notation)을 의미하고, `0x`와 같다. 

참고한 글 : [stackoverflow](https://stackoverflow.com/questions/18415904/what-does-mode-t-0644-mean)

### 1.2 sem_close
```c
#include <semaphore.h>
int	sem_close(sem_t *sem);
```
이름있는 세마포어에 할당된 자원들을 해제하고, 세마포어 디스크립터를 의미없게 하는(없애는)함수. 이름없는 세마포어를 전달하는 경우 undefine behavior
1. 인자
  - `sem_t *sem` : 세마포어 디스크립터
2. 반환값
  - 성공시 `0`, 실패시 `-1` 반환하고 errno설정.

### 1.3 sem_wait
```c
#include <semaphore.h>
int	sem_wait(sem_t *sem);
```
세마포어(sem)의 값(가용 자원)이 0 이상라면 값을 1낮추고 임계 영역 내로 진입시킨다. 만약 0이라면 block하여 대기큐에서 해당 프로세스를 기다리게 한다. 가용한 자원이 1이상이 되면 인터럽스 시그널에 의해 대기큐에 있는 프로세스중 하나가 임계구역 내로 진입하게 된다.
1. 인자
  - `sem_t *sem` : 세마포어 디스크립터
2. 반환값
  - 성공시 `0`, 실패시 `-1` 반환하고 errno설정하며 세마포어의 상태는 변하지 않는다.


### 1.4 sem_post
```c
#include <semaphore.h>
int	sem_post(sem_t *sem);
```
특정 프로세스/스레드의 임계영역 사용이 끝난 후, 세마포어(sem)의 값을 증가시켜 다음 프로세스/스레드가 가용하게 하는 함수.
1. 인자
  - `sem_t *sem` : 세마포어 디스크립터
2. 반환값
  - 성공시 `0`, 실패시 `-1` 반환하고 errno설정.

### 1.5 sem_unlink

```c
#include <semaphore.h>
sem_t *	sem_unlink(const char *name);
```
이름있는 세마포어의 이름을 제거하는 함수. sem_close하여 자원들 반환후 sem_unkink하여 이름을 없애야한다(순서 지켜야함). 
1. 인자
  - `const char *name` : 
2. 반환값
  - 성공시 `0`, 실패시 `-1` 반환하고 errno설정하며 세마포어의 상태는 변하지 않는다.

### 1.6 sem_close와 sem_unlink
sem_unlink를 통해 특정 세마포어를 삭제한다고 했을때, 특정 프록세스가 그 세마포어를 사용하고 있고, sem_close가 호출될때까지 삭제가 지연된다(= sem_close가 호출되면 삭제된다). sem_close또한, sem_unlink없이 호출되었다면 단독으로 세마포어를 제거하진 않는다고 한다...

참고한 곳 : [HY's TIL blog](https://hysimok.github.io/posts/til2021/2021-02-10/)

----

전체적인 참고 : [Jseo Doodle 블로그](https://bigpel66.oopy.io/library/42/inner-circle/9), [yechoi tistory](https://yechoi.tistory.com/55), [JongeunKeum-Dev vlog](https://velog.io/@jongeun/Philosophers-Day-05)