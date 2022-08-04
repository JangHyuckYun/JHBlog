---
title: Intellij에 Eclipse Project적용하면서 났던 오류들
date: "2022-03-25"
desc: "Intellij에 Eclipse Project 적용하면서 생겨났던 오류, 해결법 등에 대해 적어봤습니다."
alt: "Intellij에 Eclipse Project적용하면서 났던 오류들"
category: "Spring"
tags: [ "Spring", "EgovFramework", "Intellij", "Eclipse Project", "Eclipse" ]
thumbnail: ./images/eclipseToIntellij.png
---

# Intellij에 Eclipse Project 적용 하기까지

우리 회사에선 주로 Spring 짤 때 Eclipse를 사용해서 짜고 만든 프로젝트들은 svn(subversion)에서 관리한다. (git 쓰고 싶어요..)
Spring 쪽 코드를 짤 땐 괜찮은데 javascript 쪽 코드를 짤땐 개인적으로 되게 불편했다. (내가 eclipse를 잘 몰라서 그럴수도 있다.) 
Intellij로 한번 짜보려고 Eclipse Project를 Intellij로 옮기는 것을 전에도 한번 시도해봤었다.
그때는 처참히 패배하고 시도를 포기하였다. 요번에 다시 사업을 시작하게 되면서 관련 코딩을 할 준비를 해야 했는데
eclipse로 js 코드를 짜고 싶지 않아 다시 한번 도전 해봤다. 결과부터 말하자면 '겨우 성공' 이다.

---

# 오류 종류

Intellij에 Eclipse Project를 적용하며 났던 문제가 크게 2가지였던거 같다.

- pom.xml `repository -> egovframe url` 관련 오류
- 실행 시 `src/main/resources` 경로 안의 파일들 인식 못하는 오류


`※비교 확인을 위해 Eclipse에서 실행할 프로젝트 폴더 (이하 폴더A), Intellij에서 실행할 프로젝트 폴더(이하 폴더B)로 나누어서 비교해봤다.`

---

# pom.xml `repository -> egovframe url` 관련 오류

여기 오류는 의외로 쉽게 해결했다.

![pom.xml](src/posts/blog/development/Programming-language/Spring/images/pom_xml_egov.png)

위의 사진이 pom.xml의 일부분이다. 여기서 원래 url 들이 **https**가 아닌 **http**로 되어있었다.
Eclipse에서는 별 문제 없이 됬었는데 Intellij에서 실행하였더니 오류가 났었다. 간단하게 `http -> https` 로 수정하였더니 잘 되었다.

> **폴더A**에서도 똑같이 바꾸고 실행해봐도 별 문제없이 돌아갔다.

---

# 실행 시 `src/main/resources` 경로 안의 파일들 인식 못하는 오류
겨우겨우 maven을 통해 라이브러리까지 받는걸 성공하고, 톰캣 적용하고 실행을 해봤다.
잘 되나 싶더니만 에러가 났다. AnalyticsService에서 관련 json 파일을 `src/main/resources`에서 읽어오지 못한것이 원인이였다.
똑같은 프로젝트인데 **Eclipse**에서는 잘 되고 **Intellij**에서는 안되니 진짜 미치는줄 알았다.

## 해결책 실마리 발견

몇시간 동안 희망을 찾아 해매며 구글링을해도 잘 모르겠어서 포기하던 찰나였으나 StackOverflow에서 어느 한 글에 `src/main/resources`의 파일들은
build시 `target/classes` 에 옮겨진다? 였나 비슷한 문구를 발견했다. 바로 확인작업에 들어갔다.
실제로 `폴더A`에서 `target/classes`폴더에 들어가보면 resources의 파일들이 있는 반면,
`폴더B -> target/classes`쪽에는 resources 관련 파일들이 없었다!


## 해결책

해결책을 찾기까지 과정은 힘들었지만 막상 알고나니 되게 허무했다. (기초? 이론 등의 중요성을 깨닫게 되기도 하였다.)  
결론부터 말하면 IDE 설정의 문제였다. 먼저 `Settings -> Build, Execution, Deployment -> Compiler -> Java Compiler`에 들어간다.

![setting - java compiler](src/posts/blog/development/Programming-language/Spring/images/javaCompiler_down.png)

아래쪽을 보면 위에 사진처럼 생긴 부분이 있을것이다.
여기에 실행할 Module(Maven Project)을 넣어주고 적용 후 실행하면 매우 잘 된다!(안되면 Rebuild 후 재실행)

---

# 마치며

몇시간을 해매도 못찾겠어서 아무 생각없이 세팅을 뒤지다가 혹시나 싶어 건드린게 매우 잘 되어 되게 놀랐었다.
실행 후에 들었던 생각은 "Java 관련해 컴파일 되는 과정을 좀 더 공부했었더라면? 실행 과정에 대해 좀 더 알았더라면?" 이런 생각들이
머릿속을 맴돌았다. 회사에선 주로 Spring(Spring boot X ... ㅠ)을 사용하니 평소에 사용하던 Intellij로 작업하고 싶어 도전해봤다.
개인적으로는 React, Typescript 쪽을 중점으로 공부하고 있어 멀리 뒀었는데.. 아무래도 Spring 쪽도 다시 봐야겠다는 생각이 든 계기였다.
