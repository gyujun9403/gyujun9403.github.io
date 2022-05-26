---
layout: post
title: "Compile에서 virtual까지"
description: "코드가 컴파일되서 링킹되고 실행되어 메모리에 올라간이후 실행되었을때 virtual키워드는 어떻게 처리될지"
date: 2022-5-25
tags: [cpp, compile, linking, class, memory, virtual]
comments: true
share: true
---

컴파일러/링킹/ELF등등 내용은 흐름에 필요한 내용만 간략하게 적을 예정.

참고 서적 및 내용 : [컴파일러 개발자가 들려주는 C이야기](https://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9788966263318), []


그리고, 해당 파일들을 컴파일러로 하여 실행 파일을 만들어 낼것이다.

컴파일러는 ~~의 집합니다.

### 1.1 전처리기
1. .c파일의 #include가 각 헤더의 내용들로 치환된다.
2. .c파일의 매크로를 #define의 두번째 인자로 치환한다.

이외에 #undef, #ifdef, #error, #pragma, #line...등등을 처리한 .i파일을 생성한다.

### 1.2 컴파일
컴파일러는 .i파일을 해석하여 고수준의 언어를 저수준의 언어(어셈블리)로 변환한다. 그 결과 생긴것이 .s파일이며, 사용자 환경에 따라 저수준 언어 동작 방식이 다르므로 서로다른 운영체게간에 호환되지 않는다. 

고수준의 언어를 저수준의 언어로 해석하는 도중 발생하는게 컴파일 오류이며, 코드의 문법상의 문제이다.

### 1.3 어셈블
어셈블러가 어셈블리어인 .s파일을 해석하여 기계어(바이너리파일))로 변환한다. 이때 생긴것이 오브젝트파일(.o, .obj)이다.

오브젝트파일 및 실행파일(추가로 DDL)등을 유닉스/리눅스에서는 ELF파일이라고 하며, 윈도우에서는 PE파일이라고 한다. 특히 오브젝트 파일은 윈도우에서는 PE형식의 .obj, 리눅스/유닉스에서는 ELF형식의 .o이 생성된다.

대략적인 형태는 아래 링킹 이미지에 표시. 오브젝트 파일 까봤는데... .obj는 정보도 적고, 뭔가 구조도 달라서 그냥 넘어가는게 좋을듯 하다. 리버싱 공부하는거 아니고선 잘 모르겠음...

우리에게 필요한건 오브젝트 파일이 **PE형식**이며, 변수 및 함수가 각 **기능별 섹션으로 나누어져 있다는 것**이다. 

https://redthing.tistory.com/entry/PE-%ED%8C%8C%EC%9D%BC%EC%9D%98-%EA%B5%AC%EC%A1%B0

