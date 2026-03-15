### Context
개발 중 습득한 단편적인 지식이나 메모를 체계적으로 정리하기 위해, Discord를 입력 창구로 활용하고 GitHub 레포지토리에 자동으로 TIL(Today I Learned) 문서를 생성 및 인덱싱하는 자동화 워크플로우를 구축했습니다. n8n의 노드 기반 설계를 활용하여 수동 작업 없이 메시지 전송만으로 문서화 전 과정을 자동화하는 것이 핵심 목표입니다.

### Core
이 워크플로우는 n8n의 트리거, AI 에이전트, 그리고 GitHub API 노드를 연쇄적으로 연결하여 작동합니다.

* **Discord Trigger Node**: 특정 채널의 메시지를 실시간으로 감지하여 데이터를 수신합니다.
* **AI Agent Node (LLM)**: 수신된 비정형 텍스트를 분석하여 정해진 TIL 구조(Context, Core, Insight)로 변환합니다.
* **GitHub Node (File Push)**: 생성된 마크다운 내용을 `YYYY-MM-DD-제목.md` 형식의 파일로 지정된 레포지토리에 푸시합니다.
* **README Indexing Logic**: 
    * GitHub 노드로 기존 `README.md` 파일을 호출합니다.
    * 'Edit Fields' 노드를 통해 파일 하단 또는 인덱스 섹션에 새 TIL 파일 링크를 추가합니다.
    * 업데이트된 내용을 다시 GitHub에 덮어쓰기(Update) 합니다.
* **Discord Notification**: 모든 작업 완료 후, 처리 결과와 GitHub 링크를 사용자에게 다시 전송합니다.

```json
// n8n 워크플로우 개념 구조 예시
{
  "nodes": [
    { "type": "n8n-nodes-base.discordTrigger", "name": "Discord Message" },
    { "type": "n8n-nodes-base.aiAgent", "params": { "prompt": "Convert this to TIL format..." } },
    { "type": "n8n-nodes-base.github", "params": { "operation": "create", "filePath": "={{$now.format('YYYY-MM-DD')}}-{{$json.title}}.md" } },
    { "type": "n8n-nodes-base.github", "params": { "operation": "update", "filePath": "README.md" } },
    { "type": "n8n-nodes-base.discord", "params": { "content": "TIL Successfully Updated!" } }
  ]
}
```

### Insight
n8n을 처음 사용해보았으나, 프로그래밍의 기본인 입력(Input)과 출력(Output)의 흐름만 이해하고 있다면 개발자에게 이보다 강력한 도구는 없을 것이라 판단됩니다. 코드를 직접 작성하는 대신 노드를 연결하는 것만으로 복잡한 API 연동을 해결할 수 있어 생산성이 극대화됩니다. 특히 AI Agent 노드를 중간에 배치함으로써 단순 데이터 전달을 넘어 '지식의 구조화'까지 자동화할 수 있다는 점이 가장 큰 강점입니다. 향후 다양한 외부 API(Notion, Slack 등)와 결합하여 개인용 기술 비서를 고도화할 계획입니다.

**출처:** 
* [n8n Discord Integration Guide](https://n8n.io/integrations/discord/)
* [n8n GitHub Node Documentation](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.github/)
* [n8n AI Agent Node Overview](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.ai-agent/)