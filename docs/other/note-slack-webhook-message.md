---
title: '[Note] Slack WebHook Message'
---

如果不需要互動功能，只需要傳送純文字訊息到 Slack，這個方案適合你。

如果要建立可互動的 Bot，請參考最後面的 [Reference](#reference)。

## 前置步驟

1. 有 Slack 帳號
2. 擁有(加入)至少一個 Workspace (非自建工作區可能需要申請安裝 App 權限)
3. 建立接收訊息的 Channel

## 申請 WebHook 流程

### 1. 進入後台首頁
  到 [Slack - API](https://api.slack.com/) 登入帳號，並進入 App 列表頁

### 2. 建立 App
  ![Create an app from scratch](https://imgur.com/vqAZnl8.jpeg "Create an app from scratch")

### 3. 初步設定
  設定 App 名稱(後面可以再詳細設定)，選擇 Workspace

  ![Set basic config](https://imgur.com/bhzIzRZ.jpeg "Set basic config")

### 4. Optional, 設定訊息機器人的 Profile
  在 App 後台首頁找尋 `Display Information` 區塊並設定

  ![Set detail bot config](https://imgur.com/92KjgGx.jpeg "Set detail bot config")

### 5. 啟用 & 新增 Webhook
  在 App 後台中尋找並進入 `Incoming Webhooks` 頁面，啟用功能，新增 Webhook

  ![active Webhook](https://imgur.com/7hJeQiz.jpeg "active Webhook")

### 6. 設定接收訊息的 Channel
   
  ![set message receiver](https://imgur.com/Oe3RVe4.jpeg "set message receiver")

## 發送訊息

成功跑完流程後，在 `Incoming Webhooks` 頁面下半部會得到一串 `Webhook URL`，使用這個 URL 發送 `POST Request`

![Get Webhook URL](https://imgur.com/L1IuifN.jpeg "Get Webhook URL")

發送 POST Request 時需夾帶 JSON 格式的 Body

```json
{
  "text": "Hello, World!"
}
```

發送的字串可以使用一定程度的 markdown 格式，具體請參考 [文件](https://api.slack.com/reference/surfaces/formatting)。

![Sample](https://imgur.com/fOalmVQ.jpeg "Sample")

## Reference
- [用python寫出slack機器人](https://andrewlearningnote.com/用python寫出slack機器人/)
- [為自己寫程式 - 簡單寫個 .NET Slack 訊息發送程式](https://blog.darkthread.net/blog/slack-api/)
