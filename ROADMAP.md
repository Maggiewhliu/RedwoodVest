# ROADMAP.md — RedwoodVest

**Version:** v0.2  
**Status:** Active roadmap  
**Primary owner:** Maggie  
**Initial platform:** Telegram  
**Primary timezone:** Asia/Taipei  
**Core stack:** Node.js + Telegraf + SQLite  
**Source of truth:** `PRD.md` + `ARCHITECTURE.md` + `CLAUDE.md`
**Official brand / GitHub repo:** RedwoodVest
**Website:** redwoodvest.com
**Official proprietary score:** Redwood Signal Score

---

# 1. Roadmap Objective

這份 Roadmap 的目的不是把功能塞得越多越快，而是確保 RedwoodVest 可以：

1. **先穩定跑起來**
2. **先只服務 Maggie 自己**
3. **所有市場資料都是真實、可驗證的**
4. **Tesla 永遠最高優先**
5. **Meta 固定追蹤**
6. **先完成每日實際會用的工作流**
7. **再加入 Options / Max Pain**
8. **再加入 Congress / Dark Pool**
9. **最後才恢復社群、VIP 與多平台**

---

# 2. Development Rules

所有 Phase 都遵守：

```text
Build small
→ Test
→ Verify data
→ Accept milestone
→ Move forward
```

禁止：

```text
一次做很多功能
→ 最後一起測
```

每一個 Milestone 都必須有明確驗收條件。

---

# 3. Overall Phase Map

```text
Phase 0  Foundation / Recovery
   ↓
Phase 1  Maggie Telegram MVP
   ↓
Phase 2  Options Intelligence + Day Trade
   ↓
Phase 3  Alternative Data + Advanced Radar
   ↓
Phase 4  Community / VIP / Monetization
   ↓
Phase 5  Multi-Platform Expansion
```

---

# 4. Phase 0 — Foundation / Recovery

## Goal

建立新的乾淨 repo：

```text
RedwoodVest
```

原本 repo：

```text
maggie_invests_bot
```

保留為 reference，不直接大改。

---

## Milestone 0.1 — Repo Initialization

### Tasks

- [ ] 建立新 GitHub repo `RedwoodVest`
- [ ] 放入：
  - [ ] `PRD.md`
  - [ ] `ARCHITECTURE.md`
  - [ ] `CLAUDE.md`
  - [ ] `ROADMAP.md`
- [ ] 建立：
  - [ ] `README.md`
  - [ ] `VISION.md`
  - [ ] `DECISIONS.md`
  - [ ] `.gitignore`
  - [ ] `.env.example`
  - [ ] `package.json`
- [ ] 建立基本資料夾結構

### Acceptance

- [ ] Repo 結構清楚
- [ ] 文件全部可讀
- [ ] `.env` 不會被 commit
- [ ] Claude 能從文件理解專案方向

---

## Milestone 0.2 — Clean Boot

### Goal

先讓 Bot 真正跑起來。

### Tasks

- [ ] 修正原本可復用的 Telegraf 啟動邏輯
- [ ] 建立 `src/index.js`
- [ ] 建立 Telegram adapter
- [ ] 建立 env loader
- [ ] 建立基本 logger
- [ ] 建立 SQLite 初始化
- [ ] `/start` 正常回覆
- [ ] 僅 Maggie 可使用
- [ ] 其他使用者顯示 `Private beta`

### Acceptance

```text
npm install
npm start
```

都成功。

Telegram：

```text
/start
```

有回覆。

Bot 不 crash。

---

## Milestone 0.3 — Real Market Data Foundation

### Goal

建立第一個可信的真實市場資料流程。

### Tasks

- [ ] 建立 `MarketDataProvider` interface
- [ ] Polygon 作為 primary
- [ ] Twelve Data 作為 fallback
- [ ] Alpha Vantage 作為第二 fallback
- [ ] 建立 normalized quote model
- [ ] 建立 timestamp
- [ ] 建立 provider 欄位
- [ ] 建立 freshness
- [ ] 建立錯誤處理
- [ ] 建立 retry / timeout
- [ ] 建立 cache

