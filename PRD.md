# RedwoodVest — Product Requirements Document (PRD)

**Version:** v0.2  
**Status:** Draft for implementation  
**Primary owner:** Maggie  
**Initial platform:** Telegram  
**Core stack to preserve:** Node.js + Telegraf + SQLite + Scheduler  
**Primary timezone:** Asia/Taipei  
**Primary use case:** Maggie 先自用，穩定後再擴大成社群、會員網站與多平台產品
**Official brand:** RedwoodVest
**Website:** redwoodvest.com
**Official proprietary score:** Redwood Signal Score

---

## 1. 產品定位

RedwoodVest 不是單純的「查股價 Bot」，而是一個 **AI 驅動的美股投資決策助手**。

第一階段先服務單一使用者 Maggie，透過 Telegram 主動推送與指令查詢，將分散的市場資訊整理成可以快速閱讀、快速判斷、快速進入研究的決策介面。

產品核心不是讓 AI 猜市場，而是：

1. 先取得真實、可驗證的市場資料
2. 將不同來源的資料標準化
3. 用規則與量化條件做初步篩選
4. 最後才讓 AI 做摘要、歸納與解釋
5. 任何缺少資料的地方必須明確顯示「資料不足 / 暫無資料」，不得補寫

---

## 2. 產品目標

### 2.1 Phase 1 目標：Maggie 自用版

第一版必須先做到：

- 每天台北時間 07:00 自動推送隔夜美股摘要
- Tesla 固定為最高優先追蹤標的
- Meta 固定列入重點追蹤
- 可查詢至少 S&P 500 成分股
- 可建立個人 Watchlist
- Watchlist 財報日提醒
- 七巨頭快速整理
- 今日多空市場摘要
- 強勢股 / 弱勢股篩選
- 成交量較大的可交易觀察標的
- 交易策略視角：
  - 強勢追蹤
  - 逢低布局觀察
  - 平衡配置
- AI 決策摘要：
  - 長線關注
  - 短線觀望
  - 風險警告
  - 投資組合重點觀察
- 所有輸出都只可使用真實資料

### 2.2 長期目標

穩定後逐步擴充為：

- 多使用者 Telegram 社群
- 會員分級與 VIP
- LINE
- WhatsApp
- Discord
- Slack
- Microsoft Teams
- Email
- RedwoodVest.com 會員網站與個人 Dashboard
- Web Dashboard
- iOS / Android App

核心後端只維持一套，各平台只是不同入口。

---

## 3. 核心產品原則

### 3.1 Data First

AI 不得成為市場資料來源。

所有價格、漲跌幅、成交量、財報日、期權資料、Max Pain、國會交易、暗池、大盤指數、殖利率、VIX 等，都必須先由真實資料服務取得。

### 3.2 Fail Closed

當資料來源失敗時：

- 顯示資料暫時無法取得
- 保留上一筆資料時，必須明確標示資料時間
- 不得自動填入推測值
- 不得用 AI 補數字

### 3.3 Source + Timestamp

重要數據需保留：

- data source
- fetched_at
- market timestamp
- freshness status

### 3.4 AI 只負責解釋

AI 可以：

- 摘要
- 分類
- 比較
- 找出衝突訊號
- 說明可能原因
- 將多來源資料整合成自然語言

AI 不可以：

- 自行生成價格
- 自行生成成交量
- 自行生成 Max Pain
- 自行生成國會交易
- 自行生成暗池成交
- 把推測寫成事實
- 保證某檔股票會漲或跌

---


## 3.5 Redwood Signal Score

**Redwood Signal Score** 是 RedwoodVest 唯一正式對外使用的自有綜合評分名稱。

它的用途不是宣稱「上漲機率」，而是衡量某檔股票在特定時間點的 **研究與關注優先級**。

對外可以顯示：

```text
Redwood Signal Score: 82 / 100
```

內部可由多個可量化 component 組成，例如：

- price / momentum
- relative volume / liquidity
- volatility
- catalyst / event
- options positioning（Phase 2）
- risk
- sector / market context
- Congress / Dark Pool signals（Phase 3）

核心規則：

1. Score input 必須全部來自可驗證資料。
2. AI 不得直接決定或捏造 Score。
3. Score 必須由 deterministic code 計算。
4. 權重與完整公式屬於 RedwoodVest proprietary logic。
5. 前端、Telegram、LINE、WhatsApp 等 channel 只接收必要結果，不暴露完整公式。
6. Score 必須附 `calculated_at` 與必要的資料 freshness 狀態。
7. 若關鍵輸入缺失，系統應降低 confidence 或顯示資料不足，不可補值。
8. Score 只能作為研究排序與決策輔助，不可包裝成保證報酬或必然漲跌。

