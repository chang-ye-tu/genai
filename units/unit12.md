# 第 12 單元（12/14 – 12/27）｜影像與聲音的生成策略

> 本頁內容同步張貼於 iLearn「第 12 單元」區塊的「學習指引」頁面。
> 本單元流程：**① 看教材 → ② 線上測驗**（發布當天即可作答）。本單元沒有討論活動。
> 本單元所有活動截止：**12/27（日）23:59**。這是最後一個有新教材與測驗的單元。

## 本單元學習目標

完成本單元後，你應該能夠：

1. 說明影像／聲音為什麼不用像素／取樣點接龍，以及 token（細胞 vs 原子）作為生成單位的意義。
2. 描述 Tokenizer／Detokenizer 的訓練目標，以及「像」的三種定義（Regression、Perceptual、Adversarial loss）。
3. 解釋 MaskGIT 與 Next Scale Prediction 如何改變接龍的「順序」。
4. 說明離散 token 在課程所引設定中的品質極限、對連續 token 做「單一向量的 MSE 回歸」為什麼會塌到平均（沒有標準答案），以及生成模型「從容易取樣的簡單分佈（常用常態分佈）變換到目標分佈」的核心想法。
5. 用自己的話說明 Flow Matching 的訓練與生成過程，以及「接龍骨幹＋生成頭」如何讓兩條技術路線匯聚。

## 教材

| # | 教材 | 類型 | 長度 | 來源 |
|---|------|------|------|------|
| 1 | 【必看】第 9 講：影像和聲音上的生成策略 — Diffusion/Flow-matching 系列和接龍 (Autoregressive) 這兩條世界線的交會 <https://youtu.be/ccqCDD9LqCA> | 影片 | 123 分 | 李宏毅《生成式人工智慧與機器學習導論 2025》 |
| 2 | 【必讀】投影片 Generation.pdf <https://speech.ee.ntu.edu.tw/~hylee/GenAI-ML/2025-fall-course-data/Generation.pdf> | 投影片 | 101 頁 | 同上 |
| 3 | 【選看】第 15 講：為什麼語言模型用文字接龍，圖片生成不用像素接龍呢？— 淺談生成式人工智慧的生成策略 <https://youtu.be/QbwQR9sjWbs>（投影片 <https://speech.ee.ntu.edu.tw/~hylee/genai/2024-spring-course-data/0517/0517_strategy.pdf>） | 影片 | 45 分 | 李宏毅《生成式 AI 導論 2024》 |
| 4 | 【選看】第 17 講：有關影像的生成式 AI（上）— AI 如何產生圖片和影片 <https://youtu.be/5H2bVEmYDNg> | 影片 | — | 同上 |
| 5 | 【選看】第 18 講：有關影像的生成式 AI（下）— 快速導讀 VAE, Flow, Diffusion, GAN <https://youtu.be/OYN_GvAqv-A> | 影片 | — | 同上 |
| 6 | 【選讀單元】第 10 講：語音語言模型發展史 <https://youtu.be/CbIPjrOj2Tc>（2025 年的技術從 1:42:00 開始；投影片 <https://speech.ee.ntu.edu.tw/~hylee/GenAI-ML/2025-fall-course-data/SpeechLLM.pdf>） | 影片 | 149 分 | 李宏毅《生成式人工智慧與機器學習導論 2025》 |
| 7 | 【選做】Flow Matching 範例程式（課程投影片附） <https://colab.research.google.com/drive/162Z6c_3d3GCRBeRwCjS7N_mrIonccEgz> | Colab | — | 同上 |

## 重點提示（Highlights）

- 兩條技術路線：**Diffusion／Flow Matching** 與 **Autoregressive（接龍）**；本講的主軸是 2025 年它們如何走在一起。
- 影像的基本單位是像素、影片是一連串的畫面（frame）、聲音是取樣點；**像素／取樣點接龍**（Pixel RNN／CNN）可行但太慢。
- 需要「適合生成的基本單位」——**token**（細胞 vs 原子）；**Tokenizer→token 接龍→Detokenizer**，訓練目標是重建「越接近越好」，但可能失真。語音常用 **RVQ**（多層量化，像進位）；影像 token 不一定要 2-D 排列（32 個 token 也能重建）。
- 什麼叫「像」：**表面上的像**（Regression loss，對平移很敏感）、**感知上的像**（Perceptual loss）、**模型難以分辨的像**（Adversarial loss／GAN）；課程介紹的 codec 把三者相加，這是常見組合而非必然（不同系統可能另有 codebook／commitment、KL 等項）。
- 接龍不一定由左而右：**MaskGIT** 每輪同時預測、只留信心最高的 K 個；**Next Scale Prediction** 從小圖到大圖，由粗到細。
- **離散 token 的極限**（課程所引設定：8192 種離散 token vs 16 維連續 token 的重建比較）：token 表示不完整，接龍訓練再好也沒用 → 改用**連續 token**（離散不必然較差、連續 bottleneck 也不是無損，品質取決於 codebook、token 數與 decoder）；但對連續 token 直接做**單一向量的 MSE 回歸**行不通——「世界沒有標準答案」，單點預測會塌到多種合理答案的平均（模糊、四不像）。問題出在「確定性單點預測」，不在平方誤差本身。
- **生成模型**（VAE、GAN、Flow、Diffusion、Flow Matching）：把一個容易取樣的簡單分佈（常用常態分佈）變換成目標分佈。**Flow Matching**：在雜訊 x0 與資料 x1 之間內插，學向量場 v = x1 − x0——它的訓練目標本身就是對向量場做平方誤差回歸，可見平方誤差用對地方仍然可行；生成時分若干步走，步數如「層數」。
- **匯聚**：Transformer 接龍 + **Generation head**（Diffusion／Flow-matching head，小型 MLP）產生連續 token；代價是每個 token 要多次 iteration，「減少 iteration」是關鍵研究問題；可搭配 MaskGIT，並用於語音、音訊、影片。

