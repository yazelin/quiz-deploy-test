# 台灣汽車駕照筆試（示範）

**線上：<https://yazelin.github.io/quiz-deploy-test/>**

這是 [ipas-ai-quiz](https://github.com/yazelin/ipas-ai-quiz) 換一份題庫做出來的示範站，用途是證明「**換題庫 → 推 repo → 開 Pages**」這條路真的走得通，並且把每一步的實際耗時與踩雷記下來。

> **題目是 AI 自撰的 20 題示範題，不是官方試題。**答案僅供示範，不要拿來準備考照。

這份 README 是 2026-09-23 週三直播「用 AI 做自己的刷題網站」的配套文件。**講義 PDF 一匯出就凍住了，指令會過期；這裡是活的，以這份為準。**

---

## 你會得到什麼

| | 內容 | 要辦什麼帳號 |
|---|---|---|
| **第一段** | 完整的刷題站：練習、智慧複習、Leitner 錯題本、計時模擬考、弱點統計、筆記匯出、裝到手機離線刷。有網址可以分享。 | **只要 GitHub** |
| **第二段** | 再加跨裝置同步與每日提醒推播：手機刷一半換電腦接著刷。 | 再加 Cloudflare |

**第二段不做，站一樣是完整的。** `app.js` 的 `SYNC_URL` 留空時，同步與推播自動停用，其他功能一個都不少。

---

## 先準備三件 AI 做不到的事

| 你要先準備 | 為什麼 AI 代勞不了 | 第一段 | 第二段 |
|---|---|---|---|
| GitHub 帳號，然後 `gh auth login` | 瀏覽器授權，要你本人按同意 | 要 | 要 |
| Cloudflare 帳號，然後 `npx wrangler login` | 同上 | 不用 | 要 |
| 你的題庫 | 只有你知道題目從哪來、能不能散布 | 要 | 要 |

**互動式登入是 AI 唯一做不到的事**，其餘它自己來——包括開 GitHub Pages。

**不需要進 Cloudflare 後台做任何設定，也不需要網域。** 官方文件寫明帳號自帶一個 `workers.dev` 子網域，讓你「不必先接自己的網域就能部署 Workers」。這個示範站整條路（建資料庫、建表、部署、設金鑰）全部在終端機完成，一次都沒開後台。註冊後若被引導「新增網站／輸入網域」，跳過即可。

關於信用卡：官方文件寫「使用者預設就有 Workers 免費方案」，付費方案要自己主動加購（每月最低 5 美元），D1 的文件也明講免費方案會一直存在。按文件我們用到的都在免費額度內。**但官方沒有一句明說註冊不需要信用卡，而這個示範站是用一個很久以前辦的帳號做的，不能證明新帳號的情況。** 如果你註冊時被要求綁卡，歡迎回報。真的要綁卡就只做第一段。

---

## 第一段：讓站活著

實測從 clone 到站打得開，**約六分鐘**，其中指令只佔不到一分鐘：

| 步驟 | 實際耗時 |
|---|---|
| `git clone` | 2 秒 |
| 換題庫與站名（見下方清單） | 約 1 分 |
| `gh repo create --source=. --push` | 4 秒 |
| `gh api … /pages` 開 Pages | 1 秒 |
| Pages 首次建置到打得開 | 31 秒 |

```bash
git clone --depth 1 https://github.com/yazelin/ipas-ai-quiz.git site && cd site

# ……照下面那張清單換乾淨……

node tools/check-questions.mjs      # 必須全過
node core.test.mjs                  # 必須 PASS

gh repo create <你的repo> --public --source=. --push
gh api -X POST repos/<你的帳號>/<你的repo>/pages \
  -f 'source[branch]=main' -f 'source[path]=/'
```

`gh api` 那行就把 Pages 開起來了，**不必進 GitHub 設定頁點**。

### 換裝清單：十五處，只有一處會被機器擋下來

這是整件事真正花力氣的地方。**其餘十四處畫面上完全正常**，你不會自己發現。

| 要換的地方 | 不換會怎樣 | 機器擋得住嗎 |
|---|---|---|
| `questions.json` | 題庫還是別人的 | — |
| `app.js` 的 `SYNC_URL` | 進度會同步到別人的後端 | 不會 |
| `assets/` 裡的舊圖 | 變孤兒圖 | **會，`check-questions` exit 1** |
| `<link rel="canonical">` | **等於告訴 Google 你這頁是原站的複本，你的站不會被收錄** | 不會 |
| `og:url`、`og:image` | 分享出去的卡片指向原站 | 不會 |
| `<title>`、`meta description`、`og:description` | 瀏覽器分頁與分享卡還是原站文案 | 不會 |
| `apple-mobile-web-app-title` | 裝到手機後 App 名稱是別人的 | 不會 |
| `core.js` 的匯出筆記標題 | 匯出的檔案標題是別人的 | 不會，要按匯出才露出 |
| `sw.js` 的推播通知標題 | 推播顯示別人的站名 | 不會，要真的收到推播才露出 |
| 每題「這題有誤？」的 issue 連結 | 你的使用者跑到原作者 repo 開 issue | 不會 |
| footer 的討論區／意見回饋／GitHub 連結 | 同上 | 不會 |
| promo-footer 的 `REPO` 變數 | 開站統計算到原站頭上 | 不會 |
| `.github/workflows/` | **你的 repo 每天自動開一個跟你無關的 issue** | 不會 |
| `robots.txt`、`sitemap.xml`、`manifest.json` | 都寫死原站網址 | 不會 |
| `build.html`（題庫 API 說明頁） | 同上 | 不會 |
| `sw.js` 的 `CACHE` 版號 | 使用者拿到舊版快取 | 不會 |

所以**貼給 AI 的那段話要把「換乾淨」逐項寫出來，只說「改成我的」它一定會漏**。

---

## 第二段：跨裝置同步與推播

指令總共跑不到 20 秒：

```bash
cd worker && npm i                                     # 1 秒
npx wrangler d1 create <你的資料庫名>                    # 2 秒,回傳 database_id
npx wrangler d1 execute <你的資料庫名> --remote --file schema.sql   # 4 秒
npx web-push generate-vapid-keys --json                # 1 秒
npx wrangler deploy                                    # 8 秒
npx wrangler secret put VAPID_PRIVATE                  # 3 秒,貼私鑰
```

然後把 Worker 網址填回 `app.js` 的 `SYNC_URL`、bump `sw.js` 的 `CACHE`，再推一次。

### 這一段的坑跟第一段性質不同：指令全都會成功，錯的是設定檔裡一個名字

| 坑 | 症狀 |
|---|---|
| **binding 名稱必須是 `DB`** | `wrangler d1 create` 成功後會印一段設定叫你貼進 `wrangler.toml`，裡面寫的是 `binding = "<你的資料庫名>"`。**照貼就死**——程式用的是 `env.DB`，部署會過、一打 API 就 500。**這是整段最容易死的地方。** |
| Worker 名稱要全域唯一 | `wrangler.toml` 的 `name` 沿用原本的會撞名 |
| 先 `deploy` 再 `secret put` | 反過來的話 Worker 還不存在，`wrangler` 會跳出互動提問，自動化情境會卡住 |
| 有多個 Cloudflare 帳號時 | 非互動模式直接失敗，要在 `wrangler.toml` 寫 `account_id` 或設 `CLOUDFLARE_ACCOUNT_ID` |
| D1 免費層最多 10 個資料庫 | 做第二、第三個站前先 `npx wrangler d1 list` 看一眼 |
| VAPID 金鑰 | 公鑰要填 `wrangler.toml` 的 `[vars]` 與 `app.js` **兩個地方**；**私鑰只進 `wrangler secret`，絕對不要進 git**。`worker/node_modules/` 與 `worker/.wrangler/` 要進 `.gitignore` |

### 怎麼確認第二段真的成功

不要只看部署訊息，直接打端點：

```bash
U=https://<你的worker>.workers.dev
curl -s -o /dev/null -w "%{http_code}\n" $U/sync/fox-river-1234        # 404 還沒寫過,正常
curl -s -o /dev/null -w "%{http_code}\n" -X PUT \
  -H 'Content-Type: application/json' -d '{"v":1,"q":{},"updatedAt":1}' \
  $U/sync/fox-river-1234                                               # 200
curl -s $U/sync/fox-river-1234                                         # 讀得回剛才那包
curl -s -o /dev/null -w "%{http_code}\n" $U/sync/not_a_valid_code      # 400 擋掉壞碼
```

再開兩個瀏覽器做一次真的跨裝置：A 答一題、打開設定頁（會上傳），把同步碼抄到 B 按「套用此碼」，B 應該拿到 A 的進度。這個示範站就是這樣驗過的。

---

## 題庫哪裡來，才是真正的門檻

誰出的題、能不能重製、能不能公開散布，這是你自己要處理的。自己出的題、公開授權的題、官方明確允許重製的題最安全。

程式碼 MIT，來自 [ipas-ai-quiz](https://github.com/yazelin/ipas-ai-quiz)（林亞澤）。

---

## 更新紀錄

- **2026-09-22**：修掉示範題庫的答案分佈——原本 20 題裡有 18 題的正解都是 (B)，全部選 B 就 90 分。選項順序打散成 A/B/C/D 各 5 題，題目與正解文字一字未變。另同步上游的手機分頁換行修正（六顆分頁在 430px 以下會擠成兩行，只有 430 放得下）。README 改寫成直播配套文件。
- **2026-09-21**：建立。第一段與第二段都實測走完，記下實際耗時與換裝清單。
