---
layout: post
title: Prologue Epilogue
subtitle: function prologue & epilogue, Calling convention
categories: Pwnable
tags: [Function Call, Function Prologue, Function Epilogue, Calling convention]
---

### ✅ Function Prologue & Epilogue
오랜만에 포너블을 다시보니 하나도 기억이 나지 않아 기초부터 한 번 정리해보려고 한다.<br>
64bit Ubuntu 환경(22.04.02 LTS)에서 함수의 프롤로그 에필로그부터 함수 호출 규약까지 간단하게 살펴보자. 

<br>

아! 우선 프롤로그(Prologue)는 함수가 호출되기 전 원래의 실행 흐름으로 돌아가기 위해 이전 함수의 상태(ex. 베이스포인터)를 저장하고 새로운 스택프레임을 만들어 변수나 레지스터를 쓸 수 있도록 준비하는 작업이다.

반대로 에필로그(Epilogue)는 함수의 실행이 끝나 사용한 스택프레임을 정리하고 원래의 실행 흐름으로 돌아가기 위한 작업이다.



이제 바로 코드를 통해 알아보자. <br>
아래의 코드를 통해 **함수의 프롤로그 & 에필로그**가 어셈블리어로 어떻게 수행되는지랑 **함수의 호출 규약(Calling convention)**에 대해 살펴보자.

```C
// gcc -o basic basic.c -fno-pie -no-pie

#include <stdio.h>

int main(){
	char name[10];

	printf("[!] Main Started!\n");
	printf("[*] Input name: \n");
	read(0, name, 9); // name 변수에 문자열 저장

	func(name, 1, 2, 3, 4, 5, 6);
	return 0;
}

void func(char *name, int num1, int num2, int num3, int num4, int num5, int num6){
	int sum;

	printf("Hi, %s", name);
	sum = num1 + num2 + num3 + num4 + num5 + num6;
	printf("Sum: %d", sum);
}
```

위의 코드로 테스트해보자.

```cmd
gcc -o basic basic.c -fno-pie -no-pie
```
PIE가 걸려있으면 디버깅할 때 주소가 보기 싫으니 PIE는 꺼두도록 하자.

컴파일하고 _checksec_ 로 바이너리의 보호기법을 확인해보면 아래와 같다.
```cmd
[*] 'basic'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```



gdb로 고고~!
main 함수에서 func 함수로 진입하기 전 어셈블리는 아래와 같다.
```gdb
   0x4011ea <main+84>     lea    rax, [rbp - 0x12] // $rbp-0x12 = coco
   0x4011ee <main+88>     sub    rsp, 8
   0x4011f2 <main+92>     push   6
   0x4011f4 <main+94>     mov    r9d, 5
   0x4011fa <main+100>    mov    r8d, 4
 ► 0x401200 <main+106>    mov    ecx, 3
   0x401205 <main+111>    mov    edx, 2
   0x40120a <main+116>    mov    esi, 1
   0x40120f <main+121>    mov    rdi, rax
   0x401212 <main+124>    mov    eax, 0
   0x401217 <main+129>    call   func
```

#### ✅ Calling Convention

우선 함수 호출 규약부터 살펴보자.

<div style="width: 100%; max-width: 800px; min-height: 50px; background-color: #ffebee; border-left: 5px solid #f44336; padding: 15px; margin: 10px 0; box-shadow: 0 2px 4px rgba(0,0,0,0.1); box-sizing: border-box;">
  <strong>⚠️ 중요</strong>
  리눅스 환경의 함수 호출 규약(System V AMD64 ABI), rdi → rsi → rdx → rcx → r8 → r9 순으로 레지스터에 인자를 전달한다.
  그 뒤 인자부터는 스택을 활용한다.
</div>

어셈블리어를 보면 순서대로 아래와 같이 func 함수에 값을 전달하고 있다.<br>
rdi = "coco"라는 문자열을 저장한 주소 <br>
esi(rsi) = 1 <br>
edx(rdx) = 2 <br>
ecx(rcx) = 3 <br>
r8d(r8) = 4 <br>
r9d(r9) = 5 <br>
7번째 인자인 6은 Stack에 값을 넣는다.(push 6) <br>
(※ 숫자데이터는 4바이트만 써도 충분하기 때문에 8바이트 레지스터가 아닌 4바이트 레지스터를 쓰는 것 같다.)

<br>

gdb로 Func 함수 호출 직전으로 가보면 다음과 같다.
```gdb
 ► 0x401217 <main+129>    call   func                      <func>
        rdi: 0x7fffffffe1ee ◂— 0xa6f636f63 /* 'coco\n' */
        rsi: 0x1
        rdx: 0x2
        rcx: 0x3
─────────────────────[ REGISTER ]─────────────────────
*RAX  0x0
 RBX  0x0
 RCX  0x3
 RDX  0x2
 RDI  0x7fffffffe1ee ◂— 0xa6f636f63 /* 'coco\n' */
 RSI  0x1
 R8   0x4
 R9   0x5
```

