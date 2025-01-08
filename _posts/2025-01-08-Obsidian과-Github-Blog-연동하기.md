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

<br><br><br><br>

### 결국은 그냥 편해서




요즘은 또 자기 PR의 시대라고 하지 않던가..

![](assets/img/posts/2025-01-08-Obsidian%EA%B3%BC-Github-Blog-%EC%97%B0%EB%8F%99%ED%95%98%EA%B8%B0/b7c9d5d7df307f0bc71ac271444f5504_MD5.jpeg)

사실 뭐 이런 저런 핑계고 그냥 Obsidian이 쓰기 편해서가 가장 크다

근데 또 하는김에 기록을 외부에 노출시키기도 해야겠다는 생각이 들었다

자기 PR의 시대라나 뭐라나.. 

## 일단 ㄱㄱㄱ
![](assets/img/posts/2025-01-08-Obsidian%EA%B3%BC-Github-Blog-%EC%97%B0%EB%8F%99%ED%95%98%EA%B8%B0/8f644ea298dce8864886a77b2756b1a4_MD5.jpeg)작성 구조는 생각보다 간단하다

글을 작성하고 commit push하면 Github에 작성한 글이 올라가고, Github Actions가 돌아 Jekyll을 build해 웹페이지로 글을 올린다


<br><br><br><br><br>
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

<br><br><br>


### 2. Github Pages 생성

![](assets/img/posts/2025-01-08-Obsidian%EA%B3%BC-Github-Blog-%EC%97%B0%EB%8F%99%ED%95%98%EA%B8%B0/222edcf79f354ff56bad72f44f52b692_MD5.jpeg)


이제 저장소를 생성한다
Github Pages를 사용하려면 저장소 이름을 `[사용자 Github 이름].github.io`으로 작성해야만 한다


![](assets/img/posts/2025-01-08-Obsidian%EA%B3%BC-Github-Blog-%EC%97%B0%EB%8F%99%ED%95%98%EA%B8%B0/43ba9c43bdf6ded60e4345ec4c586893_MD5.jpeg)
settings로 와보면 Pages라는 섹션이 있다

내부로 들어가 Build and deployment를 Github Actions로 선택해주자


<br><br><br>

### 3. 프로젝트 구성
이제 옵시디언에 Vault를 만들고 압축한 파일을 해당 Vault 위치에 해제한다


![프로젝트 구조](assets/img/posts/2025-01-08-Obsidian%EA%B3%BC-Github-Blog-%EC%97%B0%EB%8F%99%ED%95%98%EA%B8%B0/6b1ce9e13afeb8d682bc23a4bea8f20b_MD5.jpeg)_프로젝트 구조_


여기서 필요한 설정값들은 각 테마마다 설정 방식이 달라 생략하도록 하겠다

기본적으로 프로젝트 루트의 `_config.yml`에 설정값을 작성하는 편이더라

