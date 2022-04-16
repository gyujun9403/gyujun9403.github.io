---
layout: post
title: "게임서버 공부_네트워크"
description: "책 내용 + 공부/실습한거"
date: 2022-03-10
tags: [network, gameserver]
comments: true
share: true
---
## 2.1 컴퓨터 네트워크를 구성하는 기기
![](https://media.fs.com/images/community/upload/kindEditor/202105/24/lan-switch-application-1621848069-kpDihgwSTn.jpg)
로컬 지역 네트워크 LAN : 네트워크 스위치 하나를 사에에 두고 별 모양으로 단말기가 연결되어있는 형태. star topology. 가운데 스워치로 데이터를 전달하고, 스위치는 목적지인 단말로 이를 전달.
OSI모델의 표준만 지켜주면 제조사/통신규약이 달라도 네트워크가 가능.
LAN이 성립하러면 OSI모델의 2 계층(Layer 2)를 지키면 되고, 이만 지켜준다면 회선의 종류(랜선, 무선, 전화선...)등등 상관이 없다.
### 2.1.1 OSI 모델
[좋은 정보 + 이미지](https://www.stevenjlee.net/2020/02/09/%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-osi-7%EA%B3%84%EC%B8%B5-%EA%B7%B8%EB%A6%AC%EA%B3%A0-tcp-ip-4%EA%B3%84%EC%B8%B5/)
<!--![osi7](/images/OSI-TCP-Model-v1.png)-->

![](https://media.fs.com/images/community/upload/kindEditor/202107/29/original-seven-layers-of-osi-model-1627523878-JYjV8oybcC.png)![data](https://elguber.files.wordpress.com/2010/05/osi_layers.jpg)

![osi flow](https://flylib.com/books/2/629/1/html/2/images/1578702410/graphics/01fig03.gif)

#### [네트워크 엑세스 계층]
물리적인 장치가 신호를 처리하고 특정 장치를 찾아가는 영역.
- 1계층 : 물리계층. 신호의 물리적 형태()
	- 단위 : 
	- 목적지 : 물리적인 연결(랜선, 와이파이)
- 2계층 : 스위치 등에 의해 MAC주소를 찾아가는 계층
	- 단위 : 프레임
	- 목적지 : MAC주소

#### [인터넷 계층]
IP등을 이용해 라우팅을 하는 계층.
- 3계층 : Packet단위로 상위계층 데이터를 쪼개고, 라우팅하여 목적지까지 전송.
	- 단위 : 패킷
	- 목적지 : IP주소

#### [전송 계층]
port를 통해 특정 프로그램까지 찾아가게하고 데이터의 신뢰성 정도(TCP/UDP)를 설정한다. 상위 계층(응용)이 전달된 데이터를 잘 가져다 쓸 수 있게 한다.
- 4계층
	- 단위 : 세그먼트
	- 목적지 : 포트

#### [응용 계층]
사용자가 실질적으로 application을 통해 소통하는 계층. OSI는...
- 5, 6, 7계층	
	- 단위 : data/mesege
	- 5계층 : 응용프로그램들간 동기화, 로그인관리(유지, 로그아웃 등)
	- 6계층 : 인코딩(음원 및 영상), 및 복호화/암호화
	- 7계층 : 유저가 사용하는 어플리케이션.


게임 프로그램을 만드는건 5~7계층 중 어딘가를 정의하는 것.
소켓프로그래밍은 ???계층의 ?를 프로그래밍하는 것이다.

### 2.1.2 OSI 모델의 계층 2
각 단말기들은 LAN사이즈로 연결되며, 각각은 LAN내부에서 고유하다(MAC주소). 데이터를 헤더(주소)와 테일로 감싸서 프레임으로 만듦거나/이를 해석함.
### 2.1.3 OSI 모델의 계층 3
서로 다른 LAN이 연결된 WAN에서의 통신. 라우터(혹은 공유기)가 IP(인터넷 프로토콜)주소를 통해 적합한 경로를 탐색(라우팅)하여 LAN으로 데이터를 전달한다. LAN에서는 2L스위치나 공유기가 적합한 MAC주소를 가진 장비로 전달한다.

## 2.2 인터넷
이러한 스위치와 라우터가 연결된 망 전체를 인터넷이라고 한다.
## 2.3 컴퓨터 네트워크 데이터
네트워크나 서버 앱을 개발할때 프레임이나 패킷을 직접 다룰일은 많이 없다(물리적인 장치가 알아서 처리하므로). 대신 스트림과 메시지라는 것을 많이 다루게 된다.
### 2.3.1 스트림 형식
스트림 : 한쪽에서 다른 한쪽으로 연결된 데이터 흐름. 그 방향이 중요한거지, 어떤 단위로 끊을지 정해지지 않았기 때문에 스트림이라 표현하는듯하다.

데이터를 보낼때와 받을때 순서, 받은 개수 등등 <u>중간과정은 다를수 있지만 결론적으로 이를 종합하면 같은 방식의 통신 형식</u>.
### 2.3.2 메시지 형식
메시지 : 각 메세지를 정확히 구분해서 보낸때와 받을때 <u>중간과정 결과 모두 동일한 방식의 통신 형식</u>.

이러한 스트림/메세지 형태를 직접 다룰일은 많이 없다. OS부분에서 알아서 처리하기 때문이다.

큰 스트림이나 메세지를 전송하는 경우, 외부에서 볼때는 한번에 그 데이터가 왔다갔다 하는것처럼 보인다. 하지만 내부에서는 그 큰 데이터를 쪼개서 보낸다.
![](https://upload.wikimedia.org/wikipedia/commons/c/cd/PDU_Fragmentation-en.png)

https://thebook.io/006884/ch02/04/
## 2.4 컴퓨터 네트워크 식별자
![IPv4 vs IPv6](https://23nkt41oktaj11pvu09p7og1-wpengine.netdna-ssl.com/wp-content/uploads/2016/01/ipv4-ipv6-600x600.jpg) 출처 : [asmed](https://asmed.com/comparison-of-ipv4-and-ipv6/)

- IP 서브넷 마스크 : IP의 Net-id와 Host-id를 구분하기 위한 마스크. class를 사용할수도 있지만,서브네팅을 하게되면 class는 무의미해진다.
- IP 프리픽스 : IP의 Net-id와 Host-id를 구분하기 위해 사용하는 값. 마스크보다 더 간략하게 표현할수 있어서 prefix방식을 더 사용한다. 마스크의 경우 1111.1111....이런 값을 가지는데, 연속된 1의 개수가 16개라면 /16으로 표기하는 것.

[자세한 이론](http://www.ktword.co.kr/word/abbr_view.php?nav=2&id=846&m_temp1=5246), [참조:bignet 블로그](https://bignet.tistory.com/49)

포트
> 인터넷 프로토콜 스위트에서 포트(port)는 운영 체제 통신의 종단점이다. 이 용어는 하드웨어 장치에도 사용되지만, 소프트웨어에서는 네트워크 서비스나 특정 프로세스를 식별하는 논리 단위이다. 주로 포트를 사용하는 프로토콜은 전송 계층 프로토콜이라 하며, 예를 들어 전송 제어 프로토콜(TCP)와 사용자 데이터그램 프로토콜(UDP)가 있다. 각 포트는 번호로 구별되며 이 번호를 포트 번호라고 한다. 포트 번호는 IP 주소와 함께 쓰여 해당하는 프로토콜에 의해 사용된다. -위키피디아-
                                            
## 2.5 컴퓨터 네트워크의 품질과 특성
### 2.5.1 네트워크의 품질을 저해하는 것들
- 라우터는 여러 네트워크 장비들이 지나가는 통로이다. 이러한 라우터 차리할수 있는 것 이상으로 패킷이 몰리면(과부하) 패킷이 유실되고 품질이 저해된다.
- 신호가 약하거나 잡음에 노출되면 패킷의 정보가 오염되어 유실이 발생한다.

### 2.5.2 전송 속도와 전송 지연 시간
레이턴시 : 두 기기간 데이터를 최소량 전달할때 걸리는 시간(ms).

- 매체(선성 등)의 상태가 안좋으면 데이터 오염 -> 재전송 -> 레이턴시 증가
- 신호가 스위치/라우터를 거치면서 처리되는 시간은 레이턴시에 더 큰 영향을 미친다. -> 거리가 멀더라도 더 적은 기기를 거치면 더 빠를 수 있다.

성능 지표    
- 두 최종 목적지 사이의 레이턴시 = 중간의 모든 네트워크 기기의 레이턴시의 총합
- 두 최종 목적지 사이의 스루풋 = 중간의 모든 네트워크 기기 중 가장 작은 스루풋

### 2.5.3 네트워크 품질 기준 세 가지
- 스루풋 : 전송속도. 단위 시간당 전송되는 데이터의 총량.
- 패킷 유실률 : 전송되는 데이터 중 유실된는 비율.
- 레이턴시 : 전송된 데이터가 도달하기까지 걸린 시간.

### 2.5.4 무선 네트워크의 품질
CSMA참고.
## 2.6 컴퓨터 네트워크에서 데이터 보내기와 받기
### 2.6.1 UDP 네트워킹
User Datagram Protocol. 사용자가 정의한 테이타그램을 보내는 통신 프로토콜.

UDP통신을 하기 위해서는 송신측과 수신측이 각각 소켓을 생성하고, 포트를 하나씩 점유해야한다.

메세지 형태로 데이터가 전송된다. 테스트 해보려고 했는데 도저히 안됬음...

### 2.6.2 TCP 네트워킹
[TCP 헤더의 내용. 이외에도 굉장히 좋은 내용들.](https://evan-moon.github.io/2019/11/10/header-of-tcp/)

TCP는 통신을 시작하기 전에 연결을 먼저 해야하는 연결 지향형인 통신이다. 처음 연결이 추가되지만 데이터 유실이 없음을 보장한다.
![](/images\network\TCP.png)
![](/images\network\TCP2.jpg)

코드 참고 : https://pony11.tistory.com/10, 변수명이 이상한 것들이 있음. 수정하고 사용.

## 2.7 패킷 유실 시 UDP와 TCP에서 현상
- UDP : 하나의 패킷이라도 유실되면, 데이타그램 자체가 유실된다(). -> 데이타그램 전체 유실. 하지만 레이턴시 = 네트워크기기의 레이턴시
- TCP : 패킷을 하나 받을때마다 ASK를 보내기 때문에, 송신측에서 ASK를 받지 못하면 다시 패킷을 전송한다. -> 지연시간 발생. 따라서 레이턴시 = 데이터기기의 레이턴시 + <u>패킷 유실율 * 재전송 대기시간</u>

위와 같은 이유때문에 중요한 정보나 채팅 등 유실되면 안되지만 느려도 되는 데이터, UDP는 탄막이나 음성 등 잠깐의 정보 유실은 괜찮지만 느리면 안되는 데이터에 사용.

## 2.8 주로 사용하는 메시지 형식
TCP , UDP를 통해 전달되는 데이터의 형태는 주로 HTTP나 JSON같은 표준화된 형식일수 있고, 혹은 바이너리 형태의 데이터일수 있다.

- 메타데이터가 있는 경우 : HTTP, JSON처럼  메시지 내부에 데이터에 대한 정보데이터도 포함하는 경우. 데이터의 양이 더 많지만, 데이터의 의미가 확실하고 순서에 얽메이지 않아 확장성이 좋다.
```json
{
    "name": "식빵",
    "family": "웰시코기",
    "age": 1,
    "weight": 2.14
}
```
- 메타데이터가 없는 경우 : 필요한 정보만 갈수 있지만, 확장성이 떨어진다(데이터가 추가되거나, 바뀌는 경우).
```
{
    "식빵",
    "웰시코기",
    1,
    2.14
}
```

메타데이터가 있는 데이터로 통신하는 경우, 서버측 업데이트로 인해 데이터의 순서나 내용이 추가되어도 클라이언트 측에서 정상작동 된다.

하지만 메타데이터가 데이터의 경우 서버측에서 보내는 데이터가 변경되면 클라이언트도 업데이트를 통해 반영시켜야 한다.

## 2.9 네트워크 주소 변환

NAT 라우터 = 공유기

통신사에 인터넷 신청을 하면 하나의 IP를 부여받는다(자주 바뀌는 IP긴함). 그리고 일반적으로 이 하나의 IP에 공유기를 연결하여 여러대의 유선장비와 무선장비를 연결한다. 이때 공유기와 연결된 장비들은 공유기에 의해 내부IP를 부여받는다. 

#### 포트매핑
![](https://upload.wikimedia.org/wikipedia/commons/6/63/Network_Address_Translation_%28file2%29.jpg)
내부아이피로 외부의 서버에 접속하는 방법. 포트매핑을 통해 사설IP를 가진 장비로도 외부의 서버와 통신할수 있다.
1. 내부IP가 192.168.100.3인 기기의 3855번 포트에서 외부의 서버(209.131.36.158:80)에게 패킷을 보낸다(192.168.100.3:3855 -> 209.131.36.158:80).
2. 공유기를 통과하면서 출발지가 공유기IP(142.12.131.7)의 6282번포트로 매핑된다((142.12.131.7:6282 -> 209.131.36.158:80).
3. 서버는 패킷을 받고 처리하여 응답을 142.12.131.7:6282로 보낸다.
4. 공유기는 6282포트로 받은 입력을 테이블에 매핑되어 있는 192.168.100.3:3855로 되돌려준다.

-> 외부의 서버는 요청을 한 기기의 사설 IP를 알 필요가 없다.

#### 포트포워딩
![](https://upload.wikimedia.org/wikipedia/commons/thumb/5/5f/NAPT-en.svg/1920px-NAPT-en.svg.png)
공유기IP를 가지고 외부에서 내부IP에 접속하는 방법.
1. 공유기등 NAT 라우터각 포트마다 서버의 포트들이 테이블에 매핑된다
	- 공유기 1024포트에 10.1.1.1:1024
	- 공유기 1025포트에 10.1.1.2:1024
	- 공유기 1026포트에 10.1.1.3:1033
	- 공유기 1027포트에 10.1.1.3:1034(이미지엔 없지만 가능함)
2. 외부에서 라우터(대표IP)의 특정포트에 신호를 보내면, 이를 테이블에 매핑된 곳으로 보낸다.

https://mommoo.tistory.com/107

https://www.stevenjlee.net/2020/02/09/%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-osi-7%EA%B3%84%EC%B8%B5-%EA%B7%B8%EB%A6%AC%EA%B3%A0-tcp-ip-4%EA%B3%84%EC%B8%B5/