# 14. Longest Common Prefix

> 상태: SOLVED  
> 플랫폼: LeetCode  
> 언어: C  
> 문제 링크: https://leetcode.com/problems/longest-common-prefix/description/  
> 최초 작성일: 2026-08-08  
> 마지막 수정일: 2026-08-08

---

## 1. 간단 문제 설명

여러 문자열이 주어졌을 때, 모든 문자열이 공통으로 가지고 있는 가장 긴 접두사(Longest Common Prefix)를 찾는 문제다.

- 입력: 문자열 배열 `strs`
- 출력: 모든 문자열의 가장 긴 공통 접두사
- 핵심 조건:
  - 첫 번째 문자열을 기준 문자열로 사용할 수 있다.
  - 같은 위치의 문자를 다른 모든 문자열과 비교한다.
  - 하나라도 다른 문자가 나오면 그 직전까지가 공통 접두사다.
- 주요 예외 규칙:
  - 첫 문자부터 다르면 빈 문자열 `""`을 반환한다.
  - C 문자열의 끝은 반드시 `'\0'`으로 표시해야 한다.

### LeetCode

https://leetcode.com/problems/longest-common-prefix/description/

---

## 2. 내가 진행한 코드 및 부족한 부분

### 내가 작성한 코드

```c
size_t str_len(const char* str) {

    size_t i = 0;

    while(str[i] != '\0'){

        ++i;

    }

    return i;

}


char* longestCommonPrefix(char** strs, int strsSize) {

    char* result = calloc(200, sizeof(char));

    if (result == NULL) return calloc(1, sizeof(char));

    result[0] = '\0';

    if (strsSize == 0) {

        return result;

    }

    const char* std_str = strs[0];

    size_t std_str_len = str_len(std_str);

    for (size_t std_str_ind = 0; std_str_ind <= std_str_len; ++std_str_ind){

        for(int ind = 1; ind < strsSize; ++ind){

            if (strs[ind][std_str_ind] != std_str[std_str_ind])

                return result;

        }

        result[std_str_ind] = std_str[std_str_ind];

    }

    return result;

    // first, allocate 200byte char for using output,
    // and check the letter,
    // first char in first str, and campare other str's first char
    // if all same save the letter in allocated output
    // and increase the index, and repeat,
    // but while compare the other str's char, is '\0'
    // end the function,
}
```

### 접근 방법

첫 번째 문자열을 기준 문자열로 정했다.

```text
strs[0][0] → 다른 모든 문자열의 0번째 문자와 비교
strs[0][1] → 다른 모든 문자열의 1번째 문자와 비교
strs[0][2] → 다른 모든 문자열의 2번째 문자와 비교
...
```

모든 문자열의 현재 문자가 같으면 해당 문자를 `result`에 저장한다.

하나라도 다른 문자가 발견되면 지금까지 저장한 `result`가 가장 긴 공통 접두사이므로 즉시 반환한다.

또한 `strlen()`을 사용하지 않고 `str_len()`을 직접 구현하여 C 문자열이 `'\0'`을 기준으로 끝난다는 점을 직접 확인했다.

### 부족한 부분

#### 1. 결과 버퍼 크기를 `200`으로 고정했다

```c
char* result = calloc(200, sizeof(char));
```

문자열에는 실제 문자뿐 아니라 마지막의 `'\0'`을 저장할 공간도 필요하다.

즉 길이가 `N`인 문자열을 저장하려면 최소한 다음 크기가 필요하다.

```text
N개의 문자 + '\0'
= N + 1 byte
```

고정 크기 `200`을 사용하는 것보다 기준 문자열의 길이를 구한 뒤 다음처럼 필요한 만큼 할당하는 것이 안전하고 명확하다.

```c
char* result = malloc(std_str_len + 1);
```

#### 2. `<= std_str_len`으로 `'\0'`까지 반복문에서 처리한다

기존 코드:

```c
for (size_t std_str_ind = 0;
     std_str_ind <= std_str_len;
     ++std_str_ind)
```

이 구조에서는 일반 문자뿐 아니라 기준 문자열의 마지막 `'\0'`도 비교한다.

