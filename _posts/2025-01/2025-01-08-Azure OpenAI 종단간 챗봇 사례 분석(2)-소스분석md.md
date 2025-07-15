---
title: "Azure OpenAI 종단간 챗봇 사례 분석(2)-소스분석"
date: 2025-01-08
tags: [마이크로소프트, Microsoft, Build 2025, Azure AI Foundry, Azure, Azure AI Foundry SDK, Azure OpenAI Studio, Azure OpenAI Service, Azure Machine Learning, Azure App Service, Azure Key Vault, Azure Monitor]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

해당 GitHub 리포지토리 `Azure-Samples/openai-end-to-end-basic` 를 기반으로 각 구성 요소별 분석해서 노트를 작성했다. 



## 1. 리포지토리 개요

* **목적**: Azure AI Foundry 기반 간단한 종단 간(End-to-End) 채팅 애플리케이션의 참조 구현. Azure OpenAI 모델과 WebApp, 에이전트, API 통합 예제 포함 
* **라이선스**: MIT (저작권 및 라이선스 고지 필수)
* **활동**: 최근 3개월 내 Dependabot 의존성 업데이트 및 CodeQL 자동 검사 수행
* **오픈 이슈**: `Parameterize the model version based on the region` 관련 이슈 오픈(2024년 11월)



## 2. 주요 구조

/
├── .devcontainer/               – 개발 컨테이너(config)
├── .github/                     – 워크플로우: Dependabot, CodeQL, 보안 검사
├── docs/, media/                – 문서 및 다이어그램
├── infra-as-code/bicep/         – Bicep 인프라 정의(Azure App Service, AI Foundry 등)
├── website/
│   ├── chatui/                  – 웹 UI 코드 (HTML, JS, CSS, .NET backend 포함)
│   └── …                        
├── bicep/                       – 추가 인프라 코드
├── CONTRIBUTING.md             – 기여 안내 :contentReference[oaicite:9]{index=9}
├── README.md                   – 프로젝트 소개 및 참조
├── LICENSE.md, SECURITY.md     – 라이선스 및 보안 정책



## 3. infra-as-code/bicep 폴더 분석

* **Bicep**

  * 정의

    * **Microsoft Azure**에서 공식 지원하는 **ARM 템플릿의 상위 수준 추상화 언어**
    * JSON 기반의 복잡한 ARM Template을 **더 간결하고 사람이 읽기 쉬운 문법**으로 표현
    * Azure 리소스를 코드로 정의하고 관리할 수 있도록 해줌

  * 장점

    * ARM 템플릿보다 간결하고 읽기 쉬움
    * Visual Studio Code 지원 (자동 완성, 오류 감지)
    * ARM JSON으로 자동 컴파일됨 (Azure와 완벽 호환)
    * 반복, 조건, 모듈화 기능으로 확장 가능

  * 배포 방법

    ```bash
    # CLI로 리소스 그룹에 배포
    az deployment group create \
      --resource-group my-rg \
      --template-file main.bicep
    ```

* **배포 대상**:

  - Azure App Service: 채팅 웹 앱 호스팅
  - Azure AI Foundry Agent Service: 봇 에이전트
  - Azure AI Search: grounding data source

* **기능**

  * `infra-as-code/bicep` 폴더의 `main.bicep`는 **웹‑챗 앱 + AI Orchestration 환경**을 한 번에 배포할 수 있는 종합 인프라 구성 템플릿

  * App Service부터 OpenAI, AI Search, 모니터링까지 포함한 **end-to-end 챗봇 인프라**를 자동화하며, 모듈화와 확장성을 고려한 설계

    | 기능                       | 목적                                                         |
    | -------------------------- | ------------------------------------------------------------ |
    | **인프라 자동화**          | 단일 명령(`az deployment`)으로 전 인프라 구성 배포           |
    | **요청자 ID 및 추적 설정** | `baseName`, `yourPrincipalId` 등의 파라미터로 일관된 네이밍 및 권한 관리 |
    | **생산성 및 유지관리**     | Bicep의 모듈화, parameter화를 통해 DevOps 파이프라인에 유리  |
    | **RAG 지원 아키텍처**      | AI Search + OpenAI 연동으로 Retrieval-Augmented Generation 흐름 구축 |

