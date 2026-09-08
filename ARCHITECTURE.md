# RedwoodVest — Technical Architecture

**Version:** v0.2  
**Status:** Initial technical blueprint  
**Primary platform:** Telegram  
**Primary timezone:** Asia/Taipei  
**Core stack:** Node.js + Telegraf + SQLite + Scheduler  
**Design goal:** 保留原本可用架構，逐步重構成資料可信、可擴充、多平台的投資決策助手
**Official brand / repository:** RedwoodVest
**Website:** redwoodvest.com
**Official proprietary score:** Redwood Signal Score

---

# 1. Architecture Goals

RedwoodVest 的架構必須同時滿足以下目標：

1. **保留原本 Node.js + Telegraf + SQLite 架構**
2. **避免整個重寫**
3. **市場資料與 AI 完全分離**
4. **所有市場數據都必須來自真實 API**
5. **AI 只能解讀，不得生成市場數字**
6. **Telegram 第一版先服務 Maggie 自己**
7. **未來可擴充到 LINE / WhatsApp / Discord / Slack / Web / App**
8. **任何單一 API 掛掉，不得讓整個 Bot 掛掉**
9. **所有排程都明確使用 Asia/Taipei**
10. **Tesla 為最高優先追蹤標的**
11. **Meta 為固定追蹤標的**
12. **開發順序必須分階段，每階段可驗收**

---

# 2. High-Level Architecture

```text
                        ┌─────────────────────┐
                        │      Telegram       │
                        └──────────┬──────────┘
                                   │
                                   ▼
                        ┌─────────────────────┐
                        │   Bot Adapter Layer │
                        │    Telegraf.js      │
                        └──────────┬──────────┘
                                   │
                                   ▼
                        ┌─────────────────────┐
                        │ Application Layer   │
                        │ Commands / Reports  │
                        │ Watchlist / Alerts  │
                        └──────────┬──────────┘
                                   │
                 ┌─────────────────┼─────────────────┐
                 │                 │                 │
                 ▼                 ▼                 ▼
       ┌────────────────┐ ┌────────────────┐ ┌────────────────┐
       │ Market Data    │ │ Options Data   │ │ Event Data     │
       │ Providers      │ │ Providers      │ │ Providers      │
       └───────┬────────┘ └───────┬────────┘ └───────┬────────┘
               │                  │                  │
               └──────────────┬───┴──────────────┬───┘
                              │                  │
                              ▼                  ▼
                    ┌──────────────────┐  ┌──────────────────┐
                    │ Normalization    │  │ Data Validation  │
                    │ Layer            │  │ Freshness Check  │
                    └────────┬─────────┘  └────────┬─────────┘
                             │                     │
                             └──────────┬──────────┘
                                        ▼
                              ┌──────────────────┐
                              │ Rules / Scoring  │
                              │ Engine           │
                              └────────┬─────────┘
                                       │
                                       ▼
                              ┌──────────────────┐
                              │ AI Summary Layer │
                              │ No data creation │
                              └────────┬─────────┘
                                       │
                                       ▼
                              ┌──────────────────┐
                              │ Formatter Layer  │
                              │ Telegram output  │
                              └──────────────────┘
```

---

# 3. Core Design Principle

## 3.1 AI 必須在資料層之後

錯誤架構：

```text
Telegram -> AI -> AI 自己猜資料 -> 回覆
```

正確架構：

```text
Telegram
   ↓
Real API
   ↓
Normalized JSON
   ↓
Validation
   ↓
Rule Engine
   ↓
AI Summary
   ↓
Telegram
```

AI 永遠不能直接負責「產生市場資料」。

---

# 4. Recommended Repository Structure