### First Test Symbol

```text
TSLA
```

### Acceptance

輸入：

```text
/quote TSLA
```

回傳：

- [ ] symbol
- [ ] price
- [ ] change
- [ ] change %
- [ ] volume
- [ ] provider
- [ ] data timestamp

API 全失敗時：

```text
資料暫無
```

不得出現假價格。

---

# 5. Phase 1 — Maggie Telegram MVP

## Goal

做出 Maggie 每天真的會打開使用的 Telegram 投資助理。

---

## Milestone 1.1 — Core Stock Commands

### Commands

- [ ] `/start`
- [ ] `/help`
- [ ] `/quote TSLA`
- [ ] `/quote META`
- [ ] `/quote NVDA`
- [ ] `/market`

### Market Universe

至少支援：

```text
S&P 500
```

之後可以擴大到美股主要上市股票。

### Acceptance

- [ ] S&P 500 股票可查
- [ ] 無效 symbol 有合理錯誤訊息
- [ ] 所有資料有 timestamp
- [ ] 無 fake data

---

## Milestone 1.2 — Tesla Priority

### Goal

讓 TSLA 成為產品第一順位。

### Tesla Card

至少包含：

- [ ] 最新價格
- [ ] 漲跌幅
- [ ] volume
- [ ] relative volume
- [ ] high / low
- [ ] 重大新聞
- [ ] 下一次財報日
- [ ] RSI（如可得）
- [ ] 重大事件
- [ ] risk flags

### Rule

Tesla 永遠不因排行榜而被排到後面。

### Acceptance

每日摘要與相關報告：

```text
TSLA first
```

---

## Milestone 1.3 — Meta Fixed Watch

### Goal

Meta 固定追蹤。

### Includes

- [ ] price
- [ ] change
- [ ] volume
- [ ] news
- [ ] earnings
- [ ] risk

### Acceptance

每日摘要固定有 META。

---

## Milestone 1.4 — Watchlist

### Commands

```text
/watchlist
/add AAPL
/remove AAPL
```

### Default Watchlist

```text
TSLA — priority 100
META — priority 90
```

### Acceptance

- [ ] 新增成功
- [ ] 刪除成功
- [ ] 重複新增不 crash
- [ ] restart 後資料仍在
- [ ] Watchlist 依 priority 排序

---

## Milestone 1.5 — Magnificent Seven

### Stocks

```text
AAPL
MSFT
GOOGL
AMZN
NVDA
META
TSLA
```

### Output

- [ ] 今日漲跌
- [ ] 平均漲跌幅
- [ ] 最強
- [ ] 最弱
- [ ] volume anomalies
- [ ] major news
- [ ] risk flags

### Display Rule

Tesla 仍然第一順位顯示。

---

## Milestone 1.6 — Strong / Weak Stock Scanner

### Goal

每天找出：

```text
Strongest 3–5
Weakest 3–5
```

### Strong Inputs

可使用：

- price strength
- relative volume
- sector strength
- breakout
- catalyst
- news

### Weak Inputs

可使用：

- relative weakness
- sell volume
- breakdown
- negative catalyst
- sector weakness

### Acceptance

每檔都有：

- [ ] symbol
- [ ] price
- [ ] change %
- [ ] volume
- [ ] relative volume
- [ ] reason
- [ ] source timestamp

---

## Milestone 1.7 — High-Volume Trading Watch

### Goal

建立「可操作觀察池」。

不是自動買入清單。

### Rank Inputs

- Dollar Volume
- Relative Volume
- Intraday Range
- Catalyst
- Liquidity

### Output

```text
5–10 stocks
```

### Acceptance

- [ ] 避免低流動性 microcaps 排到前面
- [ ] 每檔有流動性依據
- [ ] 每檔有入選原因

---