동작 자체는 가능하지만, 문자열 문자 비교와 문자열 종료 처리가 한 반복문 안에 섞여 있어 의도가 조금 덜 명확하다.

다음처럼 실제 문자만 비교하는 편이 이해하기 쉽다.

```c
i < std_str_len
```

그리고 반복이 끝난 뒤 직접:

```c
result[i] = '\0';
```

을 넣는다.

#### 3. `calloc()`의 0 초기화에 의존하는 부분이 있다

문자 불일치를 발견하면 바로:

```c
return result;
```

한다.

그런데 불일치 위치에 직접 `'\0'`을 쓰지 않아도 현재 코드가 문자열로 정상 동작하는 이유는 `calloc()`이 메모리를 처음부터 모두 0으로 초기화했기 때문이다.

즉 다음 상태가 이미 만들어져 있다.

```text
result:
f l o w \0 \0 \0 ...
```

따라서 `return result`가 가능하다.

틀린 코드는 아니지만, 개선 코드에서는 다음처럼 종료 문자를 직접 기록하는 편이 의도가 명확하다.

```c
result[i] = '\0';
return result;
```

#### 4. 메모리 할당 실패 처리

기존 코드:

```c
if (result == NULL)
    return calloc(1, sizeof(char));
```

첫 번째 메모리 할당이 실패했는데 다시 `calloc()`을 호출하는 방식은 적절하지 않다.

두 번째 할당도 실패할 수 있으며, 메모리 할당 실패와 정상적인 빈 문자열 결과를 구분하기 어려워진다.

다음처럼 처리하는 것이 더 명확하다.

```c
if (result == NULL)
    return NULL;
```

#### 5. `200`이라는 Magic Number

현재 코드에서는:

```c
calloc(200, sizeof(char));
```

처럼 문제 조건에 의존하는 숫자가 코드 안에 직접 들어가 있다.

문제 조건이 변경되거나 다른 코드에서 함수를 재사용하면 문제가 될 수 있다.

기준 문자열 길이를 이용하면 이런 고정값 자체가 필요하지 않다.

### 잘한 부분

- 첫 번째 문자열을 기준으로 세로 방향으로 비교하는 핵심 알고리즘을 정확히 잡았다.
- 하나라도 다른 문자를 발견하면 즉시 반환하는 구조가 적절하다.
- `str_len()`을 직접 구현하면서 `'\0'` 기반 C 문자열 구조를 확인했다.
- 문자열 길이를 매번 다시 계산하지 않고 한 번만 계산했다.
- 입력 문자열 자체를 수정하지 않고 별도의 결과 문자열을 만들어 반환했다.

---

## 3. 개선된 코드

### 핵심 아이디어

기존 알고리즘은 그대로 유지한다.

```text
1. 첫 번째 문자열을 기준 문자열로 정한다.
2. 기준 문자열 길이 + 1만큼 결과 메모리를 할당한다.
3. 기준 문자열의 각 문자를 다른 모든 문자열의 같은 위치 문자와 비교한다.
4. 하나라도 다르면 현재 위치에 '\0'을 넣고 반환한다.
5. 모두 같으면 해당 문자를 result에 복사한다.
6. 끝까지 통과하면 마지막에 '\0'을 넣고 반환한다.
```

### 개선 코드

```c
#include <stdlib.h>

size_t str_len(const char* str)
{
    size_t i = 0;

    while (str[i] != '\0') {
        ++i;
    }

    return i;
}

char* longestCommonPrefix(char** strs, int strsSize)
{
    if (strsSize == 0) {
        return calloc(1, sizeof(char));
    }

    const char* std_str = strs[0];
    size_t std_str_len = str_len(std_str);

    char* result = malloc((std_str_len + 1) * sizeof(char));

    if (result == NULL) {
        return NULL;
    }

    size_t i;

    for (i = 0; i < std_str_len; ++i) {

        for (int j = 1; j < strsSize; ++j) {

            if (strs[j][i] != std_str[i]) {
                result[i] = '\0';
                return result;
            }
        }

        result[i] = std_str[i];
    }

    result[i] = '\0';

    return result;
}
```

### 기존 코드와 차이

