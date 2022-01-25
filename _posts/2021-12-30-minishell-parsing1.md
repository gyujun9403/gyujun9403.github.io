---
layout: post
title: "Minishell Parsing01"
description: "42서울 과제 Minishell의 파싱부 조건 정리 입니다."
date: 2021-12-28
tags: [42Seoul, minishell]
comments: true
share: true
---
# MiniShell 과제 파싱부 구상...
해당 문서를 작성한 이후 파싱부에서 아주 많은 내용이 추가되고 변경되었다. 따라서 해당 문서는 폐지하고, 새로운 문서에서 정리한다.
## 1. 개요

Minishell에서 가장 복잡하고 변수가 많은 부분은 파싱부라고 해도 과언은 아니다. 해당 부분을 구현함에 있어서 마구잡이로 코드를 작성하기보다는, 정리하고 거기에 맞춰서 코드를 쓰는게 더 빠르고 정확한 길이라 판단되어 해당 글을 작성한다.

## 2. Parsing 진리표

| 문자 \ flag | ' Flg | " Flg | $ Flg | Flg 없음 |
| :----: | :----: | :----:| :----: | :----: |
| white | 무시 | 무시 | 환경변수대체, 바로뒤면그냥$ | cat, 다음줄 |
| ' |FlgOFF, cat|무시|환경변수대체, FlgON|cat, FlgON|
| " |무시|FlgOFF, cat|환경변수대체, FlgON|cat, FlgON|
| $ |무시|중첩|환경변수대체, FlgON|cat, 환경변수|
| \| |무시|무시|환경변수대체, 다음리스트|cat, 다음리스트|
| > |무시|무시|환경변수대체, 다음은파일|cat, 다음은파일|
| >> |무시|무시|환경변수대체, 다음은파일|cat, 다음은파일|
| < |무시|무시|환경변수대체, 다음은파일|cat, 다음은파일|
| << |무시|무시|환경변수대체, 다음은파일|cat, 다음은파일|


- 다음줄 : char**에서 다음 줄을 추가하고 cursor가 이를 가르키게
- 무시 : 해당 문자를 텍스트 그대로 인식한다.
- cat : cursor가 가르키는 문자열에 지금까지의 문자열 cat
- 중첩 : flg가 중복되는 경우는 "FlgON된 상태에서 &가 입력되는 상태 뿐이다.
- 환경변수대체 : 환경변수 이름으로 인식하고 있던 문자열을 검색해서 해당하는 문자를 cursor가 가르키는 부분에 cat한다.
- FlgON / FlgOFF : 해당 문자에 대응되는 Flg를 ON/OFF한다.
- 다음리스트 : |의 경우 다름 리스트로 이동하고, cursor가 이를 가르키게 한다.
$는 cat할때 해당 환경변수가 가지고 있는 값으로 변환되어 cmds에 저장되게한다.

그렇다면... 이제 위의 조건들을 어떻게 코드로 구현하느냐

## 3. 동작방식 및 수도코드
- ast에서 cursor가 지목하는 문자열에 cat하는 방식으로 구현.
- cursor의 위치는 문자열의 시작주소만을 가르키며, flg없이 whitespace가 오는 경우 다음(줄)로 넘어간다.
- 화이트 스페이스가 있으면 전부 넘긴 후, 커서를 다음줄로 옮긴다.
- idx위치에서 slide(오프셋)을 증가시키면서 한번에 cat하여 malloc 및 free의 빈도를 줄인다.
- cat 할때는 단순히 cursor에만 cat하면 해결되게 코드를 작성한다.
- result를 받고서 해야하는 동작은 다음과 같다.
	1. 다음 문자로 넘어가기(경우에 따라 1~2칸) -> ~하는 경우
	2. idx부터 idx+slide - 1까지를 커서에 cat한다. 이때 slide == 0인 경우 아무것도 붙이지 아니하며,  $가 ON인상태에서는 환경변수에서 찾아서 cat한다.
	-> 근데 slide == 0인겨우 strlcat에 넣으면 알아서 아무것도 안붙일거같은데?
	3. 한글자 넘기고 다음 리스트로 간다(명령어 or 파일로 세팅)
	4. cat할때 현재위치 -1까지 cat한다. x

