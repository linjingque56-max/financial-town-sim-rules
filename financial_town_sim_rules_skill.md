---
name: financial-town-sim-rules
description: Design and build modular town-building business simulation games with embedded financial literacy, real-time supply-chain accounting, location and logistics economics, passive+pity random events, and 3-tier financial reconciliation reviews (MoneyTown / MoneyCraft operational rules). Use whenever the user asks to build or extend a financial simulation game, business sandbox, town economy engine, or accounting education gameplay.
---
# Financial Town Simulation Rules

Architect and implement an accessible, all-ages business simulation town where financial accounting principles (revenue recognition, moving-average COGS, inventory valuation, accounts receivable/payable, fixed OPEX vs. CAPEX, location rent vs. distance shipping cost, price elasticity, and debt financing) are embedded into every supply-chain transaction and post-cycle business review.

## When to Use

- The user wants to build, expand, or refactor a financial town simulation game (such as MoneyTown or MoneyCraft) or an educational business sandbox.
- The user needs a deterministic economic simulation engine that reconciles accrual-basis Profit and Loss (P&L) against cash-basis Cash Flow statements.
- The user wants gameplay mechanics combining terrain location bonuses, distance-based shipping costs, fixed capacity with price-only control, event-triggered financial narration, and passive+pity random business events.

## Steps

### 1. Required Functional Modules and UI Architecture

Structure the simulation into six coordinated functional modules:

1. **Dual-Layer Onboarding Matrix (Persona x Industry)**:
   - **Step 1 (Cognitive Tier Selection)**:
     - **Tier 0 (Beginner / Low Financial Literacy)**: Starts in an `8x8` Town (cap 6 buildings). Cash-only sales (`payLater = false`), hides complex stress-test tools, focuses on `Revenue - COGS = Gross Profit`.
     - **Tier 1 (Intermediate / Medium Literacy)**: Starts in a `10x10` County (cap 9 buildings). Unlocks credit sales (`payLater = true`), fixed operating expenses (`OPEX`), price elasticity, and the financial lens.
     - **Tier 2 (Advanced / High Literacy)**: Starts in a `12x12` Regional City (cap 13 buildings, promotable to `14x14` Metropolis). Unlocks bank credit lines, supply-chain shock tests, and A/B parallel branch comparison.
   - **Step 2 (Industry & Initial Flagship Store Selection)**:
     - Provide distinct industry supply chains (`Raw Producer [r] -> Processor [p] -> Retail Store [s] -> Household [h]` plus `Bank [b]`):
       - **Food (`food`)**: High turnover, thin margin, short credit term (`1` cycle).
       - **Pharma (`pharma`)**: High margin, medium credit term (`2` cycles), heavy receivable monitoring.
       - **Manufacturing (`mfg`)**: High capital intensity, heavy inventory value, long credit term (`3` cycles).
2. **Permanent "My Flagship Store" (`🚩 我方主店`) Ownership Anchor**:
   - Designate the initial retail store (`role === 's'`) with `isMine = true`.
   - Render a floating `🚩 我方` flag badge and golden roof crest above it on the map, a dedicated one-click focus badge (`#myShopBtn`) in the top HUD, and separate HUD readouts for `我方现金` and `我方净利` alongside town-wide totals.
3. **Streamlined Stage and 3-Tab Inspector Layout**:
   - **Top Command Bar**: Keep operational tools concise (`🔗 连接交易`, `🔍 围观视角`, `🎲 情报悬赏`, `⚡ 原料涨价15%`, `🧾 账期+2轮`, `⚖ 方案对比`, `⚙ 存档/重开`) and display a one-click cancel banner (`#modeCancelBar`) whenever a placement or connection mode is active.
   - **Standalone Floating Finance Dock**: Place `🔎 财务透视镜` and `📊 复盘` in a dedicated top-right floating dock over the map stage so core financial inspection is always one click away.
   - **Right 3-Tab Inspector Dock**:
     - `🛠️ 站点工作台` (Station Workbench: terrain/location tags, read-only fixed capacity, editable unit price and credit terms, connected route shipping rates, and highlighted valid actions).
     - `🗺️ 晋级与版图` (City Tier Promotion checklist, on-demand territory expansion, and challenge scenario switcher).
     - `📖 课堂与日志` (Itemized accounting voucher ledger stream).
4. **Contextual Action Highlighting (`.act-hi`)**:
   - When the player clicks any station on the map, highlight its ground tile and trigger a pulsing gold/pink glow on every valid button applicable to that station (`🔍 围观视角`, `🔗 连接交易`, `⚖ 方案对比`, `🏦 借款 100`).
