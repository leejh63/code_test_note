# 20. Valid Parentheses

> 상태: SOLVED  
> 플랫폼: LeetCode  
> 언어: C  
> 문제 링크: https://leetcode.com/problems/valid-parentheses/  
> 최초 작성일: 2026-08-09  
> 마지막 수정일: 2026-08-09

---

## 1. 간단 문제 설명

문자열 `s`에는 다음 여섯 종류의 문자만 들어온다.

```text
( ) [ ] { }
```

주어진 괄호 문자열이 올바른 괄호 구조인지 판단한다.

### 유효 조건

1. 열린 괄호는 같은 종류의 닫힌 괄호로 닫혀야 한다.
2. 나중에 열린 괄호가 먼저 닫혀야 한다.
3. 모든 닫힌 괄호에는 대응하는 열린 괄호가 존재해야 한다.

예:

```text
"()"      → true
"()[]{}"  → true
"(]"      → false
"([])"    → true
"([)]"    → false
```

### 입력 조건

```text
1 <= s.length <= 10^4
```

### LeetCode

https://leetcode.com/problems/valid-parentheses/

---

# 2. 처음 생각한 접근

처음에는 Stack을 바로 사용하지 않고 문자열 자체에서 이전 열린 괄호를 다시 찾는 방법을 생각했다.

대략적인 사고 과정은 다음과 같았다.

```text
문자열을 처음부터 끝까지 확인

열린 괄호
→ 일단 지나감

닫힌 괄호
→ 앞쪽으로 이동
→ 아직 사용되지 않은 가장 최근 열린 괄호를 찾음
→ 현재 닫힌 괄호와 비교
```

여기서 핵심 문제는 다음이었다.

```text
이미 한 쌍을 이룬 괄호는 어떻게 건너뛸 것인가?
```

이를 위해 뒤쪽에서 앞으로 탐색하며 다음과 같은 카운터를 생각했다.

```text
닫힌 괄호 → +1
열린 괄호 → -1
```

앞쪽으로 이동하면서 이미 완성된 괄호 쌍은 상쇄하고, 아직 대응되는 닫힌 괄호가 없는 열린 괄호를 찾는 방식이다.

이 방법은 실제로 구현할 수 있지만, 닫힌 괄호를 만날 때마다 이전 영역을 다시 탐색하기 때문에 반복 탐색이 발생한다.

이후 문제의 핵심이 다음과 같다는 것을 발견했다.

```text
닫힌 괄호를 발견했을 때 필요한 것은

"가장 최근에 열린 괄호"
```

즉 다음 구조이다.

```text
Last In First Out
LIFO
```

따라서 Stack이 적합한 자료구조라고 판단했다.

---

# 3. 내가 처음 작성한 Stack 풀이

## 내가 작성한 코드

```c
#define MAX_STACK_SIZE 10000

typedef struct s_stack {
    char regi[MAX_STACK_SIZE];
    size_t top;
} t_stack;

void stack_init(t_stack* stack){
    stack->top = 0;
    stack->regi[stack->top] = '\0';
}

char stack_top(t_stack* stack)
{
    return stack->regi[stack->top];
}

bool stack_pop(t_stack* stack, char* out_put)
{
    if (stack->top == 0)
    {
        if (out_put != NULL)
            *out_put = '\0';
        return false;
    }

    if (out_put != NULL)
        *out_put = stack_top(stack);

    stack->regi[stack->top] = '\0';
    stack->top = stack->top - 1;

    return true;
}

bool stack_push(t_stack* stack, char in_put)
{
    if (stack->top == MAX_STACK_SIZE)
        return false;

    stack->top = stack->top + 1;
    stack->regi[stack->top] = in_put;

    return true;
}

bool isValid(char* s)
{
    t_stack stack;
    stack_init(&stack);

    char stk_top;
    size_t i = 0;
    bool valid;

    while (s[i] != '\0')
    {
        if (s[i] == '(' || s[i] == '[' || s[i] == '{')
        {
            valid = stack_push(&stack, s[i]);

            if (!valid)
                return false;
        }
        else
        {
            stk_top = stack_top(&stack);

            if (stk_top == '\0'
                || stk_top == '(' && s[i] != ')'
                || stk_top == '[' && s[i] != ']'
                || stk_top == '{' && s[i] != '}')
            {
                return false;
            }

            valid = stack_pop(&stack, NULL);

            if (!valid)
                return false;
        }

        ++i;
    }

    valid = stack_pop(&stack, NULL);

    return !valid;
}
```

