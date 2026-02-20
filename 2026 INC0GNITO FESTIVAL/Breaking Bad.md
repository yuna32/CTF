Breaking Bad
============


일단 제공된 txt파일 열어보면

```
When I heard the learn’d astronomer,
When the proofs, the figures, were ranged in columns before me,
When I was shown the charts and diagrams, to add, divide, and measure them,
When I sitting heard the astronomer where he lectured with much applause in the lecture-room,

How soon unaccountable I became tired and sick,
Till rising and gliding out I wander’d off by myself,
In the mystical moist night-air, and from time to time,
Look’d up in perfect silence at the stars.
```


### 정적 분석

제공된 elf 파일을 ida에서 열어보면 main은

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  char s[264]; // [rsp+0h] [rbp-110h] BYREF
  unsigned __int64 v5; // [rsp+108h] [rbp-8h]

  v5 = __readfsqword(0x28u);
  init_buffer();
  if ( g_key_len > 0 )
  {
    printf("input: ");
    if ( fgets(s, 256, _bss_start) )
    {
      s[strcspn(s, "\r\n")] = 0;
      if ( (unsigned int)strlen(s) == 54 )
      {
        if ( (unsigned int)check_input((__int64)s) )
          puts("You're Goddamn Right");
        else
          puts("Wrong answer. Better call Saul!");
        return 0;
      }
      else
      {
        puts("Wrong length. Better call Saul!");
        return 1;
      }
    }
    else
    {
      return 1;
    }
  }
  else
  {
    puts("gale_notebook.txt missing");
    return 1;
  }
}
```

사용자로부터 256바이트 이내의 입력을 받은 뒤 두 가지 조건을 확인    
* 길이 검증: 입력값이 정확히 54글자여야 함.
* 내용 검증: check_input(s) 함수의 반환값이 True여야 함

암호화 알고리즘을 담당하는 두 함수 enc_byte와 swap4를 보면

```c
__int64 __fastcall enc_byte(char a1, int a2)
{
  return (unsigned __int8)swap4((unsigned __int8)(a2 + g_key[a2 % g_key_len]) ^ a1);
}
```


```c
__int64 __fastcall swap4(unsigned __int8 a1)
{
  return (a1 >> 4) | (16 * (unsigned int)a1);
}
```

각 입력 문자는 enc_byte 함수를 통해 암호화되어 target_0 배열과 비교

$$enc\_byte(char, idx) = swap4((idx + g\_key[idx \pmod{g\_key\_len}]) \oplus char)$$


swap4 함수는 8비트 데이터의 상위 4비트와 하위 4비트를 교체하는 역할

키 생성 메커니즘을 담당하는 init_buffer 와 extract_spaces를 보면

```c
unsigned __int64 init_buffer()
{
  int spaces; // [rsp+8h] [rbp-2018h]
  int i; // [rsp+Ch] [rbp-2014h]
  _DWORD v3[4]; // [rsp+10h] [rbp-2010h] BYREF
  unsigned __int64 v4; // [rsp+2018h] [rbp-8h]

  v4 = __readfsqword(0x28u);
  spaces = extract_spaces("gale_notebook.txt", (__int64)v3, 2048);
  if ( spaces > 0 )
  {
    if ( spaces > 256 )
      spaces = 256;
    g_key_len = spaces;
    for ( i = 0; i < spaces; ++i )
      g_key[i] = LOBYTE(v3[i]) ^ (7 * i);
  }
  else
  {
    g_key_len = 0;
  }
  return v4 - __readfsqword(0x28u);
}
```


```c
__int64 __fastcall extract_spaces(const char *a1, __int64 a2, int a3)
{
  int v4; // eax
  int v6; // [rsp+2Ch] [rbp-14h]
  unsigned int v7; // [rsp+30h] [rbp-10h]
  int v8; // [rsp+34h] [rbp-Ch]
  FILE *stream; // [rsp+38h] [rbp-8h]

  stream = fopen(a1, "rb");
  if ( !stream )
    return 0xFFFFFFFFLL;
  v6 = 0;
  v7 = 0;
  while ( 1 )
  {
    v8 = fgetc(stream);
    if ( v8 == -1 )
      break;
    if ( v8 == 32 && (int)v7 < a3 )
    {
      v4 = v7++;
      *(_DWORD *)(a2 + 4LL * v4) = v6;
    }
    ++v6;
  }
  fclose(stream);
  return v7;
}
```

init_buffer 함수는 gale_notebook.txt 파일을 바이너리 모드로 읽어 g_key를 생성
* extract_spaces: 파일 내에서 공백(0x20)이 나타나는 모든 Index을 추출
* g_key 연산: $g\_key[i] = (i\text{번째 공백의 오프셋}) \oplus (7 \times i)$


### 동적 분석 및 문제 해결


정적 분석 후 파이썬으로 역산을 시도했을 때 처음 5글자(`INCOG`)만 맞고 그 이후가 깨지는 현상이 발생

* 원인: gale_notebook.txt에 포함된 특수 기호(`’`)가 UTF-8에서 **3바이트**를 차지하고, 윈도우 스타일의 **CRLF(`\r\n`)** 줄바꿈이 사용되어 단순 텍스트 복사 시 인덱스가 틀어짐.    



정확한 g_key 값을 확인하기 위해 GDB를 사용하여 init_buffer 실행 직후의 메모리를 덤프

```bash
pwndbg> b *main
pwndbg> r
pwndbg> x/5xb &g_key
0x555555558080: 0x04  0x01  0x02  0x05  0x06

```

이 실시간 메모리 값을 통해 파일의 바이트 구조를 역으로 추적하여 정확한 키 생성 환경을 파악



### 최종 해독 스크립트

```python
def swap4(n):
    return ((n & 0x0F) << 4) | ((n & 0xF0) >> 4)

# 1. target_0 데이터 추출
target_0 = [0xD4, 0xC4, 0x74, 0x74, 0xD4, 0xA5, 0x96, 0x44, 0x34, 0x8F, ...] 

# 2. gale_notebook.txt 바이너리 분석 (CRLF + UTF-8 반영)
with open("gale_notebook.txt", "rb") as f:
    content = f.read()
space_indices = [i for i, b in enumerate(content) if b == 0x20]

# 3. g_key 및 플래그 복구
flag = ""
for i in range(54):
    pos = space_indices[i]
    key_val = (pos ^ (7 * i)) & 0xFF
    # 역산: target -> swap4 -> XOR with (i + key)
    flag += chr(swap4(target_0[i]) ^ ((i + key_val) & 0xFF))

print(f"Final Decoded Flag: {flag}")

```


<img width="985" height="75" alt="image" src="https://github.com/user-attachments/assets/5925e08d-3169-4b8a-82cf-b040a5e8cf57" />



