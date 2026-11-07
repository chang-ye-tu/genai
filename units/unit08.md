# 第 8 單元（11/16 – 11/29）｜深度學習：訓練類神經網路的訣竅

> 本頁內容同步張貼於 iLearn「第 8 單元」區塊的「學習指引」頁面。
> 本單元流程：**① 看教材 → ② 線上測驗**（發布當天即可作答）。本單元沒有討論活動。
> 本單元所有活動截止：**11/29（日）23:59**。

**本單元其他重要事項**
- 實作作業 HW4 截止：**11/22（日）23:59**（寬限至 11/29）。
- 實作作業 HW5「預訓練一個小 GPT」：11/16 開放，**12/06（日）23:59 截止**（寬限至 12/13）——本單元的學習率、過度擬合等訣竅會在裡面看到。

## 本單元學習目標

完成本單元後，你應該能夠：

1. 用老師的「三欄表」（方法名／改了哪一個步驟／帶來什麼好處）分析任何一個訓練技巧。
2. 依 training loss 與 validation loss 的表現，判斷問題出在 optimization 還是 generalization，並選擇對應的技巧。
3. 說明 Adagrad、RMSProp、Momentum、Adam 與 learning rate scheduling 各自解決什麼問題。
4. 說明 dropout、weight decay、data augmentation、更多資料等 generalization 技巧的原理與使用時機。
5. 說明 CNN、skip connection、normalization 為什麼屬於「改變函式搜尋範圍」，以及預訓練為什麼是一種初始化。
6. 解釋為什麼分類要用 cross-entropy 而不用 accuracy 當 loss，並用直觀方式描述 backpropagation 在做什麼。

## 教材

| # | 教材 | 類型 | 長度 | 來源 |
|---|------|------|------|------|
| 1 | 【必看】第 6 講：一堂課搞懂訓練類神經網路的各種訣竅 <https://youtu.be/mPWvAN4hzzY> | 影片 | 126 分 | 李宏毅《生成式人工智慧與機器學習導論 2025》 |
| 2 | 【必讀】投影片 TrainingTip.pdf <https://speech.ee.ntu.edu.tw/~hylee/GenAI-ML/2025-fall-course-data/TrainingTip.pdf> | 投影片 | 83 頁 | 同上 |
| 3 | 【必看】Backpropagation, intuitively <https://youtu.be/Ilg3gGewQ5U> | 影片 | 12 分 | 3Blue1Brown（可開中文字幕） |
| 4 | 【選看】ML Lecture 7: Backpropagation <https://youtu.be/ibJpTrp5mcE> | 影片 | 31 分 | 李宏毅《機器學習》 |
| 5 | 【選看】Dropout（從 1:10:27 起）<https://youtu.be/xki61j7z-30?t=4227> | 影片片段 | — | 李宏毅《機器學習》 |
| 6 | 【選看】Normalization <https://youtu.be/BABPWOkSbLE> | 影片 | — | 李宏毅《機器學習》 |
| 7 | 【選看】Pretrain 的詳細介紹 <https://youtu.be/lMIN1iKYNmA> | 影片 | — | 李宏毅《機器學習》 |

## 重點提示（Highlights）

- **三欄表**：聽到任何技巧都問「改了哪一個步驟（我要什麼／我有哪些選擇／選一個最好的）」與「帶來什麼好處（Better Optimization：training loss 更低；Better Generalization：validation loss 更低）」。
- **先診斷再開藥**：深度網路的 training loss 比線性模型還高 → optimization 沒做好；training loss 低但 validation loss 高 → overfitting，要改善 generalization。
- **Optimizer 家族**（都是 Better Optimization）：不同參數需要不同 learning rate → Adagrad 用過去 gradient 平方總和；同一參數的 gradient 會變 → RMSProp 讓近期 gradient 影響較大；gradient 很小會停住 → Momentum 加上慣性；**Adam = RMSProp + Momentum**；learning rate scheduling：warm up「探索地形」、decay「準備停下來」。
- **Dropout**：訓練時隨機丟神經元、測試時火力全開；Better Generalization 但 Worse Optimization，只在 overfitting 時用。
- **Initialization**：起始位置決定走到哪個最低點；**預訓練**用 pretext task 的大量易得資料先訓練，再當下游任務的初始化，對 optimization 與 generalization 都有幫助。
- **改變函式搜尋範圍**：CNN 用「pattern 比整張圖小」「同樣 pattern 會在不同位置出現」兩個觀察縮小範圍（Better Generalization）；skip connection 與 normalization 讓 loss 地形更平滑（Better Optimization）。
- **改變「我要什麼」**：分類不能用 accuracy 當 loss（gradient 為 0，無法下坡），要用 softmax + cross-entropy；更多資料、data augmentation（小心標籤是否還正確、Mixup）、semi-supervised、parameter regularization 都是 Better Generalization。
- **Weight decay**：在一般的 gradient descent 下，把 λΣθ² 正規化代入更新式就得到 θ ← (1 − 2ηλ)θ − ηg，所以「L2 正規化」與「每步把參數衰減一點」在 vanilla GD／SGD 中等價；但對 Adam 這類 adaptive optimizer 兩者並不等價——把 L2 加進 loss 交給 Adam 並不等於 AdamW，AdamW 是把衰減項獨立（decoupled）加在更新式上（Loshchilov & Hutter, 2019）。
- **Backpropagation**：用連鎖律從輸出層往回算出 loss 對每個參數的 gradient，供 gradient descent 使用。