---

## 접근 방법

여는 괄호를 발견하면 Stack에 넣는다.

```text
(
[
{
```

닫는 괄호를 발견하면 Stack의 가장 위에 있는 괄호와 비교한다.

예:

```text
입력: ([])

'('
Stack:
(

'['
Stack:
( [

']'
Stack Top = '['
→ 정상
→ pop

')'
Stack Top = '('
→ 정상
→ pop

문자열 끝
Stack Empty
→ true
```

이 문제의 핵심인:

```text
가장 최근에 열린 괄호가
가장 먼저 닫혀야 한다.
```

를 Stack의 LIFO 구조로 바로 표현할 수 있다.

---

# 4. 처음 작성한 코드의 문제점

## 4.1 `regi[0]`을 사용하지 않는다

초기 상태는:

```c
stack->top = 0;
```

이다.

그런데 push에서는:

```c
stack->top = stack->top + 1;
stack->regi[stack->top] = in_put;
```

을 사용한다.

따라서 첫 번째 데이터는:

```text
regi[1]
```

에 들어간다.

결과적으로:

```text
regi[0]
```

은 Stack 데이터 저장에 사용되지 않는다.

---

## 4.2 배열 범위를 넘어갈 수 있다

배열이:

```c
char regi[10000];
```

이라면 사용할 수 있는 인덱스는:

```text
0 ~ 9999
```

이다.

하지만 기존 코드는:

```c
if (stack->top == MAX_STACK_SIZE)
    return false;

stack->top = stack->top + 1;
stack->regi[stack->top] = in_put;
```

으로 되어 있다.

`top == 9999`라면 검사에 걸리지 않는다.

그다음:

```text
top = 10000
```

이 되고:

```c
stack->regi[10000]
```

에 접근한다.

이는 배열 범위를 벗어난 접근이다.

---

## 4.3 `top`의 정의가 복잡하다

처음 구현에서는:

```text
top = 마지막 데이터의 인덱스
```

처럼 사용하면서:

```text
top == 0
```

을 Empty 상태로도 사용했다.

더 단순한 정의는 다음과 같다.

```text
top = 현재 저장된 원소 개수
    = 다음 push가 사용할 인덱스
```

이렇게 정의하면:

```text
top == 0
→ Empty

top == MAX_STACK_SIZE
→ Full

마지막 데이터
→ regi[top - 1]
```

로 명확해진다.

---

# 5. 수정한 Stack 풀이

## 개선 코드

```c
#define MAX_STACK_SIZE 10000

typedef struct s_stack
{
    char regi[MAX_STACK_SIZE];
    size_t top;
} t_stack;

void stack_init(t_stack* stack)
{
    stack->top = 0;
}

char stack_top(t_stack* stack)
{
    if (stack->top == 0)
        return '\0';

    return stack->regi[stack->top - 1];
}

bool stack_pop(t_stack* stack, char* output)
{
    if (stack->top == 0)
    {
        if (output != NULL)
            *output = '\0';

        return false;
    }

    stack->top = stack->top - 1;

    if (output != NULL)
        *output = stack->regi[stack->top];

    stack->regi[stack->top] = '\0';

    return true;
}

bool stack_push(t_stack* stack, char input)
{
    if (stack->top >= MAX_STACK_SIZE)
        return false;

    stack->regi[stack->top] = input;
    stack->top = stack->top + 1;

    return true;
}

bool isValid(char* s)
{
    t_stack stack;
    stack_init(&stack);

    char stk_top;
    size_t i = 0;

    while (s[i] != '\0')
    {
        if (s[i] == '(' || s[i] == '[' || s[i] == '{')
        {
            if (!stack_push(&stack, s[i]))
                return false;
        }
        else
        {
            stk_top = stack_top(&stack);

            if (stk_top == '\0'
                || (stk_top == '(' && s[i] != ')')
                || (stk_top == '[' && s[i] != ']')
                || (stk_top == '{' && s[i] != '}'))
            {
                return false;
            }

            if (!stack_pop(&stack, NULL))
                return false;
        }

        ++i;
    }

    return stack.top == 0;
}
```

---