* **main.bicep 소스 구조:** 배포 파이프라인의 핵심 템플릿으로, 다음 주요 리소스들을 생성함

  ```bash
  targetScope = 'resourceGroup'
  
  @description('The region in which this architecture is deployed. Should match the region of the resource group.')
  @minLength(1)
  param location string = resourceGroup().location
  
  @description('This is the base name for each Azure resource name (6-8 chars)')
  @minLength(6)
  @maxLength(8)
  param baseName string
  
  @description('Assign your user some roles to support fluid access when working in the Azure AI Foundry portal and its dependencies.')
  @maxLength(36)
  @minLength(36)
  param yourPrincipalId string
  
  @description('Set to true to opt-out of deployment telemetry.')
  param telemetryOptOut bool = false
  
  // Customer Usage Attribution Id
  var varCuaid = '6aa4564a-a8b7-4ced-8e57-1043a41f4747'
  
  // ---- New resources ----
  
  @description('This is the log sink for all Azure Diagnostics in the workload.')
  resource logAnalyticsWorkspace 'Microsoft.OperationalInsights/workspaces@2025-02-01' = {
    name: 'log-workload'
    location: location
    properties: {
      sku: {
        name: 'PerGB2018'
      }
      retentionInDays: 30
      forceCmkForQuery: false
      workspaceCapping: {
        dailyQuotaGb: 10 // Production readiness change: In production, tune this value to ensure operational logs are collected, but a reasonable cap is set.
      }
      publicNetworkAccessForIngestion: 'Enabled'
      publicNetworkAccessForQuery: 'Enabled'
    }
  }
  ...
  ```

  * **App Service plan** (P1v3) 및 **Web App** 혹은 **API 앱 서비스 인스턴스**
  * **Azure Storage account** – 로그 및 애플리케이션 데이터를 저장
  * **Application Insights** – 모니터링 및 원격 분석
  * **Azure AI Foundry 프로젝트** – 에이전트 실행 컨테이너
  * **Cognitive Services 계정** (OpenAI) – GPT-4o 모델 배포를 위한 Azure OpenAI 서비스
  * **Azure AI Search 서비스** – 에이전트의 grounding 데이터 조회용
  * **Managed Identity** – 서비스 간 안전한 인증용
  * 추가적으로 필요한 경우 **Key Vault**, **Log Analytics** 같은 리소스 생성 가능 옵션 포함

* 모듈 분리 가능성

  * 개별 리소스를 Bicep 모듈로 쪼개 재사용 및 구조화가 가능함. (예: appService.bicep, storage.bicep 등)



## 4. website/chatui 폴더 상세

* **구성**:

  - 정적/동적 웹 UI (채팅 인터페이스 및 백엔드 API 호출)

  - `.NET` 기반 API layer + JavaScript chat front-end.

* **기능**:
  * **Easy Auth** 통한 Azure AD 인증
  * API서버는 Foundry 에이전트 호출
  * 에이전트는 Azure AI Search로 grounding data 조회
  * OpenAI 모델에 context + 질문 보내고 응답 수신
  * Application Insights 로 사용자 요청 및 에이전트 로그 수집
* **CI/CD**: Dependabot 자동 업데이트, CodeQL 취약점 검사

* **wwwroot/Program.cs**

  ```c#
  using Microsoft.Extensions.Options;
  using Azure.AI.Agents.Persistent;
  using Azure.Identity;
  using chatui.Configuration;
  
  var builder = WebApplication.CreateBuilder(args);
  
  builder.Services.AddOptions<ChatApiOptions>()
      .Bind(builder.Configuration)
      .ValidateDataAnnotations()
      .ValidateOnStart();
  
  builder.Services.AddSingleton((provider) =>
  {
      var config = provider.GetRequiredService<IOptions<ChatApiOptions>>().Value;
      PersistentAgentsClient client = new(config.AIProjectEndpoint, new DefaultAzureCredential());
  
      return client;
  });
  
  builder.Services.AddControllersWithViews();
  
  builder.Services.AddCors(options =>
  {
      options.AddPolicy("AllowAllOrigins",
          builder =>
          {
              builder.AllowAnyOrigin()
                     .AllowAnyMethod()
                     .AllowAnyHeader();
          });
  });
  
  var app = builder.Build();
  
  app.UseStaticFiles();
  
  app.UseRouting();
  
  app.MapControllerRoute(
      name: "default",
      pattern: "{controller=Home}/{action=Index}/{id?}");
  
  app.UseCors("AllowAllOrigins");
  
  app.Run();
  ```

