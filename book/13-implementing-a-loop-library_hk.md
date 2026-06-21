# 13. 實作一個 Loop 庫

> **本章內容：** 你會學識點樣*建造*一個屬於自己嘅 Loop 庫（Loop Library）——一個經過策展、納入版本控制嘅可重用 Loop 定義集合，俾你嘅團隊去瀏覽、複製同運行。我哋會定義一個單一嘅 **Loop 條目 schema（綱要）**，令每一個 Loop 都用同一種方式去描述，鋪排一個儲存佢哋嘅儲存庫結構，展示點樣令個庫可被搜尋，並且——最重要嘅一步——展示點樣用第 08 章嗰套機制，將一個靜態嘅庫條目變成一個*運行中*嘅 Loop。讀完之後，你就有能力攞第 12 章嗰 45 個形態（或者你自己嘅），將佢哋變成一個可執行嘅庫。

## 由一個目錄走向一個庫

第 12 章係一個*目錄*——一次唯讀嘅導覽，看其他人寫嘅 Loop。而一個 **Loop 庫** 係更強嘅嘢：佢係你自己儲存庫入面一件活生生嘅產物，當中每一個條目都係結構化資料，而唔係散文，於是佢可以被搜尋、被驗證、被納入版本控制，並且——最關鍵嗰步——被*執行*。呢個分別，同本書一直返嚟覆去講嗰個一樣：散文*描述*一個 Loop；而一個庫就*定義*佢，精確到足以令一部機器運行得起。

實作一個庫，意味住要做三個決定，然後將佢哋接駁埋一齊：

1. **一個 schema（綱要）**——一種一致嘅形狀去描述每一個 Loop，令啲條目可以互相比較、又可被機器讀取。
2. **一個儲存（store）**——啲條目住喺邊度，以及佢哋點樣被組織、被版本化、被審查。
3. **一個 runner（運行器）**——一個庫條目點樣變成一個由第 06 章組件砌成嘅運行中 Loop。

本章其餘部分就按呢個次序逐個講。呢度所有嘢都直接建基於第 08 章嗰個生產級 Loop；個庫，就係令你毋須每次都重寫編排（orchestration），都可以定義出好多個咁樣嘅 Loop 嘅嘢。

## Loop 條目 schema

一個庫嘅基礎，就係一份大家一致同意嘅描述，講清楚一個 Loop *係乜*。請留意，第 12 章每一個條目都隱含噉答咗同樣嘅問題——邊個寫、佢做乜、佢點樣運作、佢幾時停。將呢啲問題變成明確嘅欄位，你就有咗一個 schema。一個好嘅 schema 會映照返八個組件（第 06 章），令一個庫條目同一個可運行嘅 Loop 一對一噉對應。

下面係一個用 Markdown 檔案加 YAML frontmatter（前置資料）表達嘅 schema——既俾人睇得明，*又*俾機器解析得到，而呢個正正就係一個庫條目所需要嘅嘢：

```markdown
---
# --- 身分與發現（人同搜尋點樣搵到呢個 loop）---
id: production-error-sweep            # 穩定、唯一、kebab-case
name: The production error sweep      # 對人友善嘅標題
author: Matthew Berman                # 標注
category: engineering                 # engineering|evaluation|design|content|operations
tags: [observability, bugfix, pull-request]
maturity: stable                      # draft|experimental|stable|deprecated
version: 1.2.0                        # 「呢個定義」嘅語意化版本號

# --- loop 契約（對應第 06 章嘅組件）---
purpose: >
  喺生產環境搵出、修正並驗證有可行動性嘅錯誤，然後開出一個 PR。

trigger:                              # 自動化／觸發器組件
  type: schedule                      # schedule|event|manual
  spec: "0 * * * *"                   # 每小時；type 為 manual 時忽略

generator:                            # 生成器 agent 組件
  prompt_file: prompts/production-error-sweep.md
  inputs: [log_source, repo]          # 呼叫者必須提供嘅參數

evaluator:                            # 評估器組件（獨立於生成器）
  objective_gate: "npm test"          # 必須以 0 結束
  critic: optional                    # 評分準則／評論者關卡（如有）
  rubric_file: rubrics/bugfix.md

isolation:                            # worktrees／隔離組件
  strategy: git-worktree
  base_branch: main

state:                                # 狀態／記憶組件
  file: .loop-state/production-error-sweep.md

stopping_conditions:                  # 停止條件組件
  success: "可行動嘅錯誤已修正 並且 測試通過 並且 已開 PR"
  no_op: "搵唔到可行動嘅錯誤"
  max_iterations: 10
  max_cost_usd: 4.00

connectors: [github, log-provider]    # 連接器組件
skills: [skills/triage.md]            # 技能／知識組件

source: https://signals.forwardfuture.ai/loop-library/
---

## 佢做乜

俾瀏覽個庫嘅人睇嘅散文描述。呢段保持簡短；上面嗰個 frontmatter 先係
俾機器讀取嘅契約。

## 護欄與註記

運行呢個 loop 嘅人應該知道嘅嘢：成本預期、波及範圍、何時*唔應該*用、
所需嘅權限。
```