Redwood Signal Score 未來會成為 RedwoodVest 重要的產品差異化與歷史資料資產。

---

## 4. 初始使用者

### Primary User

**Maggie**

需求特徵：

- 主動關注美股
- Tesla 為最高優先標的
- Meta 固定追蹤
- 同時看長線、波段與短線
- 會參考期權相關指標
- 會做 1 分鐘 / 3 分鐘級別的短線交易觀察
- 希望早上快速知道隔夜發生什麼
- 不想自己打開很多不同網站找資料

---

## 5. Phase 1 功能需求

# 5.1 Telegram Bot 基礎

第一版先只服務 Maggie。

建議指令：

- `/start`
- `/help`
- `/market`
- `/quote TSLA`
- `/analyze TSLA`
- `/tesla`
- `/watchlist`
- `/add TSLA`
- `/remove TSLA`
- `/earnings`
- `/weekly`
- `/daytrade TSLA`

第一版不需要：

- 群組驗證
- 會員層級
- VIP
- 收款
- 推薦制度
- 社群審核
- Dashboard UI

---

# 5.2 每日 07:00 隔夜市場摘要

**執行時間：Asia/Taipei 07:00**

報告必須按照固定順序輸出。

## Section A — 全球市場雷達

至少包含：

- S&P 500
- Nasdaq
- Dow Jones
- Russell 2000（如資料可得）
- 日經
- 台股
- 恆生
- 上證
- 歐洲主要指數
- 美元指數
- 美國 10 年期公債殖利率
- VIX
- 黃金
- Bitcoin

最後產出：

**Global Risk Tone**
- Risk On
- Neutral
- Risk Off

並附一句理由。

---

## Section B — Tesla Priority Card

Tesla 必須永遠排在第一個。

至少包含：

- 最新價格
- 漲跌幅
- 成交量
- 相對平均成交量
- 盤前 / 盤後狀態（如資料可得）
- VWAP（如資料可得）
- RSI
- 近期重大新聞
- 下一次財報日
- 重大事件
- Options activity
- Put / Call
- Open Interest
- Implied Volatility
- Max Pain
- Gamma / dealer positioning（資料可得時）
- Dark Pool（Phase 3）
- Congress Trading（Phase 3）

並提供：

- 強勢訊號
- 弱勢訊號
- 風險項目
- 今日最重要觀察價位 / 區間

> 價位必須由真實市場 / 期權資料計算，不得由 AI 自行生成。

---

## Section C — Meta Fixed Watch

Meta 固定追蹤，內容可比 Tesla 精簡，但至少包含：

- 價格
- 漲跌
- 成交量
- 重大新聞
- 財報
- Options / Max Pain（可得時）
- 風險提醒

---

## Section D — Magnificent Seven

固定整理：

- AAPL
- MSFT
- GOOGL
- AMZN
- NVDA
- META
- TSLA

輸出：

- 漲跌排行
- 平均漲跌幅
- 最強
- 最弱
- 異常成交量
- 技術面異常
- 重大事件

Tesla 仍需第一順位呈現，不受原本排序限制。

---

# 5.3 今日市場總結

每日整理：

### 多頭股票

由系統依據真實數據篩選 3–5 檔。

條件可包含：

- 當日相對強勢
- 成交量放大
- relative volume
- 突破
- Options activity
- 催化事件
- 正向新聞
- sector strength

### 空頭股票

由系統篩選 3–5 檔。

條件可包含：

- 相對弱勢
- 放量下跌
- 跌破關鍵均線
- bearish options activity
- 財報風險
- 負面事件
- sector weakness

每檔需列：

- symbol
- price
- change %
- volume
- relative volume
- reason
- source timestamp

---

# 5.4 高成交量「操作觀察池」

目的不是直接下單，而是幫 Maggie 快速找到流動性足、短線值得觀察的標的。

每日至少輸出 5–10 檔。

排序依據可包含：

- Dollar Volume
- Relative Volume
- Intraday Range
- Options activity
- Bid / Ask liquidity（如資料可得）
- News catalyst
- Volatility

需避免把低流動性小型股排在前面。

---

# 5.5 Day Trade 模式

指令：

`/daytrade TSLA`

