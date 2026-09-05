# 第 5 單元（10/19 – 11/01）｜解剖大型語言模型：AI 的腦科學

> 本頁內容同步張貼於 iLearn「第 5 單元」區塊的「學習指引」頁面。
> 本單元流程：**① 看教材 → ② 線上測驗**（發布當天即可作答，不受討論限制）**→ ③ 討論區發文＋回覆 → ④ 討論簽到**（完成討論後開放，1 題 1 分）。順序可自行安排。
> 本單元所有活動截止：**11/01（日）23:59**。10/26（一）為補假日，第 6 單元教材於 11/02 發布。
> 本單元同時開放 HW3（見下方「本單元其他事項」）。

## 本單元學習目標

完成本單元後，你應該能夠：

1. 描述語言模型從 prompt 到下一個 token 的完整流程：tokenization → embedding 表 → 多層 Transformer → LM head → softmax（含 temperature）。
2. 區分 token embedding 與 contextualized embedding，並舉例說明 embedding 空間中「方向」的含意。
3. 解釋 attention layer 的兩個步驟（query／key 找關聯、value 加權彙整），以及 positional embedding、multi-head、causal attention 各自的用途。
4. 說明 feed-forward layer 的運算，以及它作為「key-value 記憶體」儲存事實的觀點。
5. 舉例說明三種「看見模型在想什麼」的工具：拒絕向量（representation steering）、Logit Lens、Patchscopes，並說出 probing 與「直接問模型」的限制。

## 教材

| # | 教材 | 類型 | 長度 | 來源 |
|---|------|------|------|------|
| 1 | 【必看】第 3 講：解剖大型語言模型 <https://youtu.be/8iFvM7WUUs8> | 影片 | 124 分 | 李宏毅《生成式人工智慧與機器學習導論 2025》 |
| 2 | 【必讀】投影片 LLMunderstand.pdf <https://speech.ee.ntu.edu.tw/~hylee/GenAI-ML/2025-fall-course-data/LLMunderstand.pdf> | 投影片 | 68 頁 | 同上 |
| 3 | 【必看】How might LLMs store facts <https://youtu.be/9-Jl0dxWQs8> | 影片 | 22 分 | 3Blue1Brown（可開中文字幕） |
| 4 | 【選看】AI 的腦科學 — 語言模型內部運作機制剖析 <https://youtu.be/Xnil63UDW2o>（投影片 <https://speech.ee.ntu.edu.tw/~hylee/ml/ml2025-course-data/model_inside.pdf>） | 影片 | 109 分 | 李宏毅《生成式 AI 時代下的機器學習 2025》 |
| 5 | 【選看】第 11 講：大型語言模型在「想」什麼呢？— 淺談大型語言模型的可解釋性 <https://youtu.be/rZzfqkfZhY8>（投影片 <https://speech.ee.ntu.edu.tw/~hylee/genai/2024-spring-course-data/0503/0503_explain.pdf>） | 影片 | 45 分 | 李宏毅《生成式 AI 導論 2024》 |
| 6 | 【選用】老師的範例程式（觀察開源模型內部）<https://colab.research.google.com/drive/1uU9aW020lhaqk236E_my4ObiCzzc0eKn?usp=sharing> | Colab | – | 李宏毅 |

## 重點提示（Highlights）

- 本講**沒有訓練任何模型**，全部是「觀察已訓練好的模型」——這是可解釋性研究的基本態度。
- 流程：文字 → **tokenization**（token 編號）→ **embedding 表**查出向量 → 一層一層 Transformer（**many layers = deep learning**）→ **LM head（unembedding）** 算出每個 token 的分數（logit）→ **softmax** 變成機率。
- **Temperature**：分數除以 T 再做 softmax；T 越大分佈越平均（創意模式），T 越小越集中。
- **Token embedding** 只看 token 本身（同 token 同向量、意思近則向量近）；經過各層後成為 **contextualized embedding**（「來吃蘋果」與「使用蘋果」的「果」不再相同）。
- Embedding 空間**特定方向有特定含意**：Emb(冷) − Emb(cold) + Emb(small) ≈ Emb(小)；投影到低維還能看到句法樹與世界地圖。
- **拒絕向量**：某一層「拒絕情況的平均 − 沒拒絕情況的平均」；加到正常請求上模型就拒絕，從有害請求上減掉模型就回答。哪一層？每層都試。這類方法叫 representation／activation engineering（steering）。
- **Logit Lens**：對每一層做 unembedding，看每層「暫時最想輸出」的 token（發現 LLaMA 內部偏向英文）。**Patchscopes**：把某層表示貼進另一個問句，讓模型「說出」向量裡的內容。
- **Transformer = self-attention + feed-forward**（老師的說法：「不是發明 attention，而是拿掉 attention 以外的東西」——指拿掉 RNN；Transformer 仍保有 feed-forward、殘差連接、層正規化與位置編碼）。
- **Attention 兩步驟**：① query·key 內積找出會影響「果」意思的 token（原論文還會除以 √d_k 再 softmax，投影片省略）；② softmax 權重 × value 加總。**Positional embedding** 補上順序與距離（投影片示範的「每個位置一個向量、相加」是原始做法；現代 LLM 多用 RoPE）；**multi-head** 抓不同面向（形容詞、數量）；**causal attention** 只看左邊；輸入越長運算越多，所以有長度上限。
- **Feed-forward layer**：多層全連接網路，投影片畫成 ReLU(Wx + b) 再接第二層 W′（原始 Transformer 為 max(0, xW₁+b₁)W₂+b₂），第一層的每一列是一個神經元；「Transformer Feed-Forward Layers Are Key-Value Memories」——3Blue1Brown 用「Michael Jordan 打籃球」示範 MLP 如何儲存事實，並介紹 superposition。
- （2024 第 11 講）**Probing**：在某層接一個小分類器測試該層有沒有某種資訊；**直接問模型**它在想什麼很方便，但「說出來的話不保證可信」。