有兩個設計原則令呢個 schema 行得通：

- **每一個組件都係一個欄位。** `trigger`、`generator`、`evaluator`、`isolation`、`state`、`stopping_conditions`、`connectors` 同 `skills` 呢幾個鍵，正正就係第 06 章嗰八個組件。所以一個庫條目，就係*一份完整、宣告式嘅 Loop 描述*——關於點樣運行佢嘅任何嘢，都唔會住喺呢個檔案以外。
- **prompt 係一個引用，而非內嵌文字。** 實際嘅 agent 指令住喺一個獨立嘅 `prompt_file`。咁樣可以令 prompt 跨 Loop 重用、可以獨立做 diff（差異比對），並且毋須觸碰 Loop 嘅元資料就可以編輯。

呢個就係第 08 章嗰個 `loop.config.yaml` 嘅諗法，加以推廣：唔係一個 config 對一個 loop，而係一個 schema，俾庫入面每一個 loop 去遵循。

### 必需 vs. 可選欄位

唔係每一個 loop 都需要每一個欄位，但有一小撮核心欄位應該係**強制**嘅，咁個庫先永遠唔會含有一個運行唔到嘅條目：

| 欄位 | 必需？ | 點解 |
|-------|----------|-----|
| `id`、`name`、`author`、`category` | 必需 | 身分、標注、發現 |
| `purpose` | 必需 | 一句話、可被檢查嘅意圖 |
| `generator.prompt_file` | 必需 | 冇生成器就冇 loop |
| `stopping_conditions` | 必需 | 一個無界限嘅 loop 係一個缺陷，唔係一個條目 |
| `evaluator` | 強烈建議 | 冇佢，「完成」就只係 agent 自己講 |
| `trigger`、`isolation`、`state` | 可選（有預設）| 有合理預設值；可逐 loop 覆寫 |
| `connectors`、`skills`、`tags`、`source` | 可選 | 豐富化同整合 |

一個驗證器（下面會講）應該拒絕任何缺咗必需欄位嘅條目。最重要嘅一條規則：**冇任何條目可以缺少停止條件。** 呢一條約束，正正就係令一個由眾多自主 Loop 組成嘅庫，唔會變成一個由眾多失控進程組成嘅庫嘅關鍵。

## 儲存個庫

定咗 schema 之後，儲存就好直接：一個喺版本控制之下、由條目檔案組成嘅目錄。將純檔案放入 git，你就免費得到歷史、審查、分支同 diff——而呢啲特性，正正就係令 Loop 本身值得信任嘅同一批特性。

一個可行嘅儲存庫結構：

```
loop-library/
├── README.md                  # 生成嘅索引（見「令佢可被搜尋」）
├── schema/
│   └── loop.schema.json       # frontmatter 必須滿足嘅 JSON Schema
├── loops/
│   ├── engineering/
│   │   ├── production-error-sweep.md
│   │   ├── docs-sweep.md
│   │   └── test-stabilizer.md
│   ├── evaluation/
│   │   └── self-improving-champion.md
│   ├── design/
│   │   └── ui-ux-score.md
│   ├── content/
│   └── operations/
├── prompts/                   # 條目所引用嘅生成器 prompt
│   └── production-error-sweep.md
├── rubrics/                   # 條目所引用嘅評論者／評估器評分準則
│   └── bugfix.md
└── skills/                    # 共享嘅技能／知識檔案
    └── triage.md
```

重要嘅組織選擇：

- **一個 loop 一個檔案，按類別分資料夾。** 呢個映照返第 12 章嗰五大類，並且就算完全冇任何工具，個庫都仍然可以當作一棵純目錄樹去瀏覽。
- **prompt、評分準則同技能都係共享、第一等公民嘅檔案。** 因為條目係*引用*佢哋，所以同一個評分準則或技能可以支撐好多個 loop，而改善其中一個，就會改善每一個用佢嘅 loop。
- **一個可被機器檢查嘅 schema 住喺儲存庫入面。** `schema/loop.schema.json` 令 CI 可以喺每一次改動時驗證每一個條目——個庫自己嘅品質關卡，而呢個關卡本身，就只係將一個評估器套用喺個庫之上。