主要服務 Maggie 的 1 分鐘 / 3 分鐘短線判讀。

資料可包含：

- 1m candles
- 3m candles
- price
- volume
- relative volume
- VWAP
- opening range
- intraday high / low
- options flow
- nearest heavy open interest
- Max Pain
- implied volatility
- put/call
- large block / sweep（資料可得時）

AI 輸出重點：

- 現在偏多 / 偏空 / 盤整
- 支持該判斷的資料
- 目前主要風險
- 需要觀察什麼才會改變判斷

禁止直接輸出「買進 / 賣出」保證式訊號。

---

# 5.6 Watchlist

Maggie 可自行管理。

指令：

- `/watchlist`
- `/add TSLA`
- `/remove TSLA`

每檔股票保存：

- symbol
- priority
- added_at
- notes（optional）

預設：

1. TSLA — highest priority
2. META — fixed priority

---

# 5.7 股票搜尋

至少支援 S&P 500。

指令：

`/quote NVDA`

回傳：

- price
- open
- high
- low
- previous close
- change
- change %
- volume
- relative volume
- market cap（可得時）
- next earnings
- latest major news
- data timestamp

---

# 5.8 `/analyze <symbol>`

比 `/quote` 更完整。

整合：

- 行情
- technical indicators
- earnings
- major news
- options
- Max Pain
- sector performance
- relative strength
- risk events

最後給：

- Bullish evidence
- Bearish evidence
- Neutral / uncertain evidence
- Key things to watch

AI 必須把「資料」與「解讀」分開呈現。

---

# 5.9 財報提醒

Watchlist 中的股票：

### T-1 Day

提醒：

- 明天財報
- 預計公布時段
- 市場共識（如資料來源允許）
- options implied move（如可得）
- Max Pain
- 近期波動

### Post Earnings

財報公布後：

- EPS
- Revenue
- guidance
- surprise
- after-hours move
- management highlights

所有財報數字必須有來源。

---

## 6. 每週報告

每週輸出一份 Weekly Radar。

時間先做成可設定，不在 v0.1 寫死。

---

# 6.1 Tesla Weekly Outlook

Tesla 永遠第一。

包含：

- weekly return
- average volume
- relative volume
- key news
- options positioning
- Max Pain
- major OI strikes
- IV
- earnings / events
- analyst changes（可得時）
- Congress activity（Phase 3）
- Dark Pool（Phase 3）

最後給出：

- Bull case
- Bear case
- Base case
- Risks
- Next week watch items

不得使用「必漲 / 必跌」。

---

# 6.2 Meta Weekly Outlook

Meta 固定第二層重點。

結構與 Tesla 類似，但可以稍微精簡。

---

# 6.3 Potential Movers

每週找出可能出現較大波動的股票。

不是「預測一定上漲」，而是：

**High-Conviction Watchlist**

可分：

- Potential Upside Candidates
- Potential Downside Candidates
- High Volatility Candidates

每類 3–5 檔。

依據至少兩項真實訊號：

- earnings
- news catalyst
- options positioning
- unusual volume
- relative strength
- sector momentum
- institutional / analyst action
- congress transaction
- dark pool
- technical setup

---

# 6.4 強勢 / 弱勢股票

每週輸出：

- Strongest 3–5
- Weakest 3–5
- 平均漲跌幅
- sector distribution
- volume quality

---

## 7. 交易策略提醒

產品需要固定提供三種視角。

### 7.1 強勢追蹤

找出：

- price strength
- volume confirmation
- relative strength
- sector confirmation
- options confirmation

輸出：

- symbol
- reason
- catalyst
- invalidation risk

---

### 7.2 逢低布局觀察

找出：

- 基本面 / 事件面沒有明顯惡化
- 短期明顯回檔
- 技術或期權有潛在支撐
- 流動性足夠

只標示為「Pullback Watch」，不得直接視為買進建議。

---

### 7.3 平衡配置

顯示：

- sector strength
- market concentration
- growth vs value
- large cap vs small cap
- risk-on / risk-off
- portfolio concentration risk

---

## 8. AI 智能建議模組

輸出分類：

### A. 長線持有候選
可保留「長線持有」標籤，但必須附註：

> 此為研究分類，不代表投資建議。

### B. 短線觀望

代表：

- 高波動
- direction unclear
- event risk
- options conflict

### C. 風險警告

列出：

- upcoming earnings
- legal/regulatory
- unusual volatility
- options skew
- liquidity issue
- breaking news
- large gap risk