- 기존:
  - 결과 버퍼를 `200 byte`로 고정 할당
  - `calloc()`의 0 초기화에 일부 의존
  - `<= std_str_len`으로 `'\0'`까지 반복문에서 비교
  - 첫 번째 `calloc()` 실패 시 다시 `calloc()` 호출

- 개선:
  - `std_str_len + 1`만큼 필요한 크기만 할당
  - 문자열 종료 위치에 `'\0'`을 명시적으로 기록
  - 실제 문자만 `i < std_str_len` 범위에서 비교
  - 메모리 할당 실패 시 `NULL` 반환

- 개선 효과:
  - 버퍼 크기 관리가 명확해진다.
  - C 문자열 종료 처리 의도가 분명해진다.
  - `calloc()`의 초기화 특성에 덜 의존한다.
  - 기존의 단순한 알고리즘은 그대로 유지한다.

### 복잡도

문자열 개수를 `N`, 첫 번째 문자열의 길이를 `L`이라고 하면:

- 시간 복잡도: `O(N × L)`
- 추가 공간 복잡도:
  - 결과 문자열 포함: `O(L)`
  - 결과 문자열을 제외한 보조 공간: `O(1)`

최악의 경우 첫 번째 문자열의 모든 문자에 대해 나머지 모든 문자열을 비교한다.

---

## 4. 질문 사항과 답

### Q1. 내가 작성한 풀이에서 부족한 점은 무엇인가?

**A.**

핵심 알고리즘에는 큰 문제가 없다.

가장 중요한 수정점은 알고리즘보다 C의 문자열 및 메모리 처리다.

특히 다음 부분을 수정할 필요가 있다.

```text
1. result를 200 byte로 고정 할당한 점
2. 문자열 끝의 '\0'을 위한 공간을 명확히 확보해야 하는 점
3. calloc의 0 초기화에 암묵적으로 의존한 점
4. <= std_str_len으로 종료 문자까지 비교하는 구조
5. 메모리 할당 실패 후 다시 calloc을 호출하는 처리
```

따라서 다른 알고리즘으로 바꾸기보다는 현재 풀이를 유지하면서 메모리와 문자열 종료 처리를 명확하게 만드는 방향이 적절하다.

---

### Q2. 다른 사람들의 코드와 비교하면 어떤 차이가 있는가?

**A.**

세 코드 모두 핵심 알고리즘은 비슷하다.

모두 첫 번째 문자열의 각 위치를 기준으로 다른 문자열의 같은 위치 문자를 비교한다.

#### 다른 코드 1

```c
int i = 0;
int cntPre = 0;
int flag = 0;

while(strs[0][i] != '\0'){
    for(int j = 1; j < strsSize; j++){
        if(strs[0][i] != strs[j][i]) flag = 1;
    }

    if(flag) break;
    cntPre++;
    i++;
}

if(cntPre == -1) return "";

char* result = malloc((cntPre + 1) * sizeof(char));

for(int i = 0; i < cntPre; i++){
    result[i] = strs[0][i];
}

result[cntPre] = '\0';

return result;
```

좋은 점:

```c
malloc((cntPre + 1) * sizeof(char));
```

처럼 실제 공통 접두사의 길이를 알아낸 뒤 정확한 크기만 할당한다.

아쉬운 점:

```c
if (cntPre == -1)
```

은 `cntPre`가 0에서 시작하고 증가만 하므로 실제로 참이 될 수 없다.

또한 불일치를 발견한 뒤에도 내부 반복문을 즉시 `break`하지 않아서 이미 결과가 결정된 이후의 비교를 계속할 수 있다.

---

#### 다른 코드 2

```c
char* longestCommonPrefix(char** strs, int strsSize) {
    if (strsSize == 0) return "";

    for (int i = 0; i < strlen(strs[0]); i++) {

        char c = strs[0][i];

        for (int j = 1; j < strsSize; j++) {

            if (i == strlen(strs[j]) || strs[j][i] != c) {
                strs[0][i] = '\0';
                return strs[0];
            }
        }
    }

    return strs[0];
}
```

좋은 점:

별도의 결과 문자열을 만들지 않고 첫 번째 문자열을 직접 잘라 결과로 사용하므로 추가 메모리 할당이 필요하지 않는다.