### 喺 CI 中驗證條目

個庫應該以佢所宣揚嘅標準去要求自己。一個喺合併前對照 schema 去驗證每一個條目嘅檢查，係最低門檻：

```bash
#!/usr/bin/env bash
# validate-library.sh —— 拒絕任何打破契約嘅 loop 條目。
set -euo pipefail

fail=0
for entry in loops/**/*.md; do
  # 抽取 YAML frontmatter 並對照 JSON Schema 驗證。
  if ! yq --front-matter=extract '.' "$entry" \
        | ajv validate -s schema/loop.schema.json --data - >/dev/null 2>&1; then
    echo "SCHEMA FAIL: $entry"
    fail=1
  fi

  # 硬規則：每一個條目都必須宣告一個停止條件。
  if ! yq -e --front-matter=extract '.stopping_conditions' "$entry" >/dev/null 2>&1; then
    echo "NO STOPPING CONDITION: $entry"
    fail=1
  fi

  # 被引用嘅 prompt 檔案必須存在，否則個條目運行唔到。
  prompt=$(yq -r --front-matter=extract '.generator.prompt_file' "$entry")
  if [ ! -f "$prompt" ]; then
    echo "MISSING PROMPT: $entry -> $prompt"
    fail=1
  fi
done

exit $fail
```

呢個腳本將 schema 一節嗰三條不可妥協嘅原則編成程式碼：有效嘅形狀、一個停止條件，以及一個存在嘅 prompt。將佢放入 CI，個庫就永遠唔可以合併一個運行唔到、或者會永遠運行落去嘅條目。

## 令個庫可被搜尋

一個庫，只有當人哋可以快速搵到啱用嗰個 Loop 嗰陣，先至無愧於佢個名。因為每一個條目都帶住結構化嘅 frontmatter，所以搜尋大致上就係讀取嗰啲元資料、再將佢投射成可瀏覽嘅嘢嘅問題。

兩個低成本嘅機制，幾乎涵蓋咗所有需要：

- **一個生成嘅索引。** 一個細小嘅腳本讀取每一個條目嘅 frontmatter，寫出一個分組、有連結嘅 `README.md`——標題、作者、目的、標籤——就好似第 12 章嗰個目錄噉。喺 CI 中重新生成佢，個索引就永遠唔會過時。
- **分面、基於標籤嘅篩選。** 因為 `category`、`tags` 同 `maturity` 都係結構化欄位，你毋須一個資料庫都可以篩選個庫：「顯示標籤為 `bugfix` 嘅 stable 工程 loop」就係一句對 frontmatter 嘅查詢。

一個最精簡嘅索引生成器：

```python
# build_index.py —— 由 loop 嘅 frontmatter 重新生成 README.md。
import pathlib, frontmatter   # `python-frontmatter`
from collections import defaultdict

by_category = defaultdict(list)
for path in pathlib.Path("loops").rglob("*.md"):
    post = frontmatter.load(path)
    by_category[post["category"]].append((post, path))

lines = ["# Loop Library\n", "_生成嘅索引——請勿手動編輯。_\n"]
for category in sorted(by_category):
    lines.append(f"\n## {category.title()} Loops\n")
    for post, path in sorted(by_category[category], key=lambda p: p[0]["name"]):
        # 每一行：名稱（連結）、作者、一句話目的、標籤。
        tags = ", ".join(post.get("tags", []))
        lines.append(
            f"- [{post['name']}]({path}) — *{post['author']}* — "
            f"{post['purpose'].strip()}  `{tags}`"
        )

pathlib.Path("README.md").write_text("\n".join(lines))
```

對於一個細團隊嚟講，咁樣已經足夠;結構化嘅 frontmatter 意味住你之後可以升級去全文搜尋或者一個 web UI，而毋須改動條目本身。嗰啲元資料*就係*個搜尋索引。

## 將一個庫條目變成一個運行中嘅 Loop

呢一步，正正就係分隔一個真正嘅庫同一個花俏筆記資料夾嘅地方。一個庫條目係一份*宣告*；運行佢，就意味住將嗰份宣告交俾一個 **runner（運行器）**，由佢去組裝第 06 章嗰啲組件、並驅動第 07 章嗰條生命週期——亦即係第 08 章嗰個一模一樣嘅生產級 Loop，只係而家由條目去參數化，而唔係寫死。

