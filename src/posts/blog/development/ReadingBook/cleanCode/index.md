---
title: CleanCode(클린코드) 책 읽고 느낀점, 배운점 
date: "2022-03-15"
desc: "CleanCode 관련 책을 읽으면서 어떤것들을 배웠는지 적었습니다."
alt: "Clean Code book"
category: "ReadingBook"
tags: [ "2022" ,"Book", "Clean Code", "Programming" ]
thumbnail: ./images/codeClean.jpg
---

`※ 초반부분만 보고 느낀점, 배운점 등을 적었다. 업데이트 예정`

# 배운점

요점은 중복 줄이기, 표현력 높이기, 초반부터 간단한 추상화 고려하기 등이다.

## 변수, 함수 등 이름을 명확하게

책에서는 변수이름을 막 지은 특정 코드를 예시를 들며 설명하였다. 실제로 막 지은 코드와 이름만 바꾼 코드를 보니 왜 이름을 의미있고, 명확하게 써야 하는지 알 것 같았다.

```typescript
// 작성 언어: typescript
// 안좋은 예
function getThem() {
    let list: Array<number[]> = [];

    arrs.forEach((arr: number[]) => {
        if (arr[0] === 4) {
            list.push(arr);
        }
    });

    return list;
}

```

책에 나온 예시를 비슷하게나마 typescript로 짜봤다. 변수도 별로 없고, 하는 기능도 별로 없다. 하지만 이해하기 어려웠다. 여기서 드는 의문은

1. arrs엔 어떤것이 들어있는가?
2. arrs에서 0번째 값이 왜 중요한가?
3. 값 4는 어떤 의미인가?
4. 함수가 반환하는 `list` 변수를 어떻게 사용하는가?

등의 의문이 든다. 위 코드 샘플엔 해당 코드에 대한 설명을 남길수 있었음에도 남기지 않았다. 
책에서는 이 코드를 지뢰찾기 게임을 예로 다음 코드를 보여주었다. 그럼 arrs는 게임판을 가리키고, 배열에서 0번째
값은 칸 상태를 뜻하고, 값 4는 깃발이 꽂힌 상태를 가리킨다. 각 개념에 이름만 붙여도 코드가 완전 나아진다.

```typescript
// 작성 언어: typescript
// 좋은 예시
const STATUS_VALUE:number = 0;
const FLAGGED = 4;

function getFlaggedCells() {
    let flaggedCells: Array<number[]> = [];

    gameBoard.forEach((cell: number[]) => {
        if (cell[STATUS_VALUE] === FLAGGED) {
            flaggedCells.push(arr);
        }
    });

    return flaggedCells;
}

```
나는 위에 말을 볼 땐 잘 이해하지 못했는데 좋은 예시의 코드를 보고는 바로 이해가 되었었다.
안좋은 예시의 코드를 봤듯이 프로세스는 바뀌지 않고 단지 이름만 바뀌었음에도 보기가 되게 쉬어졌음을 느꼈다.
이걸 보면서 이름짓기의 중요성을 깨닫게 되었다.

### 그릇된 정보를 피하라

이 부분을 보면서 가장 기억났더 부분이 소문자 L 이나 대문자 O 를 사용하 변수였다. 아래 코드에서 보듯 문자 L은 숫자 1처럼 보이고,
대문자 O는 숫자 0처럼 보여 되게 햇갈렸다.

```typescript
//작성 언어: typescript
let O:number = 0;
let l:number = 1;
let Ol:number = O + l;

let a:number = l;

if(O === l) {
    a = Ol;
} else {
    l = 01;
}

3
```
이와 같은 변수를 도처에서 사용하는 코드를 실제로 보았다고 책은 말해주었다. 어떤 경우는 코드작성자가 글꼴을 바꿔 차이를 드러내는
해결책을 제안했다는데.. 봐도봐도 너무 햇갈린다. 다시 한번 이름 짓기의 중요성을 깨닫게 되었다.

### 발음하기 쉬운 이름을 사용해라

코드리뷰를 할 때 자신이 짠 코드를 설명할 것이다. 여기서 어떠한 변수, 함수 등에 대한 이름을 말할 때 `genymdhms` 란 변수 이름을 말한다면
"젠 와이 엠 디 에이취 엠 에스"라고 또는 "젠 야 무다 힘즈" 등으로 어렵게 발음할 것이다. 벌써부터 말 할 생각에 어지럽다.
다음 코드를 예제로 보면
```typescript

//1번
interface DtaRcrd102 {
    genymdhms:Date;
    modymdhms:Date;
    pszqint?:int;
}

//2번
interface Customer {
    generationTimestamp:Date;
    modificationTimestamp:Date;
    recordId:number;
}
```
두번째 코드는 지적인 대화가 가능해진다. 몇번 코드가 좋아 보이는가?

### 검색하기 쉬운 이름을 사용해라
