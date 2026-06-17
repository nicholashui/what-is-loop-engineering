# 08. 建構一個 Loop（迴圈）：實戰指南

> **本章內容：** 你將由「理解一個 Loop（迴圈）」進一步走到「*建構*一個」。
> 我們由最簡單而又能跑得起的東西開始——一段本著 Ralph Loop 精神的純 bash
> 迴圈——然後將它演化成一個生產級（production-grade）系統，具備隔離
> （isolation）、獨立評估器（Evaluator，評估器）、持久化狀態（State/Memory，
> 狀態／記憶），以及明確的停止條件（Stopping Condition，停止條件），最後再
> 提煉出設計一個 Loop 的目標、評估準則同護欄（guardrails，護欄）時通用嘅模式。
> 到本章結尾，你就有能力自己砌出一個運作中嘅 Loop，並且懂得思考點樣將佢加固。

## 由概念到建構

之前嘅章節畀咗你各個組件（第 06 章），亦示範咗佢哋點樣運轉（第 07 章）。本章
則係將一把螺絲批放入你手裡。理解一個 Loop 最快嘅方法，就係砌出能跑得起嘅最細
嘅一個，親眼睇清楚佢喺邊度脆弱，然後逐一加入結構去修補每一個弱點。

所以我哋刻意由細開始。下面第一個例子就係*最小化*嘅 Loop：寥寥幾行 bash，捕捉
咗核心概念——呼叫一個 agent、將結果餵返畀佢、再呼叫——除此之外乜都冇。佢刻意
缺少咗每一項生產層面嘅考量。將佢赤裸裸咁睇清楚，就會令嗰啲缺失嘅考量變得一目
了然，而咁樣亦正正係為咗鋪墊隨後嘅生產級版本。

## 一個最小化 Loop：Ralph 風格嘅 Bash 例子

Ralph Loop（第 05 章）係本書中每一個 Loop 嘅精神祖先：一個簡單嘅 shell 迴圈，
反覆咁將新鮮嘅 context 加埋上一次運行所產生嘅任何嘢交畀一個 agent，並一直跑到
工作完成為止。以下就係一個本著嗰種精神嘅最小化 Loop。

```bash
#!/usr/bin/env bash
# minimal-loop.sh — 一個 Ralph 風格嘅迴圈：呼叫 agent、回收佢嘅輸出、再重複。

PROMPT="Implement the tasks in TODO.md. Make the test suite pass."
context=""                                   # 將之前嘅輸出／錯誤帶住向前傳

# 迴圈條件：一直跑到出現一個哨兵字串（sentinel）標誌住工作已完成為止。
# 喺呢度，agent 喺佢認為自己完成嗰陣會寫出字面文字 "ALL DONE"。
while ! grep -q "ALL DONE" last_output.txt 2>/dev/null; do

  # Agent 呼叫：將固定嘅 prompt 加埋上一次迭代回收返嚟嘅 context
  # 一齊發送畀編碼 agent，並捕捉佢列印出嘅所有嘢
  #（佢嘅 diff 摘要、命令輸出，以及任何錯誤）。
  agent --prompt "$PROMPT

Previous attempt output and errors:
$context" > last_output.txt 2>&1

  # 將之前嘅輸出／錯誤回收入下一次迭代嘅 context。
  # 下一輪嗰陣，agent 會睇到佢頭先做咗啲乜、出咗啲乜錯，
  # 所以接住落嚟嗰次嘗試會比上一次掌握得更多資訊。
  context="$(cat last_output.txt)"

done

echo "Loop finished."
```

真正做嘢嘅只有三行，而每一行都對應番前面章節嘅一個概念：

- **迴圈條件**就係嗰個 `while ! grep -q "ALL DONE" ...` 嘅判斷。呢個就係 Loop
  最粗糙形態嘅*停止條件*（Stopping Condition）：個循環一直重複，直到 agent 嘅
  輸出入面出現一個哨兵字串為止。佢同第 07 章生命週期入面嗰個「繼續定停止」嘅
  決定係同一回事，只係簡化成單一一次字串比對。
