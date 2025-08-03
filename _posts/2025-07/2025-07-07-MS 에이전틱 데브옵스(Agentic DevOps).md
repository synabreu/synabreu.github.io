---
title: "MS 에이전틱 데브옵스(Agentic DevOps)"
date: 2025-07-07
tags: [마이크로소프트, Microsoft, Build 2025, Azure AI Foundry, Azure, Azure AI Foundry SDK, Azure OpenAI Studio, Azure OpenAI Service, Azure Machine Learning, Azure App Service, Azure Key Vault, Azure Monitor, Agentic DevOps, Github Copilot, DevOps, MLOps, Software Factory]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

Microsoft의 50 주년을 기념하면서 처음 가졌던 비전이 '소프트웨어 공장(Software Factory)' 라고 한다. Microsoft는 처음부터 개발자에 의해, 개발자를 위해 만들어진 회사다. 이제 51 번 째 해를 맞이하며 마이크로소프트는 그 소프트웨어 공장이 어떤 모습이어야 하는지를 새롭게 상상하고 있으며, 그 중심에는 바로 사용자가 있다고 강조했다. 

에이전틱 데브옵스도 이러한 사용자 중심의 문화와 관치 않다. 왜냐하면, 이번에 깃허브 코파일럿(GitHub Copilot)에 새로운 코딩 에이전트가 추가되었다.  기획부터 프로덕션까지 소프트웨어 개발의 전 과정이 지능형 에이전트를 통해 새롭게 재구성되고 있으며, 그 중심에는 사용자가 있다. 지난 마이크로소프트 빌드 2025 행사에서는 소프트웨어 개발 분야에서 일어나고 있는 큰 변화를 다루고, 함께 만들어 갈 미래를 공개했다. 

하지만 애플리케이션과 시스템이 점점 더 복잡해지고, 요구사항은 늘어나며, 기술 부채가 쌓이면서 그 마법은 희미해지기 시작한다. 코딩은 유지보수로 바뀌고, 개발의 흐름은 정체된다. 마이크로소프트는 이러한 굴레에서 벗어나 다시 개발의 즐거움, 몰입, 그리고 창조의 마법을 되찾을 수 있도록 돕고자 강조한다. 

그러한 깃허브 코파일럿은 지능형 에이전트가 개발 전 과정을 자동화하고 협업하는 차세대 데브옵스 방식을 '에이전틱 데브옵스(Agentic DevOps)' 라고 부른다. 비교적 새롭게 나온 용어이지만, 데브옵스나 ML옵스에 대한 개념의 이해나 운영 경험이 있다면 당황할 필요가 없다. 이번 노트에서는 AI 에이전트 중심의 진화된 데브옵스인 MS 에이전틱 데브옵스에 대해 정리해 보겠다. 



## 1. 새 코딩 에이전트 개발 배경

* 개발자가 된다는 것은 마치 마법사가 되는 것과 같다. 가능성을 상상하고, 그것을 코드로 현실화하는 일이다. 

* 그러나 애플리케이션과 시스템이 점점 더 복잡해지고, 요구 사항은 늘어나며, 기술 부채가 쌓이면서 그 마법은 희미해지기 시작한다.

* 코딩은 유지보수로 바뀌고, 개발의 흐름은 정체된다. 마이크로소프트는 이러한 굴레에서 벗어나 다시 개발의 즐거움, 몰입, 그리고 창조의 마법을 되찾을 수 있다. 

  

## 2. 에이전틱 워크플로우 등장

