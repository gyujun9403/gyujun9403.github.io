---
layout: post
title: "Minishell redirections"
description: "42서울 과제 Minishell과제 수행을 위한 리다이렉션 공부 내용 입니다."
date: 2022-01-14
tags: [42Seoul, minishell, redirection]
comments: true
share: true
---
## 리다이렉션
리다이렉션은 `STDIN`, `STDOUT`, `STDERR`를 다른 스트림이나 파일로 우회할수 있는 명령어이다. 꺽쇠 부호 `>`, `<`를 통해 방향을 지정하며, `[프로그램] [리다이렉션] [파일/스트림]`의 형태로 구성된다.

## 리다이렉션의 동작 예시
- `cmd > file` : (표준)출력을 파일에 덮어씌운다.
- `cmd >> file` : (표준)출력을 파일에 이어붙인다(cat).
- `cmd < file` : cmd의 (표준)입력을 파일로 사용한다.
- `cmd << DELIMITER` : DELIMITER입력이 나올때까지의 모든 입력을 cmd로 전달한다.

## 리다이렉션과 파이프의 차이
리다이렉션과 파이드 둘다 표준입력/출력/에러의 방향을 우회하는 명령어이다. 하지만 그 도착지에서 차이가 발생한다.
 - 파이프(`|`) : (표준)출력을 프로그램/유틸리티로 넘김
 - 리다이렉션(`>`) : (표준)출력을 파일/스트림으로 넘김

## HEREDOC(here document)

> 컴퓨팅에서 히어 도큐먼트(here document, here-document), 히어 텍스트(here-text), 히어독(heredoc), 히어이즈(hereis), 히어 스트링(here-string), 히어 스크립트(here-script)는 파일 리터럴 또는 스트림 리터럴이다. 마치 별도의 파일인 것처럼 취급하는 소스 코드의 한 부분이다. <b>이 용어는 비슷한 문법을 사용하는 여러 줄의 스트링 리터럴을 구성하기 위해서도 사용되며 텍스트 안의 줄 바꾸기와 기타 공백(들여쓰기 포함)을 보존한다.</b></p>
...</p>
히어 도큐먼트는 파일 또는 문자열로 취급이 가능하다. 일부 셸은 이것들을 Printf 리터럴로 취급하며 이로써 리터럴 내에서 변수 치환과 명령어 치환을 가능케 한다.
유닉스 셸에서 기원한 가장 일반적인 히어 도큐먼트 문법은 <<와 그 뒤에 붙는 구분 문자 식별자(종종 EOF 또는 END), 그리고 다음 줄에 인용 텍스트가 시작되며 인용 텍스트를 닫을 때에는 똑같은 구분 문자 식별자가 위치하게 된다. 이러한 문법은 히어 도큐먼트가 스트림 리터럴임을 의미하며 문서의 내용은 선행 명령dml stdin(표준 스트림)으로 리다이렉트된다. 히어 도큐먼트 문법은 "다음 파일로부터 입력을 받는" <인 입력 리다이렉션 문법과 유사하다.</p>
[위키백과](https://ko.wikipedia.org/wiki/%ED%9E%88%EC%96%B4_%EB%8F%84%ED%81%90%EB%A8%BC%ED%8A%B8)
...

heredoc은 유닉스에서 여러 줄(\n)의 블록을 전달할수 있게 하는 리다이렉션의 유형이다.

### 동작 예시
해당부분은 **zsh과 bash의 동작이 상당히 상이하다.**
zsh는 여러개의 리다이렉션이 걸린경우 순서대로 전부 처리하지만, bash는 마지막거만 처리한다(하지만, >, >>의 경우 파일은 만든다.)

1. `cat <infile`
```
indfile
```

2. `cat <<END`
```
heredoc
```
[zsh]pipe로 연결된 heredoc 개수마다 `pipe heredoc>`, `pipe pipe heredoc>` 으로 표시됨.

[bash]몇개의 heredoc이 있던지간에 `>`로 표시됨.

3. `$ cat < infile << END`
```
[zsh]
infile
heredoc
```
```
[bash]
heredoc
```

4. `$ cat -n infile | cat << END`
```
[zsh]
	1 infile
heredoc
```
```
[bash]
heredoc
```
[zsh]infile이 echo되기 전에 heredoc 입력 커서(`pipe heredoc>`)가 먼저 출력되고 heredoc 종료 후 이후 프로세스가 수행됨.

[bash]두번째 `cat << END`에 표준입력으로 파이프와 heredoc이 들어왔지만, 파이프는 완전히 무시되고 heredoc만이 입력된 모습.



6. `echo hi | cat -n <<END | cat <<END`
```
[zsh]
    1  hi
    2  heredoc1
heredoc2
```
```
[bash]
heredoc2
```
[zsh]우선 커서로 `pipe pipe heredoc>`이 출력되고, 첫번째 END까지 받은 입력은 `cat -n <<END`부분에, 이후부터 두번째 END까지 받은 입력은 `cat <<END`에 들어간다.

[bash]나장 나중에 입력되는 표중 입력만이 유효하므로 싹 무시되고 `cat <<END`만이 유효해진다...

7. `cat infile <<END`
```
[bash]
infile
```
infile은 cat의 인자이다. 이건 cat프로그램 자체에서 정의하는 동작이다.

8. `cat <<END <infile <<END`
```
[bash]
heredoc2
```
bash에선 여러개의 입력이 들어와도 싹 무시되고 마지막거만 유효하다...

[참고 1](https://profq.tistory.com/8)
[참고 2](https://rottk.tistory.com/entry/Redirection%EA%B3%BC-Pipe%EC%9D%98-%EC%B0%A8%EC%9D%B4%EA%B0%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80%EC%9A%94)
[참고 3](https://woorld52.tistory.com/11)
[참고 4](https://jjeongil.tistory.com/1577)