runner 嘅工作，係一次忠實嘅翻譯，逐個欄位嚟：

| schema 欄位 | runner 用佢嚟做乜 | 第 07 章階段 |
|--------------|------------------------------|------------------|
| `trigger` | 決定幾時開始（cron、webhook 或人手呼叫）| 階段 1 —— 發現／觸發工作 |
| `isolation` | 每次嘗試喺一個用完即棄嘅分支上建立一個 git worktree | 階段 2 —— 喺隔離中孵化 |
| `generator.prompt_file` + `inputs` | 渲染 prompt 並呼叫 agent | 階段 3 —— agent 行動 |
| `evaluator` | 運行客觀關卡，然後評論者／評分準則關卡 | 階段 4 —— 評估 |
| `state.file` | 持久化進度，並將佢循環入下一個 prompt | 階段 5–6 —— 回饋、持久化 |
| `stopping_conditions` | 決定 繼續／合併並停止／升級 | 階段 7 —— 決定 |

一個消費庫條目嘅 runner 係咁樣——留意佢就係第 08 章嗰個編排器，只係將佢嘅常數換成由條目讀取：

```bash
#!/usr/bin/env bash
# run-loop.sh <entry.md> [key=value ...] —— 將任何庫條目當作一個 Loop 嚟運行。
set -euo pipefail

ENTRY="$1"; shift
fm() { yq -r --front-matter=extract "$1" "$ENTRY"; }   # 讀取一個 frontmatter 欄位

# --- 由條目解析出 loop 契約（冇任何寫死嘅 loop 邏輯）---
GOAL=$(fm '.purpose')
PROMPT_FILE=$(fm '.generator.prompt_file')
TEST_CMD=$(fm '.evaluator.objective_gate')
RUBRIC=$(fm '.evaluator.rubric_file // ""')
BASE=$(fm '.isolation.base_branch // "main"')
STATE=$(fm '.state.file')
MAX_ITERS=$(fm '.stopping_conditions.max_iterations')
MAX_COST=$(fm '.stopping_conditions.max_cost_usd')

iteration=0; spend=0; context=""
mkdir -p "$(dirname "$STATE")"; touch "$STATE"

# --- 同第 08 章一樣嘅心跳，而家完全由條目驅動 ---
while (( iteration < MAX_ITERS )) && (( $(echo "$spend < $MAX_COST" | bc -l) )); do
  iteration=$((iteration + 1))
  branch="loop/$(fm '.id')-attempt-$iteration"
  worktree=".loop-worktrees/$branch"
  git worktree add -b "$branch" "$worktree" "$BASE"          # 隔離（階段 2）
  pushd "$worktree" >/dev/null

  # 生成器（階段 3）：渲染被引用嘅 prompt + 狀態 + 回饋 + 呼叫者輸入。
  { cat "../../$PROMPT_FILE";
    echo; echo "Goal: $GOAL";
    echo "Caller inputs: $*";
    echo "Progress so far:"; cat "../../$STATE";
    echo "Previous evaluation feedback: $context";
  } > .loop-prompt.txt
  agent --prompt-file .loop-prompt.txt

  # 評估器（階段 4）：獨立嘅客觀關卡，然後可選嘅評分準則關卡。
  if eval "$TEST_CMD"; then tests_pass=true; else tests_pass=false; fi
  score=1.0
  [ -n "$RUBRIC" ] && score=$(agent --review --rubric "../../$RUBRIC" --format score)
  spend=$(echo "$spend + $(agent --last-cost)" | bc -l)

  # 決定（階段 7）：接受並停止，或者持久化回饋並繼續。
  if [ "$tests_pass" = true ] && (( $(echo "$score >= 0.8" | bc -l) )); then
    popd >/dev/null
    git merge --no-ff "$branch" -m "Loop $(fm '.id'): accepted attempt $iteration"
    echo "Accepted on iteration $iteration."; break
  fi
  context="tests_pass=$tests_pass score=$score"
  printf '\n## Iteration %s\n- tests: %s\n- score: %s\n' \
    "$iteration" "$tests_pass" "$score" >> "../../$STATE"   # 持久化（階段 5-6）
  popd >/dev/null
  git worktree remove --force "$worktree"
done
echo "Loop $(fm '.id') ended after $iteration iteration(s); spend \$$spend."
```

重點唔係呢個特定嘅腳本——重點係嗰份*分離*。runner 只寫**一次**，並包含咗所有嘅編排；而庫條目就只包含*宣告*。加多一百個 loop，runner 都唔使改。呢個就係 schema 帶嚟嘅回報：一個 loop 變成咗資料，而運行任何一個 loop，都係同一條程式碼路徑。

