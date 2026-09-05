# 第 3 單元（10/05 – 10/18）｜從語言模型到 AI Agent：上下文工程與深度思考

> 本頁內容同步張貼於 iLearn「第 3 單元」區塊的「學習指引」頁面。
> 本單元流程：**① 看教材 → ② 線上測驗**（發布當天即可作答，不受討論限制）**→ ③ 討論區發文＋回覆 → ④ 討論簽到**（完成討論後開放，1 題 1 分）。順序可自行安排。
> 本單元所有活動截止：**10/18（日）23:59**（HW1 寬限期也在 10/18 結束，請提早安排）。

## 本單元學習目標

完成本單元後，你應該能夠：

1. 說明 Context Engineering 與 Prompt Engineering 的異同，以及「只訓練人類、不訓練模型」的意涵。
2. 列出 context 的組成（user prompt、system prompt、對話歷史、長期記憶、外部資訊、工具使用、reasoning）並說明各自的角色。
3. 用文字接龍解釋語言模型如何「使用工具」（含 Computer Use）與「深度思考」（reasoning）。
4. 區分一問一答、agentic workflow 與 AI agent，並說明 agent 為什麼會面臨輸入過長的問題（lost in the middle、context rot）。
5. 說明 Context Engineering 的三個基本招數：挑選（select）、壓縮（compress）、多代理（multi-agent），並舉例應用。

## 教材

| # | 教材 | 類型 | 長度 | 來源 |
|---|------|------|------|------|
| 1 | 【必看】第 2 講：上下文工程 (Context Engineering) — AI Agent 背後的關鍵技術 <https://youtu.be/lVdajtNpaGI> | 影片 | 110 分 | 李宏毅《生成式人工智慧與機器學習導論 2025》 |
| 2 | 【必讀】投影片 Agent.pdf <https://speech.ee.ntu.edu.tw/~hylee/GenAI-ML/2025-fall-course-data/Agent.pdf> | 投影片 | 77 頁 | 同上 |
| 3 | 【選看】一堂課搞懂 AI Agent 的原理 <https://youtu.be/M2Yg1kwPpts>（投影片 <https://speech.ee.ntu.edu.tw/~hylee/ml/ml2025-course-data/ai_agent.pdf>） | 影片 | 101 分 | 李宏毅《生成式 AI 時代下的機器學習 2025》 |
| 4 | 【選看】DeepSeek-R1 這類大型語言模型是如何進行「深度思考」（Reasoning）的？ <https://youtu.be/bJFtcwLSNxI>（投影片 <https://speech.ee.ntu.edu.tw/~hylee/ml/ml2025-course-data/reasoning.pdf>） | 影片 | 78 分 | 同上 |
| 5 | 【選看】解剖小龍蝦 — 以 OpenClaw 為例介紹 AI Agent 的運作原理 <https://youtu.be/2rcJdFuNbZQ>（投影片 <https://speech.ee.ntu.edu.tw/~hylee/ml/ml2026-course-data/intro.pdf>） | 影片 | 83 分 | 李宏毅《機器學習 2026》 |
| 6 | 【選看】AI Agent (1/3)：核心技術 Context Engineering 基本概念解說 <https://youtu.be/urwDLyNa9FU> | 影片 | 53 分 | 同上 |

## 重點提示（Highlights）

- **改變參數叫「訓練」；本單元只訓練人類**：Context Engineering 是在「假設模型沒問題」的前提下準備合適的輸入——每個人都做得到。
- **Context Engineering vs Prompt Engineering**：相同概念、不同重點。**Prompt Engineering** 關注一段輸入怎麼寫：輸入格式、神奇咒語；**Context Engineering** 關注系統性、動態地選取與管理模型所見的 context（放什麼進去、清什麼出來，常由語言模型自動完成）。核心目標：**避免塞爆 context**。
- **Context 裡有什麼**：user prompt（任務說明、指引、條件、風格、前提、範例——語言模型不會讀心術）、**system prompt**（服務商設定的身分、規則、安全限制、知識截止……）、對話歷史（短期記憶）、長期記憶、其他資料源的資訊（RAG）、工具使用、reasoning。**參數不是 context**。
- **使用工具是文字接龍**：模型接出 `<tool>…</tool>` 只是文字，外部程式執行後把 `<output>…</output>` 放回輸入再接龍；Computer Use 的工具是滑鼠鍵盤、輸入是螢幕畫面。
- **Reasoning**：模型自己產生的「腦內小劇場」（規劃、嘗試、驗證），也占 context，使用者可能看不到。
- **一問一答 → Agentic Workflow（固定 SOP）→ AI Agent（自己決定步驟、靈活調整）**；從語言模型的角度，agent 就是 `sys prompt, obs 1, action 1, obs 2, …` 一直接龍。
- **挑戰：輸入過長**。能讀上百萬 token ≠ 能讀懂；RAG 資料太多反而看不下去；**Lost in the Middle**（比較記得開頭與結尾）；context rot。
- **三招**：**Select**（RAG、reranking、Tool RAG、Memory RAG——不要把所有工具說明與所有記憶都塞進去）、**Compress**（把歷史摘要；重要結果放進長期儲存日後 RAG；遙遠的記憶隨風而逝）、**Multi-Agent**（每個 agent 的 context 只有自己的任務，lead 只收回報）。
- **OpenClaw（選看）**：「AI Agent 中不是 AI 的部分」——身分檔、記憶檔、SKILL 按需讀取、壓縮、心跳，都是 Context Engineering；模型「說記住了」但沒用工具寫檔＝記了個寂寞；`exec` 能執行任何指令 → 安全風險（第 11 單元再談）。

