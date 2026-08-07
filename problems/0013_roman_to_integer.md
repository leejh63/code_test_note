# 0013. Roman to Integer

> 상태: SOLVED  
> 플랫폼: LeetCode  
> 언어: C  
> 문제 링크: https://leetcode.com/problems/roman-to-integer/  
> 최초 작성일: 2026-08-07  
> 마지막 수정일: 2026-08-07

---

## 1. 간단 문제 설명

로마 숫자로 표현된 문자열 `s`를 정수로 변환하는 문제이다.

| 문자 | 값 |
|---|---:|
| I | 1 |
| V | 5 |
| X | 10 |
| L | 50 |
| C | 100 |
| D | 500 |
| M | 1000 |

일반적으로 큰 값부터 작은 값 순으로 배치하여 모두 더한다.

```text
VIII = 5 + 1 + 1 + 1 = 8
```

단, 다음 6개 조합은 앞의 작은 값을 뒤의 큰 값에서 빼는 형태로 사용한다.

```text
IV = 4
IX = 9
XL = 40
XC = 90
CD = 400
CM = 900
```

예:

```text
MCMXCIV
= M + CM + XC + IV
= 1000 + 900 + 90 + 4
= 1994
```

### LeetCode

https://leetcode.com/problems/roman-to-integer/

---

## 2. 내가 진행한 코드 및 부족한 부분

### 처음 작성한 코드

문제에서 제시한 6개의 뺄셈 조합을 직접 검사하는 방식으로 구현했다.

```c
int romanToInt(char *s)
{
    int str_ind = 0;
    int number = 0;
    char roman = s[str_ind];

    while (roman)
    {
        if (roman == 'I')
        {
            if (s[str_ind + 1] == 'V')
            {
                number += 4;
                str_ind += 2;
                roman = s[str_ind];
                continue;
            }

            if (s[str_ind + 1] == 'X')
            {
                number += 9;
                str_ind += 2;
                roman = s[str_ind];
                continue;
            }

            number += 1;
        }
        else if (roman == 'V')
            number += 5;

        else if (roman == 'X')
        {
            if (s[str_ind + 1] == 'L')
            {
                number += 40;
                str_ind += 2;
                roman = s[str_ind];
                continue;
            }

            if (s[str_ind + 1] == 'C')
            {
                number += 90;
                str_ind += 2;
                roman = s[str_ind];
                continue;
            }

            number += 10;
        }
        else if (roman == 'L')
            number += 50;

        else if (roman == 'C')
        {
            if (s[str_ind + 1] == 'D')
            {
                number += 400;
                str_ind += 2;
                roman = s[str_ind];
                continue;
            }

            if (s[str_ind + 1] == 'M')
            {
                number += 900;
                str_ind += 2;
                roman = s[str_ind];
                continue;
            }

            number += 100;
        }
        else if (roman == 'D')
            number += 500;

        else if (roman == 'M')
            number += 1000;

        roman = s[++str_ind];
    }

    return number;
}
```

### 접근 방법

문제에 명시된 예외 조합을 그대로 코드로 옮겼다.

```text
IV → +4
IX → +9
XL → +40
XC → +90
CD → +400
CM → +900
```

이 방식도 정상적인 풀이이고 문자열을 한 번 순회하므로 시간 복잡도는 `O(n)`이다.

### 부족한 부분

성능 자체가 큰 문제는 아니지만 코드 구조를 더 단순화할 수 있다.

- 6개의 예외를 각각 조건문으로 처리한다.
- `I`, `X`, `C`에서 비슷한 조건문이 반복된다.
- 두 글자 조합을 처리하기 위해 `str_ind += 2`, `continue` 등 인덱스를 직접 관리한다.
- 6개의 예외가 가지고 있는 공통 규칙을 일반화하지 못했다.

즉, **개별 예외를 하나씩 처리하는 방식**이다.

---

### 이후 직접 작성한 코드

공통 규칙을 이해한 뒤 뒤에서 앞으로 순회하는 방식으로 다시 작성했다.

