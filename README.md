# 字民 ZIMIN — Macao Visual Archive

澳門街頭招牌的視覺檔案：227 件，每件是一組**雙聯**——上面是街上的照片，
下面是同一塊招牌被重新描字（簡體轉繁體）、平塗在取自該招牌底色上的版本。

- 架構：React 19 + TypeScript + Vite，手寫 CSS，**不引入 UI 框架**
- 路由：hash（`#/index`），這樣 `file://` 直接雙擊也能跑
- 產出：`vite build` 後由 `scripts/inline.mjs` 把 JS 內嵌進 `index.html`，再另出一個「單檔版」

---

## 快速開始

```bat
cd web
pnpm install
pnpm dev        :: http://localhost:5173
pnpm build      :: 產出 dist/（約 11 MB，含圖片）
```

單檔版（所有圖片內嵌，拷到哪都能開）：

```bat
node scripts/single-file.mjs   :: 產出 ..\字民_ZIMIN_單檔版.html（約 17 MB）
```

---

## 目錄

```
web/
├─ src/
│  ├─ pages/          Home / Index / Connections / Redesign
│  ├─ components/     EntryCard、SiteNav、SiteHeader、StreetSection…
│  ├─ data/           archive.json（227 件）、panels.json（面板尺寸）、tags.json
│  ├─ lib/            router、archive、redesign、channel
│  └─ styles/         tokens.css + 各頁 CSS
├─ public/
│  ├─ img/thumb/      227 張縮圖（390×520）
│  ├─ panel/          227 張街景用圖（完整雙聯，高 320）
│  └─ favicon.svg
├─ scripts/           prepare_web.py（資料）、inline.mjs（內嵌）
└─ tools/             serve.ps1、如果打不開_用這個.cmd
```

---

## 頁面

| 路由 | 內容 |
|---|---|
| `#/` | HOME：227 張縮圖散在紙上，右側分類面板，點開看紀錄 |
| `#/index` | INDEX：227 行目錄，可搜尋、排序、按分類篩選 |
| `#/connections` | CONNECTIONS：原序／分類／顏色／形態四種看法 |
| `#/redesign` | RE-DESIGN：01 字頻標本 · 02 一條街 |

### 02 STREET（一條街）

在 `components/StreetSection.tsx`。227 件依**件號**橫向連續排開，不按時間、不分組。

| 規則 | 值 |
|---|---|
| 圖 | **完整雙聯，不裁切**（見下） |
| 高 | 150 px（手機 130 px） |
| 寬 | 按原圖比例，上限 4 倍高 |
| 間隙 | 3 px |
| 全長 | 33,342 px |

- 橫向滾動；底部量表每 20 件一段，可點擊跳段
- 滑過任一塊 → 件號與街名；點一下 → 該件的完整紀錄
- 面板圖在 `public/panel/001–227.jpg`（4.6 MB），尺寸表在 `src/data/panels.json`
- 單檔版會把這 227 張內嵌成 `window.__PANELS__`

#### 為甚麼不裁「平塗那一半」

原本想把每件的下半（重描的平塗）切出來當街景，試了三版都不行：

1. **看底色佔比** → 大字招牌走到字的行就停，切掉字的頭
2. **看整塊平均色** → 左右雙聯判錯，切掉右邊的字
3. **看邊緣密度找接縫** → 227 件裡 65 件判不出，退回硬切

每一版都有幾十件被削掉筆畫，而這批資料就是要看字。
所以改成**整張不裁**：每一塊都是完整的雙聯，代價是畫面一半是照片。

---

## 資料管線

```
OUT/（原始 CSV）→ scripts/prepare_web.py → web/src/data/archive.json
```

`prepare_web.py` 會檢查 227 件、916 字、543 獨體、121 街、18 無街，
並確保 `titleRaw` / `addressRaw` 與原 CSV 逐字一致。**不要手改 JSON。**

街景圖（`public/panel/`）由 `scripts/frieze_build5.py` 從 `archive_extract/` 產生，
再由 `scripts/panels_json.mjs` 寫出尺寸表。

### 作者（逐件上傳者）

CSV 沒有上傳者欄位，所以另外從 are.na API 取：

```
GET https://api.are.na/v2/channels/group5-1vk3ydy1fq4/contents?per=100&page=1..3
→ node scripts/uploaders.mjs → src/data/uploaders.json（以 arenaId 對應）
```

227 件全部對上，共 **49 位**上傳者。每件的 `user` 與 `connected_by_user_slug` 完全相同，
所以「作者」＝把這張圖放進該 channel 的人，不是 channel 擁有者。

注意：`prepare_web.py` 重跑不會動到這個欄位——`uploaders.json` 是獨立檔案，
要更新就重跑 `uploaders.mjs`。
---

## 部署

**GitHub Pages**（`字民_GitHub上傳.zip`）：解壓後把 `index.html`、`assets/`、`img/`、`panel/`、`.nojekyll`
上傳到 repo 根目錄，Settings → Pages → main / (root)。

**離線**：用單檔版 `字民_ZIMIN_單檔版.html`，或解壓 `字民_ZIMIN_網站.zip` 後跑 `如果打不開_用這個.cmd`。

---

## 已知問題

- 這台機器的 Vite 檔案監看偶爾卡住（`EBUSY … .tmpdir`）：改了檔案畫面沒更新就重啟 `pnpm dev`；
  `pnpm build` 直接讀硬碟，不受影響。
- 直接用 `file://` 開、但 `img/` 與 `panel/` 沒跟著走時，頁面會顯示「找不到圖片（N 張）」，
  不會只留白屏。
- 目前面板與縮圖是兩套檔案（縮圖 390×520、面板高 320）；若之後只留一套，可省約 8 MB。