이후 [Ruby](https://www.ruby-lang.org/ko/documentation/installation/)를 설치한다(jekyll이  ruby 기반인거 같다)

```sh
brew install rbenv
echo 'eval "$(rbenv init -)"' >> ~/.zshrc

rbenv install 3.4.1
rbenv global 3.4.1

bundler install

```



그리고는 서비스를 로컬에서 실행해보자

```sh
jekyll serve
```


![](assets/img/posts/2025-01-08-Obsidian%EA%B3%BC-Github-Blog-%EC%97%B0%EB%8F%99%ED%95%98%EA%B8%B0/4c595e85ea355a672f6e484b70c559bb_MD5.jpeg)


### 4. 옵시디언 세팅

아래의 세 plugin이 필요하다
- [Git](obsidian://show-plugin?id=obsidian-git): 변경사항 버전관리 자동화
- [Local Images Plus](obsidian://show-plugin?id=obsidian-local-images-plus): 캡쳐 이미지를 따로 보관
- [Templater](obsidian://show-plugin?id=templater-obsidian): 템플릿 명령어로 다양하게 처리



![](assets/img/posts/2025-01-08-Obsidian%EA%B3%BC-Github-Blog-%EC%97%B0%EB%8F%99%ED%95%98%EA%B8%B0/890442e1a3b032f2849ada1f56fc0191_MD5.jpeg)
Git plugin같은 경우 자동 커밋 시간 정도만 지정해준다


![](assets/img/posts/2025-01-08-Obsidian%EA%B3%BC-Github-Blog-%EC%97%B0%EB%8F%99%ED%95%98%EA%B8%B0/f74aa5aff495e78af11c25ed600c9857_MD5.jpeg)
Local Images Plus는 붙여넣기 했을때 이미지 파일이 저장되어야할 위치로 지정해준다

지정해주지 않는다면 프로젝트가 이미지 파일 경로를 정상적으로 인식 못할수도 있으니 주의하자

마지막으로 포스팅 템플릿을 만들어보자

템플릿 문법은 [공식문서](https://silentvoid13.github.io/Templater/)에 잘 나와있다

```markdown
---

title: <%*

let userTitle = await tp.system.prompt("새 게시물의 제목을 입력하세요");

if (!userTitle) {

userTitle = "new-post";

}

  

let hyphenTitle = userTitle.replace(/\s+/g, "-");

  

tR += hyphenTitle;

%>

date: <% tp.date.now("YYYY-MM-DD") %>

categories:

tags:

---

  

<%*

const newFileName = `${tp.date.now("YYYY-MM-DD")}-${hyphenTitle}`;

tR += await tp.file.rename(newFileName);

%>

  

<script src="https://giscus.app/client.js"

data-repo="YeaChan05/YeaChan05.github.io"

data-repo-id="********"

data-category="Announcements"

data-category-id="**********"

data-mapping="pathname"

data-strict="0"

data-reactions-enabled="1"

data-emit-metadata="0"

data-input-position="bottom"

data-theme="preferred_color_scheme"

data-lang="ko"

crossorigin="anonymous"

async>

</script>
```


간략하게 설명하자면, 파일 생성시에 사용자에게 파일명을 먼저 입력받아 파일을 완성하는데, 이때 파일 제목은 YYYY-MM-DD-제목으로 작성한다(공백은 하이픈으로 대체한다)

위와같이 제목을 작성하는 이유는 jekyll에서 지정한 [포스팅 파일 양식](https://jekyllrb.com/docs/posts/)이니 따라야 한다

또, 아래에 스크립트가 보일텐데 Github Discussion으로 작동하는 댓글 라이브러리인 [giscus](https://giscus.app/ko)이다



![](assets/img/posts/2025-01-08-Obsidian%EA%B3%BC-Github-Blog-%EC%97%B0%EB%8F%99%ED%95%98%EA%B8%B0/59e3fb9d9d4ded837b844eda22f72d04_MD5.jpeg)
이제 여기에 단축키 설정까지 한다면 완벽하다

<br><br><br><br><br>

>[Chirpy Jekyll Thema](https://github.com/cotes2020/jekyll-theme-chirpy)
>
>[Jekyll Docs](https://jekyllrb.com/docs/)
>
>[Github Blog를 위한 Obsidian Settings](https://a2256014.github.io/dev/2024/12/01/Obsidian-Settings.html)
>
>[Chirpy 테마로 GitHub 블로그 만들기 ④](https://jason9288.github.io/posts/github_blog_4/)



<script src="https://giscus.app/client.js"
        data-repo="YeaChan05/YeaChan05.github.io"
        data-repo-id="R_kgDONnXleQ"
        data-category="Announcements"
        data-category-id="DIC_kwDONnXlec4Cl2a5"
        data-mapping="pathname"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="preferred_color_scheme"
        data-lang="ko"
        crossorigin="anonymous"
        async>
</script>