## 변경된 Stack 구조

처음:

```text
top = 0

regi
[ ][ ][ ][ ]
 ^
 다음 저장 위치
```

`(` push:

```text
regi[0] = '('
top = 1

[(][ ][ ][ ]
    ^
```

`[` push:

```text
regi[1] = '['
top = 2

[(][[][ ][ ]
       ^
```

따라서 Stack Top은:

```c
stack->regi[stack->top - 1]
```

이다.

---

## 복잡도

문자열의 각 문자를 한 번씩 확인한다.

```text
시간 복잡도: O(n)
공간 복잡도: O(n)
```

최대 Stack 배열:

```text
10,000 × sizeof(char)
= 10,000 byte
≈ 9.77 KiB
```

이다.

---

# 6. 추가 풀이 1 — 입력 문자열 자체를 Stack으로 사용

## 아이디어

별도로:

```c
char stack[10000];
```

을 만들지 않고 이미 전달받은 문자열 `s`의 앞부분을 Stack으로 사용할 수 있다.

문자열은 왼쪽에서 오른쪽으로 읽는다.

따라서 이미 읽고 처리가 끝난 앞쪽 영역의 원래 값은 더 이상 필요하지 않다.

두 개의 위치만 관리한다.

```text
read
→ 현재 입력을 읽는 위치

top
→ Stack의 다음 저장 위치
```

구조를 그림으로 보면:

```text
┌──────────────────┬──────────────────────┐
│ Stack으로 재사용 │ 아직 읽지 않은 문자열 │
└──────────────────┴──────────────────────┘
0                top                    read
```

여는 괄호:

```c
s[top] = s[read];
++top;
```

닫는 괄호:

```c
s[top - 1]
```

과 비교한 후:

```c
--top;
```

한다.

---

## 코드

```c
bool isValid(char* s)
{
    size_t read = 0;
    size_t top = 0;

    while (s[read] != '\0')
    {
        if (s[read] == '(' || s[read] == '[' || s[read] == '{')
        {
            s[top] = s[read];
            ++top;
        }
        else
        {
            if (top == 0)
                return false;

            char open = s[top - 1];

            if ((open == '(' && s[read] != ')')
                || (open == '[' && s[read] != ']')
                || (open == '{' && s[read] != '}'))
            {
                return false;
            }

            --top;
        }

        ++read;
    }

    return top == 0;
}
```

---

## 예시

입력:

```text
()[{
```

`()`까지 처리했다면:

```text
top  = 0
read = 2
```

이다.

현재:

```text
index   0 1 2 3
        ( ) [ {
            ^
           read
```

`[`는 여는 괄호이므로:

```c
s[top] = s[read];
```

즉:

```c
s[0] = s[2];
```

가 된다.

문자열 메모리는:

```text
기존
( ) [ {

변경
[ ) [ {
```

가 된다.

`index 0`의 원래 `(`는 이미 처리가 끝났기 때문에 없어져도 상관없다.

---

## 장단점

장점:

```text
별도 Stack 배열 없음
추가 공간 O(1)
시간 O(n)
```

단점:

```text
입력 문자열 s를 수정한다.
```

따라서 입력 데이터를 그대로 보존해야 하는 경우에는 사용할 수 없다.

---

# 7. 추가 풀이 2 — 최초 아이디어인 뒤쪽 재탐색

## 아이디어

Stack을 따로 저장하지 않는다.

닫힌 괄호를 발견할 때마다 현재 위치에서 앞쪽으로 이동하면서 아직 사용되지 않은 가장 최근 열린 괄호를 찾는다.

뒤로 탐색할 때:

```text
닫힌 괄호
→ count++

열린 괄호
→ count > 0
   → count--
   → 이미 다른 닫힌 괄호와 대응되는 괄호

→ count == 0
   → 아직 대응되지 않은 가장 최근 열린 괄호
```

가 된다.

---

## 예시

다음까지 정상적으로 처리됐다고 해보자.

```text
( [ { } ]
          ^
```

현재 `]`보다 앞쪽을 다시 탐색하면:

```text
}
→ count = 1

{
→ count = 0

[
→ count == 0인 상태에서 열린 괄호 발견
→ 현재 ]와 비교할 후보
```

따라서:

```text
[ + ]
```

가 정상적인 쌍인지 확인할 수 있다.

---

## 코드