## Milestone 1.8 — Earnings Reminder

### Watchlist T-1

財報前一天：

- [ ] report date
- [ ] report time
- [ ] EPS estimate（if available）
- [ ] revenue estimate（if available）
- [ ] expected volatility（if available）
- [ ] risks

### Post-Earnings

- [ ] actual EPS
- [ ] actual revenue
- [ ] surprise
- [ ] guidance
- [ ] after-hours move

### Acceptance

資料缺少：

```text
資料來源未提供
```

不得補寫。

---

## Milestone 1.9 — Global Market Radar

### Includes

至少盡可能支援：

- S&P 500
- Nasdaq
- Dow
- Russell 2000
- Nikkei
- Taiwan
- Hang Seng
- Shanghai
- Europe
- USD Index
- U.S. 10Y
- VIX
- Gold
- Bitcoin

### Output

```text
Risk On
Neutral
Risk Off
```

並附依據。

---

## Milestone 1.10 — 07:00 Overnight Brief

### Schedule

```text
07:00 Asia/Taipei
```

### Final Section Order

```text
1. Global Market Radar
2. Tesla Priority
3. Meta Fixed Watch
4. Magnificent Seven
5. Bullish Candidates
6. Bearish Candidates
7. High-Volume Watch
8. Earnings / Major Events
9. Risk Alerts
10. Today's Action List
```

### Acceptance

- [ ] 07:00 準時
- [ ] 只有 Maggie 收到
- [ ] 不重複發送
- [ ] provider 掛掉不 crash
- [ ] Tesla first
- [ ] 有資料時間
- [ ] 無 fake numbers

---

## Milestone 1.11 — Daily Action List

### Format

```text
Priority 1 — TSLA
Why watch:
What matters:
Invalidation:

Priority 2 — ...
```

### Ranking

可依：

- catalyst
- volume
- volatility
- earnings
- relative strength
- risk

### Acceptance

每日產出：

```text
3–5 priorities
```

Tesla 預設 Priority 1。

---

## Milestone 1.12 — AI Summary Layer

### Goal

AI 只做「理解與解釋」。

### AI Input

只能是 structured payload。

### AI Output

可做：

- summary
- comparison
- interpretation
- conflict detection

### Validator

檢查：

- [ ] AI 是否新增 payload 沒有的數字
- [ ] AI 是否新增不存在的 ticker
- [ ] AI 是否使用保證性語言
- [ ] AI 是否把 interpretation 當 fact

### Fallback

AI 失敗：

```text
Rule-based summary
```

### Acceptance

```text
AI fabricated number = 0
```

---


## Milestone 1.13 — Redwood Signal Score v1

### Goal

建立 RedwoodVest 第一版 proprietary ranking layer。

正式對外名稱：

```text
Redwood Signal Score
```

### Phase 1 Inputs

v1 先使用目前已有、可驗證的資料，例如：

- price / momentum
- relative volume
- liquidity
- volatility
- catalyst
- risk
- market / sector context

Options、Congress、Dark Pool 等資料依 Phase 2 / 3 再加入，不可用 placeholder 假裝存在。

### Requirements

- [ ] deterministic calculation
- [ ] score 0–100
- [ ] confidence
- [ ] score version
- [ ] calculated_at
- [ ] top factors
- [ ] risk factors
- [ ] unit tests
- [ ] no AI-generated score
- [ ] proprietary weights server-side only

### Acceptance

TSLA、META 與其他候選標的可以得到可追溯的 Redwood Signal Score。

資料不足時：

```text
Score unavailable
```

或降低 confidence，不得自行補值。

---

# 6. Phase 1 Completion Gate

Phase 1 必須全部達成：

- [ ] Bot 穩定啟動
- [ ] Maggie-only access
- [ ] TSLA highest priority
- [ ] META fixed watch
- [ ] S&P 500 quote
- [ ] Watchlist
- [ ] Magnificent Seven
- [ ] Strong / Weak scanner
- [ ] High-volume watch
- [ ] Earnings reminders
- [ ] Global radar
- [ ] 07:00 brief
- [ ] Daily Action List
- [ ] AI no-fabrication validator
- [ ] Redwood Signal Score v1
- [ ] 0 production mock data