```text
RedwoodVest/
│
├── README.md
├── PRD.md
├── ARCHITECTURE.md
├── ROADMAP.md
├── VISION.md
├── CLAUDE.md
├── DECISIONS.md
├── .env.example
├── .gitignore
├── package.json
│
├── src/
│   │
│   ├── index.js
│   │
│   ├── app.js
│   │
│   ├── config/
│   │   ├── env.js
│   │   ├── constants.js
│   │   └── featureFlags.js
│   │
│   ├── bot/
│   │   ├── telegram/
│   │   │   ├── telegramBot.js
│   │   │   ├── commands.js
│   │   │   ├── handlers.js
│   │   │   └── formatter.js
│   │   │
│   │   └── adapters/
│   │       └── botAdapter.js
│   │
│   ├── application/
│   │   ├── marketBriefService.js
│   │   ├── stockAnalysisService.js
│   │   ├── watchlistService.js
│   │   ├── earningsAlertService.js
│   │   ├── weeklyReportService.js
│   │   ├── dayTradeService.js
│   │   └── actionListService.js
│   │
│   ├── services/
│   │   ├── market/
│   │   │   ├── marketDataService.js
│   │   │   ├── polygonProvider.js
│   │   │   ├── twelveDataProvider.js
│   │   │   └── alphaVantageProvider.js
│   │   │
│   │   ├── options/
│   │   │   ├── optionsService.js
│   │   │   ├── maxPainService.js
│   │   │   └── optionsProvider.js
│   │   │
│   │   ├── news/
│   │   │   ├── newsService.js
│   │   │   └── newsProvider.js
│   │   │
│   │   ├── earnings/
│   │   │   ├── earningsService.js
│   │   │   └── earningsProvider.js
│   │   │
│   │   ├── congress/
│   │   │   ├── congressService.js
│   │   │   └── congressProvider.js
│   │   │
│   │   ├── darkpool/
│   │   │   ├── darkPoolService.js
│   │   │   └── darkPoolProvider.js
│   │   │
│   │   └── global/
│   │       ├── globalMarketService.js
│   │       └── globalMarketProvider.js
│   │
│   ├── domain/
│   │   ├── normalizers/
│   │   │   ├── quoteNormalizer.js
│   │   │   ├── optionsNormalizer.js
│   │   │   └── newsNormalizer.js
│   │   │
│   │   ├── validators/
│   │   │   ├── marketDataValidator.js
│   │   │   ├── freshnessValidator.js
│   │   │   └── completenessValidator.js
│   │   │
│   │   └── models/
│   │       ├── StockQuote.js
│   │       ├── MarketSnapshot.js
│   │       ├── OptionsSnapshot.js
│   │       └── AnalysisPayload.js
│   │
│   ├── scoring/
│   │   ├── momentumScore.js
│   │   ├── weaknessScore.js
│   │   ├── liquidityScore.js
│   │   ├── volatilityScore.js
│   │   ├── riskScore.js
│   │   └── opportunityScore.js
│   │
│   ├── ai/
│   │   ├── aiClient.js
│   │   ├── summarizer.js
│   │   ├── promptBuilder.js
│   │   ├── responseValidator.js
│   │   └── prompts/
│   │       ├── dailyBriefPrompt.js
│   │       ├── stockAnalysisPrompt.js
│   │       ├── weeklyPrompt.js
│   │       └── dayTradePrompt.js
│   │
│   ├── reports/
│   │   ├── dailyBrief.js
│   │   ├── weeklyRadar.js
│   │   ├── teslaReport.js
│   │   ├── metaReport.js
│   │   ├── magnificentSevenReport.js
│   │   ├── marketSummaryReport.js
│   │   └── dayTradeReport.js
│   │
│   ├── scheduler/
│   │   ├── scheduler.js
│   │   ├── dailyJobs.js
│   │   ├── earningsJobs.js
│   │   └── weeklyJobs.js
│   │
│   ├── database/
│   │   ├── database.js
│   │   ├── migrations.js
│   │   └── repositories/
│   │       ├── watchlistRepository.js
│   │       ├── alertRepository.js
│   │       ├── cacheRepository.js
│   │       └── settingsRepository.js
│   │
│   ├── cache/
│   │   └── cacheService.js
│   │
│   ├── observability/
│   │   ├── logger.js
│   │   ├── metrics.js
│   │   └── errorReporter.js
│   │
│   └── utils/
│       ├── time.js
│       ├── numbers.js
│       ├── retry.js
│       └── rateLimit.js
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── fixtures/
│
└── data/
    └── bot.db
```

---

# 5. Application Layer

Application Layer 是整個系統的「產品邏輯」。

它不應該知道 Polygon、Twelve Data、OpenAI 等供應商細節。

例如：

```text
marketBriefService
```

只需要知道：

```text
取得市場資料
取得 Tesla 資料
取得 Meta 資料
取得七巨頭
取得 watchlist
取得重大新聞
取得財報事件
取得 options 資料
計算 scoring
送給 AI 摘要
格式化
推送
```