- **Agent 呼叫**就係嗰個 `agent --prompt "..."` 嘅調用。呢個就係*生成器 agent*
  （Generator Agent，生成器 agent）親力親為做嘢——讀任務、改檔案、跑命令——
  並列印出結果。agent 列印嘅所有嘢（`> last_output.txt 2>&1`）都會被捕捉，包括
  錯誤。
- **Context 回收**就係嗰行 `context="$(cat last_output.txt)"`，再加上將
  `$context` 嵌入到下一個 prompt 入面。呢個就係令一個 Loop 成為一個*迴圈*嘅回饋
  路徑：上一次迭代嘅輸出同錯誤會被摺疊入下一次迭代嘅 context 入面，於是每一次
  嘗試都係建基於上一次，而唔係盲住嚟由零開始。

### 呢個最小化 Loop 刻意缺少咗啲乜

呢個例子之所以有用，正正係因為佢留低咗啲乜。佢經過刻意設計，缺少咗：

- **冇隔離（isolation）。** agent 直接喺工作目錄入面改檔案。冇 git worktree、
  亦冇獨立分支（branch），所以一次差勁嘅迭代可以搞污糟你個工作區，而你冇任何
  嘢可以回退（roll back）到。
- **冇獨立評估器（Evaluator）。** 所謂「完成」係 agent *自己話*完成就完成——
  嗰個 `grep` 搵 `ALL DONE` 完全信賴 agent 自己嘅聲稱。冇任何外部嘅嘢去跑測試
  或者審查個 diff，所以呢個 Loop 分辨唔到好嘅工作同自信滿滿嘅廢話。
- **冇持久化狀態（State/Memory）。** 唯一嘅記憶就係單單一個
  `last_output.txt`，而佢每一次迭代都會被覆寫。冇耐久嘅進度記錄、冇喺崩潰之後
  仍然存活嘅任務清單，亦冇辦法由停低嘅地方恢復。

佢亦冇真正嘅預算（budget，預算）護欄：如果 agent 永遠唔列印嗰個哨兵字串，個
迴圈就會永遠跑落去。呢啲唔係要原地修補嘅 bug——佢哋正正係下一節嘅生產級 Loop
所要去填補嘅缺口。請將呢個最小化版本記喺心入面，當作我哋即將要加固嘅基準線。

## 一個生產級 Loop：填補四個缺口

我哋而家將呢個最小化 Loop 演化成你可以信任佢無人值守咁運行嘅嘢。結構係一樣
嘅——呼叫生成器 agent、將結果摺疊入下一次嘗試——但上一節入面每一個缺口，都由
一件明確嘅機制去填補。每一項新增都直接對應番第 06 章嘅八個組件之一，以及第 07
章七階段生命週期嘅其中一個階段。

喺我哋建構嘅過程中，將兩套參照框架同時擺喺眼前會有幫助：

| 我哋新增嘅生產層面考量 | 第 06 章組件 | 第 07 章生命週期階段 |
|---------------------------|----------------------|----------------------------|
| 喺一個用完即棄嘅 worktree／分支入面跑每一次嘗試 | Worktrees/Isolation（worktree／隔離） | 階段 2 — 喺一個隔離嘅工作區生成 agent |
| 用一個獨立檢查去把關（gate）接受與否 | Evaluator（評估器） | 階段 4 — 評估器檢查輸出 |
| 喺一個耐久嘅檔案入面記錄進度 | State/Memory（狀態／記憶） | 階段 6 — 持久化狀態 |
| 喺成功、達到門檻或者預算上限時終止 | Stopping Condition（停止條件） | 階段 7 — 決定 繼續／生成／停止 |