## 實例（Worked example）

假設你在做一個「鳥／狗／貓」影像分類器，發生下面兩種情況：

1. **深度網路的 training loss 一直降不下來，甚至比線性模型高。** 這不是 overfitting（連訓練資料都沒學好），而是最佳化失敗。可以嘗試：換成 Adam／AdamW、加上 warm up + decay 的 learning rate scheduling、在深層網路加 skip connection 與 normalization。此時**不要**加 dropout（會讓 optimization 更差）。
2. **training loss 很低，validation loss 卻很高。** 這是 overfitting。可以嘗試：蒐集更多有代表性的資料、做 data augmentation（翻轉、模糊；但「向右轉」路標不能左右翻）、加 dropout、加 weight decay（AdamW）、或改用 CNN 這種用 domain knowledge 縮小函式範圍的架構。

順便算一次 weight decay：loss 加上 λΣθ² 後，gradient 多了 2λθ，更新式變成 θ ← θ − η(g + 2λθ) = (1 − 2ηλ)θ − ηg。也就是每次更新前先把參數乘上一個略小於 1 的數，「衰減」一點，這就是 weight decay 的由來。（這個推導只適用於 vanilla gradient descent；若 loss 寫成 (λ/2)Σθ²，係數就是 1 − ηλ。用 Adam 時，把 L2 加進 loss 與直接衰減參數不再等價，這正是 AdamW 存在的理由。）

## 動手練習（不計分，約 15 分鐘）

打開老師投影片附的範例程式 <https://colab.research.google.com/drive/1XPIU-I77dXL9W74jnevmfb8K8XoEPRso>（Adagrad／RMSProp／Momentum 的比較）：

1. 把 learning rate 改成 10 倍與 1/10 倍，觀察哪一個「飛出地圖」、哪一個「走不到谷底」。
2. 分別打開／關閉 momentum，觀察在 gradient 很小的區域會不會停下來。
3. 把觀察結果記下來，延伸思考時會用到。

## 延伸思考（選做，不計分、不需繳交）

從本單元介紹的技巧、或你在別處聽過的技巧（例如 early stopping、label smoothing、gradient clipping、learning rate warm-up、Mixup）中挑一個，用老師的三欄表分析它：

1. 它改了哪一個步驟（我要什麼／我有哪些選擇／選一個最好的）？
2. 它帶來的是 Better Optimization 還是 Better Generalization？為什麼？
3. 舉一個「用錯時機」的情境，說明會發生什麼事（例如在 training loss 都還很高時就加 dropout）。

提示：常見錯誤：把所有 optimizer 技巧都當成能防止 overfitting；把 dropout 當成萬靈丹。

本單元沒有討論活動；想交流的話可以貼到「課程問題 Q&A」討論區，不計分。

## 本單元測驗（「U08 測驗」）

- 題庫隨機抽 10 題選擇題，作答 **1 次**；每題答錯可看提示再試（最多 3 次，每次扣該題 1/3 分）。正確答案於測驗關閉後才會顯示。
- 交卷後會顯示每個選項的解說。範圍：第 6 講影片與投影片、3Blue1Brown 短片。
- 截止：11/29（日）23:59。本學期 12 次測驗取最佳 10 次計分。

## 本單元時間預估

| 活動 | 時間 |
|------|------|
| 必看影片 | 138 分 |
| 投影片瀏覽 | 20 分 |
| 動手練習 | 15 分 |
| 線上測驗 | 20 分 |
| **合計** | **約 3 小時 13 分** |

## 延伸資源

- AdamW 原始論文 *Decoupled Weight Decay Regularization* <https://arxiv.org/abs/1711.05101>。
- ResNet（skip connection）原始論文 <https://arxiv.org/abs/1512.03385>；loss 地形視覺化 <https://arxiv.org/abs/1712.09913>。
- Mixup <https://arxiv.org/abs/1710.09412>。
- CNN 完整介紹（李宏毅《機器學習》）<https://youtu.be/OP5HcXJg2Aw?t=1686>。