5. **Role-Specific Animated Micro-View (`🔍 围观视角`)**:
   - Smoothly zoom the camera (`scale(2.75)`) into the selected station and open a 2x HiDPI cross-section scene tailored to the station's role (`r` farm/mine with windmill, sprinkler, and truck; `p` factory with smokestacks and moving conveyor belt through a reactor; `s` storefront with multi-shelf live stock and customer speech bubbles; `b` bank vault with gold bars; `h` cozy community house).
   - Render all signboards, customer thought bubbles, and transaction popups using native HTML DOM elements over the canvas for vector-sharp typography, and provide `◀ 上一站 / 下一站 ▶` and `-￥1 / +￥1` quick controls.
6. **Event-Triggered Financial Narrator (`🧮 账房算盘叔`)**:
   - Place a narrator widget at the bottom-left of the map stage. Keep it idle until a specific accounting event occurs for the first time (raw production capitalization, cash COGS transfer, credit sale accrual, shipping fee expense, fixed OPEX rent, loan interest, or price elasticity buyer refusal).
   - Pop up a bouncing red exclamation mark (`❗`) and speech bubble when triggered; clicking opens a blackboard modal explaining the exact double-entry bookkeeping (`借 / 贷`) and cost attribution (`生产成本 / 主营业务成本 COGS / 销售费用 / 管理费用 / 财务费用`).

### 2. Deterministic Economic Engine and 5-Stage Cycle Pipeline

Run the town economy continuously without a manual "Run 1 Round" button, offering a 4-state rhythm toggle (`🐢 0.5x 慢放对账` ~12s/cycle, `⏱ 1x 舒缓运转` ~7s/cycle, `⚡ 2x 快进` ~3.5s/cycle, and `⏸ 暂停对账`). Execute each cycle (`step(state)`) in five deterministic stages:

1. **Stage 0 - Bank Credit Line Auto-Disbursement (`b -> r/p/s`)**:
   - If a `Bank` node is wired to an enterprise whose cash falls below the safety threshold (`< ￥45`) and total debt is within the tier credit cap (`200 * tierScale`), automatically disburse a `￥60` working capital loan (`cash += 60`, `debt += 60`, bank `cash -= 60`). Record a financing voucher (`借：库存现金 / 贷：短期借款`) and note that loan principal does not count as revenue or profit.
2. **Stage 1 - Debt Interest and Fixed Operating Expenses (`OPEX` & Wage Recycling)**:
   - Deduct `2%` per-cycle interest on outstanding `debt` (`借：财务费用 / 贷：库存现金`).
   - In Tier 1+, deduct per-cycle fixed store rent and base payroll (`fixedOpex * opexMod * locationOpexMul * buffOpexMul`).
   - Recycle `75%` of paid fixed payroll into the town's `wagePool` so enterprise spending sustains household purchasing power.
3. **Stage 2 - Matured Accounts Receivable Settlement (`bills`)**:
   - For every unpaid bill where `due <= currentRound`, settle `min(bill.amount, buyer.cash)`.
   - Decrease buyer `cash` and `payable`, increase seller `cash`, and decrease seller `receivable` (`借：库存现金 / 贷：应收账款`). Emit a bad-debt liquidity warning if buyer cash is insufficient.
4. **Stage 3 - Primary Raw Material Production and Macroeconomic Wage Flow**:
   - Raw producers (`role === 'r'`) produce `q = min(effectiveCapacity, floor(cash / effectiveUnitCost))` units.
   - Deduct `q * effectiveUnitCost` from `cash` and capitalize it into inventory asset value `stockValue += q * effectiveUnitCost` (`借：原材料存货 / 贷：库存现金` - never expense unsold production as a loss).
   - Recycle `70%` of raw production expenditure into `wagePool`, then distribute `baseHouseholdIncome + (wagePool / householdCount)` to all `Household` (`h`) nodes.