生成器 agent（第 06 章）依然係處於中心嘅工作者；啟動腳本嘅觸發器
（trigger）就係自動化／觸發器（Automation/Trigger）組件；而由評估器回收返入
下一個 prompt 嘅回饋，就係生命週期階段 5（將回饋餵返並重複）。改變咗嘅地方
就係：呢啲嘢冇一樣再係隱含嘅。

### 步驟 1：將設定外部化

首先我哋將 Loop 嘅各個旋鈕由腳本入面抽出嚟，放入一個設定檔。喺呢度，**停止條件**
係以資料嘅形式存在，而唔係埋藏喺邏輯入面，咁樣就令預算可以被審計，亦易於調校。

```yaml
# loop.config.yaml — Loop 嘅宣告式（declarative）控制面板。
goal: "Implement the next unchecked task in TODO.md and make the suite green."

generator:
  command: "agent --prompt-file"   # 我哋所調用嘅生成器 agent（第 06 章）
  model: "your-coding-agent"

evaluator:
  # 評估器係獨立於生成器嘅：佢「唔會」信賴
  # agent 自己嘅匯報。佢會跑真實命令並為結果評分。
  test_command: "npm test --silent"   # 客觀把關：必須以 0 退出
  critic_command: "agent --review"     # 可選嘅第二意見／評分準則（rubric）評分器
  rubric_threshold: 0.8                # 只有當評審分數 >= 0.8 先接受

stopping_conditions:
  max_iterations: 12        # 硬性上限：迴圈次數絕不超過呢個數
  max_cost_usd: 5.00        # 預算護欄：如果開支超出呢個數就停
  stop_when_tests_pass: true # 成功退出：測試套件全綠 同埋 達到評分準則門檻

isolation:
  base_branch: "main"
  worktree_root: ".loop-worktrees"  # 每一次迭代喺呢度攞到自己嘅 worktree

state_file: "TODO.md"        # 喺迭代之間持久化嘅進度
```

呢度每一個欄位都係一項被明確化嘅生產層面考量。單單係 `stopping_conditions`
呢個區塊，就已經填補咗最小化版本嗰個「永遠跑落去」嘅缺口：呢個 Loop 而家有一個
成功退出、一個評分準則退出，*同埋*兩個獨立嘅預算上限。

### 步驟 2：協調器（orchestrator）腳本

下面嘅腳本會讀取嗰個設定檔，並運行加固咗嘅 Loop。請將內聯註解當作一次導覽
嚟讀——每一個有標籤嘅區塊都係四項新增之一。

