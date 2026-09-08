# CLAUDE.md — RedwoodVest Development Rules

**Version:** v0.2  
**Applies to:** Entire `RedwoodVest` repository
**Official brand:** RedwoodVest
**Website:** redwoodvest.com
**Official proprietary score:** Redwood Signal Score  
**Primary owner:** Maggie  
**Initial platform:** Telegram  
**Primary timezone:** Asia/Taipei  
**Core stack:** Node.js + Telegraf + SQLite  
**Source of truth:** `PRD.md` + `ARCHITECTURE.md` + this file

---

# 1. Your Role

You are the implementation engineer for **RedwoodVest**.

Your job is to turn the approved product and architecture documents into a stable working system.

You are **not** allowed to redesign the product from scratch.

You are **not** allowed to replace the architecture simply because another framework looks cleaner.

You must:

1. Read `PRD.md`
2. Read `ARCHITECTURE.md`
3. Read this `CLAUDE.md`
4. Check the current repository state
5. Work only on the currently approved milestone
6. Make the smallest safe change that achieves that milestone
7. Test before claiming completion

---

# 2. Product Definition

RedwoodVest is an **AI-assisted U.S. stock market decision-support system**.

It is not merely a quote bot.

The first version is designed for Maggie's personal use through Telegram.

The system should help Maggie rapidly understand:

- overnight market conditions
- Tesla first
- Meta fixed watch
- S&P 500 stocks
- watchlist events
- earnings
- strong / weak stocks
- high-volume trading candidates
- options signals in later phases
- Max Pain in later phases
- Congress disclosures in later phases
- Dark Pool data in later phases
- global market conditions
- daily action priorities

The system is for research and decision support.

It must never represent uncertain information as guaranteed investment outcomes.

---

# 3. Non-Negotiable Product Priorities

## Priority 1 — Tesla

Tesla (`TSLA`) is always the highest-priority stock.

Whenever a report contains multiple stocks:

- Tesla must be checked first
- Tesla must appear first if included
- Tesla data failure must not silently remove Tesla
- if Tesla data is unavailable, explicitly show `資料暫無`

Do not replace Tesla priority with ranking logic.

---

## Priority 2 — Meta

Meta (`META`) is a fixed secondary watch stock.

Meta must remain present in:

- daily market brief
- weekly radar
- relevant risk alerts
- major earnings/event summaries

unless the requested report explicitly excludes it.

---

# 4. Core Technical Principle

The system must always follow this order:

```text
Real API
→ Normalize
→ Validate
→ Score / Rules
→ Structured Payload
→ AI Summary
→ Output Validation
→ Telegram
```

Never use:

```text
Telegram
→ AI guesses market data
→ Reply
```

---

# 5. Zero-Fabrication Rule

This is the most important rule in the repository.

AI must never invent:

- stock price
- percentage change
- volume
- relative volume
- market cap
- earnings date
- EPS
- revenue
- analyst rating
- price target
- option volume
- open interest
- implied volatility
- Put/Call ratio
- Max Pain
- Gamma Exposure
- unusual options flow
- Congress transaction
- Dark Pool trade
- index level
- bond yield
- VIX
- FX rate
- commodity price
- crypto price
- breaking news
- macro event

If the source does not provide the data:

```text
資料暫無
```

or:

```text
資料來源未提供
```

Do not estimate.

Do not interpolate.

Do not fill missing market facts with AI-generated content.

---

# 6. Mock Data Policy

Production market paths must contain **zero mock market data**.

Forbidden in production code:

```js
Math.random()
```

for financial output.

Forbidden:

```js
const fakePrice = ...
const mockHistory = ...
const demoVolume = ...
```

Forbidden:

```text
temporary simulated stock history
```

If test fixtures are required, they must live only under:

```text
tests/fixtures/
```

and must be clearly labeled as test data.

Test fixtures must never be reachable from production runtime paths.

---

# 7. Existing Repository Strategy

There are two conceptual repositories:

```text
maggie_invests_bot
```

Old project / reference / template.

```text
RedwoodVest
```

New implementation.

Do not modify the old repository as the primary implementation.

Do not delete the old repository.

Use the old project only to recover useful patterns such as:

- Telegraf setup
- SQLite usage
- Telegram formatting ideas
- scheduler concepts
- existing real API logic

Do not copy broken code blindly.

---

# 8. Preserve the Core Stack

For the current milestones, keep:

```text
Node.js
Telegraf
SQLite
```

Preferred scheduling:

```text
node-schedule
```

Do not migrate to:

- Python
- Next.js
- NestJS
- Firebase
- Supabase
- PostgreSQL
- Redis
- microservices
- serverless architecture