### D. 投資組合重點觀察

列出目前最值得花時間追蹤的 3–5 檔。

Tesla 預設最高優先。

---

## 9. 今日行動清單

每天報告最後輸出：

### Priority 1
Tesla

### Priority 2–5
系統依照：
- catalyst
- volume
- options
- earnings
- volatility
- risk

排序。

每個標的只回答三件事：

1. 為什麼今天值得看
2. 今天要看哪個資料
3. 什麼情況代表原本判斷失效

---

## 10. Options / Max Pain 模組

Tesla 為最高優先。

需要支援：

- option chain
- calls
- puts
- volume
- open interest
- put/call
- IV
- high OI strikes
- unusual flow
- Max Pain

後續可增加：

- gamma exposure
- dealer positioning
- IV skew
- expected move

Max Pain 必須由 option chain 真實資料計算或由可信資料源提供。

不得由 AI 推算不存在的資料。

---

## 11. Congress Tracker — Phase 3

追蹤美國國會議員公開申報股票交易。

功能：

- latest transactions
- buys
- sells
- politician
- filing date
- transaction date
- amount range
- ticker
- Watchlist overlap
- TSLA / META overlap

每日 / 每週可產出：

**Congress Alert**

若 Watchlist 有新增重要申報，優先顯示。

注意：

Congress filing 往往不是即時交易資料，UI 必須清楚標示：

- transaction date
- disclosure date

不能誤導成即時資金流。

---

## 12. Dark Pool Radar — Phase 3

功能：

- large off-exchange trades
- unusual notional volume
- price level
- repeated blocks
- dark pool concentration
- Watchlist overlap

Tesla / Meta 優先。

輸出只描述：

- 發生了什麼
- 規模
- 價格區間
- 與近期價格的關係

不得自動解讀為機構一定買進或一定賣出。

---

## 13. 全球市場雷達

需設計為獨立 service。

未來可支援：

- US
- Taiwan
- Japan
- Hong Kong
- China
- Europe
- FX
- Rates
- Commodities
- Crypto

Telegram 第一版只顯示最重要摘要。

---

## 14. Data Provider 架構

目前 repo 已經有：

- Polygon
- Twelve Data
- Alpha Vantage

新的版本不應把應用層直接綁死單一 provider。

建議建立 adapter interface：

```text
MarketDataProvider
OptionsDataProvider
NewsProvider
EarningsProvider
CongressProvider
DarkPoolProvider
GlobalMarketProvider
```

任何 provider 都可以被替換。

例如：

```text
services/providers/
  polygonProvider.js
  twelveDataProvider.js
  alphaVantageProvider.js
```

Options / Congress / Dark Pool 供應商留到 Phase 2 / 3 再決定。

---

## 15. Data Integrity Contract

每一筆分析輸入 AI 前，先轉成 structured JSON。

範例：

```json
{
  "symbol": "TSLA",
  "market_data": {},
  "options": {},
  "earnings": {},
  "news": [],
  "sources": [],
  "fetched_at": ""
}
```

AI system prompt 必須包含：

> You must only use facts supplied in this JSON.  
> If a fact is missing, say it is unavailable.  
> Never invent prices, percentages, dates, volumes, option values, analyst ratings, filings, or market events.

---

## 16. 系統架構

建議：

```text
Telegram
   |
Bot Adapter
   |
Application Layer
   |
------------------------------------------------
Market Data
Options
News
Earnings
Watchlist
Scoring
Congress
Dark Pool
Global Market
------------------------------------------------
   |
Normalized Data
   |
Rules / Scoring Engine
   |
AI Summary Layer
   |
Formatter
   |
Telegram
```

AI 永遠在資料取得與規則處理之後。

---

## 17. 建議 Repo 結構

```text
RedwoodVest/
│
├── README.md
├── PRD.md
├── VISION.md
├── ARCHITECTURE.md
├── ROADMAP.md
├── CLAUDE.md
├── .env.example
├── package.json
│
├── src/
│   ├── bot/
│   │   ├── telegram.js
│   │   └── commands.js
│   │
│   ├── services/
│   │   ├── market/
│   │   ├── options/
│   │   ├── earnings/
│   │   ├── news/
│   │   ├── congress/
│   │   ├── darkpool/
│   │   └── global/
│   │
│   ├── scoring/
│   │   ├── momentumScore.js
│   │   ├── riskScore.js
│   │   └── opportunityScore.js
│   │
│   ├── ai/
│   │   ├── summarizer.js
│   │   └── prompts.js
│   │
│   ├── reports/
│   │   ├── dailyReport.js
│   │   ├── weeklyReport.js
│   │   └── teslaReport.js
│   │
│   ├── scheduler/
│   │   └── jobs.js
│   │
│   ├── database/
│   │   └── database.js
│   │
│   └── config/
│       └── env.js
│
└── tests/
```

