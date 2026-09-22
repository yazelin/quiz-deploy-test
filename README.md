# 台灣汽車駕照筆試（示範）

**這是 9/23 週三直播的預習素材。** 用 [ipas-ai-quiz](https://github.com/yazelin/ipas-ai-quiz) 換一份題庫做出來的，用途是證明「換題庫 → 推 repo → 開 Pages」這條路真的走得通。

- 線上：<https://yazelin.github.io/quiz-deploy-test/>
- **題目是 AI 自撰的 20 題示範題，不是官方試題**，答案僅供示範，不要拿來準備考照。
- 兩段都接上了：站本身（Tier 1）＋ 跨裝置同步與推播（Tier 2，Cloudflare Worker + D1）。

## 這是怎麼做出來的

從 clone 到站活著，指令大約六分鐘。完整清單與每一步的耗時寫在直播講義第 5 節。

```bash
# 第一段:站本身
git clone --depth 1 https://github.com/yazelin/ipas-ai-quiz.git site && cd site
#   換 questions.json 成自己的題庫(依 AGENTS.md 的 schema)
#   app.js 的 SYNC_URL 留空 → 同步與推播自動停用,站仍然完整
#   站名、title、description、og、canonical、manifest、robots、sitemap 全換
#   刪掉 assets/ 的舊圖與 .github/workflows
node tools/check-questions.mjs && node core.test.mjs
gh repo create <你的repo> --public --source=. --push
gh api -X POST repos/<你的帳號>/<你的repo>/pages -f 'source[branch]=main' -f 'source[path]=/'

# 第二段:跨裝置同步與推播(選做,要 Cloudflare 帳號)
cd worker && npm i
npx wrangler d1 create <你的資料庫名>        # 把 database_id 填進 wrangler.toml
npx wrangler d1 execute <你的資料庫名> --remote --file schema.sql
npx web-push generate-vapid-keys --json     # 公鑰填 wrangler.toml 與 app.js
npx wrangler deploy
npx wrangler secret put VAPID_PRIVATE       # 私鑰只放這裡,不要進 git
#   最後把 Worker 網址填回 app.js 的 SYNC_URL
```

**最容易死的一步**：`wrangler d1 create` 成功後會印一段設定叫你貼進 `wrangler.toml`，裡面寫 `binding = "<你的資料庫名>"`。**照貼就死**——程式用的是 `env.DB`，部署會過、一打 API 就 500。binding 必須維持 `DB`。

程式碼 MIT，來自 [ipas-ai-quiz](https://github.com/yazelin/ipas-ai-quiz)（林亞澤）。
