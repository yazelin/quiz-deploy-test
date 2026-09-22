# 台灣汽車駕照筆試題庫

**線上：<https://yazelin.github.io/quiz-deploy-test/>**

**880 題官方試題**，來源是交通部公路局「新版汽車筆試題庫」**115.5.29 公告版**。

這是 [ipas-ai-quiz](https://github.com/yazelin/ipas-ai-quiz) 換一份題庫做出來的站，用途是示範「**拿一份真的官方 PDF → 變成能離線刷、會幫你排錯題的網站**」這條路真的走得通，並把每一步的實際耗時與踩雷記下來。2026-09-23 週三直播「用 AI 做自己的刷題網站」的配套文件。

> **講義 PDF 一匯出就凍住了，指令會過期；這份 README 是活的，以這份為準。**

## 關於這份題庫

- **來源**：[公路局筆試題庫](https://www.thb.gov.tw/cl.aspx?n=12) 的「汽車筆試題庫＿公告115.06.9」PDF。
- **版權**：依**著作權法第 9 條第 1 項第 5 款**，「依法令舉行之各類考試試題及其備用試題」**不得為著作權之標的**。駕照考驗依《道路交通安全規則》辦理。
- **官方題庫只給題目與答案，沒有解析**，所以這個站的每一題都沒有詳解。要有解析就得自己寫——那是真正的成本，不是技術問題。
- **原始 1,090 題，這裡收 880 題。**少掉的是「圖中白色虛線為：」那類**標誌題**：題目本體是一張圖，純文字抽不出來，掛不上圖就是無法判讀的廢題，寧可不收。
- 台灣駕照筆試是**三選一**，不是四選一。
- 半形括號（例如「吊扣**(銷)**駕照」）是官方原文，刻意不改成全形：**正確性優先於排版一致性**。

準備考照請以[官方題庫](https://www.thb.gov.tw/cl.aspx?n=12)為準。

---

## 拿去做你自己的

### 先準備三件 AI 做不到的事

| 你要先準備 | 為什麼 AI 代勞不了 | 第一段 | 第二段 |
|---|---|---|---|
| GitHub 帳號，然後 `gh auth login` | 瀏覽器授權，要你本人按同意 | 要 | 要 |
| Cloudflare 帳號，然後 `npx wrangler login` | 同上 | 不用 | 要 |
| 你的題庫 | 只有你知道題目從哪來、能不能散布 | 要 | 要 |

**不需要進 Cloudflare 後台，也不需要網域，註冊也不用綁信用卡**（2026-09-22 實際走過一次註冊確認）。 官方文件寫明帳號自帶 `workers.dev` 子網域。這個站整條路（建資料庫、建表、部署、設金鑰）全部在終端機完成。註冊後若被引導「新增網站／輸入網域」，跳過即可。

### 第一段：讓站活著

**貼給 AI 的那段話：**

```
請 clone https://github.com/yazelin/ipas-ai-quiz 到這個資料夾，並讀它的 AGENTS.md。

跑 node tools/rebrand.mjs --name 「◯◯◯」 --repo 我的帳號/我的repo 換裝，
把 questions.json 依它的 schema 換成我的題庫（來源與格式先問我），
跑 node tools/check-questions.mjs 與 node core.test.mjs 都要過，
再用 gh 建 repo、推上去、開 Pages，把網址給我。
```

它實際會跑的東西在下面，出問題時對照用。

**換裝現在有工具了，不用手動改十七個地方：**

```bash
git clone --depth 1 https://github.com/yazelin/ipas-ai-quiz.git site && cd site
node tools/rebrand.mjs --name "你的站名" --repo 你的帳號/你的repo   # --dry-run 可先試跑
#   換 questions.json 成你的題庫(schema 見 AGENTS.md)
node tools/check-questions.mjs && node core.test.mjs
gh repo create <你的repo> --public --source=. --push
gh api -X POST repos/<你的帳號>/<你的repo>/pages -f 'source[branch]=main' -f 'source[path]=/'
```

`rebrand.mjs` 會處理站名、`title`／`description`／`og`／**`canonical`**、PWA 的 App 名稱、**`STORE_KEY`**、匯出檔名、回報連結、promo 計數、Service Worker 快取前綴、`SYNC_URL` 清空，以及刪掉原站的帶圖題資產與原站專用的自動化。跑完會自我檢查殘留。

> **`STORE_KEY` 是最容易漏、後果最嚴重的一項。** GitHub Pages 同一個帳號底下所有站共用一個 origin，`localStorage` 也就共用。沿用原站的 key，你的站會讀到、也會污染原站的進度，還會把對方的資料 PUT 到你的 Worker。這個站 2026-09-22 實測撞過。

`canonical` 沒換的話，等於告訴 Google 你這頁是原站的複本，你的站不會被收錄。畫面上完全看不出來。

### 第二段：跨裝置同步與推播（選做）

**貼給 AI 的那段話：**

```
現在幫我加上跨裝置同步與推播，照 AGENTS.md 的「自架同步/推播後端」做。

一、進 worker/ 跑 npm i，用 wrangler 建一個 D1 資料庫（名字我等一下給你），
把它回傳的 database_id 填進 worker/wrangler.toml，
同時把 wrangler.toml 的 name 改成一個沒被別人用過的名字。
注意：binding 要維持 DB，不要照 wrangler 印出來的建議改成資料庫名，程式用的是 env.DB。

二、灌 worker/schema.sql 建表，然後 npx wrangler deploy（要先部署，secret 才放得進去）。

三、跑 npx web-push generate-vapid-keys --json 產一對金鑰：
公鑰填進 worker/wrangler.toml 的 [vars] VAPID_PUBLIC，以及 app.js 最上面的 VAPID_PUBLIC，兩個地方都要；
私鑰用 npx wrangler secret put VAPID_PRIVATE 存進去，不要寫進任何檔案、不要 commit。

四、把 Worker 網址填回 app.js 的 SYNC_URL，把 sw.js 的 CACHE 版號加一，推上去。

最後用 curl 驗四個端點給我看：沒寫過的碼要回 404、PUT 要 200、GET 要讀得回剛才那包、亂碼要回 400。
```

**金鑰是這一段最容易卡住的地方，所以咒語裡把去向寫死了：公鑰要填 `wrangler.toml` 與 `app.js` 兩個地方，私鑰只能進 `wrangler secret`。**

它實際會跑的指令，總共不到 20 秒：

```bash
cd worker && npm i                                     # 1 秒
npx wrangler d1 create <你的資料庫名>                    # 2 秒,回傳 database_id
npx wrangler d1 execute <你的資料庫名> --remote --file schema.sql   # 4 秒
npx web-push generate-vapid-keys --json                # 1 秒
npx wrangler deploy                                    # 8 秒
npx wrangler secret put VAPID_PRIVATE                  # 3 秒,貼私鑰
```

然後把 Worker 網址填回 `app.js` 的 `SYNC_URL`、bump `sw.js` 的 `CACHE`，再推一次。

**這一段的坑跟第一段性質不同：指令全都會成功，錯的是設定檔裡一個名字。**

| 坑 | 症狀 |
|---|---|
| **binding 名稱必須是 `DB`** | `wrangler d1 create` 成功後印的設定寫 `binding = "<你的資料庫名>"`，**照貼就死**——程式用 `env.DB`，部署會過、一打 API 就 500 |
| Worker 名稱要全域唯一 | 沿用原本的會撞名 |
| 先 `deploy` 再 `secret put` | 反過來 Worker 還不存在，`wrangler` 會跳互動提問 |
| 有多個 Cloudflare 帳號 | 非互動模式直接失敗，要帶 `CLOUDFLARE_ACCOUNT_ID` |
| D1 免費層最多 10 個資料庫 | 先 `npx wrangler d1 list` 看一眼 |
| VAPID | 公鑰填兩處，**私鑰只進 `wrangler secret`，不要進 git** |

### 怎麼確認第二段真的成功

不要只看部署訊息，直接打端點，再開兩個瀏覽器做一次真的跨裝置（A 答一題打開設定頁上傳，同步碼抄到 B 按「套用此碼」）。

```bash
U=https://<你的worker>.workers.dev
curl -s -o /dev/null -w "%{http_code}\n" $U/sync/fox-river-1234        # 404 還沒寫過,正常
curl -s -o /dev/null -w "%{http_code}\n" -X PUT -H 'Content-Type: application/json' \
  -d '{"v":1,"q":{},"updatedAt":1}' $U/sync/fox-river-1234             # 200
curl -s $U/sync/fox-river-1234                                         # 讀得回
curl -s -o /dev/null -w "%{http_code}\n" $U/sync/not_a_valid_code      # 400
```

---

## 抓這份 PDF 本身就踩到的兩個坑

**一、官方站擋自動抓取。** 公路局在 Imperva WAF 後面，`curl` 回 `Incapsula incident ID`，連 Playwright 真 Chrome 直連檔案也 404。要先讓瀏覽器進站解掉挑戰、拿到 cookie，再用同一個 context 抓檔。**iPAS 的官方 PDF 可以直接 `curl`，公路局的不行**——同樣是官方來源，難度差很多。

**二、[政府資料開放平臺](https://data.gov.tw/dataset/30305)那份清單已經過期五年。** 裡面的檔名寫著 `1101021`（民國 110 年），連結全部 404。現行連結要去公路局的題庫頁抓。

---

程式碼 MIT，來自 [ipas-ai-quiz](https://github.com/yazelin/ipas-ai-quiz)（林亞澤）。題目為公路局公告試題。

## 更新紀錄

- **2026-09-22（再補）**：確認 Cloudflare 註冊不會要求信用卡，原本文件只敢寫「官方沒明說」。
- **2026-09-22（晚上）**：兩段都補上可以直接複製的咒語，第二段把 VAPID 公私鑰的去向寫死在裡面。原本只有指令清單、沒有咒語，金鑰產出來之後要放哪也完全沒交代。
- **2026-09-22（下午）**：題庫換成**真的官方題庫**——公路局 115.5.29 公告版，880 題。原本那 20 題 AI 自撰示範題退場。同步支援三選一（上游 PR #48）與 `rebrand.mjs` 換裝工具（上游 PR #47）。
- **2026-09-22（上午）**：修掉同 origin 的 `localStorage` 撞 key（會讀到也會污染原站進度）；修掉示範題庫 18/20 正解都是 (B) 的問題；同步上游的手機分頁換行修正。
- **2026-09-21**：建立。第一段與第二段都實測走完。