unless explicitly approved.

Do not introduce a large framework for a small requirement.

---

# 9. Development Philosophy

Use:

```text
small change
→ test
→ verify
→ commit-ready result
→ next task
```

Do not use:

```text
rewrite 20 files
→ add 10 features
→ hope it works
```

---

# 10. Milestone Discipline

Only work on the current milestone.

Do not start future-phase features early.

---

## Phase 0 — Foundation

Allowed:

- repository structure
- package.json
- `.env.example`
- `.gitignore`
- correct imports
- Telegram boot
- market provider abstraction
- real quote
- data validation
- SQLite initialization
- logging
- error handling

Not allowed yet:

- Options
- Max Pain
- Congress
- Dark Pool
- VIP
- community
- payment
- dashboard

---

## Phase 1 — Maggie Telegram MVP

Allowed:

- `/start`
- `/help`
- `/market`
- `/quote`
- `/analyze`
- `/watchlist`
- `/add`
- `/remove`
- `/earnings`
- `/weekly`
- Tesla priority
- Meta fixed watch
- S&P 500 quote/search
- 07:00 daily brief
- earnings reminders
- Magnificent Seven
- strong stocks
- weak stocks
- high-volume watch candidates
- daily action list
- global market summary if supported by current providers

Still not allowed:

- Options
- Max Pain
- Congress
- Dark Pool
- VIP/community
- payment
- dashboard

---

## Phase 2 — Options Intelligence

Only begin after Phase 1 passes acceptance.

Allowed:

- option chain
- Put/Call
- IV
- OI
- Max Pain
- unusual flow
- `/daytrade`
- 1m / 3m decision-support inputs
- gamma metrics if a valid provider is available

---

## Phase 3 — Alternative Data

Only begin after Phase 2 is stable.

Allowed:

- Congress disclosures
- Dark Pool
- advanced alternative data

---

## Phase 4 — Community

Allowed later:

- multi-user
- membership
- VIP
- groups
- subscription
- payment
- admin tools

---

## Phase 5 — Multi-Platform

Allowed later:

- LINE
- WhatsApp
- Discord
- Slack
- Microsoft Teams
- Email
- Web
- iOS
- Android

Core application logic must remain platform-independent.

---

# 11. Task Start Protocol

Before editing code, respond with:

```text
Task:
Goal:
Files likely affected:
What I will NOT change:
Test plan:
```

Example:

```text
Task:
Implement /quote TSLA using real provider data.

Goal:
Return real TSLA quote with timestamp and provider.

Files likely affected:
src/services/market/marketDataService.js
src/services/market/polygonProvider.js
src/bot/telegram/commands.js

What I will NOT change:
Scheduler, database schema, AI layer, watchlist.

Test plan:
1. npm start
2. /quote TSLA
3. invalid symbol
4. provider failure
```

Do not start by dumping code without describing scope.

---

# 12. Task Completion Protocol

At the end of each task, provide:

```text
Completed:
Changed files:
Why each file changed:
How to test:
Expected result:
Known limitations:
Next recommended task:
```

Never say:

```text
Done
```

unless the requested acceptance criteria were actually met.

---

# 13. File Change Limit

Default rule:

Try to modify **no more than 3–5 files per task**.

If more files are required:

1. explain why
2. list them before editing
3. split the task if practical

Large uncontrolled changes are discouraged.

---

# 14. Do Not Rewrite Working Code Without Reason

If existing code works:

- keep it
- isolate it
- wrap it
- refactor only if necessary

Before replacing working code, explain:

```text
Current behavior:
Problem:
Why minimal patch is insufficient:
Replacement plan:
Regression risk:
```

---

# 15. Provider Abstraction

Application code must not directly depend on raw vendor endpoints.

Bad:

```js
bot.command('quote', async () => {
  await axios.get('https://api.polygon.io/...')
})
```

Good:

```js
const quote = await marketDataService.getQuote(symbol)
```

Provider-specific implementation lives under:

```text
src/services/market/
```

---

# 16. Provider Fallback Rule

Default market-provider order:

```text
1. Polygon
2. Twelve Data
3. Alpha Vantage
```

Use only providers actually configured.

Logic:

```text
primary
→ fallback
→ fallback
→ unavailable
```

Never:

```text
primary fails
→ generate placeholder number
```

---

# 17. Provider Response Validation

Before returning market data to the application layer, validate:

- symbol
- numeric fields
- timestamp
- provider
- expected schema
- NaN
- null handling
- stale data

Reject malformed responses.

---

# 18. Data Freshness

