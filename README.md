[README.md](https://github.com/user-attachments/files/28105965/README.md)
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
| 📅 行事曆 | N-1 起7天待辦與跟進事項，點擊查看完整紀錄 |
| 👥 客戶 | 所有客戶列表、搜尋 |
| ⚙️ 設定 | Token 設定、PWA 安裝、資料庫 ID 查詢 |

### 📅 行事曆說明
- 顯示**昨天（N-1）起共 7 天**的拜訪與跟進事項
- 🟢 **綠點** = 拜訪日期落在該天
- 🟡 **黃點** = 下次跟進日期落在該天
- **N-1（昨天）** 卡片灰化顯示，今天起全彩
- 點擊卡片 → 展開詳情（客戶名稱、拜訪類型、摘要、需求、下一步行動）

---

## 🗄️ Notion 資料庫 ID 對照表

| 資料庫 | ID |
|--------|-----|
| 📋 客戶資料庫 | `830789dad67c4629b1aac7343980aeaa` |
| 📅 拜訪記錄 | `cf0ba45c2eef40928737d6d84e41478e` |
| 📅 每日任務 | `c8cdef9c13d24499bfa95eafa62c0748` |
| 📋 預估試算表 | `07a4837ef85041debc5a665355e514ae` |
| 💰 佣金明細記錄 | `e1f8ecbceeb5461e87d5fec84ab05cae` |

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
2. 若無法連線（DNS 錯誤）→ 依上方步驟重新部署
3. 更新 `index.html` 第 1105 行的網址 → 上傳 GitHub

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
1. 點底部分享按鈕（□↑）
2. 選「加入主畫面」
3. 點「新增」完成

**Android / Chrome：**
1. 點右上角 ⋮ 選單
2. 選「新增至主畫面」
3. 點「新增」完成

---

## 🔄 更新維護

**更新 index.html 步驟：**
1. 修改 `index.html`
2. 上傳至 GitHub 取代舊版
3. GitHub Pages 約 1～2 分鐘自動更新
4. 瀏覽器強制重新整理：`Ctrl + Shift + R`

---

## 📝 版本紀錄

| 日期 | 更新內容 |
|------|---------|
| 2026/05/21 | 行事曆圖例常態顯示（🟢 拜訪 / 🟡 跟進），置頂固定 |
| 2026/05/21 | N-1 昨天行程灰化顯示，N 起全彩 |
| 2026/05/21 | 修正時區偏移問題（localDateStr 取代 toISOString） |
| 2026/05/21 | 新增行事曆頁籤（N-1 起7天、點擊詳情） |
| 2026/05/21 | 修正 Proxy 換為 notion-proxy.light545252.workers.dev |
| 2026/05/21 | 新增保單簽約拜訪類型 |

---

*by 亮｜南山人壽業務 CRM*