### 1.4 링킹 및 실행파일
![linking 1](https://www.tenouk.com/Bufferoverflowc/Bufferoverflow1_files/image018.png)
![linking 2](/images\cpp\obj_linker.png)


각 변수 및 함수가 섹션(.bss, .data, 등에 따라 구분)되어 있고, 링킹때 필요한 부분만 쏙쏙 챙겨서 실행파일에 합친다.

#### 1.5 PE파일
![.exe PEview](/images\cpp\PE_readonly_section.png)
PE파일을 뷰어로 열어서 확인한 이미지. 출력을 위해 코드에 기입한 문자열이 .rdata영역에 저장되어 있다.

PE파일은 헤더와 바디로 구분되어 있고, 바디는 섹션으로 구성되어있다.
- 헤더 : 각 섹션의 크기, 정보를 담고 있음
- 섹션 : 실질적인 데이터와 실행코드가 포함되는 영역.
	- .text :프로그램의 실행코드 포함하는 섹션. IAT정보도 포함
	- .data	: 전역변수, static변수 할당을 위해 존재하는 섹션.
		- .bss : 초기화 되지 않은 전역변수를 담는 섹션. 보통 .data 섹션에 병합됨.
	- .rdata : 문자열, 상수 등 읽기전용 데이터를 담는 섹션. <u>가상함수 테이블(vtable)</u>도 .rdata에 포함된다(위치 계산은 [여기](https://www.sysnet.pe.kr/2/0/11167) 참고).
	- .rsrc : 리소스 데이터가 포함되는 섹션
	- .idata : Import할 DLL와 관련 API들의 정보를 담는 섹션
	- .edata : EXPORT할 API대한 정보를 담는 섹션.


[섹션별 정보 참고](https://unabated.tistory.com/entry/%EA%B0%9C%EB%85%90-%EC%9D%B4%ED%95%B4-PE-%ED%8C%8C%EC%9D%BC%EC%9D%98-%EA%B5%AC%EC%84%B1)

## 2. 메모리 로딩

![PE to memory](https://t1.daumcdn.net/cfile/tistory/2401E242588322871F) 왼쪽이 PE포멧. PE파일의 자세한 내부 구조는 [해당 블로그](https://m.blog.naver.com/luuzun/50190920011)참조. 

 > PE파일이 로드될때, VMM은 페이지파일을 사용하지 않고, PE파일 자체를 페이지파일처럼 가상 주소 공간에 그대로 매핑
 [참고](https://redthing.tistory.com/entry/PE-%ED%8C%8C%EC%9D%BC%EC%9D%98-%EA%B5%AC%EC%A1%B0)

파일을 메모리로 올리는것을 로드라고 하며, 이는 로더에 의해 수행된다. 로더는 다음과 같은 동작을 수행한다고 한다.
1. 파일의 정보를 읽고 유효성 검사를 수행
2. 메모리 요구사항을 계산 및 프로세스 설정
	- 프로그램 실행을 위한 기본 메모리(프로세스당 할당된)할당
	- 보조 저장장치에서 메모리로 주소공간을 복사한다.
	- 실행파일의 .text 및 .data섹션을 메모리로 복사한다.

위의 예시를 보면 PE의 섹션을 거의 그대로 메모리로 올리는걸 알 수 있다. 사실 PE는 기계어로 되어있으므로, 이 자체가 컴퓨터가 이해하는 동작 명령인 것이다.

대부분 그대로 올라가지만, NULL의 경우 NULL padding을 하여 크기가 증가한다(효율적으로 처리할 수 있는 단위로 만듦).

## 3. 프로그램의 메모리 구조.
![](https://www.oreilly.com/library/view/learning-malware-analysis/9781788392501/assets/ba1d4dd8-ba89-4e7c-a7ee-6432a84cd179.png) <br>[이미지 참조](https://www.oreilly.com/library/view/learning-malware-analysis/9781788392501/813ebede-c3ab-41be-9cdd-32d4baac9d37.xhtml)

위처럼 메모리상에 올라간 PE파일(프로그램)은 프로세스에 할당한 공간중 .text, .data(.bss포함) 영역이 된다.

스택은 임시적인 공간으로, 
1. 프로그램 인수(ac, av, env...)를 스택에 복사한다.
2. esp레지스터가 위 메모리 구조에서 스택의 가장 위(시작부)를 가르키도록 한다.
3. 시작루틴(mian())으로 점프하면 프로그램이 동작하기 시작한다.

- .text : 코드가 저장된 영역. .code영역이라고도 한다. 작성한 코드(함수, 클래스, 외부라이브러리 함수 등)이 이 영역에 들어간다.
- .data : 전역, 정적변수들이 저장된 영역이다.
	- .data : 초기화된 데이터가 저장된다.
	- .bss : 초기화되지 않은 데이터가 저장된다.
	- 지역변수는 스택 공간에 저장된다.
- heap : 동적으로 할당되는 메모리가. 동적으로 메모리를 할당/해제하는 syscall에 의해 사용자가 관리한다. 또한 동적 링크 라이브러리 함수들 또한 heap영역의 최상단에 할당된다...?
- stack : 호출된 함수 및 지역변수는 stack에 push되고, return시 pop된다. 재귀함수가 끝나지 않으면 stackoverflow가 발생.

heap주소는 증가하고, stack주소는 감소하게 설정하여, stack이나 heap이 overflow해도 중요 데이터는 침범하지 않게 설계가 되어있다.

[이미지 및 개념 차용](https://www.tenouk.com/Bufferoverflowc/Bufferoverflow1c.html)

## 2 virtual과 vtable

우선 개발자가 아래와 같은 파일들을 작성하고, 컴파일을 한다.
<details> 
<summary> [Base.hpp] </summary>
<div markdown="1">
```cpp
class Base
{
public:
	int a;
	Base() :a(42) {}
	virtual void listen();
	void speak();
};
```
</div>
</details>

<details> 
<summary> [Base.cpp] </summary>
<div markdown="1">

```cpp
#include <iostream>
#include "Base.h"

void Base::listen()
{
	std::cout << "Base listen" << std::endl;
}

void Base::speak()
{
	std::cout << "Base speak" << std::endl;
}
```

</div>
</details>

<details> 
<summary> [Derived.hpp] </summary>
<div markdown="1">

```cpp
class Derived : public Base
{
public:
	int b;
	Derived() :b(43) { a = 41; }
	virtual void listen() override;
	void speak();
};
```
</div>
</details>

<details> 
<summary> [Derived.cpp] </summary>
<div markdown="1">

```cpp
#include <iostream>
#include "Derived.h"

void Derived::listen()
{
	std::cout << "Derived listen" << std::endl;
}

void Derived::speak()
{
	std::cout << "Derived speak" << std::endl;
}
```

</div>
</details>



<details> 
<summary> [main.cpp] </summary>
<div markdown="1">

```cpp
#include <iostream>
#include "Base.h"
#include "Derived.h"


int main()
{
	Base base;
	Derived derived;
	Base* basePtr = new Base();
	Derived* derivedPtr = new Derived();
	Base* derivedPoly = new Derived();
	//std::cout << << std::endl;
	
	std::cout << "base.a :" << base.a << std::endl;
	base.listen();
	base.speak();
	std::cout << "-----------------------------------" << std::endl;
	std::cout << "derived.a :" << derived.a << std::endl;
	std::cout << "derived.b :" << derived.b << std::endl;
	derived.listen();
	derived.speak();
	std::cout << "-----------------------------------" << std::endl;
	std::cout << "basePtr->a :" << basePtr->a << std::endl;
	basePtr->listen();
	basePtr->speak();
	std::cout << "-----------------------------------" << std::endl;
	std::cout << "derivedPtr->a :" << derivedPtr->a << std::endl;
	std::cout << "derivedPtr->b :" << derivedPtr->b << std::endl;
	derivedPtr->listen();
	derivedPtr->speak();
	std::cout << "-----------------------------------" << std::endl;
	std::cout << "basePtr->a :"  << derivedPoly->a << std::endl;
	// dynamic_cast -> 동적때 결정됨.
	std::cout << "dynamic_cast<Derived*>(basePtr)->a :" << dynamic_cast<Derived*>(derivedPoly)->a << std::endl;
	std::cout << "dynamic_cast<Derived*>(basePtr)->b :" << dynamic_cast<Derived*>(derivedPoly)->b << std::endl;
	derivedPoly->listen();
	derivedPoly->speak();
	std::cout << "dynamic_cast<Derived*>(basePtr)->listen() :";
	dynamic_cast<Derived*>(derivedPoly)->listen();
	std::cout << "dynamic_cast<Derived*>(basePtr)->speak() :";
	dynamic_cast<Derived*>(derivedPoly)->speak();
}
```

</div>
</details>

code영역에 vtable 있다. 각 클래스마다 하나씩 할당되며 그 위치는...

각 virtual이 있는 인스턴스의 시작 주소에는 8byte(62bit체계)크기의 vtable의 주소를 가지고 있다. 

어떻게 함수가 vtable에 있는지 알고 접근을 하지?


이때 함수, 변수, vtable는 ~에 저장되어 있고~

vtable은 ~할 때 생성된다

런타임에 결정된다에 고찰.
특성 맴버 함수에 virtual키워드가 지정되면 이를 상속받는 클래스들은 virtual키워드를 적지 않아도 해당 함수는 virtual함수가 됨

-> 이러한 특성이 있어야, 상속하는 클래스들에서도 가상함수가 항상 같은 위치에 있게 된다.

예시에서 listen함수는 이후 상속받는 함수들이 virtual키워드를 생성한다고 하더라고, 영원히 vtable의 첫번째 위치에 있게 된다.


![](/images\cpp\virtual_funcs_call.png)
개발자가 특정 맴버 함수를 호출할때, 컴파일러는 코드를 해석하여 다음처럼 동작하게 어셈블리어를 생성한다(컴파일타임).
- 일반 함수면 해당 함수의 주소(.data)를 호출
- 가상함수이면 무조건 vtable에서 할당된 위치를 참조하게 한다.
	- 위의 사진에서 첫번째 가상함수 

따라서 , listne()을 호출할때 컴파일러는 Base클래스를 호출하던, 그 자식 클래스를 호출하던, 업캐스팅된 Base 포인터에서 함수를 호출하던 상관 없다. 그냥 주소의 시작위치 8byte가 가르키는 vtable로 가서 첫번째 주소를 참조하라고 하면 된다.

vtable자체는 컴파일 타임에서 결정되지만, 어떤 vtable을 참조하게 되는지는 런타임에 포인터에 어떤 클래스가 할당되는지에 따라 달라진다.

이렇게 일괄된 방식으로 vtable을 처리하며 다형성이 구현된다.