```c
#include <stdbool.h>

bool is_open(char c)
{
    return c == '(' || c == '[' || c == '{';
}

bool is_close(char c)
{
    return c == ')' || c == ']' || c == '}';
}

bool is_pair(char open, char close)
{
    if (open == '(' && close == ')')
        return true;

    if (open == '[' && close == ']')
        return true;

    if (open == '{' && close == '}')
        return true;

    return false;
}

int find_open(char* s, int current)
{
    int count = 0;

    for (int i = current - 1; i >= 0; --i)
    {
        if (is_close(s[i]))
        {
            ++count;
        }
        else if (is_open(s[i]))
        {
            if (count == 0)
                return i;

            --count;
        }
    }

    return -1;
}

bool isValid(char* s)
{
    int i = 0;
    int balance = 0;

    while (s[i] != '\0')
    {
        if (is_open(s[i]))
        {
            ++balance;
        }
        else
        {
            --balance;

            if (balance < 0)
                return false;

            int open_index = find_open(s, i);

            if (open_index == -1)
                return false;

            if (!is_pair(s[open_index], s[i]))
                return false;
        }

        ++i;
    }

    return balance == 0;
}
```

---

## 왜 Stack보다 느린가?

Stack 방식에서는 각 문자를 한 번 확인한다.

```text
여는 괄호 → push
닫는 괄호 → pop
```

하지만 역탐색에서는 닫힌 괄호가 나올 때마다 이미 확인했던 앞쪽 문자열을 다시 확인할 수 있다.

예를 들어 깊게 중첩된 문자열:

```text
((((()))))
```

에서는 뒤쪽의 닫힌 괄호일수록 더 많은 이전 문자를 다시 확인한다.

따라서 최악의 경우:

```text
시간 복잡도: O(n²)
추가 공간: O(1)
```

이 된다.

즉 이 방식은:

```text
메모리를 거의 사용하지 않는 대신
같은 문자열 영역을 반복 탐색한다.
```

라는 특징이 있다.

---

# 8. 추가 풀이 3 — 2bit 압축 Stack

## 시작한 의문

일반 Stack은:

```c
char regi[10000];
```

을 사용한다.

`char` 하나는 1 byte, 즉 보통 8bit이다.

하지만 실제 Stack에 저장해야 하는 열린 괄호는 단 세 종류다.

```text
(
[
{
```

3개의 상태를 표현하려면 8bit 전체가 필요하지 않다.

다음처럼 표현할 수 있다.

```text
( → 00
[ → 01
{ → 10
```

따라서 괄호 하나에 필요한 공간은:

```text
2bit
```

이다.

---

## 1 byte에 괄호 4개 저장

1 byte는:

```text
8bit
```

이다.

괄호 하나는:

```text
2bit
```

만 필요하므로:

```text
8 / 2 = 4
```

즉 `uint8_t` 하나에 열린 괄호 4개를 저장할 수 있다.

예:

```text
괄호

( [ { (

인코딩

00 01 10 00
```

하나의 byte를 비트 단위로 보면:

```text
bit
7 6 | 5 4 | 3 2 | 1 0
-----------------------
 00 |  10 |  01 |  00
```

가 된다.

---

## 필요한 배열 크기

일반 `char` Stack:

```text
10,000 × 8bit
= 80,000bit
= 10,000byte
```

2bit 압축 Stack:

```text
10,000 × 2bit
= 20,000bit
= 2,500byte
```

따라서 Stack 저장공간을 약 1/4로 줄일 수 있다.

```text
10,000 byte
      ↓
 2,500 byte
```

75% 감소한다.

---

## 코드

