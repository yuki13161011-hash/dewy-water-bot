# 💧 水滴精靈 Dewy：LINE 喝水記錄小精靈

傳一個數字就能記錄喝水量，還能查進度、比排行；聊天時，AI 小精靈會溫柔提醒你喝水。

> 設計原則：**能用規則算清楚的事交給程式，需要溫度的事交給 AI。**
> 數字（加總、排名）由程式計算，AI 只負責聊天，所以不會算錯、也不會亂編。

## 功能

| 使用者傳的訊息 | 小精靈做的事 |
|---|---|
| `300` 或 `喝水 300` | 記錄 300 ml，回覆今日進度條 |
| `今天` | 今日累計與達成率 |
| `排行` | 今日排行榜 |
| `日報` | 昨天的喝水報告（進度、排名、評語） |
| `週報` | 上週一到週日每日長條圖、達標天數、冠軍 |
| 聊天、抱怨、分享心情 | AI 以小精靈人設回應，並自然提醒喝水 |
| 「剛喝了一杯珍奶」 | AI 不編數字，引導輸入「喝水 500」 |

防呆：單次記錄限定 50～2000 ml。每日目標預設 2000 ml（程式內 `GOAL` 可調整）。

## 系統架構

```
LINE 訊息 → n8n Webhook → 過濾空事件 → 解析指令 → 取得暱稱 → 分流
  ├─ 記錄 → 寫入 Google Sheets → 讀取全部紀錄 ─┐
  ├─ 今天 / 排行 / 日報 / 週報 ────────────────┤→ 組合回覆 → LINE 回覆
  └─ 其他（聊天）→ AI Agent（Gemini）→ 整理回覆 → LINE 回覆
```

| 元件 | 用途 |
|---|---|
| LINE Messaging API | 使用者介面 |
| n8n | 流程編排 |
| Google Sheets | 資料庫（每次喝水一列） |
| Gemini（AI Agent） | 聊天與人設，不碰計算 |

## 資料表（Google Sheets）

工作表名稱 `喝水紀錄`，表頭：

| 時間 | 日期 | userId | 暱稱 | ml |
|---|---|---|---|---|

「日期」欄請設為純文字（格式 `2026-10-25`）。

## AI 人設

完整 System Prompt 見 [`system-prompt.md`](./system-prompt.md)。

## 如何自己架一份

1. 建立 LINE Messaging API channel，取得 Channel access token。
2. 建立 Google 試算表，依上方表頭建立工作表。
3. 在 n8n 匯入 `workflow.json`（Workflow → Import from file）。
4. 重新建立 Credential：LINE（Header Auth，網域限制 `api.line.me`）、Google Sheets、Gemini。
5. 把 Webhook 的 Production URL 貼到 LINE 後台，並發布 workflow。

## ⚠️ 安全提醒

本專案**不包含**任何金鑰。請勿把 Channel access token、API Key、含真實 userId 的試算表資料、完整 Webhook 網址提交到公開儲存庫。

## 踩過的坑

1. Webhook 404：測試網址只聽一次，正式使用要發布並換成 Production URL。
2. 憑證的允許網域沒填，會抓不到暱稱。
3. Switch 出口沒接線，該指令就已讀不回。
4. AI 輸出欄位是 `output`，LINE 要 `replyText`，中間要加一顆整理節點。

## 未來規劃

- 每週一 09:00 自動推播週報
- LINE 圖文選單一鍵記錄
- 載入動畫（思考中）
- 每人自訂目標、定時提醒