5. **Stage 4 - Ordered Supply-Chain Transactions (`r -> p -> s -> h`)**:
   - Sort edges upstream-to-downstream so inputs bought in the current cycle can be processed downstream.
   - **Fixed Capacity + Step-Wise Price Elasticity**:
     - Keep base `unitOutput` fixed in the workbench (only modified by city promotion, terrain bonus, or random events). Players regulate volume and margin via unit `price` and credit `terms`.
     - Compute price ratio `ratio = price / refPrice`:
       - `ratio <= 0.85`: Promotional boost (`demand = ceil(effectiveCap * 1.4)`).
       - `0.85 < ratio <= 1.25`: Standard demand (`1.0x`).
       - `1.25 < ratio <= 1.65`: Overpriced suppression (`0.55x`).
       - `ratio > 1.65`: Buyer refusal (`demand = 0`, emit refusal warning).
     - Offering credit terms (`payLater === true`) adds `+1` unit to buyer willingness and bypasses immediate buyer cash limits.
   - **Moving-Average COGS and Distance Transport Fee**:
     - Revenue `amt = q * seller.price`.
     - Moving-average cost of goods sold `bc = q * (seller.stockValue / seller.stock)`. Deduct `q` from `stock` and `bc` from `stockValue`.
     - Deduct distance shipping fee `shipFee = q * unitShipRate` from seller cash (`借：销售费用—物流运费 / 贷：库存现金`, recycling `60%` to `wagePool`).
     - If `payLater`, book `receivable += amt`, `payable += amt`, and push a bill due at `round + terms`; otherwise transfer cash immediately.
     - If the buyer is a business (`p` or `s`), add `q` to buyer `stock` and capitalize `amt` into buyer `stockValue` so upstream price changes propagate through inventory cost.

### 3. Terrain Diversity, Expandable Territory, and Logistics Economics

1. **Six Terrain Types and Location Multipliers (`tileTerrain` & `locBonus`)**:
   - `🌊 运河水域` (River Water): Non-buildable animated waterway; adjacent land tiles gain `⚓ 临水码头` (`+15%` processor/retail capacity, reduced water shipping rate).
   - `🌉 跨河石桥` (Bridge): Non-buildable crossing tile.
   - `🌾 沃土草甸` (Fertile Meadow): Raw producer output `+25%`, unit input cost `-15%`.
   - `⛰️ 矿脉山地` (Mineral Hill): Raw producer output `+20%`, non-raw maintenance OPEX `+15%`.
   - `🛣️ 黄金商街` (Commercial Avenue): Retail customer demand `+25%`, commercial rent OPEX `+20%`.
   - `🌿 平原标准地` (Standard Plain): Neutral baseline.
2. **Distance-Based Shipping Cost (`calcEdgeRoute`)**:
   - Compute Manhattan distance `dist = |ax - bx| + |ay - by|` between connected nodes.
   - Charge `0.28` per tile per unit by default; discount to `0.15` per tile if both nodes sit on riverfront docks (`⚓ 水运优惠`), or penalize to `0.42` per tile if the midpoint crosses unbridged water (`🌊 跨水绕行`). Allow players to drag buildings to new tiles to optimize location bonuses vs. shipping distances.
3. **On-Demand Territory Expansion (`🗺️ 拓展地域`)**:
   - Allow players to spend coins (`180 * (ext + 1)`) at any time to expand the isometric grid by `+2` rings and building cap by `+3`, unlocking named districts (`东郊工坊区`, `南岸贸易港`, `西山高新园`, `北辰金融湾`, `星海自由港`) independently of city-tier promotion.

### 4. Passive + Pity Random Events and Paid Intelligence Draw

1. **Passive Random Push with Cooldown and Hard Pity**:
   - Do not use a free manual event trigger. Push events passively during active gameplay using a dual mechanism:
     - **Minimum Cooldown**: Require at least `5` cycles (`pityCount >= 5`) between passive events so gameplay remains unhurried.
     - **Progressive Probability & Hard Pity**: Roll `0.08 * (pityCount - 4)` per cycle after cooldown, and guarantee a `100%` push at `PITY_LIMIT = 9` cycles (`9周期保底必出`). Filter out `lastEvtId` to prevent consecutive duplicates.
2. **Paid Random Event Draw (`🎲 情报悬赏`)**:
   - Provide a paid button starting at `￥40` (scaling by `+50%` per purchase) that deducts cash as an information expense and immediately draws a random event from the deck.