## 實例（Worked example）

句子「兩顆青蘋果」，看 attention 如何處理最後的「果」：

1. **找關聯**：「果」的 query 與每個 token 的 key 做內積，得到分數：兩 0、顆 −0.5、青 2.5、蘋 3、果 1（投影片省略了除以 √d_k 的縮放）。
2. **softmax**：分數變成權重 0.03、0.02、0.33、0.55、0.07——「蘋」和「青」影響最大。
3. **彙整**：用這些權重把各 token 的 value 加權相加，得到新的「果」的表示（已經知道自己是「青蘋果」的果）。
4. **另一個 head** 專門找數量：權重變成 兩 0.80、顆 0.20、其餘 0；兩個 head 的結果再由 W_O 合併。
5. 若沒有 positional embedding，「青山綠水紅蘋果」裡的「青」對「果」的相關性分數（內積）會和上面算成一樣的 2.5——內積只看內容、不看距離（整句的 softmax 權重仍會因其他 token 不同而不同）；所以每個 token 都要先加上位置向量。

## 動手練習（不計分，約 15 分鐘）

1. 到線上分詞工具（例如 <https://tiktokenizer.vercel.app/> 或 <https://platform.openai.com/tokenizer>）貼上「今天天氣真好！」與一句英文，看看各被切成幾個 token——中文字並不總是一字一 token。
2. 到任一聊天機器人問兩次：「『我的蘋果又當機了』裡的『蘋果』是什麼意思？」與「『我的蘋果被蟲咬了』裡的『蘋果』是什麼意思？」，體會 contextualized embedding 的效果。
3. 請機器人做一件它會拒絕的事（例如「幫我寫一封詐騙信」），再問「你為什麼拒絕？」——記下它的解釋，想想這個解釋可不可信（2024 第 11 講的最後一頁）。

## 本單元討論（討論區「U05 模型「理解」語意了嗎？」）

- 完成條件（系統自動追蹤）：**發表 1 篇主題 + 回覆 1 則同學的貼文**（回覆任何一位同學即可）。完成後「討論簽到」（1 題、1 分）自動開放，計入討論參與 5%（討論只在第 1、2、3、5、6、9、10、11、13 單元進行，9 次簽到取最佳 7 次）。
- 截止：11/01（日）23:59。

**討論題**：本單元看到了模型內部的幾種證據——同一個字在不同語境下表示不同、embedding 空間中有城市地理分佈、存在可以加減的「拒絕向量」、Logit Lens 顯示翻譯時中間層先出現英文。請選一個立場：

- 「這些證據代表模型某種程度上『理解』語意」，或
- 「這些只是統計上的相關性，不能算理解」。

用本單元**至少兩個具體證據**支持你的立場（每個證據用一兩句話說明它顯示了什麼），並提出**一個你想用 Logit Lens、Patchscopes 或拒絕向量去檢驗的問題**（例如：「模型回答台灣地理問題時，中間層是不是先想到英文地名？」）。

**回覆同學時**請針對對方引用的其中一個證據，說明它也可以被相反立場解釋的理由，或補充一個對方沒用到的證據。

**教師引導重點**（供教師發文與週末小結使用）：
- 「理解」沒有標準定義，評量重點是能否正確引用證據；避免流於信仰之爭。
- 把證據分成「表示層」（embedding、拒絕向量）與「輸出層」（Logit Lens、Patchscopes），提醒 probing 與「直接問模型」的限制。
- 連到 HW3：同學親手算的注意力權重就是這些內部證據的最小版本。
- 週末小結時各挑一篇正反立場、證據用得好的貼文表揚。

## 本單元測驗（「U05 測驗」）

- 題庫隨機抽 10 題選擇題，作答 **1 次**；每題答錯可看提示再試（最多 3 次，每次扣該題 1/3 分）。正確答案於測驗關閉後才會顯示。
- 交卷後顯示每個選項的解說。範圍：第 3 講影片與投影片、3Blue1Brown 短片，以及 2024 第 11 講的 probing 概念。
- 截止：11/01（日）23:59。本學期 12 次測驗取最佳 10 次計分。

## 本單元其他事項

- **HW3「從零實作注意力與 GPT 模型」**：10/19 開放，**11/08（日）23:59 截止**（寬限至 11/15）。
- **HW2** 10/25（日）截止（寬限至 11/01）；HW2 的一頁「檢索失敗案例分析」請於 10/25 前上傳同儕互評活動，10/26–11/01 互評 3 份（規準見 `hw/hw2-peer-review.md`）。
- 10/26（一）補假，第 6 單元教材延至 11/02 發布；第 4 單元測驗截止日為 10/25。

## 本單元時間預估

| 活動 | 時間 |
|------|------|
| 必看影片 | 146 分 |
| 投影片瀏覽 | 20 分 |
| 動手練習 | 15 分 |
| 討論區發文與回覆 | 25 分 |
| 線上測驗 | 20 分 |
| **合計** | **約 3 小時 46 分**（HW3 另計，約 2 小時） |

## 延伸資源

- Anthropic, *Scaling Monosemanticity* <https://transformer-circuits.pub/2024/scaling-monosemanticity/>：用稀疏自編碼器找出模型內部的可解釋特徵。
- Transformer 的時代要結束了嗎？介紹 Transformer 的競爭者們 <https://youtu.be/gjsdVi90yQo>（82 分）：投影片「延伸閱讀」所指的 Mamba 等新架構。
- 第 4 單元的 3Blue1Brown〈Attention in transformers〉可回頭複習 query／key／value 的幾何直覺。