## 實例（Worked example）

工具使用的通用方法（投影片的高雄氣溫例子）：

1. **System prompt**：「如果遇到根據你的知識無法回答的問題，使用工具。把工具指令放在 `<tool>` 與 `</tool>` 之間，使用後你會在 `<output>` 與 `</output>` 之間得到輸出。可用工具：`Temperature(location, time)`。」
2. **User prompt**：「2025 年 3 月 10 日下午 2:00，高雄氣溫如何？」
3. **模型接龍**：`<tool>Temperature('高雄', '2025.03.10 14:00')</tool>`——這只是一串文字。
4. **外部程式**偵測到 `<tool>`，真的去查溫度，把 `<output>攝氏 32 度</output>` 接到輸入後面。
5. **模型繼續接龍**：「2025 年 3 月 10 日下午 2:00，高雄的氣溫為攝氏 32 度。」——使用者只看到這一句。

把「Temperature」換成「搜尋」「讀檔案」「執行指令」「移動滑鼠」，就是搜尋、RAG、OpenClaw 與 Computer Use。

## 動手練習（不計分，約 15 分鐘）

1. 開一個全新對話，問模型：「請用三句話說明你是誰、你的知識截止日期、以及你被要求遵守的規則。」觀察 system prompt 的影子。
2. 找一篇約 2000 字的文章，在中間插入一句與文章無關的資訊（例如「密語是『紫色鯨魚』」），把整篇貼給模型後問「密語是什麼？」；再把那句話移到開頭或結尾試一次，比較 lost in the middle 是否發生。
3. 如果你有可用的 agent 工具（ChatGPT 的 agent 模式、Gemini CLI、Claude Code、OpenClaw 等），請它完成一個三步驟以上的任務，觀察它的每一步 action 與 observation；沒有的話，請一般聊天模型「列出完成這個任務的步驟並逐步執行」，比較差異。

## 本單元討論（討論區「U03 幫你的 agent 規劃 context」）

- 完成條件（系統自動追蹤）：**發表 1 篇主題 + 回覆 1 則同學的貼文**（回覆任何一位同學即可）。完成後「討論簽到」（1 題、1 分）自動開放，計入討論參與 5%（討論只在第 1、2、3、5、6、9、10、11、13 單元進行，9 次簽到取最佳 7 次）。
- 截止：10/18（日）23:59。

**討論題**：選一個你希望 AI agent 幫你完成的多步驟任務（例如安排一趟三天兩夜的旅行、整理一門課的筆記並產生複習題、幫社團寫活動企劃並排程）。請寫出：

1. 它的 context 需要包含哪些成分（至少四類，對應本單元的清單：user prompt、system prompt、對話歷史、記憶、外部資訊、工具、reasoning），各放什麼；
2. 你會用哪一招（select／compress／multi-agent）避免塞爆 context，具體怎麼做；
3. 你認為最可能出錯的一步是什麼，為什麼。

**回覆同學時**請指出對方的設計中有哪一段內容其實不必放進 context（或放了會出問題），並提出一個具體的改法。

**教師引導重點**（供教師發文與週末小結使用）：
- 一再回到「避免塞爆 context」：每個設計都問「這段內容一定要在 context 裡嗎？」
- 把同學的任務對應到三招：select（RAG、Tool RAG）、compress（摘要歷史）、multi-agent（拆子任務）。
- 提醒工具使用只是文字接龍，外部程式才真正執行，所以「最可能出錯的一步」常在工具輸出被誤讀。
- 週末小結時挑 2–3 個規劃清楚的貼文表揚，並示範一次 lost in the middle 怎麼發生。

## 本單元測驗（「U03 測驗」）

- 題庫隨機抽 10 題選擇題，作答 **1 次**；每題答錯可看提示再試（最多 3 次，每次扣該題 1/3 分）。正確答案於測驗關閉後才會顯示。
- 交卷後會顯示每個選項的解說。範圍：第 2 講影片與投影片為主；選看影片只考概念（例如「OpenClaw 是 AI Agent 中不是 AI 的部分」）。
- 截止：10/18（日）23:59。本學期 12 次測驗取最佳 10 次計分。

## 本單元其他事項

- **HW1「語言模型初體驗」10/11（日）23:59 截止，寬限至 10/18**：上傳 notebook 後才能作答「HW1 實作測驗」。
- 實作作業 HW2「RAG 與評量」10/05 已開放（10/25 截止，寬限至 11/01）；HW3「從零實作注意力與 GPT 模型」10/19 開放（11/08 截止）。

## 本單元時間預估

| 活動 | 時間 |
|------|------|
| 必看影片 | 110 分 |
| 投影片瀏覽 | 20 分 |
| 動手練習 | 15 分 |
| 討論區發文與回覆 | 25 分 |
| 線上測驗 | 20 分 |
| **合計** | **約 3 小時 10 分** |

## 延伸資源

- How Contexts Fail and How to Fix Them <https://www.dbreunig.com/2025/06/22/how-contexts-fail-and-how-to-fix-them.html>（投影片引用的部落格）。
- Lost in the Middle <https://arxiv.org/abs/2307.03172>；Context Rot <https://research.trychroma.com/context-rot>。
- Gemini CLI <https://github.com/google-gemini/gemini-cli>（開源 agent，可實際觀察 obs／action 迴圈）。