```bash
#!/usr/bin/env bash
# production-loop.sh — 一個加固咗嘅 Loop，具備隔離、一個獨立嘅
# 評估器、持久化狀態，以及明確嘅停止條件。
set -euo pipefail

# --- 載入設定（步驟 1 嗰個宣告式控制面板）---
GOAL=$(yq '.goal' loop.config.yaml)
TEST_CMD=$(yq -r '.evaluator.test_command' loop.config.yaml)
CRITIC_CMD=$(yq -r '.evaluator.critic_command' loop.config.yaml)
THRESHOLD=$(yq -r '.evaluator.rubric_threshold' loop.config.yaml)
MAX_ITERS=$(yq -r '.stopping_conditions.max_iterations' loop.config.yaml)
MAX_COST=$(yq -r '.stopping_conditions.max_cost_usd' loop.config.yaml)
BASE_BRANCH=$(yq -r '.isolation.base_branch' loop.config.yaml)
STATE_FILE=$(yq -r '.state_file' loop.config.yaml)

iteration=0
spend=0
context=""

# === 停止條件（第 06 章組件／生命週期階段 7）===
# 迴圈標頭將預算護欄編碼成由資料驅動嘅判斷，而唔係一個
# 盲目嘅哨兵。當我哋撞到迭代上限 或者 成本上限就停。
while (( iteration < MAX_ITERS )) && (( $(echo "$spend < $MAX_COST" | bc -l) )); do
  iteration=$((iteration + 1))
  echo "=== Iteration $iteration (spend so far: \$$spend) ==="

  # === 隔離（Worktrees/Isolation 組件／生命週期階段 2）===
  # 每一次嘗試都喺佢「自己嘅」git worktree、喺佢「自己嘅」用完即棄分支上運行。
  # 一次差勁嘅迭代絕對唔可以搞污糟 main 或者旁邊嘅嘗試；我哋只係
  # 刪除嗰個 worktree。呢個就係最小化 Loop 所缺乏嘅可逆性（reversibility）。
  branch="loop/attempt-$iteration"
  worktree=".loop-worktrees/$branch"
  git worktree add -b "$branch" "$worktree" "$BASE_BRANCH"

  pushd "$worktree" >/dev/null

  # === 生成器 AGENT（生命週期階段 3：agent 採取行動以產生輸出）===
  # 我哋將固定嘅目標、「持久化嘅」狀態檔，以及
  # 由上一次評估器運行回收返嚟嘅回饋（生命週期階段 5）一齊交畀 agent。
  cat > .loop-prompt.txt <<EOF
Goal: $GOAL

Current progress (persisted state):
$(cat "../../$STATE_FILE")

Feedback from the previous attempt's evaluation:
$context
EOF
  agent --prompt-file .loop-prompt.txt

  # === 評估器（Evaluator 組件／生命週期階段 4）===
  # 獨立驗證。agent 嘅意見喺呢度唔算數。
  # 把關 1 係客觀嘅（測試必須通過）；把關 2 係一個評審／評分準則分數。
  if $TEST_CMD; then
    tests_pass=true
  else
    tests_pass=false
  fi
  score=$($CRITIC_CMD --rubric ../../rubric.md --format score)  # 例如 0.0–1.0

  # 累計開支，用嚟畀預算停止條件。
  spend=$(echo "$spend + $(agent --last-cost)" | bc -l)

  # === 決策點（生命週期階段 7：繼續／生成／停止）===
  if [ "$tests_pass" = true ] && (( $(echo "$score >= $THRESHOLD" | bc -l) )); then
    # 成功退出：兩個把關都滿足。將呢次嘗試晉升並停止。
    popd >/dev/null
    git merge --no-ff "$branch" -m "Loop: accepted attempt $iteration"
    echo "Accepted on iteration $iteration (score $score). Stopping."
    break
  fi

  # === 持久化狀態（State/Memory 組件／生命週期階段 6）===
  # 記錄學到咗啲乜，令「下一次」迭代可以恢復而唔係
  # 盲住嚟由零開始，亦令進度可以喺崩潰之後存活。嗰啲回饋亦會
  # 被回收入下一個 prompt（生命週期階段 5）。
  context="tests_pass=$tests_pass; critic_score=$score; see notes below"
  {
    echo ""
    echo "## Iteration $iteration result"
    echo "- tests passed: $tests_pass"
    echo "- critic score: $score (threshold $THRESHOLD)"
  } >> "../../$STATE_FILE"

  popd >/dev/null
  # 丟棄被拒絕嗰次嘗試嘅 worktree；分支會保留低嚟做事後鑑證（forensics）。
  git worktree remove --force "$worktree"
done

# 如果我哋係喺冇經過「成功」break 嘅情況下跌出個迴圈，咁就係一個停止條件
#（最大迭代次數或者預算）令我哋停低——係一次受控嘅停止，而唔係失控狂奔。
echo "Loop ended after $iteration iteration(s); total spend \$$spend."
```

### 每一個缺口係點樣被填補

將四項考量逐一對照番最小化基準線：

- **隔離。** 最小化 Loop 係原地咁喺工作目錄入面改嘢。喺呢度，每一次
  迭代都攞到一個全新嘅 `git worktree`，喺一個用完即棄嘅 `loop/attempt-N` 分支
  上（Worktrees/Isolation，生命週期階段 2）。失敗嘅嘗試只係被簡單咁移除；只有
  *被接受*嘅嘗試先會被合併（merge）返入基底分支。agent 做嘅任何嘢都唔可以搞
  污糟 `main` 或者旁邊嘅嘗試。