```c
int value(char s)
{
    if (s == 'I') {
        return 1;
    } else if (s == 'V') {
        return 5;
    } else if (s == 'X') {
        return 10;
    } else if (s == 'L') {
        return 50;
    } else if (s == 'C') {
        return 100;
    } else if (s == 'D') {
        return 500;
    } else {
        return 1000;
    }
}

int romanToInt(char* s)
{
    int sum = 0;
    int n = strlen(s);

    for (int i = n - 1; i >= 0; i = i - 1)
    {
        if (i == (n - 1))
        {
            sum = value(s[i]);
        }
        else
        {
            if (value(s[i]) < value(s[i + 1]))
                sum = sum - value(s[i]);
            else
                sum = sum + value(s[i]);
        }
    }

    return sum;
}
```

이 풀이 역시 정상적으로 동작하며 시간 복잡도는 `O(n)`이다.

다만 `value()`의 마지막 `else`가 모든 나머지 문자를 `1000`으로 처리한다.

```c
else {
    return 1000;
}
```

문제에서는 입력 문자가 항상 유효하다고 보장되므로 통과할 수 있지만, 함수 자체만 보면 `'M'`을 명시하는 편이 더 정확하다.

---

## 3. 개선된 코드

### 핵심 아이디어

유효한 로마 숫자에서는 다음 공통 규칙을 사용할 수 있다.

```text
현재 값 < 다음 값
→ 현재 값을 뺀다.

현재 값 >= 다음 값
→ 현재 값을 더한다.
```

예를 들어:

```text
IV
= -1 + 5
= 4

IX
= -1 + 10
= 9

CM
= -100 + 1000
= 900
```

따라서 `IV`, `IX`, `XL`, `XC`, `CD`, `CM`을 각각 검사하지 않아도 된다.

### 개선 코드

```c
int value(char c)
{
    switch (c)
    {
        case 'I': return 1;
        case 'V': return 5;
        case 'X': return 10;
        case 'L': return 50;
        case 'C': return 100;
        case 'D': return 500;
        case 'M': return 1000;
        default:  return 0;
    }
}

int romanToInt(char *s)
{
    int result = 0;

    for (int i = 0; s[i] != '\0'; i++)
    {
        int current = value(s[i]);
        int next = value(s[i + 1]);

        if (current < next)
            result -= current;
        else
            result += current;
    }

    return result;
}
```

### 기존 코드와 차이

- 기존: 6개의 뺄셈 조합을 각각 조건문으로 처리
- 개선: 현재 값과 다음 값의 대소 관계라는 공통 규칙으로 처리
- 개선 효과: 조건문과 인덱스 관리가 줄어 코드의 구조가 단순해짐
- `value()` 결과를 `current`, `next`에 저장하여 같은 문자의 값을 반복 계산하지 않음

### 복잡도

- 시간 복잡도: `O(n)`
- 공간 복잡도: `O(1)`

문자열을 처음부터 끝까지 한 번 순회하고, 입력 크기에 비례하는 별도 자료구조를 사용하지 않는다.

---

## 4. 질문 사항과 답

### Q1. `current < next`이면 현재 값을 빼는 방식이 어떻게 가능한가?

**A.**

로마 숫자의 뺄셈 표현을 다음처럼 바꿔 생각할 수 있다.

```text
IV = 5 - 1
```

덧셈의 형태로 쓰면:

```text
-1 + 5 = 4
```

이므로 왼쪽부터 읽으면서 `I`의 다음 값이 더 크다는 것을 확인하면 현재 값 `1`을 빼고, 다음 `V`에서 `5`를 더하면 된다.

```text
IV → -1 + 5 = 4
IX → -1 + 10 = 9
XL → -10 + 50 = 40
XC → -10 + 100 = 90
CD → -100 + 500 = 400
CM → -100 + 1000 = 900
```

따라서:

```c
if (current < next)
    result -= current;
else
    result += current;
```

라는 하나의 규칙으로 처리할 수 있다.

---

### Q2. 그러면 `IM`처럼 `I` 뒤에 `M`이 있으면 빼면 안 되는 것 아닌가?

**A.**

맞다.

실제 로마 숫자 규칙에서는 단순히 작은 숫자 뒤에 큰 숫자가 있다고 해서 항상 뺄 수 있는 것이 아니다.

허용되는 뺄셈 조합은 다음 6개뿐이다.

```text
IV
IX
XL
XC
CD
CM
```

따라서 `IM`, `IL`, `XD`, `XM` 등은 올바른 로마 숫자 표기가 아니다.

단순 비교 코드에 `"IM"`을 넣으면:

