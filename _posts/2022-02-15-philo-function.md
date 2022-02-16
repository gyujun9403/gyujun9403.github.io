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

pthread_t
![pthread_t 자료구조](/images/42seoul/philo/pthread.png)
opaque type : 해당 타입에 저장되는 실제 데이터는 시스템의 고유 정보이며, 그 안의 데이터는 사용자가 접근할 수 없다. 따라서 pthread_t를 조작하기 위해서는 API가 제공해주는 thread를 다루는 함수들을 사용해야한다.
[![영상](https://www.youtube.com/watch?v=TsUOhPsZk6k/0.jpeg)](https://www.youtube.com/watch?v=TsUOhPsZk6k?t=0s)
참고 : [별준코딩](https://junstar92.tistory.com/229?category=982715), [bigpel66.oopy.io](https://bigpel66.oopy.io/library/c/etc/4)

### 1.1 pthread_create
```c
#include <pthread.h>
int pthread_create(pthread_t *thread,
                    const pthread_attr_t *attr,
                    void *(*start_routine)(void *),
                    void *arg);
```
1. 인자
- `pthread_t *thread` : 스레드 식별자. 생성된 스레드
- `const pthread_attr_t *attr` : 쓰레드 특성 지정하는 인자. 디폴트 사용시 NULL 전달.
- `void *(*start_routine)(void *)` : 분기시켜서 실행시킬 스레드 함수.
- `void *arg` :
2. 반환값
-
### 1.2 pthread_detach
### 1.3 pthread_join
### 1.4 pthread_mutex_init
### 1.5 pthread_mutex_destroy
### 1.6 pthread_mutex_lock
### 1.7 pthread_mutex_unlock
