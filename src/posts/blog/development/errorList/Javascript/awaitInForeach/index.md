---
title: Javascript - forEach에서 비동기 작업
date: "2023-03-20"
desc: "Javascript에서 foreach문 안에서 비동기 작업에 대해"
alt: "Javascript - forEach에서 비동기 작업"
category: "JavaScript"
tags: [ "2023" ,"Typescript", "JavaScript", "Toy Project", "Crawling" ]
thumbnail: ../images/awaitInForeach/logo.png
---


## 원인

Crawling 관련 프로젝트를 진행하면서 비동기 작업 처리도중 forEach문에서는 await을 기다려주지 않는 현상이 발생해서 문제가 있었다.
원인은 **Array.prototype.forEach**는 동기 작업을 초점으로 만들어져 있어, 비동기 작업이 지원되지 않기 때문에 forEach 안에 async
함수를 사용한 비동기 작업을 처리하려 해도, 기다려주지 않는 것이였다.

## 해결법

약 두 가지 방법을 찾았다.

### 1. for ... of 문 사용
현재 내가 크롤링 작업에 사용하는 방법이다.
```javascript
...
async function test() {
    const test_array = [1,2,3];
    for (const data of test_array) {
		// 비동기 작업 ...
        await ...;	
    }
}

test();
...
```
위와 같이 비동기 함수 안에서 **for ... of**문을 활용해 비동기 작업을 수행할 수 있다. ( 잘 기다려준다. )    
or  
```javascript
...
async function test() {
    const test_array = [1,2,3];
    for await (const data of test_array) {
	    // 비동기 작업 ...
        new Promise()
    }
}

test();
...
```

**for (const data of array) { ... }** 와 **for await(const data of array) { ... }** 의 차이가 궁금해서 찾아 봤더니,  
**for ... of** 문 내부의 **await**을 포함 하느냐 안 하느냐의 차이가 있었다.  
일반 **for ... of**의 경우, 내부적으로(for 문 안에서) 각 promise를 반환하는 변수 또는 함수 등에 **await**을 달아주어, 기다려야 한다.    
하지만, **for await ... of**의 경우, 내부적으로(for 문 안에서) 각 객체(변수, 함수 등)마다 **await**을 붙여주는 효과가 있어서,  
개발자가 **await**을 붙여주지 않아도, 자동적으로 붙여주는 차이가 있었다.

### 2. Array.prototype.reduce 사용
이건 따로 구글링을 찾아보며 알게 되었다. reduce문은 일반적으로
```javascript
 array.reduce((acc, data) => { return acc }, {});
```
와 같은 형태로 이루어져 있다. 여기서 사용되는 함수 옆에 **async**를 붙여줌으로써,
```javascript
 array.reduce(async (acc, data) => { 
    return acc;
 }, {});
```
비동기 작업을 기다려주게 된다. **forEach**함수와 다르게, **reduce**함수는 비동기 작업에 대응이 가능하게 만들어져 있는 것 같다.


