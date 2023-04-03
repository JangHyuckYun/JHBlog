---
title: Puppeteer - 사용하면서 어려웠던 점
date: "2023-03-22"
desc: "Puppeteer - 사용하면서 어려웠던 점"
alt: "Puppeteer - 사용하면서 어려웠던 점"
category: "Puppeteer"
tags:
  ["2023", "Typescript", "JavaScript", "Toy Project", "Crawling", "Puppeteer"]
thumbnail: ../images/puppeteer/logo.png
---

nodeJS 환경에서 크롤링을 해보고 싶었는데, 마침 **selenium**보다 빠르다고 알려져 있는 **Puppeteer**를 사용 해보았다.  
java에서 **Selenium**을 사용하던 것과는 짜는 느낌이 좀 색달랐다.  
하지만 나 혼자만 그런것인지, 생각보다 짜면서 생겨나는 어려움이 있었다.

## 목차

1. 동적 데이터 가져오기 (작성 완료)
2. 추후 작성 예정..

## 1. 동적 데이터 가져오기

> 첫 로딩 시 또는 특정 네트워크 개수에는 waitForNavigation()함수를 활용해 동적 데이터를 대기할 수 있지만,  
> 특정 상황에서는 해당 함수만으로는 안 될 경우가 존재했다.

예를 들어, 영화관 사이트의 크롤링을 진행하는 경우, 대부분의 데이터는 동적으로 불러온다.  
puppeteer에서 동적 데이터를 가져올 때 까지 기다리고, 가져오는 방법에는 여러가지가 있다.

- 페이지 자체에서 만들어진 로딩 태그가 나타나고 사라지는걸 감지해서, 로딩 태그가 나타나고 사라질 때 가져오도록 수행 하거나
- 또는 네트워크 자체를 감지해서, 네트워크를 불러올 경우, 데이터를 가져오도록 하거나
- 특정 태그의 값이 바뀌거나, 추가 되는것을 감지해서 가져오는 등 다양한 방법이 존재한다.

### 페이지 자체에서 만들어진 로딩 태그감지

이것을 감지하기 위해 나는 로딩 태그를 감지하는 커스텀 CrawlUtil 클래스를 만들고, 해당 클래스 안에 함수를 만들어 활용했다.

```typescript
export class CrawlUtil {
  ...
  static async waitForLoading(page: Page, selector: string): Promise<void> {
    await page.waitForSelector(selector, { timeout: this.DEFAULT_TIMEOUT });
    await page.waitForFunction(
      () => {
        return !document.querySelector(selector);
      },
      { timeout: this.DEFAULT_TIMEOUT }
    );
  }
  ...
}
```

puppeteer의 page와, 기다릴 로딩 태그의 selector을 인자로 넘겨준다.  
처음에, **waitForSelector()** 함수를 통해 해당 **selector**가 나타날 때 까지 대기한다.  
( 위의 코드는 해당 **selector**가 없을 시에 대한 에러 로직이 제외되어 있음.)  
그리고, **waitForFunction**함수를 통해서 해당 페이지에서, **selector**가 사라지는지 확인하는 코드를 짜준다.

#### waitForFunction(Function: boolean, options: [FrameWaitForFunctionOptions](https://pptr.dev/api/puppeteer.framewaitforfunctionoptions))

- 인자값은 함수를 받고, 해당 함수의 return값은 boolean이여야 한다.
- true일 경우 탈출하고, false일 경우, 해당 함수를 반복적으로 실행한다.
- option 인자의, timeout을 걸어줌으로써, 특정 시간까지 반복할 수 있다.

### 네트워크 통신 감지

네트워크 통신을 감지하는 방법은, 내가 알기로는 약 3가지가 있다.

#### waitForNavigation(options: [WaitForOption](https://pptr.dev/api/puppeteer.waitforoptions))

위에서 언급한 **waitForNavigation**을 사용할 수 있다.

```typescript
...
await page.waitForNavigation({ waitUntil: 'networkidle0 or networkidle2' });
...
```

- **networkidle0:** 0개 이상의 네트워크가 감지되는 경우
- **networkidle2:** 2개 이상의 네트워크가 감지되는 경우
- 위와 같이 **networkidle**값을 사용해서 네트워크 요청이 왔는지 감시할 수 있다.

#### waitForNetworkIdle(options: {idleTime?: number, timeout?: number})

- 위와 비슷한 방법으로, **networkidle**의 개수를 세세하게 지정할 수 있는걸로 알고 있다.
- 그리고 **timeout** 값을 통해 최대 특정 시간까지 대기할 수 있다.

#### waitForResponse() or waitForRequest()
...

### 특정 태그의 값이 바뀌는것 감지

이를 감지하는 방법에는 크게 2가지가 있다. (내가 찾아낸 것은 2가지다.)

#### waitForSelector() 활용

#### evaluate() + Observer + waitForFunction() 활용

이것도 편하게 사용하기 위해 커스텀으로 만들어서 사용했다.

**1. page.evaluate() 안에 window 객체로 observer 선언**

```typescript
await page.evaluate((windowObjName, selector) => {
  window[windowObjName] = null;

  const target = document.querySelector(selector);
  const observer = new MutationObserver((mutations) => {
    for (const mutation of mutations) {
      if (mutation.type === "childList" && conditionalFunctios()) {
        window[windowObjName] = true;
        observer.disconnect();
      }
    }
  });

  observer.observe(target, { childList: true });
}, ...datas);
```

**MutationObserver**을 선언해서, 특정 객체의 값이 바뀌는지 실시간으로 확인할 수 있다.  
임시로, 특정 객체의 값이 바뀌고 + 내가 정의한 조건이 맞을 경우에만 window객체의 특정 값이 **"true"** 로 바뀌게 해 두었다.

**2. waitForFunction()을 사용해, 해당 아까 선언한 window객체의 값이 true인지 체크**

```typescript
await page.waitForFunction((waitVar: string) => window[waitVar], {}, waitVar);
```

**waitForFunction()** 을 사용해서, 아까 **observer**선언한 곳에서, 값이 **"true"** 로 바뀔 때 까지 대기한다.  
( **options**에 **timeout**을 설정하지 않을 경우, 상위에서 설정한 **timeout** 값 또는 특정 값 동안 기다리게 된다. )  

> 2번의 방법의 경우, 번거롭다는 느낌이 들어서 도저히 대기를 못 할 경우에만 사용했다.
  
  
> **※ 추 후 계속 작성할 예정이다.**