它不應該直接寫：

```js
axios.get("https://api.polygon.io/...")
```

這類 provider-specific 程式碼。

---

# 6. Provider Layer

## 6.1 MarketDataProvider Interface

統一介面：

```js
class MarketDataProvider {
  async getQuote(symbol) {}
  async getQuotes(symbols) {}
  async getIntraday(symbol, interval) {}
  async getDailyHistory(symbol, days) {}
  async getIndices() {}
}
```

實作：

```text
PolygonProvider
TwelveDataProvider
AlphaVantageProvider
```

Application Layer 只使用：

```js
marketDataService.getQuote("TSLA")
```

而不在意資料究竟來自哪一家。

---

# 7. Provider Priority and Fallback

建議市場資料 priority：

```text
Primary: Polygon
Fallback 1: Twelve Data
Fallback 2: Alpha Vantage
```

Fallback 規則：

```text
1. Primary API 成功且資料新鮮 → 使用
2. Primary 失敗 → Fallback 1
3. Fallback 1 失敗 → Fallback 2
4. 全部失敗 → unavailable
5. 絕對禁止 mock value
```

範例：

```json
{
  "symbol": "TSLA",
  "price": null,
  "status": "unavailable",
  "reason": "All providers failed"
}
```

---

# 8. Normalized Data Model

所有 provider 回傳資料都先標準化。

範例：

```json
{
  "symbol": "TSLA",
  "price": 0,
  "open": 0,
  "high": 0,
  "low": 0,
  "previous_close": 0,
  "change": 0,
  "change_percent": 0,
  "volume": 0,
  "relative_volume": 0,
  "market_timestamp": "",
  "fetched_at": "",
  "provider": "polygon",
  "freshness": "fresh"
}
```

不能讓各個 report 自己處理不同 provider 的欄位格式。

---

# 9. Data Freshness Policy

每筆資料都必須知道「這是什麼時間的資料」。

建議：

### Real-time / Near Real-time

```text
fresh <= 5 minutes
stale <= 30 minutes
expired > 30 minutes
```

### Daily data

```text
fresh = same trading day
stale = previous trading day
```

UI 顯示：

```text
TSLA $xxx.xx
資料時間：09:42 ET
來源：Polygon
```

如果 stale：

```text
⚠️ 資料非即時
```

---

# 10. Watchlist Architecture

第一版 SQLite 即可。

## Table: watchlist

```sql
CREATE TABLE watchlist (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL,
  symbol TEXT NOT NULL,
  priority INTEGER DEFAULT 0,
  pinned BOOLEAN DEFAULT 0,
  notes TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(user_id, symbol)
);
```

預設：

```text
TSLA priority = 100
META priority = 90
```

所有報告：

```text
Tesla -> first
Meta -> fixed watch
Other watchlist -> priority descending
```

---

# 11. User Settings

雖然 Phase 1 只有 Maggie，也要先保留設定表。

## Table: user_settings

```sql
CREATE TABLE user_settings (
  user_id INTEGER PRIMARY KEY,
  timezone TEXT DEFAULT 'Asia/Taipei',
  daily_brief_enabled BOOLEAN DEFAULT 1,
  daily_brief_time TEXT DEFAULT '07:00',
  weekly_report_enabled BOOLEAN DEFAULT 1,
  daytrade_interval_primary TEXT DEFAULT '1m',
  daytrade_interval_secondary TEXT DEFAULT '3m'
);
```

未來多使用者時不用重寫。

---

# 12. Scheduler Architecture

使用 node-schedule 或其他穩定 scheduler。

所有 job 必須：

```text
explicit timezone = Asia/Taipei
```

---

# 13. Scheduler Jobs

## Phase 1

### Daily Overnight Brief

```text
07:00 Asia/Taipei
```

執行：

```text
fetch global market
fetch US indices
fetch TSLA
fetch META
fetch Mag7
fetch watchlist
fetch major news
fetch earnings events
calculate rankings
generate daily action list
AI summarize
send Telegram
```

### Earnings Reminder

建議：

```text
每天 08:00
每天 20:00
```

檢查：

```text
watchlist earnings in next 24h
```

### Weekly Radar

建議預設：

```text
週日 20:00 Asia/Taipei
```

時間之後可調整。

---

# 14. Report Generation Pipeline

每一份 report 走相同流程：

