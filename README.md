[README.md](https://github.com/user-attachments/files/28214858/README.md)
# 亮業務 CRM 🏢
> 南山人壽業務專用行動 CRM｜基於 Notion 資料庫的 PWA 應用

---

## 📱 系統簡介

這是一套專為南山人壽業務員設計的行動 CRM 系統，以 GitHub Pages 部署為 PWA（Progressive Web App），可安裝到手機主畫面像原生 App 一樣使用。

所有資料儲存於 Notion 資料庫，透過 Cloudflare Workers 作為 CORS Proxy 橋接 Notion API。

**線上網址：** `https://light545252-sys.github.io/insurance-crm/`

---

## 📁 檔案結構

```
insurance-crm/
├── index.html        # 主程式（PWA 單頁應用）
├── manifest.json     # PWA 安裝設定
├── icon-192.png      # App 圖示（192x192）
├── icon-512.png      # App 圖示（512x512）
└── README.md         # 本說明文件
```

---

## 🖥️ 功能頁籤

| 頁籤 | 功能 |
|------|------|
| 🏠 首頁 | 今日統計、快速記錄（關係維護、轉介紹、新觸點、保單說明、成交確認） |
| 📅 行事曆 | N-1 起7天拜訪、跟進、待辦任務整合顯示 |
| 👥 客戶 | 所有客戶列表、搜尋 |
| ⚙️ 設定 | Token 設定、PWA 安裝、資料庫 ID 查詢 |

---

## 📅 行事曆功能說明

### 顯示範圍
- **昨天（N-1）起共 7 天**

### 圖例
| 顏色 | 代表 |
|------|------|
| 🟢 綠點 | 拜訪日期 |
| 🟡 黃點 | 下次跟進日期 |
| 🔵 藍點 | 待辦任務（當天） |
| 🔴 紅點 + 橘框 | 延滯任務（未完成且已過期） |

### 待辦任務邏輯
- **延滯任務**（日期 < 今天且未完成）→ 固定出現在今天區塊最頂部
- 顯示 `⚠️ 延滯N天` 紅色標籤，精確計算天數
- 優先度標籤：🔴 急 / 🟡 本週 / 🟢 備用
- **N-1（昨天）** 所有卡片灰化顯示，今天起全彩

### 點擊卡片
- **拜訪/跟進卡片** → 詳情（客戶、類型、日期、摘要、需求、下一步行動）
- **待辦任務卡片** → 詳情 + **✅ 標記完成按鈕**
  - 有關聯客戶 → 詢問是否同步寫入拜訪記錄
  - 確認 → 任務完成 + 自動建立拜訪記錄
  - 完成後行事曆自動重整，任務消失

---

## 📋 新增待辦表單說明

### 預計完成日 — 快速選取
點選三個按鈕之一即可設定日期，無需手動輸入：

| 按鈕 | 行為 |
|------|------|
| **今日** | 自動填入今天日期 |
| **明日** | 自動填入明天日期 |
| **📅 選取日期** | 展開日期選取器，手動選取任意日期 |

---

## 🗄️ Notion 資料庫 ID 對照表

| 資料庫 | ID |
|--------|-----|
| 📋 客戶資料庫 | `830789dad67c4629b1aac7343980aeaa` |
| 📅 拜訪記錄 | `cf0ba45c2eef40928737d6d84e41478e` |
| 📅 每日任務 | `c8cdef9c13d24499bfa95eafa62c0748` |
| 📋 預估試算表 | `07a4837ef85041debc5a665355e514ae` |
| 💰 佣金明細記錄 | `e1f8ecbceeb5461e87d5fec84ab05cae` |

### 📅 每日任務欄位說明

| 欄位 | 說明 |
|------|------|
| 任務名稱 | 任務標題 |
| 任務類型 | ① 關係維護 / ② 轉介紹鋪墊 / ③ 開發新觸點 / ④ 高價值任務 |
| 優先度 | 🔴 急 / 🟡 本週 / 🟢 備用 |
| 完成狀態 | 待執行 / ✅ 已完成 / ⏸ 跳過 |
| 預計完成日 | 快選今日／明日，或手動選取日期 |
| 🔗 對象客戶 | 關聯客戶（有則完成時可同步寫入拜訪記錄） |
| 備註 | 任務說明 |
| 延滯原因 | 未如期完成的原因說明 |

---

## ⚙️ Cloudflare Workers Proxy

**用途：** 解決瀏覽器呼叫 Notion API 的 CORS 限制

**目前 Proxy 網址：**
```
https://notion-proxy.light545252.workers.dev
```

**位於 `index.html` 第 1105 行：**
```javascript
const PROXY = 'https://notion-proxy.light545252.workers.dev';
```

**Workers 程式碼（如需重新部署）：**
```javascript
export default {
  async fetch(request) {
    const url = new URL(request.url);
    const notionUrl = 'https://api.notion.com/v1' + url.pathname + url.search;
    const headers = new Headers(request.headers);
    headers.set('Notion-Version', '2022-06-28');
    if (request.method === 'OPTIONS') {
      return new Response(null, { status: 204, headers: {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'GET, POST, PATCH, DELETE, OPTIONS',
        'Access-Control-Allow-Headers': 'Authorization, Content-Type, Notion-Version',
      }});
    }
    const response = await fetch(notionUrl, {
      method: request.method,
      headers: headers,
      body: request.method !== 'GET' ? request.body : undefined,
    });
    const newHeaders = new Headers(response.headers);
    newHeaders.set('Access-Control-Allow-Origin', '*');
    return new Response(response.body, { status: response.status, headers: newHeaders });
  }
};
```

**重新部署步驟：**
1. 登入 [dash.cloudflare.com](https://dash.cloudflare.com)
2. Compute → Workers & Pages → notion-proxy → Edit code
3. 貼上上方程式碼 → Deploy
4. 將新網址更新至 `index.html` 的 `PROXY` 常數

**⚠️ 出現 `Failed to fetch` 時：**
1. 瀏覽器開啟 `https://notion-proxy.light545252.workers.dev` 確認是否正常
2. 若無法連線 → 依上方步驟重新部署
3. 更新 `index.html` PROXY 網址 → 上傳 GitHub

---

## 🔑 Notion Token 設定

1. 開啟 App → 點底部「設定」頁籤
2. 在「貼上 Notion Token」欄位輸入 Integration Token
3. 點「儲存 Token」
4. Token 格式：`ntn_xxxxxxxxxx` 或 `secret_xxxxxxxxxx`
5. Token 僅儲存於本機瀏覽器，不會上傳任何伺服器

---

## 📲 PWA 安裝方式

**iPhone / Safari：**
1. 點底部分享按鈕（□↑）→「加入主畫面」→「新增」

**Android / Chrome：**
1. 點右上角 ⋮ →「新增至主畫面」→「新增」

---

## 🔄 更新維護

1. 修改 `index.html`
2. 上傳至 GitHub 取代舊版
3. GitHub Pages 約 1～2 分鐘自動更新
4. 強制重新整理：`Ctrl + Shift + R`

---

## 📝 版本紀錄

| 日期 | 更新內容 |
|------|---------|
| 2026/05/25 | 新增待辦「預計完成日」快選按鈕（今日／明日／選取日期） |
| 2026/05/21 | 待辦任務整合行事曆：延滯提醒、完成按鈕、客戶連動寫入拜訪記錄 |
| 2026/05/21 | 每日任務新增「優先度」「延滯原因」欄位 |
| 2026/05/21 | 行事曆圖例常態顯示（🟢拜訪 / 🟡跟進 / 🔵待辦），置頂固定 |
| 2026/05/21 | N-1 昨天行程灰化顯示，N 起全彩 |
| 2026/05/21 | 修正時區偏移問題（localDateStr） |
| 2026/05/21 | 新增行事曆頁籤（N-1 起7天、點擊詳情） |
| 2026/05/21 | 修正 Proxy 換為 notion-proxy.light545252.workers.dev |
| 2026/05/21 | 新增保單簽約拜訪類型 |

---

*by 亮｜南山人壽業務 CRM*
