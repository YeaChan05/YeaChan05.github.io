---
title: Obsidian과-Github-Blog-연동하기
date: 2025-01-08
categories: ect
tags:
  - 문서
  - 일상
---
평소에는 기록을 그리 습관처럼 하지는 않지만 최근들어 [Obsidian](https://obsidian.md/)이라는 마크다운 에디터를 찾아서 사용하고 있다

notion도 있고 Velog도 있고 Tstory도 있지만 안하는데에는 이유가 있다

필자에게 이상한 습관이 있기 때문이다

그것은
1. 자동화가 안되면 안된다
2. 써보기로 한것은 반드시 계속 쓴다

인데, 기록하던 notion에 문제점이 있었다

### 자동화가 완벽하지는 않다

노션에는 다양한 자동화 처리기(특히, 요즘은 ai도 따로 제공해줘서 상당히 좋다만..)가 존재하기는 하지만 뭔가 뭔가다...

예를 들어, 파일명을 자동으로 작성해줄 트리거를 만들 수 있다던지, 반복적인 행위를 자동화할 코드 작성같은 것들이 되지 않는다

### 자잘한 오류

notion을 사용하다보면 어째서인지 가끔씩 작성한 내용이 누락되는 경우가 있다

내 실수라고 생각하고 넘어갔었는데, 주변 사람들 얘기를 들어보니 나만 그런건 아닌거 같더라..

텍스트 뿐만 아니라 이미지도 마찬가지다

velog에경우 이미지 업로드시 곧바로 CDN에 업로드 되는 형태인거 같다만, 가끔 보니 이미지가 제대로 업로드 되지 않는것 같다(사실, 이때문에 velog를 쓰기 싫은것이 크다)

### 외부 도구에 대해 친절하지 않다

정확히는 외부 API를 통해 외부 도구를 접근하는데에 있어서는 상당히 많은 지원이 있지만, 사실 이부분도 코드로 해결할 수 있다면 해결하도록 만들수도 있고, 추가로 API 의존적 문서 기록방식을 사용할 수 밖에 없게된다

사실 IDE처럼 외부 도구를 추가하도록 plugin 추가가 가능하다면 너무 좋을텐데 아직까지는 그렇지 않은거같다

\
\
\
\
	
### 결국은 그냥 편해서




요즘은 또 자기 PR의 시대라고 하지 않던가..

![](assets/img/posts/2025-01-08-Obsidian%EA%B3%BC-Github-Blog-%EC%97%B0%EB%8F%99%ED%95%98%EA%B8%B0/b7c9d5d7df307f0bc71ac271444f5504_MD5.jpeg)

사실 뭐 이런 저런 핑계고 그냥 Obsidian이 쓰기 편해서가 가장 크다

근데 또 하는김에 기록을 외부에 노출시키기도 해야겠다는 생각이 들었다

자기 PR의 시대라나 뭐라나.. 

## 일단 ㄱㄱㄱ
![](assets/img/posts/2025-01-08-Obsidian%EA%B3%BC-Github-Blog-%EC%97%B0%EB%8F%99%ED%95%98%EA%B8%B0/8f644ea298dce8864886a77b2756b1a4_MD5.jpeg)작성 구조는 생각보다 간단하다

글을 작성하고 commit push하면 Github에 작성한 글이 올라가고, Github Actions가 돌아 Jekyll을 build해 웹페이지로 글을 올린다


\
\
\
\
\
	
### 1. Jekyll 테마 다운로드

일단 테마부터 구경해보자

http://jekyllthemes.org/


![짱 많다](assets/img/posts/2025-01-08-Obsidian%EA%B3%BC-Github-Blog-%EC%97%B0%EB%8F%99%ED%95%98%EA%B8%B0/9028b99c4833a76cad817ddea2e539ba_MD5.jpeg)

필자는 [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) 테마를 선택했다

이유로는 심플한 UI와


![](assets/img/posts/2025-01-08-Obsidian%EA%B3%BC-Github-Blog-%EC%97%B0%EB%8F%99%ED%95%98%EA%B8%B0/136deff620d520f0f5efffead9bf2787_MD5.jpeg)
프로젝트 starter 지원이 있다

보통의 경우에는 해당 프로젝트의 압축파일을 다운로드 하면 된다

starter나 github template를 지원해주는 경우에는 압축파일 다운로드 말고 Github 저장소를 생성할때 연결해도 좋다

\
\
	


### 2. Github Pages 생성

![](assets/img/posts/2025-01-08-Obsidian%EA%B3%BC-Github-Blog-%EC%97%B0%EB%8F%99%ED%95%98%EA%B8%B0/222edcf79f354ff56bad72f44f52b692_MD5.jpeg)


이제 저장소를 생성한다
Github Pages를 사용하려면 저장소 이름을 `[사용자 Github 이름].github.io`으로 작성해야만 한다


![](assets/img/posts/2025-01-08-Obsidian%EA%B3%BC-Github-Blog-%EC%97%B0%EB%8F%99%ED%95%98%EA%B8%B0/43ba9c43bdf6ded60e4345ec4c586893_MD5.jpeg)
settings로 와보면 Pages라는 섹션이 있다

내부로 들어가 Build and deployment를 Github Actions로 선택해주자


\
\
\
\
	

### 