- **獨立評估器。** 「完成」唔再係 agent 自己講就算數。一個客觀
  把關（`npm test` 必須以 0 退出）同一個評審／評分準則把關（`score >= 0.8`）
  *兩者*都必須通過，個工作先會被接受（Evaluator，生命週期階段 4）。呢個係抵禦
  「自信但錯誤」輸出嘅主要防線。
- **持久化狀態。** 進度喺每一次迭代之後都會被附加到一個耐久嘅 `TODO.md`
  入面，而嗰個檔案亦會被餵返入下一個 prompt（State/Memory，生命週期
  階段 6）。呢個 Loop 可以喺崩潰之後恢復，而每一次嘗試都係建基於一段被記錄落
  嚟嘅歷史，而唔係一個被反覆覆寫嘅單一草稿檔。
- **停止條件。** 終止係明確而且多管齊下嘅：成功（兩個
  把關都綠）、用盡 `max_iterations`，或者超出 `max_cost_usd`
  （Stopping Condition，生命週期階段 7）。最小化 Loop 嗰種「永遠跑落去」嘅失效
  模式喺呢度結構上係無可能發生嘅。

留意吓有啲乜*冇*改變：個心跳依然係 生成 → 評估 → 餵返 → 重複。生產級 Loop
唔係一個有別於 Ralph 風格腳本嘅諗法——佢係同一個諗法，只係每一個隱含嘅假設都
被替換成一個明確、可檢視嘅組件。呢個就係 Loop Engineering（Loop 工程）嘅全部
要旨：攞第 06 章嘅各個部件，將佢哋沿住第 07 章嘅生命週期排列好，並令每一個
控制面板都變成你睇得到、調得到、信得過嘅嘢。

## 由各個組件建構一個 Loop：一條實戰路徑

上面兩個例子展示咗目的地。本節就係路線。你唔會坐低一次過寫好個生產級
Loop——你係慢慢將佢養大，一次一個組件，循住一個能令你喺每一步都保持掌控嘅次序。
指導原則好簡單：**由能對住你真實任務跑得起嘅最細嘅 Loop 開始，然後只有喺缺少
某個組件真係造成痛楚嗰陣，先至加入嗰個組件。**

由第 06 章八個組件去裝配一個 Loop，一個可靠嘅次序係：

1. **由生成器 agent 加一個停止條件開始。** 呢個就係第一節嗰個最小化
   Loop：一個 `while` 迴圈、一次 agent 呼叫，以及一個粗糙嘅停止判斷。即刻
   對住一個真實任務跑佢。你而家唔係要追求正確——你係要*睇到個心跳*（生成 →
   餵返 → 重複）喺你部機上由頭到尾運轉得起。
2. **跟住加入隔離，喺你信任個 Loop 處理任何真實嘢之前。** 一旦
   agent 無人值守咁改檔案，一次差勁嘅迭代就可以搞污糟你個工作區。將每一次嘗試
   包喺一個用完即棄分支上嘅 git worktree 入面（Worktrees/Isolation），令每一次
   迭代都可逆。呢個係成個系統入面最平嘅保險；要早啲加。
3. **加入評估器。** 呢個係單一最高槓桿嘅組件，因為佢正正係
   分隔開一個 Loop 同一個隨機輸出產生器嘅嘢。由你手頭上已經有嘅最強客觀信號
   開始——通常就係你現有嘅測試套件——之後先至疊加一個評審／評分準則把關（喺
   下面*評估準則*嗰度會講）。