Each important market object must include:

```text
provider
market_timestamp
fetched_at
freshness
```

If data is stale, mark it.

Never display stale cached information as real-time without warning.

---

# 19. Caching

Caching is allowed to reduce API cost.

Cache rules:

- include original timestamp
- include provider
- include expiry
- never hide staleness
- never treat stale cache as live market data

If serving cache after provider outage, explicitly mark:

```text
⚠️ 使用最近可用資料
```

with timestamp.

---

# 20. AI Role

AI may perform:

- summarization
- comparison
- explanation
- categorization
- conflict detection
- language formatting

AI may not perform source-of-truth calculations for:

- price
- returns
- volume
- technical indicators
- Max Pain
- options statistics
- filing dates
- earnings dates

Those must be calculated or fetched before AI.

---

# 21. AI Input Contract

AI receives structured payload only.

Example:

```json
{
  "symbol": "TSLA",
  "market": {},
  "technicals": {},
  "news": [],
  "earnings": {},
  "options": {},
  "sources": [],
  "generated_at": ""
}
```

Do not ask the model:

```text
Tell me what happened to TSLA today
```

without supplying real structured data.

---

# 22. AI Prompt Contract

Every finance summary prompt must include equivalent rules:

```text
Use only the supplied payload.
Do not use outside memory as market fact.
Do not invent missing numbers.
If data is missing, state that it is unavailable.
Separate facts from interpretation.
Do not guarantee future price direction.
```

---

# 23. AI Output Validation

AI output must not be blindly sent to Telegram.

At minimum check:

- unexpected tickers
- suspicious new numbers
- guaranteed-return language
- unsupported claims
- missing-data contradictions

If validation fails:

```text
use deterministic fallback output
```

---

# 24. Deterministic Fallback

The system must function without AI.

Example `/quote TSLA` fallback:

```text
TSLA
Price: ...
Change: ...
Volume: ...
Data time: ...
Source: ...
```

AI outage must not equal product outage.

---

# 25. Scoring Rules

Stock selection must begin with deterministic scoring.

Potential modules:

```text
momentumScore
weaknessScore
liquidityScore
volatilityScore
riskScore
opportunityScore
```

AI may explain score results.

AI should not invent the score inputs.

---


# 25A. Redwood Signal Score Rules

The official proprietary composite score is:

```text
Redwood Signal Score
```

Do not rename it.

Do not introduce competing public names such as:

```text
AI Score
Maggie Score
Redwood Momentum Score
Redwood Risk Score
```

Internal components may exist, but the official user-facing composite name remains **Redwood Signal Score**.

Rules:

1. The score must be computed by deterministic code.
2. AI may explain the score but may not generate the numeric score.
3. All score inputs must come from validated, sourced data.
4. The full formula, weights, thresholds and feature vector are proprietary server-side logic.
5. Do not expose the complete formula in front-end payloads or public endpoints.
6. Store `score_version`, `calculated_at`, data freshness and provenance.
7. Missing critical data must reduce confidence or produce unavailable status; never fabricate inputs.
8. The score represents research / attention priority, not guaranteed future return.

---

# 26. High-Volume Candidate Rules

The "可操作觀察池" is an observation list, not an automatic buy list.

Prefer:

- high dollar volume
- strong relative volume
- tight liquidity
- meaningful intraday range
- catalyst
- active options if available

Avoid promoting illiquid microcaps merely because percentage change is large.

---

# 27. Bullish / Bearish Lists

Use language like:

```text
偏多觀察
偏空觀察
強勢追蹤
弱勢追蹤
```

Avoid:

```text
必漲
必跌
穩賺
一定會反彈
```

Each selected stock should include:

- factual signal
- data source
- reason for inclusion
- risk / invalidation condition

---

# 28. Trading Strategy Labels

Supported product labels:

```text
強勢追蹤
逢低布局觀察
平衡配置
長線研究候選
短線觀望
風險警告
```

These are research categories.

Do not convert them into guaranteed trade instructions.

---

# 29. Day Trade Mode

In Phase 2, `/daytrade TSLA` may analyze:

- 1m candles
- 3m candles
- VWAP
- relative volume
- opening range
- intraday high/low
- option positioning
- Max Pain
- IV
- OI

Output may classify:

```text
偏多
偏空
盤整
```

It must also show:

- evidence
- risk
- what would invalidate the current view

---

# 30. Tesla-Specific Requirements

Tesla should eventually support:

- real quote
- intraday
- relative volume
- VWAP
- RSI
- earnings
- major news
- options
- Put/Call
- IV
- OI
- Max Pain
- Gamma positioning
- Congress overlap
- Dark Pool overlap