* **Controlls/ChatController.cs**

  ```ChatController.cs
  using Microsoft.AspNetCore.Mvc;
  using Microsoft.Extensions.Options;
  using Azure;
  using Azure.AI.Agents.Persistent;
  using chatui.Configuration;
  
  namespace chatui.Controllers;
  
  [ApiController]
  [Route("[controller]/[action]")]
  
  public class ChatController(
      PersistentAgentsClient client,
      IOptionsMonitor<ChatApiOptions> options,
      ILogger<ChatController> logger) : ControllerBase
  {
      private readonly PersistentAgentsClient _client = client;
      private readonly IOptionsMonitor<ChatApiOptions> _options = options;
      private readonly ILogger<ChatController> _logger = logger;
  
      // TODO: [security] Do not trust client to provide threadId. Instead map current user to their active threadid in your application's own state store.
      // Without this security control in place, a user can inject messages into another user's thread.
      [HttpPost("{threadId}")]
      public async Task<IActionResult> Completions([FromRoute] string threadId, [FromBody] string prompt)
      {
          if (string.IsNullOrWhiteSpace(prompt))
              throw new ArgumentException("Prompt cannot be null, empty, or whitespace.", nameof(prompt));
  
          _logger.LogDebug("Prompt received {Prompt}", prompt);
          var _config = _options.CurrentValue;
  
          PersistentThreadMessage message = await _client.Messages.CreateMessageAsync(
              threadId,
              MessageRole.User,
              prompt);
  
          ThreadRun run = await _client.Runs.CreateRunAsync(threadId, _config.AIAgentId);
  
          while (run.Status == RunStatus.Queued || run.Status == RunStatus.InProgress || run.Status == RunStatus.RequiresAction)
          {
              await Task.Delay(TimeSpan.FromMilliseconds(500));
              run = (await _client.Runs.GetRunAsync(threadId, run.Id)).Value;
          }
  
          Pageable<PersistentThreadMessage> messages = _client.Messages.GetMessages(
              threadId: threadId, order: ListSortOrder.Ascending);
  
          var fullText =
              messages
                  .Where(m => m.Role == MessageRole.Agent)
                  .SelectMany(m => m.ContentItems.OfType<MessageTextContent>())
                  .Last().Text;
  
          return Ok(new { data = fullText });
      }
  
      [HttpPost]
      public async Task<IActionResult> Threads()
      {
          // TODO [performance efficiency] Delay creating a thread until the first user message arrives.
          PersistentAgentThread thread = await _client.Threads.CreateThreadAsync();
  
          return Ok(new { id = thread.Id });
      }
  }
  ```