4. **加入持久化狀態（State/Memory）。** 一旦各次迭代都被隔離同把關咗，
   就畀個 Loop 一份耐久嘅記憶——一個 `TODO.md`、草稿簿（scratchpad），或者
   專案看板（project board）——令進度可以喺崩潰之後存活，亦令每一次嘗試都建基
   於一段被記錄落嚟嘅歷史，而唔係盲住嚟由零開始。
5. **將停止條件收緊成明確嘅護欄。** 用由資料驅動嘅上限去取代
   嗰個粗糙嘅哨兵：成功退出、迭代上限，以及一個預算上限（見下面*護欄*嗰度）。
6. **只有喺你嘅任務需要嗰陣，先至加入餘下嘅組件。**
   自動化／觸發器（cron、webhook、slash-command）、連接器（Connectors，例如
   issue tracker、chat、source-control host），以及技能／知識
   （Skills/Knowledge，例如一個 skills 檔、編碼標準）呢啲組件，正正係將一個你
   親手去跑嘅 Loop 變成一個自己跑嘅 Loop 嘅嘢。佢哋對一個無人值守嘅 Loop 嚟講
   係必不可少嘅，但佢哋對正確性冇任何貢獻——所以喺「生成—評估」核心變得值得
   信任之前，將佢哋押後。

呢度嘅紀律係**減法，而唔係加法**：你每加入一個組件，就係多咗一個你而家要去
理解同維護嘅控制面板，所以喺每一步嘅正確問題係「省略呢樣嘢會唔會喺我嘅任務上
造成一個具體嘅失效？」如果一個喺本機對住一個全綠測試套件就跑得起嘅 Loop 已經
做到件事，咁你就唔需要一個 webhook 觸發器或者一個 chat 連接器。只係喺你嘅任務
真係推你去到嗰一步嗰陣，先至向住個生產級例子去成長。

## 設計目標、評估準則同護欄嘅模式

有三個設計決定，決定一個 Loop 嘅成敗多過任何其他嘢：你畀佢嘅**目標**、決定佢
嘅工作幾時先算可接受嘅**評估準則**，以及界定佢行為嘅**護欄**。各個組件只係
喉管；呢三樣嘢先係工程判斷所在之處。下面嘅模式就係最值得優先去攞嚟用嘅。

### 設計目標

一個 Loop 係無人值守咁運行嘅，所以佢嘅目標必須**清晰而且可檢查
（checkable）**——要咁樣措辭，令到生成器 agent 自己嘅樂觀以外嘅某啲嘢，可以
判斷得到佢究竟係咪達成咗。

- **令目標可以由外部驗證。** 優先選一個佢嘅完成可以由機器
  確認嘅目標。*「令 `npm test` 以 0 退出，並且冇被略過嘅測試」*係可檢查嘅；
  *「改善代碼」*就唔係。一個評估器檢查唔到嘅目標，就係一個 Loop 完成唔到嘅
  目標。
- **將範圍界定喺一個連貫嘅工作單位。** 一個被導向去
  *「重構付款模組，令所有現有測試仍然通過」*嘅 Loop 會收斂；一個瞄準
  *「將代碼庫現代化」*嘅就會漫遊。狹窄嘅目標令回饋路徑喺每一次迭代都有啲具體
  嘢可以推。
- **將目標表述成一個結果，而唔係一個程序。** 描述你想要嘅
  最終狀態（「測試套件係綠嘅，而新嘅 endpoint 喺成功時回傳 201」），而唔係達到
  嗰個狀態嘅步驟。agent 揀步驟；Loop 驗證結果。

一個咁樣寫嘅目標可以直接放入生產級例子嗰個設定嘅 `goal` 欄位：

```yaml
# 一個清晰、可檢查、以結果表述嘅目標——係一個 Loop 真係完成得到嗰種。
goal: >
  Add pagination to GET /orders so it accepts ?page and ?limit query params,
  returns at most `limit` items, and keeps the existing /orders tests green.
# 可檢查：現有 + 新嘅測試就係「完成」嘅客觀信號。
# 有界定：一個 endpoint、一個行為——而唔係「改善 orders API」。
```

