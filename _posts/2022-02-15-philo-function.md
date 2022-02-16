---
layout: post
title: "Philosophers functions"
description: "42서울 Philosophers과제 함수 정리."
date: 2022-02-16
tags: [42Seoul, Philosophers, 철학자]
comments: true
share: true
---

# Philosophers과제 함수

`memset`, `printf`, `malloc`, `free`, `write`, `usleep`, `gettimeofday`, `pthread_create`, `pthread_detach`, `pthread_join`, `pthread_mutex_init`, `pthread_mutex_destroy`, `pthread_mutex_lock`, `pthread_mutex_unlock`

## 1. thread 함수
pthread.h : POSIX tread.

pthread_t와 opaque type
![pthread_t 자료구조](/images/42seoul/philo/pthread.png)
opaque type : 해당 타입에 저장되는 실제 데이터는 시스템의 고유 정보이며, 그 안의 데이터는 사용자가 접근할 수 없다. 따라서 pthread_t를 조작하기 위해서는 API가 제공해주는 thread를 다루는 함수들을 사용해야한다.
[![영상]( https://img.youtube.com/vi/TsUOhPsZk6k/0.jpg)](https://youtu.be/TsUOhPsZk6k?t=0s)
참고 : [별준코딩](https://junstar92.tistory.com/229?category=982715), [bigpel66.oopy.io](https://bigpel66.oopy.io/library/c/etc/4)

### 1.1 pthread_create
```c
#include <pthread.h>
int pthread_create(pthread_t *thread,
                    const pthread_attr_t *attr,
                    void *(*start_routine)(void *),
                    void *arg);
```
스레드를 생성하는 함수.
1. 인자
  - `pthread_t *thread` : 스레드 식별자. 생성된 스레드의 정보를 담고있는 pthread_t 포인터. 다만, 이 값은 운영체재마다 다르게 정의한다.
  - `const pthread_attr_t *attr` : 쓰레드 특성 지정하는 인자. 디폴트 사용시 NULL 전달.
  - `void *(*start_routine)(void *)` : 분기시켜서 실행시킬 스레드 함수.
  - `void *arg` : start_routine의 인자.
2. 반환값
  - 성공시 0, 실패시 error number 반환


### 1.2 pthread_detach
```c
#include <pthread.h>
int pthread_detach(pthread_t thread);
```
입력된 스레드를 바로 종료시키는 함수. 생성된 스레드의 자원은 스레드 종료사 지동으로 해제되지 않고, pthread_detach나 pthread_join을 통해서 해제시켜야한다.
1. 인자
  - `pthread_t thread` : 스레드
2. 반환
  - 성공시 0, 실패시 error number 반환


### 1.3 pthread_join
```c
#include <pthread.h>
int pthread_join(pthread_t thread, void **value_ptr);
```
입력된 스레드가 종료될때까지 기다리는 함수. 해당 스레드의 리턴값을 받아올수 있다.
1. 인자
  - `pthread_t thread` : 스레드
  - `void **value_ptr` : 스레드의 리턴값(`void *`)을 받아올 변수.
2. 반환
  - 성공시 0, 실패시 error number 반환

## 2. 뮤텍스
뮤텍스는 여러 스레드가 공유하는 자원을 보호하는 일종의 데이터 잠금장치이다. 이 뮤텍스로 보호되는 영역에는 하나의 스레드만 접근할수 있으며, 이 영역을 임계구역이라고한다.

### 2.1 pthread_mutex_init
```c
#include <pthread.h>
int pthread_mutex_init(pthread_mutex_t *mutex,
                        const pthread_mutexattr_t *attr);
```
잠그고 싶은 영역 하나다 하나의 자물쇠가 필요하다. 이때 이 자물쇠의 역할을 하는게 뮤텍스 객체?구조체(`pthread_mutex_t`)이므로 임계구역개수만큼 뮤텍스 객체 또한 초기화시켜줘야한다.
1. 인자
  - `pthread_mutex_t *mutex` : 초기화 시킬 뮤텍스 구조체
  - `const pthread_mutexattr_t *attr` : 뮤텍스의 특성을 지정하는 인자. 디폴트 사용시 NULL 전달.
2. 반환
  - 성공시 0, 실패시 error number 반환

### 2.2 pthread_mutex_destroy
```c
#include <pthread.h>
int pthread_mutex_destroy(pthread_mutex_t *mutex);
```
1. 인자
  - `pthread_mutex_t *mutex` : 파괴할 뮤텍스 구조체(사용이 끝남)
2. 반환
  - 성공시 0, 실패시 error number 반환


### 2.3 pthread_mutex_lock
```c
#include <pthread.h>
int pthread_mutex_lock(pthread_mutex_t *mutex);
```
1. 인자
  - `pthread_mutex_t *mutex` : 임계구역을 잠그기위해 사용할 뮤텍스 구조체
2. 반환
  - 성공시 0, 실패시 error number 반환

### 2.4 pthread_mutex_unlock
```c
#include <pthread.h>
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```
1. 인자
  - `pthread_mutex_t *mutex` : 해당 임계구역을 잠그기위해 사용한 뮤텍스 구조체
2. 반환
  - 성공시 0, 실패시 error number 반환
