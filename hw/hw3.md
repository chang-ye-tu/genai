# HW3 從零實作注意力與 GPT 模型（10/19 – 11/08，寬限至 11/15）

筆記本：[hw3/hw3_attention_gpt.ipynb](hw3/hw3_attention_gpt.ipynb)　取材：Raschka《Build a Large Language Model (From Scratch)》第 3、4 章，套件 `llms-from-scratch`　難度：原理型｜核心段落約 2 小時，選做另計　全部可在 CPU 執行

## 目標

把第 4、5 單元的 Transformer 講解變成可以執行的程式：內積如何變成注意力權重、因果遮罩怎麼實作、多頭注意力有多少參數，以及把 12 層堆起來後 GPT-2 small 尺寸的 1.24 億參數是怎麼算出來的。

## 步驟

核心段落為第 1–5 節，只有核心段落會出題；第 6 節為選做，不計分。

### 第 1 節 最簡單的注意力
- 六個 token 的 3 維向量（"Your journey starts with one step"）。以第 2 個 token 當 query：內積 → softmax → 加權和。
- 印出注意力分數、注意力權重（加總 1）、上下文向量；再一次算完 6×6 的權重矩陣。
- ✍️ 哪個 token 對 journey 最相關？權重加總為 1 的意義？

### 第 2 節 可訓練的 Q、K、V 與因果遮罩
- 用套件的 `SelfAttention_v2`（有 W_query／W_key／W_value）計算；手動做因果遮罩：右上三角設 −∞，softmax 後為 0；除以 √d_k。
- ✍️ 為什麼右上三角是 0？為什麼要除以 √d_k？

### 第 3 節 多頭注意力（GPT-2 尺寸）
- `MultiHeadAttention(d_in=768, d_out=768, num_heads=12, qkv_bias=True)`：印出參數量與輸入輸出形狀。

### 第 4 節 組裝 GPT-2 尺寸的模型並數參數
- `GPTModel(GPT_CONFIG_124M)`（書中設定，`qkv_bias=False`；原始 GPT-2 的 Q/K/V 有偏差項，HW5 載入官方權重時改用 `qkv_bias=True`）：總參數量、權重共享後的參數量、12 個前饋層與 12 個注意力模組各占多少、詞嵌入與位置嵌入、fp32 記憶體。
- 前向傳播：兩句話 → logits 形狀。
- ✍️ 前饋層為什麼比注意力多一倍參數？共享權重省下的等於哪個矩陣？

### 第 5 節 沒訓練過的 GPT 會說什麼
- 用隨機參數 greedy 接龍十個 token：亂碼。這正是 HW5 要補的部分。

### 第 6 節（選做）對照 HW1 的 Qwen2.5-1.5B
- 層數、維度、頭數、詞彙表、上下文、參數量對照表；連到第 4、5 單元的 KV cache 與多語詞彙表。

## 繳交

1. 執行完整份筆記本（保留輸出），命名 `hw3_學號.ipynb`，上傳至 iLearn「HW3 作業」。
2. 作答「HW3 實作測驗」（10 題，作答 1 次，答錯可看提示再試）。範圍：第 1、3、4 節的確定性數值（注意力權重、參數量、形狀）與第 2、4、5 節的概念題；第 6 節不出題。

## 常見問題

- **要不要 GPU？** 不用。整份在 CPU 幾分鐘內可跑完。
- **`pip install --no-deps` 是什麼意思？** 只裝套件本身、不裝它列的相依套件（原套件會要求 TensorFlow），Colab 已有 torch。
- **想看套件原始碼？** <https://github.com/rasbt/LLMs-from-scratch/tree/main/pkg/llms_from_scratch>，`ch03.py`、`ch04.py` 各不到 200 行。