### 設計評估準則

評估器（第 06 章）係 Loop 抵禦「自信但錯誤」輸出嘅防線。一個值得信任嘅評估器
共有一個決定性特徵：**佢獨立於生成器 agent**——佢絕不接受 agent 嘅自我匯報
作為成功嘅證明。好嘅評估通常會疊加兩種檢查：

- **一個難以造假嘅客觀把關。** 測試、型別檢查、linter、一個
  build 步驟，或者一個 schema 驗證——任何回傳一個 agent 講極都過唔到嘅明確
  通過／失敗嘅嘢。呢個係值得信任嘅評估嘅骨幹；由呢度開始。
- **一個評審／評分準則把關，去處理客觀檢查所遺漏嘅嘢。** 一個測試套件
  確認行為，但對可讀性、安全性，或者個改動係咪符合意圖，就講得好少。一個
  第二 agent 對住一份寫低嘅評分準則為個 diff 評分，就可以補番嗰個缺口。將評分
  準則保持明確並為佢做版本控制，咁樣，個 Loop 被要求達到嘅標準就係可見而且
  可調嘅。

兩個把關組合成一條接受規則——*只有喺客觀把關通過 同埋 評分準則
清過佢嘅門檻嗰陣先接受*——正正就係生產級例子嗰個決策點：

```python
# 接受規則：一個 agent 自己無法自我認證嘅獨立兩重把關檢查。
def is_acceptable(result, rubric_score, threshold=0.8):
    # 把關 1 — 客觀而且無法造假：真實嘅測試套件必須通過。
    if not result.tests_passed:
        return False
    # 把關 2 — 質性：一個獨立嘅評審必須清過評分準則門檻。
    if rubric_score < threshold:
        return False
    return True  # 兩個把關都滿足 -> 晉升呢次嘗試並停止。
```

有兩個模式喺實務上令一個評估器值得信任。**喺 agent 工作嗰個同一個隔離
worktree 入面跑客觀把關**，咁你評嘅就正正係佢所產生嘅嘢。同埋**刻意咁設定評分
準則門檻**：太低，劣質嘢（slop）就會混過去；太高，個 Loop 就會耗盡佢成個預算去
追一個佢永遠達唔到嘅分數。一個你揦得出道理嘅門檻——並隨住你嘅學習去調整——
好過一個隨意嘅門檻。

### 設計護欄

護欄就係停止條件（第 06 章）加上隔離同升級（escalation）規則，佢哋令一個無人
值守嘅 Loop 唔會搞出破壞。喺評估決定*工作是否良好*嘅地方，護欄就決定*個 Loop
被容許去到幾遠*先至有啲嘢去阻止佢。值得攞嚟用嘅模式有：

- **永遠設多過一個預算。** 同時為迭代次數*同埋*開支設上限。任何一個
  單獨用都有盲點——一個又平又卡住嘅 Loop 可以原地空轉直到耗盡佢嘅迭代次數；
  一次昂貴嘅單一嘗試可以一鋪過爆咗個金額預算。兩者一齊，就將兩種失控模式都
  框住。
- **隔離每一次嘗試，並只喺成功時先合併。** 可逆性*正正就係*一個
  護欄：如果一次被拒絕嘅嘗試係活喺一個用完即棄嘅 worktree 入面，咁一次差勁嘅
  迭代就只係令你做一次 `git worktree remove`，而唔係一個被搞污糟嘅 `main`。
- **定義一條明確嘅升級至人手（escalation-to-human）路徑。** 一個 Loop 應該
  識得幾時要停低並*去問*。當佢喺未獲接受嘅情況下耗盡佢嘅預算、喺冇改善嘅情況下
  振盪（oscillates），或者觸動咗一個高風險嘅警報線（tripwire）嗰陣，佢就必須
  交畀人手處理，而唔係靜靜雞失敗或者繼續燒 token。