```text
Raw API
   ↓
Normalized Data
   ↓
Validation
   ↓
Scoring
   ↓
Structured Payload
   ↓
AI Summary
   ↓
Response Validation
   ↓
Formatter
   ↓
Telegram
```

---

# 15. Daily Brief Payload

AI 不直接讀 API raw response。

AI 只接受結構化 payload。

例如：

```json
{
  "report_type": "daily_brief",
  "generated_at": "",
  "global_market": {},
  "tesla": {},
  "meta": {},
  "magnificent_seven": [],
  "bullish_candidates": [],
  "bearish_candidates": [],
  "high_volume_candidates": [],
  "earnings_today": [],
  "risk_events": [],
  "sources": []
}
```

---

# 16. Scoring Engine

AI 不負責選股第一階段。

先由 deterministic rules 做篩選。

---

# 17. Momentum Score

可使用：

```text
price change
relative volume
distance from VWAP
distance from moving averages
intraday breakout
sector relative strength
options confirmation
news catalyst
```

輸出：

```text
0–100
```

---

# 18. Weakness Score

可使用：

```text
negative return
high sell volume
relative weakness
breakdown
negative catalyst
bearish options
sector weakness
```

輸出：

```text
0–100
```

---

# 19. Liquidity Score

對 Maggie 的 1m / 3m 短線非常重要。

可使用：

```text
Dollar Volume
Average Daily Volume
Relative Volume
Bid/Ask spread
Options liquidity
Intraday range
```

低流動性標的不得出現在「可操作觀察池」前段。

---

# 20. Opportunity Score

建議：

```text
Opportunity Score =
Momentum
+ Liquidity
+ Catalyst
+ Options confirmation
- Risk penalty
```

不是「預測上漲機率」。

只是：

```text
今天值得花多少注意力
```

---


# 20A. Redwood Signal Score Architecture

**Redwood Signal Score** 是 RedwoodVest 唯一正式對外的綜合評分名稱。

建議架構：

```text
Validated Market Data
        +
Internal Quant Components
        +
Risk / Freshness Penalties
        ↓
RedwoodSignalScoreEngine
        ↓
score + confidence + reasons
```

建議輸出資料模型：

```json
{
  "symbol": "TSLA",
  "score": 82,
  "confidence": "high",
  "calculated_at": "",
  "data_freshness": "fresh",
  "top_factors": [],
  "risk_factors": []
}
```

重要架構規則：

1. `RedwoodSignalScoreEngine` 必須是 deterministic service。
2. AI 只能解釋 Score，不可生成 Score。
3. 完整權重、閾值與公式只存在 server-side。
4. Telegram / Web / LINE / WhatsApp 不接收完整 internal feature vector。
5. API response 僅回傳 UI 所需的 score、confidence 與有限 explanation。
6. production logs 不應完整輸出 proprietary weights。
7. future public API 若存在，也不可暴露完整公式。
8. Score version 必須可追蹤，例如 `score_version: "rss-v1"`，避免未來公式更新後無法回測。
9. 每次 Score 必須可以回溯到 input timestamp 與 provider provenance。
10. 歷史 Redwood Signal Score 應持續保存，未來可用於回測、產品研究與個人化。

建議檔案：

```text
src/scoring/
  redwoodSignalScore.js
  components/
    momentum.js
    liquidity.js
    volatility.js
    catalyst.js
    risk.js
```

內部 component 名稱不需要作為外部品牌名稱；對使用者正式呈現一律使用 **Redwood Signal Score**。

---

# 21. Tesla Priority Engine

Tesla 必須是獨立 service / report，不要只當普通 watchlist。

```text
TeslaPriorityService
```

永遠執行：

```text
market
intraday
volume
news
earnings
options
max pain
risk
major levels
```

如果 Tesla provider 資料失敗：

```text
仍保留 Tesla 卡片
但顯示 unavailable
```

不得跳過 Tesla。

---

# 22. Meta Fixed Watch

Meta 類似 Tesla，但資訊量可低一級。

```text
MetaFixedWatchService
```

---

# 23. Options Architecture — Phase 2

Options 模組獨立。

```text
OptionsService
```

統一輸出：

```json
{
  "symbol": "TSLA",
  "expiration": "",
  "put_call_ratio": null,
  "iv": null,
  "max_pain": null,
  "largest_call_oi": [],
  "largest_put_oi": [],
  "unusual_flow": [],
  "provider": "",
  "fetched_at": ""
}
```