* **Views/Home/Index.cshtml**

  ```html
  <html>
  
  <head>
    <style type="text/css">
      :root {
          --body-bg: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
          --msger-bg: #fff;
          --border: 2px solid #ddd;
          --left-msg-bg: #ececec;
          --right-msg-bg: #579ffb;
      }
  
      html {
          box-sizing: border-box;
      }
  
      *,
      *:before,
      *:after {
          margin: 0;
          padding: 0;
          box-sizing: inherit;
      }
  
      body {
          display: flex;
          justify-content: center;
          align-items: center;
          height: 100vh;
          background-image: var(--body-bg);
          font-family: Helvetica, sans-serif;
      }
  
      .msger {
          display: flex;
          flex-flow: column wrap;
          justify-content: space-between;
          width: 100%;
          max-width: 867px;
          margin: 25px 10px;
          height: calc(100% - 50px);
          border: var(--border);
          border-radius: 5px;
          background: var(--msger-bg);
          box-shadow: 0 15px 15px -5px rgba(0, 0, 0, 0.2);
      }
  
      .msger-header {
          display: flex;
          justify-content: space-between;
          padding: 10px;
          border-bottom: var(--border);
          background: #eee;
          color: #666;
      }
  
      .msger-chat {
          flex: 1;
          overflow-y: auto;
          padding: 10px;
      }
  
      .msger-chat::-webkit-scrollbar {
          width: 6px;
      }
  
      .msger-chat::-webkit-scrollbar-track {
          background: #ddd;
      }
  
      .msger-chat::-webkit-scrollbar-thumb {
          background: #bdbdbd;
      }
  
      .msg {
          display: flex;
          align-items: flex-end;
          margin-bottom: 10px;
      }
  
      .msg:last-of-type {
          margin: 0;
      }
  
      .msg-img {
          width: 50px;
          height: 50px;
          margin-right: 10px;
          background: #ddd;
          background-repeat: no-repeat;
          background-position: center;
          background-size: cover;
          border-radius: 50%;
      }
  
      .msg-bubble {
          max-width: 450px;
          padding: 15px;
          border-radius: 15px;
          background: var(--left-msg-bg);
      }
  
      .msg-info {
          display: flex;
          justify-content: space-between;
          align-items: center;
          margin-bottom: 10px;
      }
  
      .msg-info-name {
          margin-right: 10px;
          font-weight: bold;
      }
  
      .msg-info-time {
          font-size: 0.85em;
      }
  
      .left-msg .msg-bubble {
          border-bottom-left-radius: 0;
      }
  
      .right-msg {
          flex-direction: row-reverse;
      }
  
      .right-msg .msg-bubble {
          background: var(--right-msg-bg);
          color: #fff;
          border-bottom-right-radius: 0;
      }
  
      .right-msg .msg-img {
          margin: 0 0 0 10px;
      }
  
      .msger-inputarea {
          display: flex;
          padding: 10px;
          border-top: var(--border);
          background: #eee;
      }
  
      .msger-inputarea * {
          padding: 10px;
          border: none;
          border-radius: 3px;
          font-size: 1em;
      }
  
      .msger-input {
          flex: 1;
          background: #ddd;
      }
  
      .msger-send-btn {
          margin-left: 10px;
          background: rgb(0, 196, 65);
          color: #fff;
          font-weight: bold;
          cursor: pointer;
          transition: background 0.23s;
      }
  
      .msger-send-btn:hover {
          background: rgb(0, 180, 50);
      }
  
      .msger-chat {
          background-color: #ffffff;
      }
    </style>
    
    <script type="text/javascript">
      const BOT_NAME = "Chatbot";
      const PERSON_NAME = "";
  
      document.addEventListener("DOMContentLoaded", async () => {
          const chatForm = document.querySelector(".msger-inputarea");
          const chatContainer = document.querySelector(".msger-chat");
          const messageInput = chatForm?.elements?.message;
  
          const { id } = await createThread();
          const threadId = id;
  
          addChatMessage(BOT_NAME, "left", "How can I help you today?");
  
          chatForm.addEventListener("submit", async (event) => {
              event.preventDefault();
              const prompt = messageInput?.value?.trim();
  
              if (!prompt) return;
  
              messageInput.value = "";
  
              addChatMessage(PERSON_NAME, "right", prompt);
  
              try {
                  const { data } = await sendPrompt(prompt);
                  addChatMessage(BOT_NAME, "left", data);
              } catch (error) {
                  addChatMessage(BOT_NAME, "left", `Sorry, something went wrong.`);
                  console.error(error);
              }
          });
  
          async function sendPrompt(prompt) {
              const response = await fetch(`/chat/completions/${threadId}`, {
                  method: "POST",
                  headers: { "Content-Type": "application/json" },
                  body: JSON.stringify(prompt)
              });
  
              if (!response.ok) {
                  const errorMessage = await response.text().catch(() => response.statusText);
                  throw new Error(`Error sending prompt: ${errorMessage}`);
              }
  
              return response.json();
          }
  
          async function createThread() {
              const response = await fetch("/chat/threads", {
                  method: "POST",
                  headers: { "Content-Type": "application/json" }
              });
  
              if (!response.ok) {
                  const errorMessage = await response.text().catch(() => response.statusText);
                  throw new Error(`Error creating session: ${errorMessage}`);
              }
  
              return response.json();
          }
  
          function addChatMessage(name, side, text) {
              const timestamp = formatTime(new Date());
  
              const msgHTML = `
                  <div class="msg ${side}-msg">
                      <div class="msg-bubble">
                      <div class="msg-info">
                          <div class="msg-info-name">${name}</div>
                          <div class="msg-info-time">${timestamp}</div>
                      </div>
                      <div class="msg-text">${escapeHTML(text)}</div>
                      </div>
                  </div>
              `;
  
              chatContainer?.insertAdjacentHTML("beforeend", msgHTML);
              chatContainer.scrollTop = chatContainer.scrollHeight;
          }
  
          function formatTime(date) {
              return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
          }
  
          function escapeHTML(str) {
              const div = document.createElement("div");
              div.textContent = str;
              return div.innerHTML;
          }
      });
    </script>
  </head>
  
  <body>
    <section class="msger" aria-label="Chat interface">
      <header class="msger-header">
        <h1 class="msger-header-title" aria-label="Chat title">
          <i class="fas fa-comment-alt" aria-hidden="true"></i>
          <span>Chat with your orchestrator</span>
        </h1>
      </header>
  
      <main class="msger-chat" aria-live="polite" role="log">
        <!-- Chat messages dynamically appear here -->
      </main>
  
      <form class="msger-inputarea" aria-label="Send a message">
        <input
          id="messageInput"
          name="message"
          type="text"
          class="msger-input"
          placeholder="Ask me about a current event..."
          required
          autocomplete="off"
          aria-label="Type your message"
        />
        <button id="sendButton" class="msger-send-btn" type="submit">Send</button>
      </form>
    </section>
  </body>
  
  </html>
  ```

  

## 5. 참고 소스

* [openai-end-to-end-basic](https://github.com/Azure-Samples/openai-end-to-end-basic)

