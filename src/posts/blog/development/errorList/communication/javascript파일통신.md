---
title: "Javascript Fetch 사용해 File 데이터 보낼 때"
date: "2022-09-28"
desc: "회사 업무중, 서버로 데이터를 fetch함수를 활용해 보낼 때 나타난 에러 부분을, 예제코드로 바꾸었습니다."
alt: "graphql이란"
category: "JavaScript"
tags: ["JavaScript"]
thumbnail: ./../../images/js_fetch_api_logo.png
---

회사에서 글을 등록하는 페이지를 만들고 있었다. 지금까지는 string, int, json 등의 데이터만 보내어서 별 문제가 없었지만, 이미지같은 파일 데이터도 같이 보내는 과저에서 오류가 발생했다.

구글링을 통한 결과 파일 데이터도 같이 보내기 위해서는 몇가지 조건이 필요했다.

1. javascript의 fetch()함수를 활용할 때는 headers에 Content-Type을 지정하지 말 것
    1. 이유는, 개발자가 임의로 “multipart/form-data” 형식을 지정할 경우 뒤에 “—boundary” 속성값이 자동으로 지정되지 않는다고 한다. ( boundary란 파일을 첨부할 때,
       브라우저가 생성한 값으로 서버에서 받을 때 필요하다고 알고 있다. )
2. 파일 데이터와, 다른 속성값들도 같이 보내고 싶은 경우 나눠서 보낼 것 ( Controller에서 json 형식의 파일을 dto를 통해 따로 받고 싶은 경우 )

```javascript
/** front단 */

let testData = {
		name: "jang",
		image: "test.png",
		description: "content..."
	};

let testForm = new FormData();
// Blob을 활용해 보내는 이유에 대해서는 아직 공부가 필요하다.
testForm.append("key", new Blob([ JSON.stringify(test) ], {type: "application/json"}));
testForm.append("file", data.file);

fetch(url, {
	method: "POST", // GET, POST, PUT, DELETE
	body: testForm, // formData로 보내는 경우 따로 변환하여 보낼 필요가 없다고 안다.
	...
}).then(res =>
... )
;
```

```java

/** controller단 */

// ResponseResult 객체는 전달할 때 사용할 목적으로 만든 임의 객체로
// front단에서 json형식으로 받을 수 있다.
/** controller단 */

// ResponseResult 객체는 전달할 때 사용할 목적으로 만든 임의 객체로
// front단에서 json형식으로 받을 수 있다.
@ApiOperation(value = "컬렉션 등록")
@PreAuthorize("hasRole('SYSTEM')")
@ResponseBody
@PostMapping(value = "/new")
public ResponseResult regist(@RequestPart(value = "key") WriteDto key,
@RequestPart(value = "file") MultipartFile file
) throws IOException {
	return ResponseResult
	.builder()
	.statusCode(HttpStatus.SC_OK)
	.data(writeService.regist(key, file))
	.message(MessageUtil.getMessage("test.test")).build();
}
```
> 출처: https://medium.com/jaehoon-techblog/simpleblog-개발-일지-4-55a8d2a8604

---
## 에러난 원인들

1. **( Back-end )** 처음에 @RequestBody를 사용해서 파일과, json형태의 자료를 한꺼번에 받으려 해서 에러가 났던 것.
    1. **@RequestPart**를 활용하여 해결하였다.
2. **( Back-end )** Controller에서 **@RequestPart** 어노테이션을 사용할 때 안의 속성에서 **value=”…”**를 사용해야 하는데, **name=”…”** 를 사용해서 계속 매핑이 되지 않았단 것.
    1. **name**속성이 아닌 **value**속성을 활용하여 해결하였다.
3. **( Front-end )** fetch로 보낼 때 Headers에 Content-Type를 지정 한 것. ( **“multipart/form-data”** or **“application/json”** )
    1. **Content-Type**을 지정하지 않음으로써 해결했다.
4.  **( Front-end )** 데이터를 한 json안에 혹은 new FormData()를 선언해서 한 곳에 같이 넣어 보낸 것.
    1. Controller에서 RequestPart를 통해 나누어서 받으려고 할 때 프론트에서도 보낼 방식을 바꿨어야 했는데 뒤늦게 알게 되어 많이 아쉬웠다.
    

---

## 마치며

통신 관련 지식, spring에 대한 이해도 부족으로 일어난 에러들이라고 느꼈다.  
프로그래밍 언어만 공부할 게 아니라 통신 등 생각보다 공부할 게 많다고 느꼈다.  
그리고 에러코드를 제대로 훓어봤더라면 더 빨리 해결책을 찾을 수도 있었는데, 빨리 해결하려고 하다가 오히려 더 늦게 해결하게 되어 많이 아쉬웠다. 
관련 지식을 조금 더 공부하고, 다음부터 에러가 날 때는 좀 더 유심히 봐야겠다고 다짐하게 되었다.
