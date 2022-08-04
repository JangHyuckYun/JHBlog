---
title: Android 초기 설정 및 jdk 설치 ( Mac )
date: "2022-08-02"
desc: "homebrew 설치 방법부터, homebrew 활용한 jdk 설치 및 android studio 환경 변수 설정"
category: "Android Studio"
tags: [ "Gatsby", "React", "Javascript" ]
thumbnail: ./images/android_studio.png
---

# Android Studio 설치

먼저 https://developer.android.com/studio 사이트에 접속하여 안드로이드 스튜디오를 다운 받는다.

![](./images/android_studio_site.png)

## 환경변수 설정

안드로이드 스튜디오를 설치 후 해야하는 환경변수 설정이다. ( iterm에서 진행 )

맥 OS 카탈리나보다 높은 버전의 맥 OS를 사용하는 경우에는 터미널에서 **$HOME/.zshrc** 파일을,
미만 버전을 사용시 **$HOME/.bash_profile** 또는 **$HOME/.bashrc**  파일을 수정해야 한다.

```shell
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

터미널에서 vim 명령어를 통해 위의 설명한 곳으로 접속 후 위의 코드를 붙여녛기 하면 된다.

---

# JDK 설치
<br />
내가 본 강의에 따르면 jdk 버전을 1.8로 설치하라고 나온다. 하지만 1.8로 설치 후 추후에 실행 시,
버전 관련해서 오류가 발생하여 11버전으로 바꾸어 주었더니 잘 되었다.

Mac 에서는 주로 jdk를 설치할 때 Homebrew를 사용한다고 들었다.

## Homebrew 설치

다음으로 Homebrew 공식 사이트 "https://brew.sh/index_ko" 에 접속 후 메인에 바로 나와있는 명령어를 iterm( terminal )에 복붙 후 엔터 해준다.

![](./images/homebrewSite.png)

아래 사진처럼 패스워드가 나오게 되고, 맥북의 비번을 입력해주면 된다.

![](./images/homebrew_cmd_1.png)

그러면 아래 사진처럼 나오게 되고, Enter를 누르면 설치가 된다.
( xcode를 미리 설치 하였을 시 가끔 license 관련해서 에러가 나오기도 한다. 나 "https://anywaydevlog.tistory.com/13#recentComments" 여기 사이트를 보고 해결했다. )
![](./images/homebrew_cmd_2.png)

이후에 나오는 echo 관련 명령문 2개를 추가로 터미널에 입력해준다.

```shell
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/사용자/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)”
```

다음으로 마지막 명령어를 입력하면 jdk 설치는 끝난다.

```shell
brew install —cask adoptopenjdk/openjdk/adoptopenjdk11
```

---

# 마치며
  
  
이러면 세팅은 끝나는걸로 알고 있다. 추후에 부족한 점을 찾으면 수정할 예정이다.  
생각보다 맥으로 하면 윈도우를 사용했을 떄 보다 어려울 거라 생각했는데 그렇게 어렵진 않고,
더 쉬운 느낌이 든다. 하지만 키보드는 아직 적응중이다. ㅜㅜ