---

# 24. Max Pain Calculation

兩種模式：

### A. Provider 直接提供

```text
可信 provider value
```

### B. 本地計算

使用完整 option chain：

```text
strike
call open interest
put open interest
```

計算每個 strike 作為到期價時，全市場 option holder 的總 intrinsic payout。

使總 payout 最低的 strike：

```text
Max Pain
```

計算過程必須 deterministic。

AI 不參與計算。

---

# 25. Day Trade Architecture — Phase 2

指令：

```text
/daytrade TSLA
```

資料：

```text
1m candles
3m candles
volume
VWAP
relative volume
opening range
intraday high/low
options
Max Pain
IV
major OI strikes
```

流程：

```text
Market Data
↓
Intraday Indicators
↓
Options Positioning
↓
Rules
↓
AI Explanation
```

AI 只能說明：

```text
偏多
偏空
盤整
```

以及理由。

---

# 26. Congress Architecture — Phase 3

Congress 是「disclosure tracker」，不是即時資金流。

資料模型：

```json
{
  "politician": "",
  "symbol": "",
  "transaction_type": "",
  "transaction_date": "",
  "disclosure_date": "",
  "amount_range": "",
  "source": ""
}
```

必須同時顯示：

```text
Transaction Date
Disclosure Date
```

避免使用者誤以為是當天交易。

---

# 27. Dark Pool Architecture — Phase 3

Dark Pool 只呈現：

```text
trade size
notional
price
time
venue / source if available
repeated blocks
relative to current price
```

不能直接轉成：

```text
institution is buying
institution is selling
```

除非來源真的有明確方向資料。

---

# 28. Global Market Architecture

獨立 service：

```text
GlobalMarketService
```

統一輸出：

```json
{
  "us": {},
  "taiwan": {},
  "japan": {},
  "hong_kong": {},
  "china": {},
  "europe": {},
  "fx": {},
  "rates": {},
  "commodities": {},
  "crypto": {}
}
```

Telegram 只顯示摘要。

Web 未來才顯示完整 detail。

---

# 29. News Architecture

新聞不可讓 AI 自己「回想」。

NewsProvider 必須先抓資料。

每則：

```json
{
  "headline": "",
  "source": "",
  "published_at": "",
  "url": "",
  "symbols": [],
  "category": ""
}
```

AI 只摘要提供進來的新聞。

---

# 30. Earnings Architecture

Earnings provider 回傳：

```json
{
  "symbol": "",
  "report_date": "",
  "report_time": "",
  "eps_estimate": null,
  "revenue_estimate": null,
  "actual_eps": null,
  "actual_revenue": null,
  "guidance": null,
  "source": ""
}
```

缺少欄位：

```text
null
```

不能填估計值。

---

# 31. AI Layer

AI Layer 只負責：

```text
summarize
compare
explain
rank explanations
detect conflicting signals
```

不負責：

```text
market data
option calculations
financial calculations
event dates
prices
volumes
```

---

# 32. AI System Contract

所有 prompt 最前面固定加入：

```text
You are an investment research summarizer.

You may ONLY use facts contained in the supplied structured payload.

You must NEVER invent:
- prices
- percentages
- volumes
- market events
- earnings dates
- analyst ratings
- options values
- Max Pain
- Congress transactions
- Dark Pool trades

If information is missing, say:
"資料暫無" or "資料來源未提供".

Separate:
1. Facts
2. Interpretation

Never present interpretation as fact.
```

---

# 33. AI Response Validation

AI 回來後不能直接送 Telegram。

先經：

```text
responseValidator
```

至少檢查：

```text
是否出現 payload 裡不存在的 ticker
是否出現 payload 裡不存在的數字
是否出現保證性用語
是否出現不允許的 financial claim
```

如果 validator 失敗：

```text
fallback to rule-based summary
```

---

# 34. Rule-Based Fallback

AI 失敗時 Bot 仍然要能回覆。

例如：

```text
TSLA
價格：...
漲跌：...
成交量：...
Max Pain：...
主要新聞：...
```

這能避免：

```text
AI API 掛掉 = 整個 Bot 沒用
```

---

# 35. Database Architecture

Phase 1 保留 SQLite。

建議 tables：

```text
users
user_settings
watchlist
alerts
earnings_cache
market_cache
report_history
api_logs
job_logs
```