## 實例（Worked example）

**「一隻在奔跑的狗」是怎麼被畫出來的（接龍 + 生成頭）**

1. 文字條件「一隻在奔跑的狗」與已生成的影像 token 一起送進 Transformer。
2. Transformer 在每個位置輸出一個「上下文向量」；它不直接用一個確定的向量去猜下一個連續 token（那樣的 MSE 回歸會學到多種答案的平均）。
3. 生成頭從常態分佈抽一個樣本，以上下文向量為條件，依 Flow Matching 的向量場走幾步，得到下一個連續 token。
4. 重複 2–3 直到所有 token 產生，最後由 Detokenizer 還原成影像。因為每次抽的樣本不同，同一個描述會得到不同的狗——這正是「沒有標準答案」的正確處理方式。

## 動手練習（不計分，約 15 分鐘）

1. 用任一影像生成工具（Gemini、ChatGPT、或 Hugging Face 上的開源模型 Space）以同一段描述生成 4 張圖，觀察它們哪裡相同、哪裡不同。
2. 試著生成一張「畫面中有清楚中文字」或「時鐘指著 10 點 10 分」的圖，看模型是否做得到；想想這與 token 化、失真、或條件理解有什麼關係。
3. （選做）打開投影片附的 Flow Matching 範例程式，把「步數」改成 2、4、16，觀察生成品質與時間的變化。

## 延伸思考（選做，不計分、不需繳交）

貼出你用同一段描述生成的 4 張圖（或描述你觀察到的差異），並回答：

1. 用本單元「沒有標準答案 → 先產生分佈再抽樣」的觀念，解釋為什麼 4 張圖會不一樣、又為什麼它們都「合理」。
2. 挑一張你覺得「畫壞」的圖（或做不到的要求，例如中文字、時鐘），推測問題出在哪個環節：token 化失真、條件理解、還是生成過程本身？說明你的理由。
3. 表態：你認為未來影像／語音生成會以「接龍」為主、以「Diffusion／Flow」為主、還是兩者結合？用本單元內容支持你的看法。

提示：把「同一描述多種輸出」和第 1 單元的「擲骰子抽樣」連起來：文字用離散分佈抽樣，影像用生成模型從常態分佈抽樣，本質相同。

本單元沒有討論活動；想交流的話可以貼到「課程問題 Q&A」討論區，不計分。

## 本單元測驗（「U12 測驗」）

- 題庫隨機抽 10 題選擇題，作答 **1 次**；每題答錯可看提示再試（最多 3 次，每次扣該題 1/3 分）。正確答案於測驗關閉後才會顯示。
- 交卷後會顯示每個選項的解說。範圍：第 9 講影片與投影片（另含 2 題與第 15 講重疊的基本概念）。
- 截止：12/27（日）23:59。本學期 12 次測驗取最佳 10 次計分。

## 本單元其他事項

- **HW6** 12/20（日）23:59 截止（寬限至 12/27）；**HW7** 12/27（日）23:59 截止（寬限只到 12/28）。
- **第 13 單元**（12/21 發布）：課程總結（寫在該單元的學習指引頁）、學習成果分享討論區（1 主題 + 1 回覆，計入討論參與）、期末回饋單；沒有測驗。
- **期末課程回饋單**：12/21–12/28 開放（匿名，教師介面與報表不顯示身分；等 HW6 與第 13 單元開始後才填），請務必填寫，你的意見會用於下次開課的改善。

## 本單元時間預估

| 活動 | 時間 |
|------|------|
| 必看影片 | 123 分 |
| 投影片瀏覽 | 20 分 |
| 動手練習 | 15 分 |
| 線上測驗 | 20 分 |
| **合計** | **約 2 小時 58 分** |

## 延伸資源

- 李宏毅過去課程的主題影片：VAE（2016）<https://youtu.be/8zomhgKrsmQ>、GAN（2018）<https://www.youtube.com/watch?v=DQNNMiAP5lw>、Normalizing Flow（2019）<https://youtu.be/uXY18nzdSsM>、Diffusion（2023）<https://www.youtube.com/watch?v=azBugJzmz-o>。
- MIT《Introduction to Flow Matching and Diffusion Models》<https://diffusion.csail.mit.edu/>（課程投影片引用）。
- 論文：MaskGIT <https://arxiv.org/abs/2202.04200>、An Image is Worth 32 Tokens <https://arxiv.org/abs/2406.07550>、Autoregressive Image Generation without Vector Quantization <https://arxiv.org/abs/2406.11838>。
