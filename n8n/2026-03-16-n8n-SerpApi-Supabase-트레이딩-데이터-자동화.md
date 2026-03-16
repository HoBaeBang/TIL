### Context
자동화된 트레이딩 시스템 구축 시 가장 큰 허들은 실시간 금융 데이터 수집과 뉴스 데이터의 구조화입니다. 기존의 웹 스크래핑 방식은 사이트 구조 변경에 취약하고 유지보수 비용이 높습니다. n8n과 SerpApi(Search Result API), 그리고 Supabase를 결합하면 복잡한 코딩 없이도 Google Finance의 시장 지표와 CNN Business의 뉴스 감성(Sentiment) 데이터를 실시간으로 수집하고 저장하는 견고한 ETL(Extract, Transform, Load) 파이프라인을 구축할 수 있습니다.

### Core
n8n 워크플로우는 크게 데이터 수집, 데이터 가공(Parsing), 저장의 3단계로 구성됩니다.

* **수집 (SerpApi 활용)**:
  * Google Finance Engine (`google_finance`): 특정 티커(예: AAPL:NASDAQ)의 가격, 변화율, 관련 뉴스를 구조화된 JSON으로 획득합니다.
  * Google News Engine (`google_news`): `q=CNN Business [Ticker]` 쿼리를 통해 최신 뉴스 헤드라인과 링크를 추출합니다.
* **가공 (n8n Code/Set 노드)**: 수집된 파편화된 JSON 데이터에서 필요한 필드(Price, Change %, Snippet 등)만 추출하여 정규화합니다.
* **저장 (Supabase 노드)**: 정규화된 데이터를 Supabase의 PostgreSQL 테이블에 `Upsert` 방식으로 기록합니다.

```javascript
// n8n Code Node (JavaScript) 예시: SerpApi 결과 파싱
const financeData = items[0].json.market_key_results;
const newsResults = items[0].json.news_results;

return [{
    ticker: "AAPL",
    current_price: financeData.price,
    price_change: financeData.change_percent,
    top_headline: newsResults[0].title,
    source: "CNN Business",
    collected_at: new Date().toISOString()
}];
```

### Insight
n8n을 활용한 저코드(Low-code) 접근 방식은 개발 속도와 시스템 안정성 측면에서 압도적인 우위를 점합니다.
* **유지보수 효율성**: SerpApi가 Google Finance의 레이아웃 변경을 대신 처리해주므로, 사용자는 스크래퍼 코드 수정 대신 데이터 전략에만 집중할 수 있습니다.
* **확장성**: Supabase를 사용함으로써 추후 Real-time 구독 기능을 통해 웹 대시보드나 텔레그램 알림 서비스로 즉각적인 확장이 가능합니다.
* **기술적 검증**: SerpApi는 단순 검색 결과뿐만 아니라 `google_finance` 전용 엔진을 통해 시장의 요약된 지표를 제공하므로, 뉴스 데이터와 수치 데이터를 결합한 복합 투자 지표 산출에 매우 적합합니다.

**출처:**
* [SerpApi Google Finance API Documentation](https://serpapi.com/google-finance-api)
* [n8n Supabase Integration Guide](https://supabase.com/partners/integrations/n8n)
* [n8n SerpApi Node Overview](https://n8n.io/integrations/serpapi/)