完成前：

```text
禁止進 Phase 2
```

---

# 7. Phase 2 — Options Intelligence + Day Trade

## Goal

加入 Maggie 真正短線會看的期權與盤中資訊。

---

## Milestone 2.1 — Options Provider Selection

先決定資料供應商。

### Required Data

- option chain
- call volume
- put volume
- open interest
- IV
- expiration
- strikes

### Decision Criteria

- API availability
- cost
- rate limits
- latency
- historical depth
- commercial use
- reliability

完成後記錄到：

```text
DECISIONS.md
```

---

## Milestone 2.2 — Max Pain

### Goal

Tesla 優先。

### Implementation

優先：

```text
本地 deterministic calculation
```

需要：

- strike
- call OI
- put OI

### Acceptance

- [ ] 有 unit test
- [ ] AI 不參與計算
- [ ] TSLA 可顯示 Max Pain
- [ ] 資料來源與 expiration 清楚

---

## Milestone 2.3 — Options Snapshot

### Tesla

顯示：

- [ ] Put / Call
- [ ] IV
- [ ] Max Pain
- [ ] largest call OI
- [ ] largest put OI
- [ ] unusual flow（如 provider 支援）
- [ ] expected move（如可計算）

### Meta

第二順位。

---

## Milestone 2.4 — Day Trade Mode

### Command

```text
/daytrade TSLA
```

### Inputs

- 1m candles
- 3m candles
- VWAP
- volume
- relative volume
- opening range
- intraday high/low
- options
- OI
- Max Pain
- IV

### Output

```text
偏多
偏空
盤整
```

同時顯示：

- evidence
- risk
- invalidation

### Acceptance

- [ ] 不直接輸出保證式 buy/sell
- [ ] 支援 TSLA
- [ ] 支援 Watchlist stocks
- [ ] 所有數字可追溯

---

## Milestone 2.5 — Trading Strategy Module

### Labels

```text
強勢追蹤
逢低布局觀察
平衡配置
長線研究候選
短線觀望
風險警告
```

每個分類都必須由規則先篩選，再由 AI 解釋。

---

# 8. Phase 2 Completion Gate

- [ ] Options provider stable
- [ ] TSLA Options card stable
- [ ] Max Pain verified
- [ ] Put/Call verified
- [ ] IV verified
- [ ] OI verified
- [ ] `/daytrade TSLA` stable
- [ ] 1m / 3m data stable
- [ ] AI 不生成 options values

完成後才進 Phase 3。

---

# 9. Phase 3 — Alternative Data + Advanced Radar

## Goal

加入差異化資料來源。

---

## Milestone 3.1 — Congress Tracker

### Required Fields

- politician
- ticker
- transaction type
- transaction date
- disclosure date
- amount range
- source

### Output

```text
Congress Alert
```

### Priority

TSLA / META / Watchlist overlap 優先。

### Acceptance

- [ ] transaction date 與 disclosure date 分開
- [ ] 不當成即時資金流
- [ ] 不捏造交易方向或金額

---

## Milestone 3.2 — Dark Pool Radar

### Required

- symbol
- timestamp
- price
- size
- notional
- repeated blocks
- provider

### Priority

TSLA first。

### Acceptance

- [ ] 不把 dark pool print 自動解讀為買進
- [ ] 不把 dark pool print 自動解讀為賣出
- [ ] 有 source 與 timestamp

---

## Milestone 3.3 — Advanced Global Radar

擴充：

- FX
- rates
- commodities
- crypto
- sector rotation
- market breadth
- cross-asset risk signals

---

## Milestone 3.4 — Weekly Radar

### Default Structure