---

# 36. Market Cache

避免：

```text
同一分鐘重複打 API
```

建議：

```sql
CREATE TABLE market_cache (
  cache_key TEXT PRIMARY KEY,
  payload TEXT,
  provider TEXT,
  fetched_at DATETIME,
  expires_at DATETIME
);
```

---

# 37. Report History

保存每日報告。

```sql
CREATE TABLE report_history (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  report_type TEXT,
  generated_at DATETIME,
  payload TEXT,
  ai_summary TEXT,
  sent BOOLEAN,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

好處：

```text
debug
compare
追蹤 AI 是否出錯
未來做 dashboard
```

---

# 38. API Logging

每次 provider call 記：

```text
provider
operation
symbol
started_at
finished_at
latency
status
error
```

禁止 log：

```text
API key
Bot Token
secret
```

---

# 39. Error Handling

所有 provider 呼叫使用統一 wrapper。

流程：

```text
request
↓
timeout
↓
retry
↓
fallback
↓
unavailable
```

建議：

```text
timeout = 5–10 sec
retry = 1–2
```

避免無限 retry。

---

# 40. Rate Limit Handling

建立：

```text
rateLimitService
```

負責：

```text
per-provider request budget
backoff
queue
cache reuse
```

避免 Phase 1 就把免費 API 額度打爆。

---

# 41. Security

所有 secret：

```text
.env
```

例如：

```env
BOT_TOKEN=
MAGGIE_TELEGRAM_USER_ID=

POLYGON_API_KEY=
TWELVE_DATA_API_KEY=
ALPHA_VANTAGE_API_KEY=

AI_API_KEY=

OPTIONS_API_KEY=
CONGRESS_API_KEY=
DARK_POOL_API_KEY=
```

`.gitignore`：

```text
.env
bot.db
data/
logs/
node_modules/
```

---

# 42. Maggie-Only Access — Phase 1

第一版 Bot 僅接受 Maggie 的 Telegram User ID。

middleware：

```text
if ctx.from.id !== MAGGIE_TELEGRAM_USER_ID
    reject
```

回覆：

```text
Private beta.
```

未來 Phase 4 才打開多使用者。

---

# 43. Bot Adapter Layer

現在：

```text
TelegramAdapter
```

未來：

```text
LineAdapter
WhatsAppAdapter
DiscordAdapter
SlackAdapter
TeamsAdapter
EmailAdapter
WebAdapter
```

Application Layer 不應直接寫 Telegraf 格式。

應先產生：

```json
{
  "title": "",
  "sections": [],
  "actions": []
}
```

再由 Telegram formatter 轉成 Telegram message。

---

# 44. Multi-Platform Design

正確：

```text
                  Core Backend
                       |
        ┌──────────────┼──────────────┐
        │              │              │
 Telegram Adapter   LINE Adapter   WhatsApp Adapter
