---
layout: post
title: "C++ 게임개발 인강01_기본 입출력, 데이터 저장 및 연산"
description: "리마인드용 이미지, 코드위주로"
date: 2022-02-08
tags: [assembly]
comments: true
share: true
---
![assembly01](\images\game_cpp\assembly01.png)

![assembly02](\images\game_cpp\assembly02.png)

![assembly03](\images\game_cpp\assembly03.png)

## 리틀엔디엔과 빅 엔디안
![assembly04](\images\game_cpp\assembly04.png)
숫자를 저장하게 할때 0x12345678을 저장하게 명령했지만, 실제 저장방식은 0x78 0x56 0x34 0x12형태로 반대로 저장이 되어 있다. 이는 리틀엔디안 방식으로, 작은 단위가 앞에 나오도록 저장이 된다. 빅엔디안 리틀엔디안 장단점은 검색해 보는게 빠르다.

참고 : 서버에서 데이터를 날릴때, 클라이언트쪽이 리틀엔디안을 쓰는지, 빅 엔디안을 쓰는지 고려해줄 필요가 있다.

## 사칙 연산
![assembly05](\images\game_cpp\assembly05.png)
## 비트 연산/논리연산
![assembly06](\images\game_cpp\assembly06.png)