```text
1. Tesla Weekly Outlook
2. Meta Weekly Outlook
3. Strongest 3–5
4. Weakest 3–5
5. Potential Upside Watch
6. Potential Downside Watch
7. High Volatility Watch
8. Congress Alerts
9. Dark Pool Alerts
10. Next Week Action List
```

### Tesla Weekly

包含：

- weekly return
- average volume
- relative volume
- news
- earnings / events
- options
- Max Pain
- OI
- IV
- Congress
- Dark Pool
- risks

### Scenario Format

```text
Bull case
Bear case
Base case
```

禁止：

```text
必漲
必跌
```

---

# 10. Phase 3 Completion Gate

- [ ] Congress stable
- [ ] Dark Pool stable
- [ ] Weekly Radar stable
- [ ] Tesla cross-data card stable
- [ ] Meta cross-data card stable
- [ ] alternative data timestamps clear
- [ ] no misleading institutional-flow claims

---

# 11. Phase 4 — Community / VIP / Monetization

## Goal

等 Maggie 自用版真的有價值後，再恢復多人產品。

---

## Milestone 4.1 — Multi-User Core

- [ ] users
- [ ] user settings
- [ ] per-user watchlist
- [ ] authorization
- [ ] rate limiting per user

---

## Milestone 4.2 — Membership Tiers

可重新評估舊 repo：

```text
Lobby
Certified
VIP
```

但不要照搬。

先根據實際產品價值重新設計權限。

---

## Milestone 4.3 — Subscription

- [ ] plan
- [ ] billing
- [ ] expiry
- [ ] renewal
- [ ] cancellation

---

## Milestone 4.4 — Payment

依當時市場與公司條件選擇：

- ECPay
- Stripe
- other approved provider

---

## Milestone 4.5 — Community

- Telegram group
- admin tools
- onboarding
- moderation
- verification

---

# 12. Phase 4 Completion Gate

- [ ] Maggie 自用版仍正常
- [ ] 多用戶資料互相隔離
- [ ] Watchlist per user
- [ ] subscription stable
- [ ] payment stable
- [ ] community 不影響核心市場資料服務

---

# 13. Phase 5 — Multi-Platform Expansion

## Goal

將 RedwoodVest 從 Telegram Bot 變成多平台投資助理。

---

## Platform Priority

建議順序：

```text
1. Telegram
2. LINE
3. WhatsApp
4. Web
5. Email
6. Discord
7. Slack
8. Microsoft Teams
9. iOS
10. Android
```

實際順序可依使用者需求調整。

---

## Milestone 5.1 — Platform-Neutral Message Model

Core backend 先輸出：

```json
{
  "title": "",
  "sections": [],
  "alerts": [],
  "actions": []
}
```

由各 adapter 格式化。

---

## Milestone 5.2 — LINE

支援：

- daily brief
- alerts
- watchlist commands
- stock query

---

## Milestone 5.3 — WhatsApp

透過官方 WhatsApp Business Platform。

第一版支援：

- daily brief
- alert
- basic query

---

## Milestone 5.4 — Web Dashboard

等資料層穩定後才做。

用途：

- chart
- watchlist overview
- weekly history
- options view
- congress
- dark pool
- report archive

---

## Milestone 5.5 — Email

適合：

- weekly report
- long-form research
- Congress digest
- Tesla weekly digest

---

## Milestone 5.6 — Mobile App

最後再評估：

```text
iOS
Android
```

避免過早投入 app 開發。

---


## Milestone 5.7 — Growth & Distribution Loop

RedwoodVest 不只擴平台，也要建立可持續獲客循環。

建議早期路徑：

```text
X / Threads / Search / Community
        ↓
Free TSLA Radar
        ↓
Telegram / LINE
        ↓
RedwoodVest.com account
        ↓
Watchlist / Alerts
        ↓
Premium membership
```

### Early target audience

第一批優先：

- Tesla active investors
- U.S. tech-stock investors
- options-aware active investors
- users who need consolidated U.S. market information

### Channel strategy