```c
#include <stdbool.h>
#include <stddef.h>
#include <stdint.h>

#define MAX_STACK_SIZE 10000
#define STACK_BYTE_SIZE ((MAX_STACK_SIZE + 3) / 4)

#define PAREN_ROUND   0u
#define PAREN_SQUARE  1u
#define PAREN_CURLY   2u

typedef struct s_stack
{
    uint8_t regi[STACK_BYTE_SIZE];
    size_t top;
} t_stack;

void stack_init(t_stack* stack)
{
    stack->top = 0;
}

uint8_t encode_open(char c)
{
    if (c == '(')
        return PAREN_ROUND;

    if (c == '[')
        return PAREN_SQUARE;

    return PAREN_CURLY;
}

bool stack_push(t_stack* stack, char input)
{
    if (stack->top >= MAX_STACK_SIZE)
        return false;

    size_t byte_index = stack->top / 4;
    size_t bit_offset = (stack->top % 4) * 2;

    uint8_t value = encode_open(input);

    stack->regi[byte_index] &=
        (uint8_t)~(0x03u << bit_offset);

    stack->regi[byte_index] |=
        (uint8_t)(value << bit_offset);

    ++stack->top;

    return true;
}

bool stack_top(t_stack* stack, uint8_t* output)
{
    if (stack->top == 0)
        return false;

    size_t index = stack->top - 1;

    size_t byte_index = index / 4;
    size_t bit_offset = (index % 4) * 2;

    *output =
        (stack->regi[byte_index] >> bit_offset) & 0x03u;

    return true;
}

bool stack_pop(t_stack* stack, uint8_t* output)
{
    if (stack->top == 0)
        return false;

    --stack->top;

    size_t byte_index = stack->top / 4;
    size_t bit_offset = (stack->top % 4) * 2;

    if (output != NULL)
    {
        *output =
            (stack->regi[byte_index] >> bit_offset) & 0x03u;
    }

    return true;
}

bool is_match(uint8_t open, char close)
{
    if (open == PAREN_ROUND && close == ')')
        return true;

    if (open == PAREN_SQUARE && close == ']')
        return true;

    if (open == PAREN_CURLY && close == '}')
        return true;

    return false;
}

bool isValid(char* s)
{
    t_stack stack;
    stack_init(&stack);

    size_t i = 0;
    uint8_t open;

    while (s[i] != '\0')
    {
        if (s[i] == '(' || s[i] == '[' || s[i] == '{')
        {
            if (!stack_push(&stack, s[i]))
                return false;
        }
        else
        {
            if (!stack_top(&stack, &open))
                return false;

            if (!is_match(open, s[i]))
                return false;

            stack_pop(&stack, NULL);
        }

        ++i;
    }

    return stack.top == 0;
}
```

---

# 9. 2bit Stack 동작 이해

## 9.1 어느 byte를 사용할 것인가?

다음 계산을 사용한다.

```c
byte_index = top / 4;
```

하나의 byte에 괄호 4개를 저장하기 때문이다.

예:

```text
top = 0,1,2,3
→ regi[0]

top = 4,5,6,7
→ regi[1]

top = 8,9,10,11
→ regi[2]
```

---

## 9.2 byte 내부에서 어느 bit를 사용할 것인가?

```c
bit_offset = (top % 4) * 2;
```

를 사용한다.

예:

```text
top = 0
→ offset 0
→ bit 0~1

top = 1
→ offset 2
→ bit 2~3

top = 2
→ offset 4
→ bit 4~5

top = 3
→ offset 6
→ bit 6~7
```

따라서 한 byte는:

```text
7 6 | 5 4 | 3 2 | 1 0
-----------------------
 #4 |  #3 |  #2 |  #1
```

형태로 사용된다.

---

## 9.3 값을 저장하는 과정

예를 들어 `[`를 저장한다고 하자.

```text
[ = 01
```

먼저 저장할 2bit 위치를 찾는다.

가령:

```text
bit_offset = 4
```

라면:

```text
00 00 00 01
```

을 왼쪽으로 4bit 이동한다.

```text
01 << 4

00010000
```

그리고 해당 byte와 OR 연산한다.

```c
regi[byte_index] |= value << bit_offset;
```

결과적으로 원하는 bit 위치에 값이 들어간다.

---

## 9.4 기존 값 제거

같은 2bit 위치에 이전 값이 남아 있을 수도 있다.

따라서 저장 전에:

```c
stack->regi[byte_index] &=
    (uint8_t)~(0x03u << bit_offset);
```

을 사용한다.

`0x03`은:

```text
00000011
```

이다.

원하는 위치로 shift한 뒤 반전하면 해당 2bit만 `0`으로 만들 수 있는 mask가 된다.

즉:

```text
기존 2bit 제거
↓
새로운 2bit 기록
```

순서로 처리한다.

---

## 9.5 값을 읽는 과정

Stack에서 값을 꺼낼 때:

```c
(stack->regi[byte_index] >> bit_offset) & 0x03u
```

를 사용한다.

