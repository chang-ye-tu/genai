# 授權說明（Licensing）

本 repo 混合三種來源，逐目錄標示如下；`LICENSES/` 內附 Apache-2.0、MIT、CC BY-NC-SA 4.0 與 CC BY 4.0 的完整授權文本。本說明為授權盤點，不構成法律意見。公開 repo 只包含 `README.md`、本檔、`.gitignore`、`LICENSES/`、`units/` 與 `hw/`（共 38 檔）；其餘檔案與目錄只保存在教師本機。

| 路徑 | 授權 | 說明 |
|------|------|------|
| `README.md`、`units/`、`hw/*.md`、`hw/hw2/corpus.md` | CC BY-NC-SA 4.0 | 授課教師原創文字。<https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh-Hant> |
| `hw/hw1/`、`hw/hw2/*.ipynb`、`hw/hw7/` 的筆記本 | MIT（`LICENSES/MIT.txt`） | 課程自編程式；講解文字部分視同 CC BY-NC-SA。 |
| `hw/hw3/`、`hw/hw4/`、`hw/hw5/`、`hw/hw6/` 的筆記本 | Apache-2.0（`LICENSES/Apache-2.0.txt`） | 呼叫並改寫自 Sebastian Raschka 的 `llms-from-scratch`（<https://github.com/rasbt/LLMs-from-scratch>）與 `reasoning-from-scratch`（<https://github.com/rasbt/reasoning-from-scratch>），Copyright (c) Sebastian Raschka，Apache-2.0。依 Apache-2.0 §4 保留原著作權聲明，並在筆記本開頭標示「取材」與修改事實（講解文字為本課程改寫）。 |
| `hw/hw6/sms_spam_collection.zip` | CC BY 4.0（依 UCI 目錄） | UCI Machine Learning Repository「SMS Spam Collection」（Almeida & Hidalgo），<https://archive.ics.uci.edu/dataset/228/sms+spam+collection>，DOI 10.24432/C5CC84；授權依 UCI 目錄標示為 CC BY 4.0（ZIP 內附的原始 readme 第 4 節是資料集作者早期的版權／免責聲明，授權以 UCI 目錄為準）。作為 UCI 站台失效時的備份；使用時請引用原資料集。 |
| `.gitignore`、`LICENSES/*.txt` | — | 版本控制設定檔（無著作內容）與各授權條款的原文（依各自條款散布）。 |
| `quiz/`、`audits/` | 不公開 | 題庫、答案、稽核報告與回覆只存於教師本機或私有 repo，不隨公開 repo 散布（`.gitignore` 已排除，匯出工具亦不匯出）。 |
| `PLAN.md`、`doc/`、`ilearn/`、`tools/` | 不公開（教師本機） | 教師版實施計畫（`PLAN.md`）、校方檢核表與原廠 iLearn 操作指引及其對照（`doc/`）、本課程的 iLearn 建置手冊與回饋單題目（`ilearn/`）、題庫建置、筆記本產生器、繳交檢查、公開匯出工具與 Selenium 建置腳本（`tools/`）只保存在教師本機，不隨公開 repo 散布（`.gitignore` 已排除）。 |

課程引用之影片與投影片為原作者（李宏毅教授、3Blue1Brown 等）所有，本 repo 僅以網址連結引用並註明出處，未重製。

學生繳交之作品著作權屬學生本人；繳交即同意於課程內供同儕互評與教學觀摩使用。