```

錯誤：

```text
Telegram Bot code
+
LINE Bot code
+
WhatsApp Bot code
各自重新寫一次所有資料邏輯
```

---

# 45. Feature Flags

尚未完成的功能：

```text
OPTIONS_ENABLED=false
CONGRESS_ENABLED=false
DARK_POOL_ENABLED=false
COMMUNITY_ENABLED=false
```

不要因為缺 API 而讓整個 repo 無法啟動。

---

# 46. Development Environment

建議 Node：

```text
Node.js 20 LTS+
```

package scripts：

```json
{
  "scripts": {
    "dev": "node --watch src/index.js",
    "start": "node src/index.js",
    "test": "node --test",
    "lint": "eslint src tests"
  }
}
```

---

# 47. Testing Strategy

## Unit Tests

測：

```text
RSI
VWAP
Max Pain
scoring
normalizer
validator
```

## Integration Tests

測：

```text
market provider
fallback provider
database
Telegram formatter
scheduler
```

## Acceptance Tests

每個 Milestone 都需跑。

---

# 48. No-Fake-Data Test

必須有專門測試：

```text
當所有 market providers 失敗
```

Expected：

```text
price = unavailable
```

Forbidden：

```text
price = random
price = cached without timestamp
price = AI generated
```

---

# 49. Phase 0 Migration from Existing Repo

原 repo：

```text
maggie_invests_bot
```

保持不動，作為 reference。

新 repo：

```text
RedwoodVest
```

只搬可用部分：

```text
Telegraf setup
SQLite patterns
stockService concepts
scheduler concepts
Telegram formatting patterns
```

不要搬：

```text
mock data
broken imports
missing services
unfinished dashboard
VIP/community logic in Phase 1
```

---

# 50. Phase 0 Required Refactor

優先順序：

```text
1. package.json
2. .env.example
3. correct imports
4. bot boot
5. market provider
6. /start
7. /quote TSLA
8. watchlist
9. scheduler
10. 07:00 brief
```

---

# 51. Milestone 1 — Clean Boot

Acceptance:

```text
npm install works
npm start works
bot connects
/start works
/quote TSLA returns real data
provider failure returns unavailable
no mock market data
```

完成前禁止進入 Phase 2。

---

# 52. Milestone 2 — Maggie Daily Workflow

Acceptance:

```text
Tesla priority works
Meta fixed watch works
S&P 500 search works
watchlist works
07:00 brief works
earnings reminders work
strong/weak list works
high-volume watch works
```

---

# 53. Milestone 3 — Options Intelligence

Acceptance:

```text
option chain works
Max Pain works
IV works
put/call works
OI works
/daytrade TSLA works
```

---

# 54. Milestone 4 — Alternative Data

Acceptance:

```text
Congress works
Dark Pool works
data dates are clearly labeled
no misleading interpretation
```

---

# 55. Architecture Rules for Claude

Claude 在開發時必須遵守：

1. 不可整個重寫
2. 不可自行更換核心技術棧
3. 不可新增 mock market data
4. 不可新增未經要求的大型框架
5. 不可將 provider logic 混進 bot command
6. 不可讓 AI 負責市場數字
7. 不可在沒有測試前一次修改大量檔案
8. 每次最多完成一個 milestone 子任務
9. 修改前先說明影響範圍
10. 修改後必須提供測試方法
11. 若資料缺失，回傳 unavailable
12. 若 provider 失敗，先 fallback
13. 所有時間都明確處理 timezone
14. Tesla 永遠最高優先
15. Meta 保持固定 watch
16. Phase 1 不恢復 VIP / community
17. Phase 1 不做 Dashboard
18. Phase 2 前不實作 Options
19. Phase 3 前不實作 Congress / Dark Pool
20. 不得將 TODO 標示為完成

---

# 56. Architecture Decision Summary

目前正式決策：

```text
Core backend:
Node.js

Bot:
Telegraf

Database:
SQLite first

Scheduler:
node-schedule first

Initial platform:
Telegram

Primary timezone:
Asia/Taipei

Initial user:
Maggie only

Highest priority symbol:
TSLA

Fixed secondary watch:
META

Market universe:
At least S&P 500

AI role:
Summary / reasoning only

AI market data generation:
Forbidden

Options:
Phase 2

Congress:
Phase 3

Dark Pool:
Phase 3

Community/VIP:
Phase 4

Multi-platform:
Phase 5
```

---


# 56A. Proprietary Logic / Anti-Copy Architecture

RedwoodVest 不應依賴「秘密 prompt」作為主要防禦。

真正需要保護的資產是：

```text
provider orchestration
normalized historical data
Redwood Signal Score logic
ranking rules
alert logic
user personalization
historical outcomes
```

技術原則：

- API keys 永遠只存在 server-side。
- proprietary scoring weights 不送到 client。
- normalized raw dataset 不應整包暴露給瀏覽器。
- Web / Bot adapters 只取得必要 presentation payload。
- authenticated endpoints 使用 authorization + rate limiting。
- 對異常大量讀取保留 audit log。
- 未來 RedwoodVest.com 應加入 bot / scraping protection。
- 每份報告可保留 report ID / user ID / generated timestamp 以利稽核。
- proprietary ranking / derived datasets 應在 Terms of Service 中禁止未授權再散布、scraping 與 AI/ML training。
- 歷史資料與 Score version 應保存在後端，形成長期資料資產。

這些機制是保護 RedwoodVest 的產品資產，不影響使用者正常取得其訂閱範圍內的研究資訊。

---

# 57. Final Architecture Principle

RedwoodVest 的核心不是：

```text
AI 很會講股票
```

而是：

```text
真實資料
+
正確時間
+
可靠來源
+
量化篩選
+
AI 幫 Maggie 快速理解
```

任何功能如果違反這個順序，都不應該進入正式版本。