3. **12 Multi-Choice Financial Events (`EVENTS_DECK`)**:
   - Present 3 distinct choices (`A / B / C`) plus a `↩ 暂不处理，返回小镇` option on every event card, explicitly teaching a financial trade-off:
     - `📸 网红探店爆单潮` (Inventory turnover & operating leverage)
     - `🌧️ 连日暴雨与冷链物流受阻` (Preventive OPEX vs. inventory impairment & fixed capacity loss `-1`)
     - `🏢 跨国企大客户团购询单` (High-margin credit sale vs. 85% cash discount vs. 50% deposit split)
     - `🏛️ 市政消费券与产业扶持基金` (Demand-side consumer vouchers vs. supply-side CAPEX capacity grant `+2` vs. OPEX rent cut)
     - `🧾 下游客户资金周转危机` (85% debt factoring/discounting vs. maturity extension vs. inventory-for-debt swap)
     - `🏪 商业街旺铺续约与节能改造` (CAPEX energy retrofit permanently halving fixed rent vs. floating seasonal rent vs. guild freeze)
     - `🕵️ 年度财务合规审计与纳税评级` (CPA audit fee for Class-A tax credit grant vs. input VAT deduction refund)
     - `🤖 智能流水线与工坊设备革新` (Buy automated line for permanent `+2` capacity vs. sell idle machine for `+￥65` cash and `-1` capacity)
     - `🚢 运河航道疏浚与绿色物流联盟` (Equity buy-in to permanently cut town-wide shipping rates by `40%` vs. dock warehouse subsidy)
     - `⚔️ 邻镇竞品渗透与街区价格战` (Prepaid VIP stored-value cards `预收账款` vs. brand packaging upgrade raising `refPrice +20%` vs. 85% volume promo)
     - `🏦 商业银行绿色低息授信窗口` (50% loan principal forgiveness restructuring vs. interest-free supply-chain working capital vs. deposit interest income)
     - `🌾 丰收节庆与大宗原材料锁价合约` (Forward contract locking `-25%` raw input cost vs. town harvest shopping festival `+35%` demand)

### 5. Three-Tier Financial Review Dashboard and A/B Parallel Comparison

Structure the `📊 复盘` modal into three analytical layers:

1. **Layer 1 - Side-by-Side P&L vs. Cash Flow Reconciliation**:
   - **Left Column (Accrual P&L Statement)**:
     `(+) 累计营业收入 (Sales) - (-) 已售商品成本 (COGS) = (=) 经营毛利 (Gross Profit) - (-) 店租人工/运费/利息 (OPEX) = (=) 经营净利润 (Net Profit)`.
   - **Right Column (Cash Flow Reconciliation)**:
     `期初创始现金 + (+) 经营净利润贡献 - (-) 应收账款未收占压 - (-) 存货账面净增占压 + (+) 银行借款净流入 = (=) 当前店内可用现金`.
   - **Automated Diagnostic Engine**: Compare `receivable`, `stockValue` growth, and `netProfit` against `cash` to output targeted warnings (`⚠️ 应收占压预警`, `📦 存货积压提示`, `📉 经营亏损预警`, or `✅ 现金流健康`).
2. **Layer 2 - Interactive Cycle Timeline Trace (`#repBars`)**:
   - Render a bar chart of per-cycle net profit (teal for positive, gold for cycles with active special events, pink for net losses).
   - Clicking any cycle bar displays that cycle's exact historical snapshot (revenue, COGS, OPEX/shipping, net profit, ending cash, and ending receivables).
3. **Layer 3 - Same-Baseline A/B Strategy Comparison (`⚖ 方案对比`)**:
   - Deep-clone the live town state (`JSON.parse(JSON.stringify(base))`) and run two isolated 4-cycle branches without mutating the live game:
     - **Plan A (85% Cash Discount Promo)**: Cash settlement (`payLater = false`) + `15%` price discount (triggering `+40%` elastic demand volume).
     - **Plan B (Full-Price Credit Sale)**: Full price + `2-cycle` credit terms (triggering `+1` credit demand boost).
   - Compare `新增营业收入`, `新增经营毛利`, `扣除店租运费后净利`, `期末可用现金`, `周期最低现金谷值 (minC)`, and `期末应收账款占压`.
4. **Structured Challenge Scenarios (`🎭 剧本`)**:
   - Include playable crisis scenarios alongside Free Sandbox:
     - `剧本一：账期危机救援` (Start with `￥240` receivables, only `￥25` cash, and `￥100` debt; recover to `我方现金 >= ￥280`).
     - `剧本二：原料通胀生存战` (Upstream raw costs surge `+50%` and OPEX rises `+20%`; achieve town gross profit `>= ￥450` and flagship net profit `>= ￥160`).

## Gotchas

- **Confusing Purchases with Expenses**: Never deduct raw material production or inventory purchases directly from Gross Profit at the moment of purchase. Capitalize purchases into `stockValue` and transfer them to `cogs` via moving-average cost only when units are actually sold.
- **Household Cash Deadlock**: Without wage recycling, household cash depletes after a few cycles and halts all town trade. Always recycle `60%-75%` of enterprise production, OPEX, and shipping payments back into household cash pools each cycle.
- **Unelastic Pricing Loophole**: Without step-wise price elasticity (`calcElasticDemand`), players can set unit prices to `999` and drain all buyer cash in one transaction, rendering A/B discount vs. credit comparisons meaningless.