```text
-1 + 1000 = 999
```

로 계산되지만, `IM` 자체가 유효한 입력이 아니다.

LeetCode 문제에서는 입력 `s`가 항상 유효한 로마 숫자라고 보장하기 때문에:

```c
if (current < next)
```

만 검사해도 된다.

즉:

```text
실제 로마 숫자 규칙
→ current < next라고 항상 뺄 수 있는 것은 아님

LeetCode의 이 문제
→ 입력이 이미 유효하다고 보장됨
→ current < next만 확인해도 충분함
```

---

### Q3. 뒤에서 앞으로 푼 코드와 앞에서 뒤로 푼 코드에서 왜 실행시간 차이가 발생하는가?

**A.**

두 코드의 **이론적인 시간 복잡도는 모두 `O(n)`**이다.

첫 번째 코드는:

```c
int n = strlen(s);
```

를 사용하므로 먼저 문자열 길이를 구하기 위해 한 번 순회한다.

```text
strlen(s) → O(n)
for loop  → O(n)

전체 → O(n) + O(n) = O(n)
```

두 번째 코드는:

```c
for (int i = 0; s[i] != '\0'; i++)
```

로 문자열 끝을 확인하면서 바로 한 번 순회한다.

```text
for loop → O(n)
```

따라서 실제 수행하는 연산 수에는 약간의 차이가 있을 수 있지만, 둘 다 `O(n)`이다.

또한 두 코드 모두 다음과 같이 `value()`를 반복 호출한다.

```c
if (value(s[i]) < value(s[i + 1]))
    result -= value(s[i]);
```

일반적인 반복 한 번에서 `value()`가 약 3번 호출된다.

하지만 LeetCode의 이 문제는 문자열 길이가 최대 15이므로 입력 자체가 매우 작다.

따라서 LeetCode에서 보이는:

```text
0 ms
1 ms
2 ms
```

정도의 차이만으로 한 풀이가 다른 풀이보다 확실하게 빠르다고 판단하기 어렵다.

실행시간에는 알고리즘 외에도 실행 환경의 영향이 존재하기 때문이다.

결론:

```text
첫 번째 풀이 → O(n)
두 번째 풀이 → O(n)

실제 수행 연산에는 조금 차이가 있음
하지만 이 문제의 입력이 매우 작음
→ LeetCode의 몇 ms 차이는 큰 의미가 없음
```

성능 차이보다는 코드 구조와 중복 연산을 줄이는 관점에서 개선하는 것이 더 적절하다.

---

# 부록

## A. `value()` 반복 호출 줄이기

다음 코드는 같은 값 변환을 한 반복 안에서 여러 번 수행한다.

```c
if (value(s[i]) < value(s[i + 1]))
    result -= value(s[i]);
else
    result += value(s[i]);
```

다음처럼 한 번 계산한 값을 변수에 저장하면 코드가 더 명확하다.

```c
int current = value(s[i]);
int next = value(s[i + 1]);

if (current < next)
    result -= current;
else
    result += current;
```

컴파일러 최적화에 따라 실제 성능 차이는 작거나 없어질 수 있지만, 코드 자체에서는 중복 계산 의도를 제거할 수 있다.

---

## B. `if-else`와 `switch` 자체가 실행시간 차이의 핵심은 아니다

두 코드의 `value()` 구현은 각각 `if-else`와 `switch`를 사용한다.

```c
if (s == 'I')
    ...
else if (s == 'V')
    ...
```

```c
switch (c)
{
    case 'I':
    case 'V':
    ...
}
```

하지만 `switch`가 항상 `if-else`보다 빠르다고 단정할 수는 없다.

컴파일러는 조건의 형태와 최적화 옵션에 따라 비교문, 분기 구조 또는 다른 형태로 변환할 수 있다.

이 문제에서는 입력이 최대 15자이므로 이 차이를 성능의 핵심으로 볼 필요는 없다.

---

# 수정 이력

| 날짜 | 상태 | 변경 내용 |
|---|---|---|
| 2026-08-07 | SOLVED | 최초 풀이, 개선 코드 및 로마 숫자 비교 규칙 질문 정리 |
| 2026-08-07 | SOLVED | 뒤→앞 풀이 추가 및 두 구현의 실행시간 차이 Q&A 추가 |
