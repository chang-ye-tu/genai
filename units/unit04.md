# 第 4 單元（10/12 – 10/25）｜Transformer：語言模型如何做文字接龍

> 本頁內容同步張貼於 iLearn「第 4 單元」區塊的「學習指引」頁面。
> 本單元流程：**① 看教材 → ② 線上測驗**（發布當天即可作答）。本單元沒有討論活動。
> 本單元所有活動截止：**10/25（日）23:59**。

## 本單元學習目標

完成本單元後，你應該能夠：

1. 依序說明 Transformer 的五個步驟（tokenization、input layer、attention、feed forward、output layer）各自在做什麼，以及哪些步驟有需要訓練的參數。
2. 解釋 token embedding 與 positional embedding 的意義，以及為什麼需要位置資訊。
3. 說明 attention 如何用「相關性分數 → 加權和」考慮上下文，以及 causal attention 與 multi-head attention 的用意。
4. 說明 Transformer block 的堆疊與 output layer（softmax 機率分佈）如何接回第 1 單元的「文字接龍」。
5. 以 3Blue1Brown 的 query／key／value 觀點解釋注意力，並說明 temperature 與 context window 的作用。

## 教材

| # | 教材 | 類型 | 長度 | 來源 |
|---|------|------|------|------|
| 1 | 【必看】第 10 講：今日的語言模型是如何做文字接龍的 — 淺談 Transformer <https://youtu.be/uhNsUCb2fJI> | 影片 | 38 分 | 李宏毅《生成式 AI 導論 2024》 |
| 2 | 【必讀】投影片 0503_transformer.pdf <https://speech.ee.ntu.edu.tw/~hylee/genai/2024-spring-course-data/0503/0503_transformer.pdf> | 投影片 | 27 頁 | 同上 |
| 3 | 【必看】Transformers, the tech behind LLMs <https://youtu.be/wjZofJX0v4M> | 影片 | 27 分 | 3Blue1Brown（可開中文字幕） |
| 4 | 【必看】Attention in transformers, step-by-step <https://youtu.be/eMlx5fFNoYc> | 影片 | 26 分 | 3Blue1Brown |
| 5 | 【選看】Transformer 的時代要結束了嗎？介紹 Transformer 的競爭者們 <https://youtu.be/gjsdVi90yQo>（投影片 <https://speech.ee.ntu.edu.tw/~hylee/ml/ml2025-course-data/mamba.pdf>） | 影片 | 82 分 | 李宏毅《生成式 AI 時代下的機器學習 2025》 |
| 6 | 【選看】加快語言模型生成速度 (2/2)：KV Cache <https://youtu.be/fDQaadKysSA> | 影片 | 38 分 | 李宏毅《機器學習 2026》 |
| 7 | 【選看】加快語言模型生成速度 (1/2)：Flash Attention <https://youtu.be/vXb2QYOUzl4>（投影片 <https://speech.ee.ntu.edu.tw/~hylee/ml/ml2026-course-data/inference.pdf>） | 影片 | 49 分 | 同上 |

## 重點提示（Highlights）

- **模型演進**：N-gram → Feed-forward Network → RNN → **Transformer**；《Attention Is All You Need》的貢獻是「不需要 RNN（recurrence），靠 attention 考慮上下文就夠了」——注意 Transformer 仍保有 feed-forward、殘差連接、層正規化與位置編碼，並非「只有 attention」。
- **五個步驟**：① Tokenization（文字 → token；用 BPE 事先建好 token list，**這一步沒有要訓練的參數**）② Input Layer（**token embedding**：查表得到向量，意思相近的 token 向量接近，**不考慮上下文**；**positional embedding**：每個位置一個獨特向量，加到 token 向量上——這是原始 Transformer 與本講的做法；現代 LLM 多改用 RoPE 等相對位置編碼）③ **Attention**（考慮上下文）④ **Feed Forward**（整合、思考）⑤ **Output Layer**（linear + softmax → 機率分佈）。③④ 組成 Transformer Block，**× N 反覆思考**。
- **Attention 兩步**：計算相關性（有參數）得到 attention weight → 加權和集合相關資訊。同一個「果」在「蘋果電腦」與「來吃蘋果」中，因為前面的 token 不同而得到不同向量。（投影片示意圖讓每個 token 看整句，是簡化的雙向版本；實作的 causal attention 只看左邊，見下方實例。）
- **Causal attention**：實作時只看左邊的 token，因為接龍時右邊還不存在。
- **Multi-head attention**：關聯性不只一種，多組 attention 各抓不同關係。
- **3Blue1Brown 補全簡化處**：向量乘上三個矩陣得到 **query／key／value**；query·key 衡量相關性（原論文再除以 √d_k 縮放，投影片省略）→ softmax → 對 value 加權相加；**masking** 就是 causal attention；MLP（feed forward）層可能儲存事實。
- **Temperature**：高 → 分佈更平均、更隨機；低 → 更集中、更保守。**Context window**：一次能處理的 token 上限，attention 要算兩兩相關性，長度是成本與遺忘的根源。

