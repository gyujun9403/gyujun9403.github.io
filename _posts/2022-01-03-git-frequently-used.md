---
layout: post
title: "Git 자주쓰는 명령어"
description: "Git을 쓰면서 자주 쓰는 명령어들을 모음."
date: 2022-01-03
tags: [git, cmd]
comments: true
share: true
---

### 1. frok해서 push하던, 타인의 repo에서 내가 심은 잔디 가져오기
1. 우선 기록을 담을 ropo를 내 저장소에 만든다.
2. 복사할 타인의 repo를 받아온다.그냥 clone 해도 되지만, 잔디만 가져올꺼면 bare clone을 사용한다. `git clone --bare 상대repo`
3. clone한 타인의 ropo에서 내 repo로 push한다. `push --mirror 내repo`