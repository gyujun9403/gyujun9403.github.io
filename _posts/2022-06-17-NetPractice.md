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
과제에서 제시한대로 연결 가능하게 컴퓨터, 스위치, 라우터의 IP 및 마스크를 조작하는 과제. 네트워크 지식을 이용한 퍼즐이라고 생각하면 편하다.

## 1. 기본 IP개념
- 네트워크에 연결된 모든 장치는 IP를 가지고 있다.
- 라우터 내부 기준으로, 라우터로 연결된 다른 네트워크를 외부망이라고 칭한다(게이트웨이 역할).
- IP는 0.0.0.0 ~ 255.255.255.255의 범위를 가진다.
- 각각의 네트워크(LAN)은 라우터로 구분되며, 라우터 내부는 같은 네트워크 주소를 가진다.
- 반대로, 라우터의 각 인터페이스는 다른 네트워크여야 한다.
- 같은 네트워크(라우터 안)은 호스트 주소를 통해 구분된다.
- IP는 네트워크 주소와 호스트주소로 구분된다.
    - 호스트 주소가 전부 0이면 네트워크를 대표하는 주소로 예약되어 있다.
    - 호스트 주소가 전부 1이면 브로드캐스트용도로 예약되어 있다.

### 1.1 클래스
IPv4기준. IP주소를 네트워크의 크기에 맞게 할당하고자, 이를 구분해놓은것이 IP의 class이다.
![](https://miro.medium.com/max/1400/1*wbYRk65-lnwsWYSFJ656xw.png)

- A클래스 : 대규모 네트워크 환경.
    - 이때, 0.⎕︎.︎︎⎕︎.⎕︎, 127.⎕︎.︎︎⎕︎.⎕︎ IP는 각각 컴퓨터 자기자신을 나타내는 주소, 루프백 주소로 예약되어있다. 따라서 1.⎕︎.︎︎⎕︎.⎕︎ ~ 126.⎕︎.︎︎⎕︎.⎕︎의 IP만 사용가능하다.
- B클래스 : 중교모 네트워크 환경.
- **C클래스** : 소규모 네트워크 환경.
- D는 멀티캐스트용, C는 연구/개발용

- 같은 네트워크 내부는 같은 네트워크 IP 및 서브넷마스크를 가진다.
- 같은 네트워크 내부는 각각 다른 호스트 IP를 가진다.
- 클래스별로 pirvate영역이 있어서, 외부에서 해당 IP로 라우팅을 할 수 없다.
    - A class :10.⎕︎.︎︎⎕︎.⎕︎
    - B class :172.16.︎︎⎕︎.⎕︎~172︎.⎕︎.⎕︎(10101100.0001⎕︎⎕︎⎕︎⎕︎.⎕︎.⎕︎)
    - C clsaa :192.168.︎︎⎕︎.⎕︎

### 1.2 서브넷팅
기업에서 직원간 네트워크를 만들때, 인원이 1000명으로 254명이 넘어서 클래스 B로 할당해야한다고 가정하자. 만약 1000명의 인원이 모두 같은 네크워크 안에 있으면 ~문제가 발생하게 된다. 따라서 ABC클래스를 설정하고, 거기서 더 쪼갠 네트워크를 서브넷이라고 하며, 이를 나누는 과정을 서브넷팅이라고 한다.

위 표의 클래스의 네트워크부분이 전부 1인경우 디폴트 서브넷마스크, 거기서 확장되어 1이 증가하는 경우 일반적인 서브넷 마스크이다.
```
C class의 디폴트 서브넷마스크
11111111.11111111.11111111.00000000

C class의 네트워크를 나눈 서브넷마스크들(가능한 것들)
11111111.11111111.11111111.10000000
11111111.11111111.11111111.11000000
11111111.11111111.11111111.11100000
...
```

```
특정 장비의 IP가
IP : 147.168.105.254 
마스크 : 255.255.192.0

-> B class내부의 장치. 
디폴트 서브넷 마스크 : 255.255.0.0
네트워크 주소 147.168.64.0
해당 네트워크 자체를 나타내는 주소 : 147.168.64.0
브로드캐스트 주소 : 147.168.127.255
```

### 1.3 넷프에서의 IP
- 넷프를 할 때, 과제들의 네트워크는 소규모 네트워크이므로, 192.⎕︎.︎︎⎕︎.⎕︎ ~ 223.⎕︎.︎︎⎕︎.⎕︎사이의 IP를 할당한다.
- 선으로 연결되거나, 스위치를 통해 연결된 것들은 같은 네트워크주소를 가져야한다.
- 호스트 주소가 전부 0인경우와 1인경우는 제외한다.
- 서브넷마스크는 디폴트서브넷마스크보다 1이 더 많아야한다.

### 1.4 라우팅
![img](/images/42seoul/net/Net Practice-4.jpg)

라우팅 테이블
```
client A: webserv.non-real.com
    Routes :
목적지IP/마스크 => 보내야할 인터페이스
```
- 모든 입력에 대해서 적용시키고 싶을때, 목적지IP/마스크를 0.0.0.0/0으로 기입
- 목적지IP/마스크를 보내고자하는 네트워크 자체로 기입한다(목적지가 해당 네트워크에 포함되므로). -> 호스트 주소가 전부 0인 경우.
- 라우터의 각 인터페이스는 서로 다른 네트워크여야한다.
- 목적지가 라우터에 연결되어있는 IP라면, 라우터한테 도착하기만 하면 알아서 보내준다.

![](/images/42seoul/net/Net Practice-6.jpeg)
- 라우터 각 인터페이스의 IP의 네트워크 범위가 곂치면 안된다.
    - Not for me 발생.
- invalid route on host R1 : 라우팅 테이블 호스트가 이상할때(마스크가 없다던가...)
- private subnets not routed over internet : 10.0.0.1 ...?

## 2. 장치

### 2.1 컴퓨터
장치명
```
client A: 장치명
Interflace A1...
```
```
client A: webserv.non-real.com
    Routes :
목적지IP/마스크 => 보내야할 인터페이스
```

### 2.2 스위치
아이피 할당 안되어있음.

### 2.3 라우터

route R:My_Gate
```
router R: gate.non-real.com
    Routes :
목적지IP/마스크 => 보내야할 인터페이스
```
```
Interface R1
Interface R2
Interface R3
```

### 2.4 인터넷(외부망)
인터넷(외부망)과 연결되는 IP는 사설 IP여서는 안된다. 즉, ISP로 부터 할당받은 IP주소이다.
```
internet I: Internet
Routes :
 =>
```


## 풀이법?
1. 이미 입력되어 있는걸 싹 날린다
2. 네트워크 주소나, 서브넷 마스크, 라우팅 테이블에 의해 정해져 있는 정보만 기입한다......