예를 들어:

```text
regi

00100100
```

에서 필요한 값이 bit 2~3이라면:

```text
00100100 >> 2

00001001
```

이 된다.

하지만 여기에는 다른 bit도 포함되어 있다.

우리가 필요한 것은 마지막 2bit뿐이므로:

```text
00001001
&
00000011
----------
00000001
```

을 수행한다.

결과:

```text
01
```

따라서:

```text
[
```

로 해석할 수 있다.

---

# 10. 네 가지 방법 비교

| 방법 | 시간 복잡도 | 추가 공간 | 특징 |
|---|---:|---:|---|
| 뒤쪽 재탐색 | O(n²) | O(1) | 최초에 생각한 방법 |
| 일반 `char` Stack | O(n) | O(n) | 가장 단순하고 정석적 |
| 입력 문자열을 Stack으로 재사용 | O(n) | O(1) | 입력 문자열 수정 |
| 2bit 압축 Stack | O(n) | O(n) | Stack 메모리를 약 1/4로 감소 |

여기서 2bit Stack도 공간 복잡도 표기 자체는:

```text
O(n)
```

이다.

일반 Stack:

```text
n byte
```

2bit Stack:

```text
n / 4 byte
```

이기 때문이다.

Big-O에서는 상수 `1/4`를 제거하므로 둘 다 `O(n)`이다.

하지만 실제 사용되는 메모리 크기는 크게 다르다.

---

# 11. 어떤 방법이 가장 적절한가?

## 문제 풀이 기준

가장 적절한 기본 풀이는:

```text
일반 배열 Stack
```

이다.

이유:

- 시간 복잡도 O(n)
- 구현이 단순함
- Stack 개념이 명확하게 드러남
- 입력을 수정하지 않음
- 디버깅이 쉬움

---

## 메모리를 가장 적게 사용하고 싶다면

입력 변경이 허용된다는 조건에서는:

```text
입력 문자열 자체를 Stack으로 재사용
```

하는 방법이 가장 단순하면서 추가 공간도 `O(1)`이다.

---

## 비트 연산을 학습하고 싶다면

```text
2bit 압축 Stack
```

이 의미가 있다.

이 문제에서는 실질적인 성능 필요성보다는 다음 개념을 연습할 수 있다는 점이 중요하다.

```text
bit packing
bit mask
shift
AND
OR
메모리 압축
```

---

## 최초 아이디어의 의미

처음 생각한 역탐색 풀이도 틀린 접근은 아니었다.

실제로:

```text
O(n²) 시간
O(1) 공간
```

의 풀이로 구현할 수 있었다.

하지만 문제를 더 관찰하면서:

```text
"현재 닫힌 괄호가 확인해야 하는 것은
가장 최근에 열린 괄호이다."
```

라는 규칙을 발견했고, 이것이 Stack이라는 자료구조와 직접 연결됐다.

따라서 이번 문제에서 중요한 학습 과정은:

```text
직접 역탐색을 생각함
        ↓
가장 최근 열린 괄호가 중요하다는 사실 발견
        ↓
LIFO 구조 확인
        ↓
Stack 적용
        ↓
배열 Stack 직접 구현
        ↓
top 경계 조건 오류 발견 및 수정
        ↓
메모리 사용량 의문
        ↓
in-place Stack 검토
        ↓
2bit Stack까지 확장
```

으로 정리할 수 있다.

---

# 12. 질문 사항과 답

## Q1. 최대 10,000개의 배열을 만드는 것이 부담스러울 수 있지 않은가?

**A.**

```c
char stack[10000];
```

은 약 10,000 byte, 즉 약 9.77 KiB이다.

LeetCode나 일반적인 PC 환경에서는 큰 크기가 아니지만, 제한된 메모리를 생각한다면 최적화를 고민하는 것은 의미가 있다.

특히 열린 괄호는 세 종류밖에 없으므로 2bit만으로 표현할 수 있다.

---

## Q2. 처음 생각했던 `14bit 위치 + 2bit 괄호 종류` 방식은 어떤가?

**A.**

이 문제에서는 괄호의 원래 위치를 저장할 필요가 없다.

필요한 정보는:

```text
어떤 종류의 괄호가
어떤 순서로 들어왔는가
```

뿐이다.

따라서:

```text
14bit 위치
+
2bit 종류
```

