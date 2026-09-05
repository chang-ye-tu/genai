# 第 10 單元（11/30 – 12/13）｜後訓練與終身學習

> 本頁內容同步張貼於 iLearn「第 10 單元」區塊的「學習指引」頁面。
> 本單元流程：**① 看教材 → ② 線上測驗**（發布當天即可作答，不受討論限制）**→ ③ 討論區發文＋回覆 → ④ 討論簽到**（完成討論後開放，1 題 1 分）。順序可自行安排。
> 本單元所有活動截止：**12/13（日）23:59**。

**本單元其他重要事項**
- **HW6「微調：分類微調、LoRA 與遺忘」** 11/30 開放，**12/20（日）23:59 截止**（寬限至 12/27）：本單元的 LoRA 與災難性遺忘會親手做一次。
- HW5 截止：**12/06（日）23:59**（寬限至 12/13）。

## 本單元學習目標

完成本單元後，你應該能夠：

1. 說明課程所謂「後訓練」（在別人打造好的通用模型上持續更新參數）的意思，區分它與 prompt、RAG 等不改參數的方法，並舉出新知識、新技能、新概念、unlearning 四類需求；知道 post-training 與 continual／lifelong learning 有交集但不等同。
2. 用 Reliability、Generality、Locality 三個指標評量一次後訓練是否成功，並說明災難性遺忘為什麼可視為 locality 失敗的典型例子（知識編輯評測 KnowEdit 另把 portability 獨立為第四項）。
3. 解釋「最好的後訓練就是不要後訓練」的理由與決策順序。
4. 描述用 gradient descent 微調時減少遺忘的做法：限制可更新的範圍（LoRA：凍結原權重、只訓練新增的低秩矩陣）、對參數的偏好、把 locality 寫進 loss（experience replay、以模型自問自答產生 pseudo-replay 資料）。
5. 說明 model editing（ROME）、model merging（task vector 的加、減、類比）與 test-time training 的基本原理與限制。

## 教材

| # | 教材 | 類型 | 長度 | 來源 |
|---|------|------|------|------|
| 1 | 【必看】第 8 講：通用模型的終身學習（Fine-tuning, Model Editing, Model Merging, Test-Time Training）<https://youtu.be/EnWz5XuOnIQ> | 影片 | 120 分 | 李宏毅《生成式人工智慧與機器學習導論 2025》 |
| 2 | 【必讀】投影片 Post.pdf <https://speech.ee.ntu.edu.tw/~hylee/GenAI-ML/2025-fall-course-data/Post.pdf> | 投影片 | 103 頁 | 同上 |
| 3 | 【選看】機器終身學習（二）— 災難性遺忘的克服之道 <https://youtu.be/Y9Jay_vxOsM> | 影片 | 36 分 | 李宏毅《機器學習 2021》 |
| 4 | 【選看】生成式人工智慧的後訓練（Post-Training）與遺忘問題（從 33:20 起）<https://youtu.be/Z6b5-77EfGk?t=2000> | 影片片段 | — | 李宏毅《生成式 AI 時代下的機器學習 2025》 |
| 5 | 【選看】人工智慧的微創手術 — 淺談 Model Editing（從 23:04 起）<https://youtu.be/9HPsz7F0mJg?t=1384> | 影片片段 | — | 同上 |

## 重點提示（Highlights）