---

## 18. 非功能需求

### Reliability

- Bot restart 後 scheduler 必須恢復
- API error 不得讓整支 Bot crash
- provider failure 必須 fallback 或標示 unavailable

### Security

- 所有 keys 放 `.env`
- 不可 commit token
- 不可把 API key 寫死在程式碼

### Logging

至少記錄：

- provider
- endpoint type
- symbol
- fetched_at
- latency
- success / fail
- error message

### Timezone

所有排程必須明確設定：

`Asia/Taipei`

禁止依賴 server local timezone。

---

## 19. Phase Roadmap

### Phase 0 — Repo Recovery / Foundation

- 建立新 repo
- 原 repo 保留不動
- 修正 package.json
- 修正 import
- 清除 mock data
- 建立 env
- 建立 provider layer
- Telegram 可成功啟動

### Phase 1 — Maggie Telegram MVP

- 07:00 daily brief
- Tesla Priority
- Meta fixed watch
- Magnificent Seven
- S&P 500 quote
- Watchlist
- earnings reminders
- daily action list
- strong / weak stocks
- volume watch

### Phase 2 — Options Intelligence

- Max Pain
- option chain
- IV
- put/call
- OI
- flow
- daytrade mode

### Phase 3 — Alternative Data

- Congress
- Dark Pool
- advanced global radar

### Phase 4 — Multi User / Community

- authentication
- user watchlists
- tiers
- VIP
- Telegram groups
- payment

### Phase 5 — Multi Platform

- LINE
- WhatsApp
- Discord
- Slack
- Microsoft Teams
- Email
- Web
- iOS / Android

---

## 20. Phase 1 Acceptance Criteria

Phase 1 只有在以下全部成立時才算完成：

- [ ] Bot 可以正常啟動
- [ ] `/quote TSLA` 回傳真實資料
- [ ] `/quote` 至少可查 S&P 500
- [ ] `/watchlist` 正常
- [ ] `/add` / `/remove` 正常
- [ ] Tesla 固定第一優先
- [ ] Meta 固定追蹤
- [ ] 07:00 台北時間成功推送
- [ ] 七巨頭數據無假資料
- [ ] 強勢 / 弱勢股有量化依據
- [ ] 所有價格都有 timestamp
- [ ] provider 失敗不會產生假值
- [ ] 沒有 mock / fake market data
- [ ] AI 無法自行生成市場數字
- [ ] 所有 API keys 都在 `.env`

---

## 21. Claude 開發執行原則

Claude 在此 repo 工作時必須遵守：

1. 不得整個重寫現有架構
2. 每次只完成一個小里程碑
3. 每次修改前先說明：
   - 要改什麼
   - 為什麼
   - 影響哪些檔案
4. 修改後必須提供：
   - test steps
   - expected output
5. 不得留下 mock market data
6. 不得創造假 API response
7. 不得把 TODO 當成完成
8. 不可新增大量功能後才一起測
9. 每一個 Phase 必須先驗收再進下一個
10. 發現現有程式有問題時，優先修復，不要直接拋棄整個 repo

---

## 22. 產品成功指標

第一階段不是看營收。

先看：

### Reliability
- 07:00 報告成功率
- API 成功率
- Bot uptime

### Data Quality
- 錯誤市場數據 = 0
- AI fabricated number = 0

### Personal Utility
- Maggie 每天是否真的打開
- 是否降低查資料時間
- 是否能更快找到今天該關注的股票
- Tesla / Watchlist 是否能直接支援日常盯盤流程

---

## 23. 第一個真正要做的 Milestone

**Milestone 1: Clean Boot**

只做：

1. 建立新的 `RedwoodVest` repo
2. 複製可保留的舊架構
3. Bot 可啟動
4. `/start` 正常
5. `/quote TSLA` 能拿到真實 API 數據
6. 所有 mock data 移除
7. 資料錯誤時顯示 unavailable

在 Milestone 1 通過前：

**禁止開始 Congress、Dark Pool、Options Flow、VIP、Dashboard。**

這是避免再次陷入「功能很多，但 Bot 根本不能穩定使用」的核心規則。