```c
strs[0][i] = '\0';
```

을 사용하면:

```text
"flower"

↓

f l \0 w e r

↓

"fl"
```

처럼 문자열을 원하는 위치에서 끝낼 수 있다.

아쉬운 점:

입력 문자열인 `strs[0]` 자체를 수정한다.

따라서 함수 실행 전의 첫 번째 문자열을 그대로 보존해야 하는 상황에는 적합하지 않을 수 있다.

또한 반복문 안에서 `strlen()`을 여러 번 호출한다.

`strlen()`도 문자열의 시작부터 `'\0'`까지 확인하여 길이를 계산하므로 같은 문자열 길이를 반복 계산하게 된다.

이 부분에서는 길이를 한 번만 계산한 내 코드의 구조가 더 명확하다.

---

### 세 코드 비교

| 항목 | 내 코드 | 다른 코드 1 | 다른 코드 2 |
|---|---|---|---|
| 기본 알고리즘 | 첫 문자열 기준 문자 비교 | 동일 | 동일 |
| 입력 문자열 수정 | 하지 않음 | 하지 않음 | 수정함 |
| 결과용 메모리 | 별도 할당 | 별도 할당 | 추가 할당 없음 |
| 길이 반복 계산 | 없음 | 없음 | `strlen()` 반복 |
| 종료 문자 처리 | `calloc`에 일부 의존 | 명시적 | 원본 문자열에 직접 삽입 |
| 주요 개선점 | 메모리 크기/종료 처리 | 불필요 조건/조기 종료 | 입력 변경/반복 `strlen()` |

결론적으로 다른 사람의 코드를 그대로 따라갈 필요는 없다.

현재 풀이의 핵심 알고리즘은 적절하므로 C 문자열과 동적 메모리 처리를 보완하는 것이 더 좋은 학습 방향이다.

---


### Q3. 현재 풀이보다 더 좋은 알고리즘이 존재하는가?

**A.**

현재 작성한 방식은 첫 번째 문자열의 같은 위치 문자를 다른 모든 문자열과 비교하는 방식으로, 일반적으로 **Vertical Scanning**이라고 볼 수 있다.

이 방식의 최악 시간 복잡도는 문자열 개수를 `N`, 비교하는 최대 문자열 길이를 `L`이라고 할 때 다음과 같다.

```text
O(N × L)
```

모든 문자열이 동일하거나 공통 접두사가 매우 긴 경우에는 실제로 각 문자열의 많은 문자를 확인해야 하므로, 현재 접근은 이 문제에서 충분히 좋은 방법이다.

현재 풀이를 완전히 다른 알고리즘으로 교체하기보다는 다음 개선을 우선 고려할 수 있다.

```text
첫 번째 문자열을 무조건 기준으로 사용
        ↓
가장 짧은 문자열 또는 가장 짧은 문자열 길이를 먼저 확인
        ↓
그 길이까지만 비교
```

공통 접두사는 가장 짧은 문자열보다 길 수 없기 때문이다.

다만 이 방법도 최악 시간 복잡도의 차수를 본질적으로 낮추는 것은 아니며, 불필요한 비교를 줄이는 구현상의 개선에 가깝다.

다른 대표적인 접근으로는 다음이 있다.

- Horizontal Scanning
- Divide and Conquer
- Binary Search
- Trie
- 문자열 정렬 후 첫 문자열과 마지막 문자열 비교

현재 단계에서는 위 방법들이 존재한다는 것까지만 확인했으며, 각각의 구현 원리와 장단점은 별도로 학습할 필요가 있다.


# 부록

## A. `'\0'`과 C 문자열

C의 문자열은 문자 배열이며 마지막에 Null Character인 `'\0'`이 있어야 한다.

예:

```c
char str[] = "abc";
```

실제 메모리는 다음과 같다.

```text
index    0    1    2    3
value   'a'  'b'  'c' '\0'
```

따라서 `"abc"`를 저장하려면 문자 3개가 아니라 총 4 byte가 필요하다.

```text
문자 3 byte
+
'\0' 1 byte
=
4 byte
```