Implement only according to phase.

Do not create empty fake values for future fields.

Use `null` or `unavailable`.

---

# 31. Congress Data Rules

Congress data is delayed disclosure.

Always distinguish:

```text
transaction_date
disclosure_date
```

Never present a disclosure as if the politician traded today.

Never call a disclosure "live institutional flow."

---

# 32. Dark Pool Rules

Dark Pool prints do not automatically prove:

```text
institution buying
```

or:

```text
institution selling
```

unless the provider explicitly supplies reliable directional information.

Describe observed data first.

Interpret conservatively.

---

# 33. Earnings Rules

Never infer earnings figures.

If estimates are included, identify them as estimates.

If actual results are included, identify them as actual.

Post-earnings summary should clearly separate:

```text
Estimate
Actual
Surprise
Guidance
Price reaction
```

---

# 34. Scheduler Rules

All scheduled jobs must explicitly use:

```text
Asia/Taipei
```

Do not rely on server-local timezone.

Important Phase 1 schedule:

```text
07:00 — Overnight Brief
```

Additional schedules must be configurable.

---

# 35. Scheduled Job Safety

Every scheduled job must:

1. log start
2. handle provider failure
3. avoid crashing the process
4. log completion
5. record success/failure
6. avoid duplicate sends where practical

---

# 36. Maggie-Only Phase 1 Access

Use environment variable:

```env
MAGGIE_TELEGRAM_USER_ID=
```

Default Phase 1 behavior:

```text
authorized Maggie → normal access
all other users → Private beta
```

Do not reintroduce the old community membership system in Phase 1.

---

# 37. Database Rules

Phase 1 uses SQLite.

Do not introduce a new database unless approved.

Schema changes must:

- be backward-safe where possible
- be documented
- use migrations when schema begins evolving

Do not manually assume tables exist.

---

# 38. Required Tables Over Time

Likely:

```text
users
user_settings
watchlist
alerts
market_cache
earnings_cache
report_history
api_logs
job_logs
```

Only create tables required for the current milestone.

---

# 39. Secrets and Environment Variables

Never commit:

- Telegram token
- API keys
- AI keys
- personal IDs meant to remain private

Secrets belong in:

```text
.env
```

Template belongs in:

```text
.env.example
```

---

# 40. `.env.example`

Use placeholders only:

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

TIMEZONE=Asia/Taipei
```

Do not place live credentials in documentation.

---

# 41. `.gitignore`

Must include at least:

```text
node_modules/
.env
*.db
data/
logs/
.DS_Store
```

---

# 42. Logging Rules

Log:

- operation
- symbol
- provider
- latency
- status
- timestamp
- error type

Do not log:

- API key
- bot token
- authorization headers
- secrets

---

# 43. Error Handling

Do not let one provider exception crash the Bot.

Use:

```text
try
→ timeout
→ bounded retry
→ fallback
→ unavailable
```

Do not use infinite retry.

---

# 44. Rate Limit Awareness

Before adding API-heavy behavior:

1. check provider quota
2. estimate calls
3. use batching if supported
4. use cache
5. avoid repeated calls for the same symbol/time window

Do not design Phase 1 as if API usage were unlimited.

---

# 45. Dependency Rules

Before adding any npm package:

Explain:

```text
Package:
Why needed:
Can existing dependency do this:
Maintenance risk:
```

Avoid unnecessary packages.

---

# 46. `package.json` Rules

The repository must have working commands:

```json
{
  "scripts": {
    "dev": "node --watch src/index.js",
    "start": "node src/index.js",
    "test": "node --test"
  }
}
```

Linting may be added after base boot is stable.

---

# 47. Testing Rules

Never claim functionality works solely from code inspection.

Test when possible.

At minimum for each feature:

```text
happy path
bad input
provider failure
missing environment variable
```

---

# 48. Financial Calculation Tests

Deterministic calculations must have unit tests.

Examples:

- RSI
- VWAP
- relative volume
- Max Pain
- scoring
- freshness
- data normalization

---

# 49. No-Fake-Data Acceptance Test

Required scenario:

```text
All market providers fail.
```

Expected:

```text
TSLA price: unavailable
```

Forbidden:

```text
fake price
cached value without date
model-generated estimate
```

---

# 50. Telegram Message Rules

Messages should prioritize fast scanning.

Use sections.

Recommended pattern:

```text
🎯 TSLA
Price
Change
Volume
Key event

📈 Strong signals
...

📉 Weak signals
...

⚠️ Risk
...

