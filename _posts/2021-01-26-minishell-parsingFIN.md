---
layout: post
title: "minishell parsing"
description: "parsing."
date: 2022-01-26
tags: [42seoul, minishell]
comments: true
share: true
---

# Parsing
근 2달간 미니셸 과제를 하면거 가장 오랜시간 고민했던건 파싱부이다(사실 아직도 완벽하지 않다). 해당 포스트에선 파싱을 처리하면서 어떠한 (자랑하고싶은)개념을 사용했는지, 이후에 어떤개념을 통해 이를 개량할수 있는지 적어보고자한다.
## 1. 기본 동작방식
![img](/images/[minishell]parsing.001.jpeg)
문자열을 확인하는 방식은 투포인터 방식과 플래그방식을 사용.
idx는 시작점, slide는 오프셋이다.
1. 시작점에서 오프셋을 증가시켜 가면서 문자 하나씩 확인한다.
2. action_decider에서 문자에 따른 동작방식들(action_set)을 반환한다.
	- 특정한 동작을 요구하는 글자(`'`, `"`, `|`, `>` ...) : 이전까지의 입력을 버퍼로 옮김 or 새리스트 or flg여부 확인 및 세팅...
	- 일반 문자, `'`내부에 있어서 의미가 없어진 문자 : ++slide(J)
3. 반복문을 통해 동작방식들(action_set)을 차레대로 수행한다.
	- CJIJE : `C : [idx, idx + slide)범위 문자열을 버퍼로 옮김` -> `J : ++slide` -> `I : idx위치 slide로 이동` -> `J : ++slide` -> `E : idx ~ idx + slide 환경변수 대응 value 찾아서 버퍼로 옮김`


```cpp
while (종료 플래그) {
	char *action_set = action_decider(*line)
	// 전달받은 action_set을 반복문으로 하나씩 수행
	while (*action_set != '\0')
	{
		if (*action_set = 'C')
			[idx, idx + slide)범위 문자열을 buffer에 cat.;
		else if (*action_set = 'E')
			[idx, idx + slide)가 key인 환경변수 value를 buffer에 cat.;
		else if (*action_set = 'J')
			++slide;
		else if (*action_set = 'I')
			idx += slide; slide = 0;
		else if (*action_set = 'A')
			buffer에 있는 내용을 리스트 내용에 추가, buffer 재할당;
			...
		++action_set;
	}
}
```
이외에도 몇개의 동작들이 있지만 동작을 알아보는덴 필요가 없으므로 생략.

- 장점 : 조건이 추가되거나 변경시 반환되는 action_set만 변경해주면 된다. 코드가 간결해지고, action_decider 결과가 어떠한 동작들을 할지 직관적이다. 
- 단점 : action_set을 할당할 추가적인 공간 필요.

## 2. 동작 결정(action_decider)
해당 부분에서는 action_decider에 정적 변수 flgs를 사용하여, 플래그들은 ON/OFF 했다.

이때 flg의 종류는 `'`, `"`, `$`, 리다이렉션이 있고, 각 플래그는 flgs에 bit형태(`0b0001`, `0b0010`...)로 비트 연산되어 저장된다.

이러한 형태를 취한 이유는 
1. `'`, `"`내부에 있는 글자들은 의미가 달라지기 때문에 이전에 들어온 이전에 이러한 문자들이 들어왔는지 저장할 필요가 있다.
2. `"`내부에서는 `$`를 사용할수 있고, 리다이렉션할 파일 이름도 `'`, `"`를 사용하는 것 등 플래그가 중첩될수 있기 때문이다.


구현한 동작 예시(일부)
- ex: `'`flg ON 상태에서 `space`, `$`등
	- -> 일반문자로 취급하여 `J`(++slide)동작 반환
- ex: `"`flg ON 상태에서 `$`입력
	- -> `$`flg ON 
	- -> `CJI`(`[idx ~ idx + slide)`문자열을 buffer에 이어붙임, `++slide`, `idx`이동)동작들을 반환