## 實例（Worked example）

「來吃蘋果」中最後一個 token「果」要考慮上下文（依實作的 causal attention：只看自己與左邊的 token）：

1. 用相關性函數算「果」與左邊每個 token 的分數，假設得到 來 0.0、吃 0.3、蘋 0.4、果 0.3（右邊沒有 token 可看）。
2. 加權和：`0.3 × v(吃) + 0.4 × v(蘋) + 0.3 × v(果)` 就是「果」考慮上下文後的新向量。
3. 換成「蘋果電腦」，「果」的左邊只有「蘋」，分數與加權和都不同（例如 蘋 0.6、果 0.4）——同一個 token embedding，經過 attention 後分道揚鑣。至於「這是電腦公司」這件事，要等到後面的「電」「腦」位置回頭看「蘋果」時才會被吸收：資訊只往右流動。
4. 第二個 head 可能給出另一組權重（例如 0.1／0.5／0.4），抓的是另一種關係。
5. 經過 N 層 block 後，最後一個 token 的向量送進 output layer，softmax 得到下一個 token 的機率分佈——回到第 1 單元的「擲骰子」。

（投影片的示意圖把兩個「蘋」都畫成能看到整句，是為了說明「考慮上下文」而簡化的雙向版本；接龍模型實際用的是上面的單向做法。）

## 動手練習（不計分，約 15 分鐘）

1. 到 OpenAI Tokenizer <https://platform.openai.com/tokenizer> 輸入一句中文與一句意思相同的英文，觀察各被切成幾個 token、哪些字被拆開（HW1 已用程式做過同樣的事）。
2. 手算一次加權和：三個向量 (1, 0)、(0, 1)、(1, 1)，權重 0.1、0.5、0.4，結果是多少？（答案：(0.5, 0.9)）
3. 看 3Blue1Brown 第 6 集時，暫停在 attention pattern 那張圖，找出哪一個 key 對「形容詞」的 query 分數最高。

## 延伸思考（選做，不計分、不需繳交）

找一個一詞多義的中文詞（例如「蘋果」「銀行」「花」「打」），造兩個語意不同的句子。請說明：

1. 對這個詞而言，attention 應該把較高的權重放在哪些 token 上，才能分辨兩種語意（提醒：接龍模型只看左邊，若線索在右邊，就要由右邊的 token 回頭看它）；
2. 如果模型沒有 positional embedding，哪一組句子會被混淆（例如「我打他」與「他打我」），為什麼；
3. 實際把兩個句子交給任一聊天模型，請它解釋該詞在兩句中的意思，看它是否分辨成功。

提示：強調第 10 講是「大量簡化」：真實的 attention 有 query／key／value 與線性轉換，3Blue1Brown 已補全；不必背公式，但要能說出「相關性 → 加權和」。

本單元沒有討論活動；想交流的話可以貼到「課程問題 Q&A」討論區，不計分。

## 本單元測驗（「U04 測驗」）

- 題庫隨機抽 10 題選擇題，作答 **1 次**；每題答錯可看提示再試（最多 3 次，每次扣該題 1/3 分）。正確答案於測驗關閉後才會顯示。
- 交卷後會顯示每個選項的解說。範圍：第 10 講影片與投影片、3Blue1Brown 第 5、6 集。
- 截止：10/25（日）23:59。本學期 12 次測驗取最佳 10 次計分。

## 本單元其他事項

- **HW3「從零實作注意力與 GPT 模型」** 10/19（一）開放，**11/08（日）23:59 截止**（寬限至 11/15）：本單元的 attention、因果遮罩、多頭注意力會在 HW3 親手實作一次。HW2 於 10/25 截止。
- 10/26（一）為補假日：第 5 單元 10/19（一）發布後，第 6 單元延至 11/02（一）發布。

## 本單元時間預估

| 活動 | 時間 |
|------|------|
| 必看影片 | 91 分 |
| 投影片瀏覽 | 20 分 |
| 動手練習 | 15 分 |
| 線上測驗 | 20 分 |
| **合計** | **約 2 小時 26 分** |

## 延伸資源

- Transformer（上）／（下）<https://youtu.be/n9TlOhRjYoc>、<https://youtu.be/N6aRv06iv2g>（李宏毅，完整版 self-attention 與 Transformer 講解）。
- The Illustrated Transformer <https://jalammar.github.io/illustrated-transformer/>。
- Hugging Face NLP Course：Tokenizers（BPE）<https://huggingface.co/learn/nlp-course/chapter6/5>。