#### ✅ Prologue

이제는 func 함수로 들어가서 프롤로그와 에필로그를 살펴보자. <br>
si 명령어로 함수 내부로 들어갈 수 있다.

```gdb
 ► 0x40123b <func>       endbr64
   0x40123f <func+4>     push   rbp
   0x401240 <func+5>     mov    rbp, rsp
   0x401243 <func+8>     sub    rsp, 0x30
```

요 부분이 함수의 프롤로그이다. <br>
push rbp는 두 스텝으로 나눠볼 수 있는데 rsp -= 0x8 → [rsp] = rbp로 보면 된다.

즉, rsp = rsp - 0x8을 수행하고 rbp의 값을 rsp가 가르키는 곳에 저장한다.

<br>

```gdb
   0x40123b <func>       endbr64
 ► 0x40123f <func+4>     push   rbp
   0x401240 <func+5>     mov    rbp, rsp
   0x401243 <func+8>     sub    rsp, 0x30
   0x401247 <func+12>    mov    qword ptr [rbp - 0x18], rdi
   0x40124b <func+16>    mov    dword ptr [rbp - 0x1c], esi
   0x40124e <func+19>    mov    dword ptr [rbp - 0x20], edx
   0x401251 <func+22>    mov    dword ptr [rbp - 0x24], ecx
   0x401254 <func+25>    mov    dword ptr [rbp - 0x28], r8d
   0x401258 <func+29>    mov    dword ptr [rbp - 0x2c], r9d
   0x40125c <func+33>    mov    rax, qword ptr [rbp - 0x18]
─────────────────────[ STACK ]─────────────────────
00:0000│ rsp   0x7fffffffe1c8 —▸ 0x40121c (main+134) ◂— add rsp, 0x10
01:0008│       0x7fffffffe1d0 ◂— 0x6
02:0010│       0x7fffffffe1d8 —▸ 0x4011ea (main+84) ◂— lea rax, [rbp - 0x12]
03:0018│       0x7fffffffe1e0 —▸ 0x7fffffffe5b9 ◂— 0x34365f363878 /* 'x86_64' */
04:0020│ rdi-6 0x7fffffffe1e8 ◂— 0x6f63000000000064 /* 'd' */
05:0028│       0x7fffffffe1f0 ◂— 0xa6f63 /* 'co\n' */
06:0030│       0x7fffffffe1f8 ◂— 0xb33a8bc078d13000
07:0038│ rbp   0x7fffffffe200 ◂— 0x1
```
rsp가 가르키는 주소는 0x7fffffffe1c8-0x8이고 그 주소 안의 값은 0x7fffffffe200가 되어야할 것이다.


```gdb
   0x40123b <func>       endbr64
   0x40123f <func+4>     push   rbp
 ► 0x401240 <func+5>     mov    rbp, rsp
   0x401243 <func+8>     sub    rsp, 0x30
─────────────────────[ STACK ]─────────────────────
00:0000│ rsp   0x7fffffffe1c0 —▸ 0x7fffffffe200 ◂— 0x1
01:0008│       0x7fffffffe1c8 —▸ 0x40121c (main+134) ◂— add rsp, 0x10
02:0010│       0x7fffffffe1d0 ◂— 0x6
03:0018│       0x7fffffffe1d8 —▸ 0x4011ea (main+84) ◂— lea rax, [rbp - 0x12]
04:0020│       0x7fffffffe1e0 —▸ 0x7fffffffe5b9 ◂— 0x34365f363878 /* 'x86_64' */
05:0028│ rdi-6 0x7fffffffe1e8 ◂— 0x6f63000000000064 /* 'd' */
06:0030│       0x7fffffffe1f0 ◂— 0xa6f63 /* 'co\n' */
07:0038│       0x7fffffffe1f8 ◂— 0xb33a8bc078d13000
```

예상한 대로 움직였다. <br>
이후에는 rsp의 값을 rbp에 넣고, rsp-0x30으로 사용할 스택 영역을 확보한다.

```gdb
   0x40123b <func>       endbr64
   0x40123f <func+4>     push   rbp
   0x401240 <func+5>     mov    rbp, rsp
   0x401243 <func+8>     sub    rsp, 0x30
 ► 0x401247 <func+12>    mov    qword ptr [rbp - 0x18], rdi
─────────────────────[ STACK ]─────────────────────
00:0000│ rsp 0x7fffffffe190 —▸ 0x402016 ◂— '[*] Input name: '
01:0008│     0x7fffffffe198 —▸ 0x7ffff7c80faa (puts+346) ◂— cmp eax, -1
02:0010│     0x7fffffffe1a0 —▸ 0x400040 ◂— 0x400000006
03:0018│     0x7fffffffe1a8 —▸ 0x7ffff7fe283c (_dl_sysdep_start+1020) ◂— mov rax, qword ptr [rsp + 0x58]
04:0020│     0x7fffffffe1b0 ◂— 0x0
05:0028│     0x7fffffffe1b8 —▸ 0x7fffffffe200 ◂— 0x1
06:0030│ rbp 0x7fffffffe1c0 —▸ 0x7fffffffe200 ◂— 0x1
```