🕐 Data time
🔗 Source
```

Do not create extremely long unreadable walls of text.

---

# 51. Daily Brief Ordering

Default 07:00 report order:

```text
1. Global Market Radar
2. Tesla Priority
3. Meta Fixed Watch
4. Magnificent Seven
5. Bullish Candidates
6. Bearish Candidates
7. High-Volume Watch
8. Earnings / Events
9. Risk Alerts
10. Today's Action List
```

Tesla must not be buried below general market sections other than the opening global radar.

---

# 52. Daily Action List

Final daily output should prioritize attention, not issue trade orders.

Example structure:

```text
Priority 1 — TSLA
Why watch:
What data matters:
What invalidates current view:

Priority 2 — ...
```

---

# 53. Global Market Radar

Global Market Radar may include:

- US indices
- Taiwan
- Japan
- Hong Kong
- China
- Europe
- USD
- U.S. 10Y
- VIX
- Gold
- Bitcoin

Only display instruments with valid sourced data.

Do not invent unavailable international index values.

---

# 54. Documentation Update Rule

When architecture materially changes:

Update:

```text
ARCHITECTURE.md
```

When product behavior changes:

Update:

```text
PRD.md
```

When a major technical decision is made:

Update:

```text
DECISIONS.md
```

Do not let implementation drift away from documentation.

---

# 55. Decision Log Rule

Important choices should be recorded with:

```text
Date:
Decision:
Reason:
Alternatives considered:
Impact:
```

Examples:

- provider choice
- database migration
- options API choice
- multi-platform adapter design
- caching policy

---

# 56. Git Hygiene

Prefer one logical task per commit.

Suggested commit style:

```text
feat: add real TSLA quote
fix: correct market provider fallback
test: add no-fake-data provider failure test
docs: update Phase 1 acceptance criteria
```

Do not combine unrelated refactors with feature work.

---

# 57. Never Delete Large Existing Sections Silently

If removing code:

Explain:

- what is being removed
- why
- whether it is unused/broken
- whether functionality changes

Especially for recovered code from the old repository.

---

# 58. Do Not Restore Community Features Early

Old repository contains community/VIP concepts.

Phase 1 should NOT restore:

- verification flow
- group join approval
- VIP subscription
- payment
- tier management
- promotions
- community administration

These belong to Phase 4.

---

# 59. Do Not Build Dashboard Early

Dashboard is not a Phase 1 priority.

No front-end dashboard until Telegram MVP is stable.

Do not spend development time on visual UI before core data correctness is proven.

---


# 59A. Proprietary Logic Protection

When implementing RedwoodVest:

- keep API credentials server-side
- keep Redwood Signal Score weights server-side
- do not expose full normalized datasets unnecessarily
- return only fields needed by the channel/UI
- do not print proprietary weights into client logs
- do not place secret formulas in public static assets
- preserve report IDs and timestamps for auditability
- keep score versions for future backtesting
- design authenticated APIs so rate limits and access controls can be added cleanly

Do not assume that hiding prompts alone protects the product.

The long-term moat is the combination of historical data, proprietary ranking, alerts, personalization and distribution.

---

# 60. Definition of Done

A task is done only when:

- requested scope is implemented
- application still boots
- tests pass or test steps are provided
- no fake production market data was added
- errors are handled
- timestamps/source are preserved where required
- relevant docs are updated
- no unrelated functionality was silently changed

---

# 61. Phase 0 First Task

If starting from the new repository, the first engineering task is:

## Milestone 1 — Clean Boot

Required:

1. working `package.json`
2. working `.env.example`
3. working `.gitignore`
4. `npm install`
5. `npm start`
6. Telegram Bot connects
7. `/start` responds
8. `/quote TSLA` returns real provider data
9. provider failure shows unavailable
10. zero production mock market data

Do not continue to Options, Congress, Dark Pool, VIP or Dashboard before this milestone is verified.

---

# 62. First Response When Asked to Start Coding

When Maggie says:

```text
開始做
```

or asks to begin implementation, first respond with:

```text
I will start with Milestone 1 — Clean Boot.

Scope:
- package.json
- environment config
- Telegram boot
- market provider
- /start
- /quote TSLA
- provider failure handling

Not included:
- AI summaries
- Options
- Max Pain
- Congress
- Dark Pool
- VIP
- Dashboard

I will complete and test this milestone before moving on.
```

Then proceed.

---

# 63. Final Rule

When uncertain, choose:

```text
correctness over completeness
real data over impressive output
small stable feature over large unfinished feature
explicit unavailable over fabricated answer
```

This project previously suffered from functionality that appeared complete while underlying data or implementation was unreliable.

Do not repeat that pattern.

