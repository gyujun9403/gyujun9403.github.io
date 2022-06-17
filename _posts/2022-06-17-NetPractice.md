---
layout: post
title: "Net Practice"
description: "42서울 과제 Net Practice"
date: 2022-6-17
tags: [42Seoul, network, IP, subnet, subnet mask]
comments: true
share: true
---

## 0. 과제 목표
과제에서 제시한대로 연결 가능하게 컴퓨터, 스위치, 라우터의 IP 및 마스크를 조작하는 과제. network퍼즐이라고 생각하면 편하다.

## 1. 기본 IP개념
- 네트워크에 연결된 모든 장치는 IP를 가지고 있다.
- 라우터 내부 기준으로, 라우터로 연결된 다른 네트워크를 외부망이라고 칭한다(게이트웨이 역할).
- IP는 0.0.0.0 ~ 255.255.255.255의 범위를 가진다.
- 각각의 네트워크(LAN)은 라우터로 구분되며, 라우터 내부는 같은 네트워크 주소를 가진다.
- 같은 네트워크(라우터 안)은 호스트 주소를 통해 구분된다.
- IP는 네트워크 주소와 호스트주소로 구분된다.
    - 호스트 주소가 전부 0이면 네트워크를 대표하는 주소로 예약되어 있다.
    - 호스트 주소가 전부 1이면 브로드캐스트용도로 예약되어 있다.

### 1.1 클래스
IPv4기준. IP주소를 네트워크의 크기에 맞게 할당하고자, 이를 구분해놓은것이 IP의 class이다.
![](https://miro.medium.com/max/1400/1*wbYRk65-lnwsWYSFJ656xw.png)

- A클래스 : 대규모 네트워크 환경.
    - 이때, 0.⎕︎.︎︎⎕︎.⎕︎인 IP와 127.⎕︎.︎︎⎕︎.⎕︎ IP는 각각 컴퓨터 자기자신을 나타내는 주소와 루프백 주소로 예약되어있다. 따라서 1.⎕︎.︎︎⎕︎.⎕︎ ~ 126.⎕︎.︎︎⎕︎.⎕︎의 IP만 사용가능하다.
- B클래스 : 중교모 네트워크 환경.
- **C클래스** : 소규모 네트워크 환경.
- D는 멀티캐스트용, C는 연구/개발용

- 같은 네트워크 내부는 서로다른 호스트 IP를 가진다.

### 1.2 서브넷팅
기업에서 직원간 네트워크를 만들때, 인원이 1000명으로 254명이 넘어서 클래스 B로 할당해야한다고 가정하자. 만약 1000명의 인원이 모두 같은 네크워크 안에 있으면 ~문제가 발생하게 된다. 따라서 ABC클래스를 설정하고, 거기서 더 쪼갠 네트워크를 서브넷이라고 하며, 이를 나누는 과정을 서브넷팅이라고 한다.

## 2. 장치

### 2.1 컴퓨터
client A: 장치명
Interflace A1...

```
client A: webserv.non-real.com
Routes :
 =>
```

### 2.2 스위치
Switch S:Switch-1

### 2.3 라우터
route R:My_Gate
```
router R: gate.non-real.com
Routes :
 =>
```

Interface R1
Interface R2
Interface R3

![img](/images/42seoul/net/Net Practice-4.jpg)


### 2.4 인터넷(외부망)
```
internet I: Internet
Routes :
 =>
```
IP