---
title: 2025-05-04-MCP(With-Spring-AI-MCP)
date: 2025-05-04
categories: Spring AI
tags:
  - Spring
  - AI
toc: true
pin: true
sitemap:
  changefreq: daily
  priority: 1
image:
  path: assets/img/posts/2025-05-04-MCP(With-Spring-AI-MCP)/Pasted%20Image%2020250504154933_492.png
---
![](assets/img/posts/2025-05-04-MCP(With-Spring-AI-MCP)/b7c0de8bf7c9c1f244141b87773cde48_MD5.jpeg)
## MCP란

최근 Antropic에서 발표한 MCP

설명이 주저리주저리 많지만 그냥 기존의 http 통신처럼 하나의 프로토콜이 생겨난거다

다만, 이 프로토콜의 사용 주체가 조금 달라진것이다

사람이 아닌 AI agent가 그 주체인것이다

사실 프로토콜이 존재하기 전에도 ai에게 도구를 사용하도록 하는것은 가능했었다

예를들어 gpt가 이미지를 생성해준다던지, 아니면 웹에서 검색을 해오는것도 결국 ai가 도구를 사용한 것이니까

물론 그 도구를 사용하게 하기 위해서는 ai에게 도구 사용법을 숙지시켜야 했었다

하지만 MCP의 등장으로 도구를 사용하는 방식의 표준이 생긴것이다

Antropic에서는 MCP를 C-type 포트로 비유한다

실제로 어떤 도구인지는 사용자(AI agent)는 구체적으로 알 필요가 없다

단지 C-type을 지원하기만 한다면 어떤 도구든지 사용 가능한것이다

### MCP 구성요소

![](assets/img/posts/2025-05-04-MCP(With-Spring-AI-MCP)/Pasted%20image%2020250502152024.png)
MCP는 크게 세가지 구성요소가 있다고 생각하면 된다

- MCP Host: 여러 Client 인스턴스를 관리하여 AI/LLM을 통합 및 샘플링
- MCP Client: AI Agent를 실제로 사용하는 서비스
- MCP Server: 서비스 가능한 도구를 제공

### Tool
앞에서도 말했지만 기존에도 AI가 사용 가능한 도구(Tool)이라는 개념은 존재했다

그도 그럴것이 AI 자체로는 할 수 있는 작업에 제한이 많았기 때문이다

다만, 이제는 그 도구를 쉽게 적용할 수 있도록 표준이 생겼다 정도?

MCP Server가 도구를 제공한다는건 이 도구를 간단하게 접근 가능한 형태로 만들어둔다는 것이다

Tool에 어떤것이 있는지 예시를 좀 들어보자면..
- 웹 검색 
- 파일 시스템 접근
- 이메일 전송
- API 호출
- 서드파티 애플리케이션(Slack, Notion, ...~) 사용
- 데이터소스 접근
- 등등..

보이는것과 같이 애초에 안됐을리가 없는 녀석들이다


### 전송 방식
Transport Layer에서 MCP Server를 호출할때는 다양한 도구를 호출 할 수 있도록 JSON-RPC 2.0을 사용한다

뭐 언어 종속을 회피하기 위함과 비동기, 그리고 확장성을 위해서 RPC를 사용한것으로 보여진다

통신 방식에는 두가지가 있다
1. Stdio: 표준 입출력을 사용한 통신방식. 로컬 MCP Server에서 사용하기에 적합하다.
2. HTTP SSE: HTTP SSE를 사용해 서버에서 응답을 보내는 형태. 로컬이나 원격 MCP Server에서 사용 가능하다.

### MCP Connection Lifecycle
MCP의 연결 과정 라이프사이클을 살펴보자
#### 1.  초기화
1. 클라이언트가 프로토콜 버전과 기능이 포함된 `initialize` 요청 전송
2. 서버는 자신의 프로토콜 버전과 기능으로 응답
3. 클라이언트가 확인으로 `initialized` 알림 전송
4. 이후 정상적인 메시지 교환

#### 2.  메시지 교환
초기화 이후에는 아래 두가지 패턴이 지원된다.
- **요청-응답**: 클라이언트나 서버가 요청을 보내고, 상대방이 응답
- **알림**: 양쪽에서 일방향 메시지 전송 가능