- Telegram：第一個完整產品入口
- redwoodvest.com：會員、帳號、歷史資料與長期產品資產
- LINE：台灣市場的重要通知入口
- WhatsApp：海外華人 / 國際市場延伸
- X / Threads：內容獲客，不作為核心資料產品
- Email：Weekly Radar 與長篇研究

### Growth rule

免費內容提供足夠價值讓使用者建立習慣，但完整個人化、歷史、進階資料與 proprietary signals 留在會員層。

---

# 14. Product Quality Gates

所有 Phase 都要追蹤：

## Reliability

```text
Bot uptime
Scheduler success rate
Provider success rate
Message send success
```

## Data Integrity

```text
Fabricated price = 0
Fabricated volume = 0
Fabricated options value = 0
Fabricated event = 0
```

## Product Utility

Maggie 是否：

- 每天真的看
- 少開很多網站
- 更快找到今日重點
- 更快找到 TSLA 風險與機會
- 更快完成早晨市場掃描
- 更快找到當沖觀察標的

---

# 15. Current Priority Queue

目前正式優先順序：

```text
P0 — 建立新 repo
P0 — 文件放入 repo
P0 — Clean Boot
P0 — /quote TSLA
P0 — No-fake-data path

P1 — Tesla Priority
P1 — Meta Fixed Watch
P1 — Watchlist
P1 — S&P 500 query
P1 — 07:00 Overnight Brief
P1 — Strong / Weak Scanner
P1 — High-Volume Watch
P1 — Earnings Alerts
P1 — Daily Action List

P2 — Options provider
P2 — Max Pain
P2 — Put/Call / IV / OI
P2 — /daytrade TSLA

P3 — Congress
P3 — Dark Pool
P3 — Weekly Radar

P4 — Community / VIP / Payment

P5 — LINE / WhatsApp / Web / App
```

---

# 16. Immediate Next 5 Engineering Tasks

Claude 的下一步不要自由發揮。

照以下順序：

## Task 1

建立新 repo 基礎檔案：

```text
package.json
.env.example
.gitignore
src/index.js
```

---

## Task 2

建立 Telegram Clean Boot：

```text
/start
Maggie-only middleware
```

---

## Task 3

建立 MarketDataProvider：

```text
Polygon primary
Twelve Data fallback
Alpha Vantage fallback
```

---

## Task 4

建立：

```text
/quote TSLA
```

要求：

```text
real data
timestamp
provider
unavailable fallback
```

---

## Task 5

建立 automated tests：

```text
valid TSLA response
bad symbol
primary provider failure
all providers failure
no fake data
```

**這五個 Task 全部完成並驗收後，才能做 Watchlist。**

---

# 17. What We Are Deliberately NOT Doing Yet

現在不要做：

- [ ] VIP
- [ ] payment
- [ ] community group
- [ ] dashboard
- [ ] mobile app
- [ ] Congress
- [ ] Dark Pool
- [ ] Options
- [ ] Max Pain
- [ ] daytrade
- [ ] complicated AI agent architecture
- [ ] microservices
- [ ] database migration to cloud

原因：

```text
先證明核心資料與 Telegram 工作流真的可靠。
```

---

# 18. Roadmap Completion Definition

Roadmap 的成功不是：

```text
功能全部做完
```

而是每一階段都有一個可以真的使用的產品版本：

```text
Phase 0
Bot can boot

Phase 1
Maggie can use it every day

Phase 2
Maggie can use it for options / intraday decision support

Phase 3
Maggie gets alternative-data edge

Phase 4
Other users can safely join and pay

Phase 5
The same intelligence is available across multiple platforms
```

---

# 19. Final Priority Rule

任何時候出現需求衝突，使用以下排序：

```text
1. Data correctness
2. Product stability
3. Maggie daily usefulness
4. Tesla priority
5. Speed
6. Feature quantity
```

如果新功能會犧牲資料可信度或穩定性：

```text
不要做。
```