呢啲規則同一個升級掛鈎（escalation hook）一齊，活喺設定嘅
`stopping_conditions` 區塊：

```yaml
# 護欄：分層嘅預算，加上一個明確嘅交畀人手。
stopping_conditions:
  max_iterations: 12          # 為一個卡住但平嘅、原地空轉嘅 Loop 設上限
  max_cost_usd: 5.00          # 獨立於迭代次數，為一個昂貴嘅 Loop 設上限
  stop_when_tests_pass: true  # 成功退出：客觀 + 評分準則把關兩者都清過

escalation:
  # 當一個預算喺「未獲接受」嘅情況下被撞到，唔好靜靜雞失敗——交畀人手。
  on_budget_exhausted: notify-human   # 例如 開一個 issue／貼去 chat（Connectors）
  on_repeated_no_progress: 3          # 連續 N 次同一個失敗 -> 升級
```

留意吓本節入面每一個模式都連返去第 06 章嘅一個組件，以及第 07 章生命週期嘅一個
階段：目標塑造嗰啲被*發現*嘅工作（階段 1），評估準則就係*評估器檢查*（階段 4），
而護欄就係*繼續／生成／停止嘅決定*（階段 7）。將一個 Loop 設計得好，到頭來，
就係將呢三樣嘢設計得好——各個組件只係畀每一個決定一個安身之所。

## 重點摘要

- 理解一個 Loop 最快嘅方法，就係砌出能跑得起嘅最細嘅一個，
  觀察佢喺邊度爆煲，然後加入結構去修補每一個弱點。
- 一個**最小化 Ralph 風格 Loop** 只係一個 bash `while` 迴圈，佢呼叫一個生成器
  agent、捕捉佢嘅輸出同錯誤，並將佢哋回收入下一次迭代嘅 context，直到一個
  哨兵停止條件被滿足為止。
- 嗰個最小化 Loop 刻意**冇隔離、冇獨立評估器、亦
  冇持久化狀態**——咁就令嗰啲缺失嘅考量成為隨後嘅生產級 Loop 嘅動機。
- **生產級 Loop** 保持住同一個 生成 → 評估 → 餵返
  嘅心跳，但用一個明確嘅組件去填補每一個缺口：每次嘗試一個 **git worktree**
  （隔離）、一個對住測試*同埋*一個評分準則分數把關嘅**獨立評估器**、一個喺
  崩潰之後仍然存活並通知下一次嘗試嘅**持久化 `TODO.md`**，以及令失控迴圈
  變得無可能嘅**明確停止條件**（成功、最大迭代次數、預算上限）。
- 每一項生產層面嘅新增都一對一咁對應番一個**第 06 章組件**同一個
  **第 07 章生命週期階段**——建構一個 Loop 大致上就係將嗰啲部件變得
  明確而且可檢視。
- **用減法去建構一個 Loop，一次一個組件：** 由一個生成器
  agent 加一個停止條件開始，喺信任佢之前加入**隔離**，跟住加一個
  **評估器**，再加**持久化狀態**，再加明確嘅預算護欄——只有喺任務真係需要
  嗰陣，先至加入自動化／觸發器、連接器，同技能／知識組件。
- 一個 Loop 嘅成敗，繫於三個設計決定多過任何組件選擇：
  一個清晰、可檢查、以結果表述嘅**目標**；建基於一個*獨立*客觀把關（測試）加上
  一個評審／評分準則把關嘅**評估準則**；以及**護欄**——分層嘅迭代*同埋*預算
  上限、每次嘗試嘅隔離，以及一條明確嘅升級至人手路徑——令一個無人值守嘅 Loop
  識得幾時要停低並去問。

---
[< 上一章：Loop 生命週期](07-loop-lifecycle_hk.md) | [目錄](README_hk.md) | [下一章：益處及何時使用 Loop >](09-benefits_hk.md)