#### 3. 연결 종료
양쪽 중 어느 쪽이든 아래와 같은 방법으로 연결을 종료할 수 있음
- `close()` 함수를 통한 정상 종료
- 전송 연결 해제
- [오류 상황](https://modelcontextprotocol.io/docs/concepts/architecture#error-handling)



## Spring MCP

![](assets/img/posts/2025-05-04-MCP(With-Spring-AI-MCP)/Pasted%20image%2020250504162000.png)
대강의 구조는 위와 같다

사용자의 요청이 들어오면 필요에 따라 비즈니스 로직을 AI에 도움을 받는다

이때, AI가 사용할 도구를 MCP Server에서 호출한다

MCP Server는 AI가 사용할 도구들을 미리 등록하고, 이를 `ToolProvider`(`ToolCallbackProvider`)를 사용해 프로시저로 제공한다

## 예제
사용자가 검색한 기술을 AI가 적절히 학습하기 좋은 프로젝트를 Github에서 찾아와 추천해주는 서비스를 만들어봤다(빠르게 만든 프로젝트다 보니 퀄리티가 조금 아쉽지만,,,)

### MCP Client

```java
public String search(String keyword) {  
    String systemPrompt = buildSystemPrompt(keyword);  
    return chatClient.prompt(systemPrompt)  
            .call()  
            .content();  
}
```
사용자가 검색 요청을 하면 검색어를 시스템 프롬프트에 넣어 적절한 검색이 가능하도록 했다

여기서 중요한건 AI에게 MCP를 사용해 적절한 프로젝트를 추천할 수 있도록 프롬프트를 잘 짜야한다는 것이다


```java
@Configuration  
@RequiredArgsConstructor  
public class AiConfig {  
  
    @Bean  
    public ChatClient chatClient(OpenAiChatModel chatModel, ToolCallbackProvider toolCallbackProvider) {  
        return ChatClient.builder(chatModel)  
                .defaultTools(toolCallbackProvider.getToolCallbacks())  
                .build();  
    }  
}
```
이제 모델이 도구를 사용할 수 있도록 쥐어줘야한다

`ChatClient`를 사용할때 tool을 직접 주입해줘도 되지만 당장에는 단일 기능만 존재하기 때문에 bean 등록 시점에 `ToolCallbackProvider`를 주입했다

이때 사용한 AI 모델은 OpenAI의 GPT-4o-mini를 사용했다

다만 여기서 주의 해야할 점이 있는데,

```yml
spring:
  ai:  
    mcp:  
      client:
        toolcallback:  
          enabled: true
```

`ToolCallbackProvider`를 사용 가능하도록 설정을 해줘야한다

이제 MCP Server 정보를 MCP Client에게 알려주는 설정을 보자

```yml
spring:
  ai:  
    mcp:  
      client:
        sse:  
          connections:  
            server:  
              url: http://localhost:8081  
        stdio:  
          connections:  
            github:  
              command: ${NPX_PATH:npx}  
              args:  
                - "-y"  
                - "@modelcontextprotocol/server-github"  
              env:  
                GITHUB_PERSONAL_ACCESS_TOKEN: ${GITHUB_PERSONAL_ACCESS_TOKEN}
          
```

직접 만든 MCP Server와 외부에서 가져온 MCP Server 두개를 등록해뒀다

직접 만든 MCP Server도 Github을 사용하긴 하지만 학습차원에서(그리고 내 기억상으로는 Github 검색기능이 기존의 Github MCP Server에 없었다) 별도의 MCP Server를 만들어봤다

해당 MCP Server는 어차피 당장에는 로컬에서 돌릴것이기 때문에 stdio로도 충분히 동작 하겠지만 원격 환경이 사실 활용 범주가 더 넓을 거 같아서 SSE로 서버를 연결했다

참고로 외부 mcp 설정 방식은 보통 https://smithery.ai/같은 MCP Server 검색 서비스에서 제공하는  json을 yml로 위와 같이 비슷한 형태로 설정해주면 사용 가능하다

현재 프롬프트는 직접 만든 MCP Server에서 검색한 기술과 관련된 프로젝트를 가져오고, 사용자가 학습하기 적합한 프로젝트인지 확인하기 위해 stdio로 연결된 MCP Server를 사용해 소스를 확인하여 적합성을 판단한다

이후 적절한 프로젝트 몇개를 가져와 추천해준다

### MCP Server

```java
@Service  
public class GithubTool {  
  
    public static final int SIZE = 30;  
  
    @Tool(name = "search_in_github", description = "Search for appropriate results in GitHub")  
    public String search(@ToolParam(description = "검색할 키워드") SearchRequest request) {  
        String keyword = request.keyword();  
        GitHub gitHub = null;  
        try {  
            gitHub = GitHub.connectAnonymously();  
        } catch (IOException e) {  
            throw new RuntimeException(e);  
        }  
        PagedSearchIterable<GHRepository> repositories = gitHub.searchRepositories().q(keyword).list();  
  
        StringBuilder result = new StringBuilder();  
        int count = 0;  
        for (GHRepository repository : repositories.withPageSize(SIZE)) {  
            count++;  
            result.append("Repo: ").append(repository.getFullName())  
                    .append(", Stars: ").append(repository.getStargazersCount()) 
                    .append(", Description: ").append(repository.getDescription())  
                    .append(", URL: ").append(repository.getHtmlUrl())  
                    .append(", Owner: ").append(repository.getOwnerName())  
                    .append("\n");  
        }  
        return result.toString();  
    }
```

사실 MCP Server에서 별걸 하지는 않는다

tool을 호출하면 AI가 요청한 검색어를 Github에서 검색해 필요한 정보를 가져온다

이렇게 만든 tool은

```java
@Configuration  
public class ToolConfig {  
  
    @Bean  
    public ToolCallbackProvider toolCallbackProvider(GithubTool githubTool){  
        return MethodToolCallbackProvider.builder()  
                .toolObjects(githubTool)  
                .build();  
    }  
}
```

이런식으로 던져주면 된다

`Object`형태면 전부 받아주는것 같다

참고로 해당 MCP Tool은 MCP Client에서 연결 시도하게 되면

```json
{
  "jsonrpc": "2.0",
  "id": "676c8717-1",
  "result": {
    "tools": [
      {
        "name": "search_in_github",
        "description": "Search for appropriate results in GitHub",
        "inputSchema": {
          "type": "object",
          "properties": {
            "request": {
              "type": "object",
              "description": "검색할 키워드",
              "properties": {
                "keyword": {
                  "type": "string"
                }
              },
              "required": ["keyword"]
            }
          },
          "required": ["request"],
          "additionalProperties": false
        }
      }
    ]
  },
  "error": null
}
```
이런 로그가 발생한다

[Github](https://github.com/YeaChan05/mcp)

### Conclusion
솔직히 2024년 까지는 AI가 엄청나다는건 알고 있었지만 2025년은 정말 많이 다른거같다

AI 모델 성능 자체도 유의미한 수준으로 오르긴 했지만 그보다도 AI가 할 수 있는 범위가 너무 많이 늘어난거 같다

RAG를 사용한 패러다임 변화로 개인화된 AI로 기존의 모델 활용도를 높혀진거로 그치지 않고 MCP를 통해 AI가 더 넓은 범주의 행동이 가능해지더니 더 나아가 [A2A](https://developers.googleblog.com/ko/a2a-a-new-era-of-agent-interoperability/?_gl=1*j71baj*_up*MQ..*_ga*MzAzNTQzMzgxLjE3NDYzNDUzMDI.*_ga_H733Y2BZES*MTc0NjM0NTMwMi4xLjAuMTc0NjM0NTMwMi4wLjAuMA..)라는걸 Google이 발표해버렸다..

이제 사람이 사용하던 서비스를 AI가 사용할 수 있게 되었고, 심지어 AI간의 통합마저 편해진거 같다

그렇다면 백엔드 개발자가 할 수 있는것이 무엇일지를 고민해볼 필요가 있는거 같다

예시 코드를 보면 알겠지만 그냥 기존의 Controller를 사용해 접근 가능하던 서비스를 AI가 사용 가능한 도구 형태로 만든것 뿐이다

그렇기에 점점 사용자가 아닌 AI가 사용 가능한 형태로 서비스를 제공하게 되지 않을까 싶다

하지만 AI가 마음대로 사용 가능한 형태의 서비스를 만들어 주면 AI가 치명적인 실수를 저지르는 것을 방지할 수 없다

그렇기에 AI에게 어느정도의 자율을 주더라도 사용자가 중요한 부분에서는 제어권을 가질 수 있도록 서비스를 제공하고, 이 과정에서 잠재적 보안 위협 요소를 제거할 수 있어야 하지 않을까 싶다

또한, 기존의 사용자 뿐만 아니라 AI의 요청도 처리하는 서비스를 만들어야 하는 만큼 더 많은 트레픽에 대응할 수 있어야 할거 같다


<br>
<br>
<br>
갈길이 멀다...

>[Spring AI / Model Context Protocol](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-overview.html)




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