### 參數化嘅 loop

第 12 章好多目錄 loop 都有空格——「N 次成功之後停」、「[repository task]」、「[quality threshold]」。一個庫令呢啲空格變成第一等公民嘅**輸入**（`generator.inputs` 欄位），喺運行時供應：

```bash
# 將同一個庫條目針對唔同目標運行，毋須改動條目本身。
./run-loop.sh loops/engineering/production-error-sweep.md \
    log_source=prod-us-east repo=checkout-service

./run-loop.sh loops/evaluation/quality-streak.md  streak_n=20
```

一個參數化得宜嘅條目，可以取代成打近乎重複嘅複本。呢個就係一個庫點樣喺增長之中保持細小嘅方法：你加嘅係*參數*，而唔係分叉（fork）。

## 通往你第一個庫嘅務實路徑

你毋須事前就建好成個庫，就好似你喺第 08 章都唔係事前建好成個 Loop 一樣。用同樣嘅減法方式去培育佢：

1. **晉升一個行得通嘅 loop。** 攞一個你已經喺第 08 章運行緊嘅 Loop，用 schema 寫出嚟，並將佢作為第一個條目提交。一個真實嘅條目，勝過十個臆測嘅條目。
2. **runner 只寫一次。** 將你第 08 章嘅編排器推廣成一個會讀取條目嘅 runner。透過將第一個條目跑一次過佢嚟證明佢。
3. **將 schema 驗證器加入 CI。** 而家個庫就唔可以接受一個破損或者無界限嘅條目——喺第二個條目到嚟之前，品質門檻就已經就位。
4. **只有喺你真正重用之際先晉升 loop。** 當你發現自己喺度重新建造一個你之前建過嘅 loop 形態——即第 12 章其中一種形態——嗰個就係將佢以參數化方式加入個庫嘅訊號。
5. **生成索引。** 一旦你有咗一小撮條目，就接駁個索引生成器，令個庫可被瀏覽，團隊亦可以發現有乜嘢存在。

紀律同本書其他地方一樣：**當「冇將一個 loop 放入個庫」會造成真正嘅重複嗰陣，先將佢加入——唔好早過呢個時候。** 一個由三個你團隊真正會運行嘅 loop 組成嘅庫，比一個由四十五個你複製咗但從未執行過嘅 loop 組成嘅庫，更有價值。

## 重點摘要

- 一個 **Loop 庫** 將第 12 章嗰個唯讀目錄，變成一件活生生、可執行嘅產物：每一個 loop 都變成結構化、納入版本控制嘅資料，而唔係散文。
- 實作一個，意味住要將三個決定接駁埋一齊——一個 **schema**（每一個 loop 一種形狀）、一個 **儲存**（納入版本控制嘅檔案），以及一個 **runner**（執行任何條目）。
- 個 **schema** 應該令每一個第 06 章組件都成為一個明確嘅欄位（`trigger`、`generator`、`evaluator`、`isolation`、`state`、`stopping_conditions`、`connectors`、`skills`），並將 prompt 以引用檔案嘅形式儲存；一小撮核心欄位係必需嘅，而且**冇任何條目可以缺少停止條件。**
- 個 **儲存** 係一個 git 目錄，由「一個 loop 一個檔案」、按類別分資料夾嘅條目組成，配上共享嘅 prompt／評分準則／技能，以及一個喺 CI 中驗證嘅 JSON Schema——即個庫將一個評估器套用喺自己之上。
- 搜尋幾乎免費噉由結構化嘅 frontmatter 得來：一個生成嘅索引，加上按標籤／類別／成熟度嘅篩選，毋須任何資料庫。
- 個 **runner** 只寫一次，將一個條目嘅欄位翻譯到第 07 章嘅生命週期上——佢就係第 08 章嗰個生產級 Loop，只係將佢嘅常數換成由條目讀取，於是加 loop 永遠唔會改動編排。
- 參數化嘅 **輸入** 將目錄嗰啲 `[空格]` 變成運行時嘅參數，於是一個條目可以取代好多個近乎重複嘅複本，而個庫亦喺增長之中保持細小。
- 用**減法**去建個庫：晉升一個行得通嘅 loop、寫個 runner、加入 CI 驗證，然後只有喺重用真正有需要嗰陣先加條目。

---
[< 上一章：Loop 庫](12-the-loop-library_hk.md) | [目錄](README_hk.md) | [下一章：策展、營運與擴展一個 Loop 庫 >](14-operating-a-loop-library_hk.md)