수정.
- cursor대신에 버퍼를 이용. char**형의 cmds를 직접적으로 다루다보니 새로운 cmd줄을 추가해야할 때 커서 대신에 cmds에 또 접근해야되서 코드 줄수가 더 늘어나고 깔끔해보이지 않았음. 버퍼를 이용해서 cat하고 다음줄로 넘어가야 할때만 cmds에 접근하여 버퍼를 마지막줄에 추가하고 버퍼를 ""로 새로 할당하면 cmds의 접근을 최소화할 수 있다.
- 또한 action의 반환값을 문자로 반환하여, 여려 명령을 순서대로 쭉 수행하게 한다.
예를 들면 그냥 공백을 만난 경우 CJIAW를 반환하여 cat하고, jump하고, index를 옮긴다음, 새 줄을 add하여, 중복된 whitespace를 전부 제거하는 일련의 동작들을 반환하게 한다.
action의 가지수 : jump, index 옮김, addnewline, cat, env, white, newlist

CJI : cat하고, jump하고, Index를 옮김.
-> 이전까지의 텍스트들을 버퍼로 옮기고, ++slide하고 index += slide하여 방금의 문자를 다음 cat에 반영되지 않게 한다. 


| 문자 \ flag | ' Flg | " Flg | $ Flg | " \& $ Flg | Flg 없음 |
| :----: | :----: | :----:| :----: | :----: |:----: |
| white | J | J | E |EIJ| CJIAW |
| ' |FlgOFF / CJI |J|ECJI|EIJ|Flgset / CJI|
| " |J|FlgOFF / CJI| FlgON / EIJ|Flgoff / EJI|FlgON / CJI|
| $ |J|Flgset / CJI|FlgON / EJI|EJI|CJI|
| \| |J|J|EJIN|EIJ|CJIN|
| > |J|J|EJIN|EIJ|CJIN|
| >> |J|J|EJIN|EIJ|CJIN|
| < |J|J|EJIN|EIJ|CJIN|
| << |J|J|EJIN|EIJ|CJIN|



```c
while(문자열이 끝날때까지)
{
	// 다음에 오는 문자를 보고 어떻게 행동할지 + 플래그 재설정
	result = chk(&문자열[idx + slide]);
	if (result == 공백 || 파이프 || 라다이렉션들 || 코트들) {
		strlcat(cursor, 문자열[idx], slide);
		idx += slide;
		slide = 0;
	}
	if (result == 공백날리기) {
		while(공백문자)
			++idx;
		cursor = newstr(ast->txt);
	}
	else if (result == 넘어가기)
		++slide;
	else if (result == 환경변수 cat) {
		env_vlaue = findenv(env, 문자열[idx], slide);
		strcat(cursor, env_value);
		++idx;
	}
	else if (result == 파이프) {
		다음 리스트 만듦, 타입은
		cousor = addone...();
		++idx;
	}
	else if (result == 리다이렉션) {
		ast추가;
		set_redirec_type(ast, result);
		++idx;
		if (result == <<, >>)
			++idx;
	}
}
```

```c
// 단어를 읽고 검증을 요청하는 함수.
int request(char **line) {

	return responese()
}

// 정적인 플래그와, 받은 요청을 바탕으로 행동을 반환하는 함수.
// 이때 예를 들면 파이프확인 요청 신호와 파이프에 적합한 행동을 하라는 신호가 같다면 조건문처리 할 필요 없이 입력을 그대로 반환하면 된다.
int response(char ) {
	static flgs = 0b00000000;
	// 플래그가 중첩되는건 DQ에서 DL상태 뿐이다.
	if (SQ플래그 ON상태) {
		SQ입력입력시 cat하게 하고(의미를 가짐 = 입력 그대로 반환)
		이외에는 전부 무시하는 동작 반환
	}
	else if (DQ플래그 ON상태) {
		우선 DQ시 cat하게 하는 동작 반환하고
		DL만 중접플래그 시키고, 나머지는 무시 동작 반환
	}
	else if (DL플래그 ON상태) {
		화이트 : flg unset(cat)
		DQ, SQ, DL : flg unset(cat) + flg set
		파이프, 리다이렉션 : flg unset(cat) + 그대로 반환(가능한가...?) -> flg형식으로 반환하면 될듯?
	}
	else {
		각 플래그 입력이면 플래그를 set한다.
		파이프 및 리다이렉션인 경우 그대로 반환
	}
}

```

flg가 변화(ON->OFF, OFF->ON)이 되면 일단 거기까지 cat을 한다.
`echo "Hi$PATH hello"`의 경우 `$`를 만나는 순가 `Hi`까지 cat, `$`이후 화이트스페이스(공백)까지 가서 환경변수 찾고 cat, 마지막으로 `"`를 만나서 `"`플래그가 OFF되면 `hello`를 cat


## $?
> $? should expands to the exit status of the most recently executed foreground pipeline.

foreground pipeline : 포그라운드에서 실행된 파이프
즉, minishell에서 $?를 구현한다면 

