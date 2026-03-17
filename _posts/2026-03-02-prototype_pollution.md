---
layout: post
title: 프로토타입 오염(Prototype Pollution)
subtitle: Prototype Pollution
categories: WEB
tags: [JavaScript, Prototype Pollution, React2Shell]
---

## 프로토타입 오염(Prototype Pollution)
작년 하반기, React2Shell 취약점은 보안 업계에 큰 영향력을 행사했다. 이 취약점의 근본 원인 중 하나가 바로 ‘<span style="color: red; font-weight: bold">프로토타입 오염(Prototype Pollution)</span>’이고, 오늘은 해당 취약점을 정리해보고자 한다.

프로토타입 오염은 공격자가 Object.prototype과 같은 상위 프로토타입 객체에 임의의 속성을 주입함으로써 애플리케이션 내 모든 객체가 해당 속성을 상속받도록 만드는 취약점이다. 이로 인해 권한 상승, 비즈니스 로직 우회, 심지어 서버 원격 코드 실행(RCE)까지 이어질 수 있다.


### 객체란?
자바스크립트 내의 객체가 무엇인지 간단하게 알아보자. 객체는 키-값(Key-Value) 쌍으로 이뤄진 프로퍼티의 집합이다.
객체의 속성은 점 표기법(.) 또는 대괄호 표기법([])을 통해 접근할 수 있다. 또한 객체 내부에는 함수 역시 프로퍼티 형태로 선언할 수 있다.
```javascript
const user = {
    username: "coco",
    userId: 777,
    method: func() {
        ... 생략 ...
    }
}

user.username
user['userId']
```

<br>

JavaScript의 객체는 단순한 Key-Value 집합만으로 구성된 것이 아니다. <br>
우리가 선언하지 않은 속성에도 접근할 수 있는데, 이는 객체가 상위 객체와 연결되어 있기 때문이다.

예를 들어, 개발자 도구 콘솔에서 위의 객체를 선언한 뒤 user.을 입력해보자.
아래의 예시처럼 username, userId 같은 우리가 정의한 속성뿐 아니라 \_\_proto\_\_, hasOwnProperty와 같은 속성들도 함께 표시된다. <br>
이러한 속성들은 **<u>상위 객체(프로토타입)로부터 상속된 것들이다.</u>**
![Object](/assets/images/source/260302_img/Prototype1.png "Prototype Pollution 예시")

<br><br>

### 프로토타입 체인(Prototype Chain)
자바스크립트에서 모든 객체는 상위 객체와 연결되어 있다. 만약 객체에 없는 속성을 호출하면 어디서 값을 찾게 될까?
바로 상위 객체로 올라가서 해당 속성을 찾게 된다. 이 과정을 프로토타입 체인(Prototype Chain)이라고 한다.

JavaScript의 모든 객체는 상위 객체와 연결되어 있다.
그렇다면, 객체에 없는 속성을 호출하면 어디서 값을 찾게 될까?
이 탐색 과정을 <span style="color: blue; text-decoration: underline; font-weight: bold">프로토타입 체인(Prototype Chain)</span>이라고 한다.

<br>

### 프로토타입 오염(Prototype Pollution)
<span style="color: red; text-decoration: underline; font-weight: bold">프로토타입 오염(Prototype Pollution)</span>은 공격자가 외부 입력을 통해 JavaScript 객체의 상위 프로토타입에 임의의 속성을 추가하거나 수정하는 취약점으로 데이터를 파싱하거나, 객체를 병합 또는 복사할 때 주로 발생한다.

일반적으로 공격자는 `__proto__`나 `constructor.prototype` 같은 특수 키를 이용해 프로토타입 자체를 오염시킬 수 있으며, 구현된 로직에 따라 다르겠지만 <span style="color: red; text-decoration: underline; font-weight: bold"><u>권한 상승, 설정 조작, 원격 코드 실행(RCE) 등의 위험</u></span>이 발생할 수 있다.


바로, 샘플 코드로 살펴보자. (코드는 Gemini 시키자...)
```JavaScript
function merge(target, source) {
    for (let key in source) {
        // 취약점: key가 '__proto__'인지 확인하지 않음
        if (typeof source[key] === 'object' && source[key] !== null) {
            merge(target[key], source[key]);
        } else {
            target[key] = source[key];
        }
    }
    return target;
}

// 공격자가 조작한 JSON 데이터
const maliciousPayload = JSON.parse('{"__proto__": {"coco": "WallWall"}}');

const user = {};
console.log(user.coco); // undefined (아직 오염 전)

merge(user, maliciousPayload);

// 오염 발생 후 결과
console.log(user.coco); // "WallWall" (프로토타입 체인을 통해 접근)
console.log({}.coco);   // "WallWall" (새로 생성한 빈 객체도 오염됨)

Object.prototype // {coco: 'WallWall', __defineGetter__: ƒ, __defineSetter__: ƒ, hasOwnProperty: ƒ, …}
```
위와 같이 데이터를 merge하는 과정에서 키 검증이 없어 Object.prototype에 coco라는 속성이 추가된 것을 확인할 수 있다.

<br><br>

### 프로토타입 오염(Prototype Pollution) 대응방안
위와 같이 Prototype Pollution은 웹 서비스 내에서 객체를 어떻게 사용하냐에 따라 달라지지만, 클라이언트와 서버 사이드를 모두 공격할 수 있는 파급력이 높은 취약점이라고 볼 수 있다.

1. 프로퍼티 키 검증
- "\_\_proto\_\_", "constructor"와 같은 키를 허용하지 않도록 검증

2. 프로토타입 객체 변경 방지
- Object.freeze()와 Object.seal() 같은 함수 사용
(* Object.freeze: 객체 속성과 값 수정 불가, Object.seal: 새 속성 추가 불가/기존 속성 변경 가능)

3. hasOwnproperty 검증
- 객체 속성을 접근할 때, 실제 객체가 갖고 있는 값인지 확인

<br><br>

ps. Burp CE에서 DOM Invader 기능을 사용하면 취약 포인트를 쉽게 찾을 수 있다...!

<br><br>

## ✅ 참고
<a href="https://portswigger.net/web-security/learning-paths/prototype-pollution">Prototype-pollution(PortSwigger)</a><br>
<a href="https://www.hahwul.com/cullinan/attack/prototype-pollution/">Prototype-pollution blog</a>