- **後訓練＝更新參數**：課程把 post-training、continual learning、life-long learning 三個詞放在一起，指的都是「在別人打造、學習歷程不明的 foundation／chat model 上繼續更新參數」，不是用 prompt 改變行為。嚴格說，post-training 是預訓練之後所有調整（SFT、RLHF、DPO…）的統稱，continual／lifelong learning 則是資料與任務隨時間陸續到來、必須處理 stability–plasticity 取捨的範式；兩者有交集但不等同，本單元談的是它們的交集：部署後的持續調適。
- **四類需求**：新知識（總統換人）、新技能（注音文）、新概念（感情建議）、**unlearning**（忘掉版權、隱私內容）。
- **三個評量指標**：Reliability（目標達成）、Generality（舉一反三：reversibility、portability）、**Locality**（無關內容不動）。這是課程的簡化版；知識編輯評測 KnowEdit 列的是四項——reliability、generality、locality、**portability**：改寫問法（paraphrase）屬 generality，反向關係、組合推理屬 portability。**災難性遺忘＝手術成功、病人卻死了**，是 locality 失敗最典型的例子（兩者是「例子與類別」的關係，不是同義詞）。
- **最好的後訓練就是不要後訓練**：先試 in-context learning、RAG；「新知識給了沒用」常是 prompt 沒告訴模型怎麼用；確定不改參數做不到，才動手術。
- **案例**：教 LLaMA-2-Chat 中文 → 會中文但忘了拒絕有害要求；教文字 LLM 聽聲音 → 越訓練越準，卻忘了遵循 JSON 格式指令。
- **為何沒做到 locality？因為你也沒要求**：loss 只含目標句子的 token。減少遺忘的三招：限制可更新的範圍（**LoRA**：凍結原本的權重，只訓練另外加在旁邊的低秩矩陣 A、B——learns less, forgets less，但範圍太小可能達不到目標）；對參數加偏好（重要參數變動要小）；把無關問題的資料放進 loss（**experience replay**；沒有原始資料就讓模型**自問自答**——投影片說「說出你的訓練資料」，實際上模型生成的是分佈相近的替代資料，也就是 pseudo-replay，不是真正的原始訓練資料）。
- **Model editing（ROME）**：找到與知識相關的那一層 key→value，只改那裡的 W；要加約束保住其他 key（locality）。
- **Model merging**：task vector τ = θ_後訓練 − θ_原始，可**相加**（chat vector：中文 base + alignment 向量，不用資料不用訓練）、**相減**（用於 unlearning：抑制某項行為，但不保證相關知識完全無法再被抽取）、**類比**（A:B = C:D，沒有 D 的資料也能學 D）；不總是成功。
- **Test-time training**：在測試時用眼前的輸入（無標籤、無回饋）微調，例如最小化 entropy、pretext task、SUTA；c.f. reasoning 是不改參數的 test-time computing。Standard TTT 學習不累積；Continuous TTT 會累積但有遺忘，需 fast／slow update。

## 實例（Worked example）

目標：讓某個現成模型知道「魯夫吃的是尼卡果實」。

**先設計驗收題**（訓練前就要準備好）：
- Reliability：「魯夫吃的是什麼果實？」→ 尼卡果實。
- Generality：「魯夫的惡魔果實是哪一種？」→ 尼卡果實（換個問法，paraphrase）；「誰吃了尼卡果實？」→ 魯夫（reversibility）；「喬巴的海賊團團長吃了什麼果實？」→ 尼卡果實（portability；後兩者在 KnowEdit 歸為第四項 portability）。
- Locality：「喬巴吃了什麼果實？」→ 應維持原本答案；「誰是美國總統？」→ 不能被改變。

**方案一：直接 SFT 微調**。訓練資料「User：魯夫吃了什麼惡魔果實？ AI：尼卡果實」，loss 只涵蓋這句的 token，模型可能學成「吃了什麼果實」一律接「尼卡」→ locality 失敗。補救：混入「喬巴吃了什麼果實 → 人人果實」「誰是美國總統 → …」等無關資料（experience replay），或改用 LoRA 限制可更新的範圍。

**方案二：先不動手術**。把海賊王第 1044 話的段落放進 prompt（RAG），並告訴模型「請依據以下資料回答」——若這樣就能答對，根本不需要後訓練。

**方案三：model merging 思路**（用在「教中文又要保安全」的情境）：θ_中文base + (θ_chat − θ_base)，把 alignment 向量加回來，不需要任何訓練資料。

## 動手練習（不計分，約 15 分鐘）

在任一聊天機器人測試「不動參數」的替代方案：