#### ✅ Epilogue

다음으로는 에필로그이다. <br>
함수가 종료될 때 수행되는 작업으로 바로 디버거로 살펴보자.

```gdb
 ► 0x4012af <func+116>    nop
   0x4012b0 <func+117>    leave; rsp = rbp, pop rbp(rbp=[rsp], rsp+=8)
   0x4012b1 <func+118>    ret; rip = [rsp], rsp += 8
─────────────────────[ STACK ]─────────────────────
00:0000│ rsp 0x7fffffffe190 ◂— 0x500402016
01:0008│     0x7fffffffe198 ◂— 0x300000004
02:0010│     0x7fffffffe1a0 ◂— 0x100000002
03:0018│     0x7fffffffe1a8 —▸ 0x7fffffffe1ee ◂— 0xa6f636f63 /* 'coco\n' */
04:0020│     0x7fffffffe1b0 ◂— 0x0
05:0028│     0x7fffffffe1b8 ◂— 0x15ffffe200
06:0030│ rbp 0x7fffffffe1c0 —▸ 0x7fffffffe200(main의 rbp) ◂— 0x1
07:0038│     0x7fffffffe1c8 —▸ 0x40121c (main+134) ◂— add rsp, 0x10
```

에필로그는 leave와 ret명령어로 이뤄져있는데, leave와 ret도 push와 마찬가지로 두 가지로 나눠 생각해볼 수 있다.(leave; rsp=rbp, rbp=[rsp], rsp+=8 / ret; rip=[rsp], rsp+=8) <br>

rbp의 값을 rsp에 넣고, rsp가 가르키는 값을 다시 rbp에 넣는다. 이 과정이 func 호출 전 main함수의 스택 프레임을 복원하는 과정이다.

```gdb
   0x4012b0 <func+117>    leave
 ► 0x4012b1 <func+118>    ret                                  <0x40121c; main+134>
    ↓
   0x40121c <main+134>    add    rsp, 0x10
─────────────────────[ STACK ]─────────────────────
00:0000│ rsp 0x7fffffffe1c8 —▸ 0x40121c (main+134) ◂— add rsp, 0x10
01:0008│     0x7fffffffe1d0 ◂— 0x6
02:0010│     0x7fffffffe1d8 —▸ 0x4011ea (main+84) ◂— lea rax, [rbp - 0x12]
03:0018│     0x7fffffffe1e0 —▸ 0x7fffffffe5b9 ◂— 0x34365f363878 /* 'x86_64' */
04:0020│     0x7fffffffe1e8 ◂— 0x6f63000000000064 /* 'd' */
05:0028│     0x7fffffffe1f0 ◂— 0xa6f63 /* 'co\n' */
06:0030│     0x7fffffffe1f8 ◂— 0xb33a8bc078d13000
07:0038│ rbp 0x7fffffffe200 ◂— 0x1
```

이후 rsp += 8을 통해 다음 실행할 주소를 가르킨다. rsp+=8에는 main함수의 주소가 적혀있다.
rip를 통해 실행할 주소를 설정하고 rsp는 다시 증가시킨다.

```gdb
 ► 0x40121c <main+134>    add    rsp, 0x10
   0x401220 <main+138>    mov    eax, 0
─────────────────────[ STACK ]─────────────────────
00:0000│ rsp 0x7fffffffe1d0 ◂— 0x6
01:0008│     0x7fffffffe1d8 —▸ 0x4011ea (main+84) ◂— lea rax, [rbp - 0x12]
02:0010│     0x7fffffffe1e0 —▸ 0x7fffffffe5b9 ◂— 0x34365f363878 /* 'x86_64' */
03:0018│     0x7fffffffe1e8 ◂— 0x6f63000000000064 /* 'd' */
04:0020│     0x7fffffffe1f0 ◂— 0xa6f63 /* 'co\n' */
05:0028│     0x7fffffffe1f8 ◂— 0xb33a8bc078d13000
06:0030│ rbp 0x7fffffffe200 ◂— 0x1
07:0038│     0x7fffffffe208 —▸ 0x7ffff7c29d90 (__libc_start_call_main+128) ◂— mov edi, eax
```

이로써 func 함수를 호출하기 전 실행흐름으로 돌아왔다.

끝✨



### ✅ 참고
<a href="https://ko.wikipedia.org/wiki/%ED%98%B8%EC%B6%9C_%EA%B7%9C%EC%95%BD">Wiki 함수 호출 규약</a>