이 문제에서 동적 메모리를 할당할 때도 같은 원칙을 적용한다.

```c
malloc(std_str_len + 1);
```

여기서 `+ 1`은 마지막 `'\0'`을 위한 공간이다.

---

## B. `calloc()`과 `malloc()`의 차이가 이번 코드에 미친 영향

기존 코드:

```c
char* result = calloc(200, sizeof(char));
```

`calloc()`은 확보한 메모리를 0으로 초기화한다.

따라서 처음에는 개념적으로 다음과 같다.

```text
result
[0][0][0][0][0]...
```

문자를 몇 개 기록하면:

```text
[f][l][o][0][0]...
```

가 되므로 직접 `'\0'`을 넣지 않아도 현재까지 기록된 부분 뒤에 0이 남아 있다.

반면:

```c
malloc(...)
```

은 메모리를 0으로 초기화한다고 보장하지 않는다.

따라서 `malloc()`을 사용할 때는 문자열 끝을 직접 다음처럼 만들어야 한다.

```c
result[i] = '\0';
```

이번 문제를 통해 단순히 `malloc`과 `calloc`의 문법 차이뿐 아니라, 초기화 여부가 문자열 처리 방식에도 영향을 줄 수 있다는 점을 확인했다.

---


## C. 추가 학습 필요 — Longest Common Prefix 알고리즘 비교

> 현재는 개념의 존재와 대략적인 특징만 확인한 상태다.  
> 아래 항목은 이번 문제를 SOLVED로 유지하면서 이후 별도로 공부할 내용으로 남긴다.

### 현재 이해한 부분

- 현재 풀이 방식은 Vertical Scanning 계열이다.
- 첫 번째 문자열을 기준으로 같은 위치의 문자를 모든 문자열과 비교한다.
- 최악의 경우 시간 복잡도는 `O(N × L)` 수준이다.
- 공통 접두사는 가장 짧은 문자열보다 길 수 없으므로, 가장 짧은 문자열의 길이를 비교 상한으로 사용할 수 있다.
- 이 개선은 최악 시간 복잡도를 다른 차수로 바꾸는 것보다는 불필요한 비교를 줄이는 최적화에 가깝다.

### 추가로 공부할 알고리즘

- [ ] Vertical Scanning을 정확히 정리하기
- [ ] Horizontal Scanning과 현재 풀이 비교
- [ ] Divide and Conquer 방식의 동작 원리
- [ ] Binary Search를 prefix 길이에 적용하는 방법
- [ ] Trie가 Prefix 검색에 적합한 이유
- [ ] 문자열을 정렬한 뒤 첫 문자열과 마지막 문자열만 비교할 수 있는 이유
- [ ] 각 방법의 시간/공간 복잡도 비교
- [ ] LeetCode 14 단일 문제에서는 어떤 방식이 가장 적절한지 다시 판단

### 이후 확인할 핵심 질문

```text
1. Vertical Scanning과 Horizontal Scanning은 실제 비교 횟수가 어떻게 다른가?
2. 가장 짧은 문자열을 먼저 찾는 비용까지 포함하면 실제 이점은 어느 정도인가?
3. Binary Search 방식은 왜 단순 Vertical Scanning보다 반드시 빠르지 않은가?
4. Trie는 한 번의 LCP 계산에는 왜 과할 수 있는가?
5. 입력 문자열을 정렬하는 방식은 어떤 부작용과 추가 비용이 있는가?
```

현재는 이 문제의 기본 풀이와 C 문자열/메모리 처리를 우선 학습한 상태이므로, 위 알고리즘 비교는 추후 학습 항목으로 보류한다.


# 수정 이력

> 문서를 수정할 때 기존 행은 삭제하거나 덮어쓰지 않고 새 행을 아래에 추가한다.

| 날짜 | 상태 | 변경 내용 |
|---|---|---|
| 2026-08-08 | SOLVED | 최초 풀이, 코드 검토, 다른 풀이 비교 및 개선 코드 정리 |
| 2026-08-08 | SOLVED | 알고리즘 대안 검토 및 Vertical/Horizontal Scanning 등 추가 학습 필요 항목 등록 |