1. 自己編一段虛構的新知識（例如「逢甲大學 2026 年新成立的社團『量子飛盤社』社長是陳小明」），先直接問「量子飛盤社社長是誰」確認模型不知道。
2. 把這段資料貼進對話再問一次；接著改成「請只依據以下資料回答，資料中沒有的請說不知道」，比較兩種問法的差異——這就是老師說的「告訴模型如何使用新資訊」。
3. 再問一個無關問題（例如「台灣最高的山」），觀察 in-context 的做法是否影響其他答案：參數沒有改變，所以有「參數層面」的 locality；但塞進上下文的資料仍可能干擾無關問題的回答（例如模型硬把飛盤社扯進去）。把觀察記下來，討論時會用到。

## 本單元討論（討論區「U10 教會它、忘掉它：後訓練決策」）

- 完成條件（系統自動追蹤）：**發表 1 篇主題 + 回覆 1 則同學的貼文**（回覆任何一位同學即可）。完成後「討論簽到」（1 題、1 分）自動開放，計入討論參與 5%（討論只在第 1、2、3、5、6、9、10、11、13 單元進行，9 次簽到取最佳 7 次）。
- 截止：12/13（日）23:59。

**討論題**：舉一個你希望讓某個現成模型「學會新東西」或「忘掉某事」的具體情境（新知識／新技能／新概念／unlearning 擇一，可以和你在 HW6 想微調的任務有關）。請說明：

1. 這件事能不能用不改參數的方法（RAG、in-context learning、prompt 技巧）解決？你怎麼判斷？
2. 如果一定要後訓練，你會選 fine-tune（含 LoRA）、model editing、model merging 或 test-time training 中的哪一種？理由？
3. 寫出你的驗收計畫：各舉一個 Reliability、Generality、Locality 的測試問題，並說明你最擔心哪一種失敗、打算怎麼預防（例如 experience replay、自問自答資料）。

**回覆同學時**請幫對方多想一個 Locality 測試問題（一個「不該被改變」的問題），並判斷對方選的方法會不會通過。

**教師引導重點**（供教師發文與週末小結使用）：
- 常見錯誤：把「用 prompt 給資料」也叫後訓練；忘了 unlearning 也是後訓練的一種。
- 反覆強調「最好的後訓練就是不要後訓練」：先問不改參數做不做得到。
- 把同學的驗收計畫連到 HW6：LoRA 之後的接龍能力變化就是 locality 的實測。
- 週末小結時挑 2–3 個驗收題設計完整的貼文表揚，並示範一次「手術成功、病人卻死了」。

## 本單元測驗（「U10 測驗」）

- 題庫隨機抽 10 題選擇題，作答 **1 次**；每題答錯可看提示再試（最多 3 次，每次扣該題 1/3 分）。正確答案於測驗關閉後才會顯示。
- 交卷後會顯示每個選項的解說。範圍：第 8 講影片與投影片。
- 截止：12/13（日）23:59。本學期 12 次測驗取最佳 10 次計分。

## 本單元時間預估

| 活動 | 時間 |
|------|------|
| 必看影片 | 120 分 |
| 投影片瀏覽 | 20 分 |
| 動手練習 | 15 分 |
| 討論區發文與回覆 | 25 分 |
| 線上測驗 | 20 分 |
| **合計** | **約 3 小時 20 分** |

## 延伸資源

- LoRA <https://arxiv.org/abs/2106.09685>；*LoRA Learns Less and Forgets Less* <https://arxiv.org/abs/2405.09673>。
- ROME：*Locating and Editing Factual Associations in GPT* <https://arxiv.org/abs/2202.05262>；KnowEdit 知識編輯資源 <https://zjunlp.github.io/project/KnowEdit/>。
- Task arithmetic <https://arxiv.org/abs/2212.04089>；Chat vector（教 LLaMA-2-Chat 中文）<https://arxiv.org/abs/2310.04799>；MergeKit <https://github.com/arcee-ai/mergekit>。
- *Self-Distillation Bridges Distribution Gap in Language Model Fine-Tuning* <https://arxiv.org/abs/2402.13669>。
- Test-time training：TENT <https://arxiv.org/abs/2006.10726>；SUTA <https://arxiv.org/abs/2203.14222>。