로 16bit를 사용하는 것보다 그냥 종류만 2bit로 저장하는 것이 더 효율적이다.

---

## Q3. 기존 문자열을 Stack으로 활용한다는 것은 무슨 의미인가?

**A.**

이미 처리가 끝난 문자열 앞부분을 Stack 저장공간으로 덮어쓰는 방식이다.

```text
┌─────────────┬─────────────────┐
│ Stack 영역  │ 아직 읽을 문자열 │
└─────────────┴─────────────────┘
```

별도 Stack 배열이 필요하지 않지만 원래 문자열이 변경된다.

---

## Q4. 처음 생각했던 뒤쪽 재탐색 방식도 실제 구현할 수 있는가?

**A.**

가능하다.

닫힌 괄호가 나오면 앞쪽으로 다시 이동하여 이미 완성된 괄호 쌍을 건너뛰고 아직 대응되지 않은 열린 괄호를 찾을 수 있다.

다만 같은 영역을 여러 번 탐색할 수 있기 때문에 최악의 경우 시간 복잡도가 `O(n²)`이 된다.

---

## Q5. 괄호 종류를 비트로 압축해서 Stack을 만들 수 있는가?

**A.**

가능하다.

세 종류의 열린 괄호는:

```text
( → 00
[ → 01
{ → 10
```

으로 표현할 수 있다.

따라서 한 괄호당 2bit를 사용하고, 하나의 `uint8_t`에 4개의 괄호를 저장할 수 있다.

최대 10,000개 기준:

```text
일반 Stack
10,000 byte

2bit Stack
2,500 byte
```

로 줄어든다.

다만 코드가 복잡해지고 bit mask와 shift 연산이 필요하기 때문에 일반적인 문제 풀이에서는 단순한 `char` Stack 쪽이 더 읽기 쉽다.

---

# 부록

## A. 이번 문제에서 학습한 Stack의 핵심

Stack은:

```text
Last In First Out
LIFO
```

구조이다.

예:

```text
push '('

Stack
(

push '['

Stack
(
[  ← Top

pop

Stack
(  ← Top
```

Valid Parentheses에서는 가장 나중에 열린 괄호를 가장 먼저 검사해야 하기 때문에 Stack 구조와 정확하게 일치한다.

---

## B. `top` 정의의 중요성

이번 첫 구현에서 실제 버그는 알고리즘보다 Stack의 `top` 정의에서 발생했다.

처음에는:

```text
top = 마지막 데이터 인덱스
top == 0 = Empty
```

를 함께 사용했다.

수정 후:

```text
top = 저장된 데이터 개수
    = 다음 저장 위치
```

로 통일했다.

이렇게 하면:

```text
Empty
top == 0

Full
top == MAX_STACK_SIZE

Push
regi[top] = value
top++

Top
regi[top - 1]

Pop
top--
```

처럼 각 연산의 의미가 명확해진다.

---

## C. 시간과 공간의 Trade-off

이번 문제에서는 여러 풀이를 비교하면서 시간과 공간 사이의 관계도 볼 수 있었다.

```text
역탐색
메모리 ↓
반복 탐색 ↑

일반 Stack
추가 메모리 ↑
탐색 시간 ↓

2bit Stack
시간 O(n) 유지
실제 Stack 메모리 감소

in-place Stack
시간 O(n)
추가 메모리 O(1)
대신 입력 데이터 수정
```

따라서 단순히 메모리가 가장 적은 코드가 항상 가장 좋은 것은 아니다.

코드 복잡도, 입력 보존 여부, 실행 시간, 메모리 제한 등을 함께 고려해야 한다.

---

# 수정 이력

| 날짜 | 상태 | 변경 내용 |
|---|---|---|
| 2026-08-09 | SOLVED | 최초 Stack 풀이 작성 |
| 2026-08-09 | SOLVED | Stack `top` 정의 및 배열 경계 오류 수정 |
| 2026-08-09 | SOLVED | 입력 문자열 자체를 Stack으로 사용하는 O(1) 추가 공간 풀이 정리 |
| 2026-08-09 | SOLVED | 최초 아이디어인 뒤쪽 재탐색 O(n²) 풀이 구현 및 분석 |
| 2026-08-09 | SOLVED | 2bit 압축 Stack 구현, bit packing 및 mask/shift 학습 내용 추가 |