* [깃허브 코파일럿](https://github.com/features/copilot)은 이미 개발자가 코딩하는 방식을 획기적으로 바꾸었고, 그 결과 1,500만 명의 개발자들이 더 빠르게 개발하기 위해 이를 사용하고 있다. 그러나 새로운 코드를 작성하는 일은 개발 업무의 아주 일부분일 뿐이다. 
* 개발자의 하루는 시스템 설계, 문서 탐색, 디버깅, 리팩토링, 그리고 레거시 코드와의 씨름으로 가득 차 있다.
* 아이디어에서 프로덕션까지 더 빠르게 도달하도록 도와주고, 코드 품질을 높이며, 협업을 가속화하고, 보안을 강화할 뿐만 아니라 이제는 기술 부채를 해결하고 애플리케이션이 프로덕션 환경에서 원활하게 작동하도록 지원한다.



## 3. 에이전틱 데브옵스 소개

* 에이전틱 데브옵스 정의
  * 지능형 에이전트들이 개발자와 서로 협력하며 소프트웨어 개발의 전 과정을 함께 수행하는, 진화된 DevOps 모델	
  * 지능형 에이전트들은 소프트웨어 수명 주기의 모든 단계를 자동화하고 최적화 -> 마치 지치지 않는 팀원들을 개발팀에 새롭게 합류시키는 것처럼, 이들은 버그 수정, 소규모 기능 개발, 문서 작성 등을 처리해주어 개발자는 가장 중요한 일에 집중할 수 있게 됨.
* 개발 방식 비교
  * 과거의 개발 방식은 느리고 긴 릴레이였다. 기획에 몇 주, 개발에 몇 달, 출시에는 분기 단위의 시간이 필요했다.
  * 그러나 이제는 아이디어가 몇 시간 만에 프로토타입으로 구현되고, 며칠 만에 프로덕션에 배포된다.
* 에이전틱 데브옵스의 핵심 
  * 개발 속도를 높이고, 백로그를 해소하며, 기술 부채를 없애고, 앱의 보안을 강화하며, 프로덕션 환경에서도 안정적인 운영을 가능하게 한다.
  * 든 과정에서 **개발자인 당신이 중심에 있다는 것**이다. 에이전트를 지휘하고, 그들의 제안을 승인하며, 다시 멋진 개발에 몰입할 수 있는 환경을 만드는 것



## 4. 깃허브 코파일럿

![그림1 - Github Copilot](/../images/2025-07/AgenticDevOps-01.jpg)

* 깃허브 코파일럿은 단순한 코드 자동 완성을 넘어, 이미 코드 리뷰를 수행하고, 보안 취약점을 탐지하며, 코드들이 프로덕션에 도달하기 전에 해결할 수 있도록 지원함

* [에이전트 모드](https://github.com/features/copilot)

  * GitHub Copilot의 기존 기능들을 한층 강화하여, 복잡하고 여러 단계로 이루어진 코딩 작업에 매끄럽고 협업적인 방식으로 적용함
  * 전체 코드베이스를 분석하고, 여러 파일에 걸쳐 수정하며, 테스트를 생성하고 실행하고, 버그를 수정하며, 터미널 명령어까지 하나의 프롬프트로 제안 가능함
  * [Microsoft Visual Studio Code, Visual Studio,](https://visualstudio.microsoft.com/) JetBrains, Eclipse, 그리고 Xcode 등 코드 편집기에서 깃허브 에이전트 모드로 지원

* [새로운 코딩 에이전트](https://github.com/features/copilot)

  * 가장 최신의 코딩 에이전트는 GitHub Copilot을 단순한 페어 프로그래머에서 팀 기능 제공
  * 코드 리뷰, 테스트 작성, 버그 수정, 전체 사양 구현 등 다양한 작업을 수행하며, 팀의 정식 구성원처럼 깃허브 코파일럿이 행동한다. 
  * 다른 에이전트들과 협업하여 개발과 운영 과정에서 더 복잡한 문제도 함께 해결할 수 있다.
  * 감사 로그(audit log)와 브랜치 보호 기능이 기본으로 내장되어 있어, 모든 제안된 변경 사항은 배포 전에 검토하는 기능도 포함됨
  * 반복적인 작업을 대신 처리해주는 가장 새로운 팀원으로, 개발자들이 정말 중요한 일에 집중할 수 있다.

* [새로운 애플리케이션 현대화](https://techcommunity.microsoft.com/blog/appsonazureblog/reimagining-app-modernization-for-the-era-of-ai/4414793?previewMessage=true)

  * 프로덕션 배포는 끝이 아니라 시작이며, 레거시 코드를 유지 관리한다고 해서 개발 속도가 느려져서는 안됨
  * GitHub Copilot은 이제 레거시 Java 및 .NET 애플리케이션 전반에 걸쳐 코드 평가, 종속성 업데이트, 문제 해결 작업을 처리함으로써 스택 현대화를 지원함. (메인프레임 현대화도 제공될 예정)
  * GitHub Copilot은 업데이트 계획을 자동으로 생성하고 실행하며, 변경 사항에 대한 전체 가시성과 제어, 명확한 요약 정보를 제공함. -> 그 결과, 더 안전하고 유지 관리가 용이하며 비용 효율적인 애플리케이션이 탄생함
  * 기술 부채에 짓눌리지 말고, 그 부채를 해결해 나가라. -> 과거를 정리하느라 발목 잡히지 않을 때, 진정한 혁신을 위한 길이 열리고 다음을 위한 개발에 집중할 수 있음

* [새 Azure 사이트 신뢰성 엔지니어링(SRE) 에이전트](https://techcommunity.microsoft.com/blog/azurepaasblog/introducing-azure-sre-agent/4414569)

  * 개발자들은 개발에 집중하고, 문제는 코파일러 에이전트가 유지한다는 개념
  * 새로운 Azure용 SRE 에이전트는 프로덕션 시스템을 24시간 모니터링하고, 실시간으로 사고에 대응하며, 문제가 발생하면 자동으로 문제를 해결해줌으로써 개발자들이 심야 알림의 스트레스로부터 자유로워질 수 있도록 돕는다.
  * SRE 에이전트는 [Azure Kubernetes Service](https://azure.microsoft.com/en-us/products/kubernetes-service), [Azure App Service](https://azure.microsoft.com/en-us/products/app-service), 서버리스, 데이터베이스 환경 전반의 상태와 성능을 지속적으로 추적하며, 전 세계적으로 Azure 기반 서비스를 운영해온 Microsoft의 깊은 운영 지식을 기반으로 동작함.
  * SRE 에이전트는 문제의 근본 원인을 함께 분석하거나 자동으로 해결함으로써 빠른 문제 해결을 가능하게 함.
  * 복구 작업이 수행되거나 수리 항목이 식별되면, 해당 내용이 GitHub 이슈로 자동 기록되어 팀이 후속 조치를 취하고 문제를 완전히 종결할 수 있도록 돕는다.
  * 더 빠른 복구, 줄어든 야간 호출, 자가 복구 능력을 갖춘 시스템 덕분에 애플리케이션은 더욱 탄탄해지고, 팀은 휴식을 취하며 명료한 상태로 다음을 준비할 수 있다.

* [깃허브 모델(Github Models)](https://docs.github.com/en/github-models)

  * 깃허브 모델은 AI 애플리케이션 개발을 더욱 쉽게 만들어 줌

  * GitHub Copilot이 개발자가 코드를 작성하는 방식을 바꿨다면, Azure AI Foundry는 개발자가 무엇을 만들 수 있는지를 바꾸고 있다. -> GitHub 워크플로우 안에서도 깃허브 모델을 직접 활용 가능해짐.

  * 새로운 네이티브 통합을 통해 OpenAI, Meta, Microsoft, Mistal, Cohere 등 다양한 최신 모델을 GitHub 내에서 직접 실험할 수 있다.

  * 성능과 가격을 나란히 비교하고, 애플리케이션이나 에이전트에 가장 적합한 모델을 선택하며, 단순하고 통합된 API로 모델을 손쉽게 교체할 수 있다.

  * GitHub Action에서 직접 모델이나 에이전트를 호출하여 오프라인 평가 과정을 간소화하거나, 현재 열려 있는 이슈들을 요약하는 데에도 활용할 수 있다.

  * 내장된 엔터프라이즈용 가이드라인 덕분에 모델 선택은 보안성, 책임성, 팀 정책과의 정합성을 유지한다.

  * 개발자의 일상은 에이전트가 단순히 보조하는 것을 넘어서, 주도적으로 개입하고, 빠르게 대응하며, 어떤 시간에도 시스템을 안정적으로 운영하는 새로운 시대에 접어들었다.

    ``

    <iframe width="560" height="315"
            src="https://www.youtube.com/embed/7fUVPrySVwI"
            title="Agentic DevOps in Action"
            frameborder="0"
            allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
            allowfullscreen>
    </iframe>

* [Visual Studio Code에서의 GitHub Copilot을 오픈소스로 공개](https://code.visualstudio.com/blogs/2025/05/19/openSourceAIEditor)

  * GitHub Copilot이 Visual Studio Code에서 진화하면서, AI는 테스트부터 보안까지 개발자 경험의 핵심 요소로 자리 잡았다.
  * GitHub Copilot 확장 기능의 AI 기반 기능들이 이제 세계에서 가장 인기 있는 개발 도구를 구동하는 **동일한 오픈소스 저장소**의 일부가 된다. 
  * 투명성, 커뮤니티 주도의 혁신, 그리고 **AI 기반 개발의 미래를 함께 만들어 갈 수 있도록 개발자에게 더 큰 목소리를 부여하겠다는 마이크로소프트의 의지**를 반영한 것




## 5. 결론

* AI 에이전트가 중심이 되는 이 새로운 소프트웨어 개발 시대가 **클라우드로의 전환만큼이나 혁신적일 것**이라고 믿음
* AI 에이전트 변화는 마찰을 줄이고 복잡성을 낮추며, 수십 년간 팀의 발목을 잡아왔던 비용 구조를 근본적으로 재설계한다.
* 무엇보다 중요한 것은, 개발자가 자주 이야기하지 않는 한 가지를 되돌려준다는 점이 바로 **"즐거움"**이다.
* 백로그를 비우고, 반복적인 일을 내려놓고, 개발자가 진심으로 만들고 싶어 하는 것에 집중할 때, 개발자들이 단순히 개발 속도만 높이는 것이 아니라, **우리가 오랫동안 꿈꿔온 미래를 직접 열어가게 된다.**



## 6. 더 살펴 볼 것

* [BRK100](https://build.microsoft.com/en-US/sessions/BRK100?source=sessions): Reimagining Software Development and DevOps with Agentic AI
* [BRK113](https://build.microsoft.com/en-US/sessions/BRK113?source=sessions): The Agent Awakens: Collaborative Development with GitHub Copilot
* [BRK118](https://build.microsoft.com/en-US/sessions/BRK118?source=sessions): Accelerate Azure Development with GitHub Copilot, VS Code & AI
* [BRK131](https://build.microsoft.com/en-US/sessions/BRK131?source=sessions): Java App Modernization Simplified with AI
* [BRK102](https://build.microsoft.com/en-US/sessions/BRK102?source=sessions): Agent Mode in Action: AI Coding with Vibe and Spec-Driven Flows
* [BRK101](https://build.microsoft.com/en-US/sessions/BRK101?source=sessions): The Future of .NET App Modernization Streamlined with AI



## 7. 참고 블로그

* [에이전트 기반 워크플로우(agentic workflows)](https://github.blog/ai-and-ml/github-copilot/copilot-ask-edit-and-agent-modes-what-they-do-and-when-to-use-them/)
* [GitHub Copilot: Meet the new coding agent](https://github.blog/news-insights/product-news/github-copilot-meet-the-new-coding-agent/)
* [Reimagining App Modernization for the Era of AI](https://techcommunity.microsoft.com/blog/appsonazureblog/reimagining-app-modernization-for-the-era-of-ai/4414793?previewMessage=true)
* [Introducing Azure SRE Agent](https://techcommunity.microsoft.com/blog/azurepaasblog/introducing-azure-sre-agent/4414569)
* [Reinventing Software Development](https://techcommunity.microsoft.com/blog/AppsonAzureBlog/building-the-agentic-future/4414743)