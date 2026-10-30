# 08. CI/CD 與 GitHub Pages 部署

## 目標

- **CI**：每次 PR 自動執行 lint、type-check、test、build
- **CD**：合併到 `main` 後自動部署到 GitHub Pages
- 解決 SPA 部署到 GitHub Pages 常見的兩個問題：**白畫面**、**重新整理 404**

---

## 1. 觀念

```
你 push / 開 PR
      │
      ▼
┌─────────────── GitHub Actions ───────────────┐
│ ci.yml（PR 時）                                │
│   checkout → 裝 pnpm → 裝 Node → install       │
│   → lint → type-check → test → build          │
│                                               │
│ deploy.yml（push 到 main 時）                  │
│   build job：install → build → 上傳 dist       │
│   deploy job：把 dist 發佈到 GitHub Pages      │
└───────────────────────────────────────────────┘
      │
      ▼
https://<帳號>.github.io/vue3-training-rwd-web/
```

| 名詞 | 意思 |
| --- | --- |
| Workflow | 一份 `.github/workflows/*.yml`，定義「什麼時候」要「做什麼」 |
| Job | Workflow 裡的一組工作，每個 job 在**全新的虛擬機**上執行 |
| Step | Job 裡的每一步，可以是指令（`run`）或別人寫好的 Action（`uses`） |
| Action | 可重用的步驟，例如 `actions/checkout`，可以在 [GitHub Marketplace](https://github.com/marketplace?type=actions) 找到 |

---

## 2. 設定 Vite 的 `base`

GitHub Pages 的網址是 `https://<帳號>.github.io/<repo>/`，網站放在 **`/<repo>/` 子路徑**底下。
但 Vite 預設認為網站在根目錄 `/`，所以打包出來的 `<script src="/assets/xxx.js">` 會去找
`https://<帳號>.github.io/assets/xxx.js`，結果找不到檔案 → **白畫面**。

修改 `vite.config.ts`：

```ts
export default defineConfig({
  base: '/vue3-training-rwd-web/',   // ← 前後都要有斜線，名稱要和 repo 一樣
  plugins: [ /* ... */ ],
  // ...
})
```

> 如果 repo 名稱是 `<帳號>.github.io`，網站就在根目錄，`base` 維持 `'/'` 即可。

### 驗證

```powershell
pnpm build
pnpm preview
```

打開 `http://localhost:4173/vue3-training-rwd-web/`，確認畫面正常。

### 檢查 public 檔案的引用

還記得第 03 章說不要寫 `/resume.pdf` 嗎？現在你應該懂原因了。全專案搜尋 `"/` 和 `'/`，
找出所有寫死根路徑的地方，改成：

```ts
const resumeUrl = `${import.meta.env.BASE_URL}resume.pdf`
```

---

## 3. 解決「重新整理 404」

**問題**：使用者在 `/vue3-training-rwd-web/projects/weather-app` 按 F5 重新整理時，
GitHub Pages 會去找 `projects/weather-app/index.html` 這個檔案。但它不存在，因為 SPA 只有一個 `index.html`，所以會回應 404。

**解法**：GitHub Pages 找不到檔案時會顯示 `404.html`。
所以在打包後**把 `index.html` 複製一份成 `404.html`**，Vue Router 載入後就會依照網址顯示正確頁面。

這一步會在下面的 deploy workflow 裡處理。

> 這招的副作用：HTTP 狀態碼仍然是 404，對 SEO 不太好，但個人作品集網站可以接受。
> 如果很在意，就改用 `createWebHashHistory`（代價見第 06 章）。

---

## 4. 開啟 GitHub Pages

GitHub repo → **Settings** → 左側 **Pages** → **Build and deployment** → **Source** 選 **GitHub Actions**。

> 不要選 "Deploy from a branch"，那是舊做法，需要另外維護一個 `gh-pages` 分支。

---

## 5. 撰寫 deploy workflow

建立 `.github/workflows/deploy.yml`。**逐行讀懂註解**再 commit：

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]        # push 到 main 時觸發（PR 合併也算）
  workflow_dispatch:        # 允許在 Actions 頁面手動觸發

# GITHUB_TOKEN 的權限：部署到 Pages 需要這三個
permissions:
  contents: read
  pages: write
  id-token: write

# 同一時間只跑一個部署，新的進來時不要取消正在跑的
concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      # 要先裝 pnpm，下一步 setup-node 的 cache: pnpm 才找得到 pnpm
      # 沒有指定 version 時，會讀取 package.json 的 packageManager 欄位（第 02 章設定的）
      - name: Setup pnpm
        uses: pnpm/action-setup@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 24
          cache: pnpm        # 快取 pnpm store，第二次之後安裝會快很多

      # --frozen-lockfile：lockfile 和 package.json 不一致時直接失敗，確保 CI 和本機一樣
      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Build
        run: pnpm build

      # 第 3 節的 SPA 404 解法
      - name: SPA fallback
        run: cp dist/index.html dist/404.html

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: dist

  deploy:
    needs: build              # 等 build job 成功才執行
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

> **關於 Action 版本**：`@v4` 這類版本號會隨時間更新。到各 Action 的 GitHub 頁面（例如 `github.com/actions/checkout`）
> 看 README 推薦的最新主版本，有新版就換上。

### Commit 並觀察

```powershell
git add .
git commit -m "ci: add github pages deploy workflow"
git push
```

到 repo 的 **Actions** 分頁，點進正在跑的 workflow，展開每個 step 看 log。
成功後，`deploy` job 上會顯示網站網址。

---

## 6. 任務：自己寫 CI workflow

建立 `.github/workflows/ci.yml`，**這份請自己寫**。規格如下：

| 項目 | 規格 |
| --- | --- |
| 觸發時機 | 開 PR 到 `main` 時（`pull_request`） |
| 權限 | 只需要 `contents: read` |
| 步驟 | checkout → pnpm → node（含 cache）→ install → **lint** → **type-check** → **test** → build |
| 注意 | 測試要用「只跑一次」的模式，不然 CI 會卡在監看模式永遠不會結束 |
| 注意 | `pnpm lint` 裡的 `lint:oxlint` 和 `lint:eslint` 都帶 `--fix`，CI 上應該要「只檢查不修改」，想想看怎麼處理 |

完成後：

1. 開一個分支，**故意**寫一個型別錯誤，push 並開 PR，確認 CI 會失敗 ❌
2. 修好後再 push，確認變成 ✅
3. 回到第 07 章，設定 main 分支保護，把這個 CI job 設成必要檢查

<details>
<summary>卡關提示：lint 在 CI 上不要 --fix</summary>

在 `package.json` 加一個「只檢查」的指令，CI 執行這個：

```jsonc
"check:lint": "oxlint . && eslint ."
```

注意名稱**不能用 `lint:` 開頭**。因為 `pnpm lint` 是 `run-s "lint:*"`，會執行所有 `lint:` 開頭的指令，
取名 `lint:check` 的話會被一起執行到。

也可以直接在 workflow 寫 `pnpm exec oxlint . && pnpm exec eslint .`，但寫在 `package.json` 比較好，因為本機也能跑一樣的指令。

</details>

<details>
<summary>卡關提示：CI 的步驟骨架</summary>

```yaml
name: CI
on:
  pull_request:
    branches: [main]
permissions:
  contents: read
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      # checkout、pnpm、node、install 和 deploy.yml 一樣
      # 然後依序：
      # - run: pnpm check:lint
      # - run: pnpm type-check
      # - run: pnpm test:unit --run
      # - run: pnpm build-only
```

</details>

---

## 7. 常見問題排查

| 症狀 | 可能原因 | 解法 |
| --- | --- | --- |
| 網站一片白，Console 出現 js 檔 404 | `base` 沒設定或拼錯 | 檢查 `vite.config.ts` 的 `base`，大小寫要和 repo 名稱完全一樣 |
| 首頁正常，重新整理子頁面變成 GitHub 的 404 頁 | 沒有 `404.html` | 檢查 workflow 的 SPA fallback 步驟 |
| 圖片或 PDF 404 | 引用 `public/` 檔案時寫死了 `/` 開頭的路徑 | 改用 `import.meta.env.BASE_URL` |
| `pnpm install --frozen-lockfile` 失敗 | 改了 `package.json` 但沒 commit `pnpm-lock.yaml` | 本機執行 `pnpm install` 後 commit lockfile |
| `Unable to locate executable file: pnpm` | `pnpm/action-setup` 放在 `setup-node` 之後 | 調整順序 |
| deploy job 出現權限錯誤 | 少了 `permissions` 或 Pages 的 Source 沒選 GitHub Actions | 檢查第 4、5 節 |
| CI 在測試步驟卡住不動 | Vitest 進入了監看模式 | 加上 `--run` |

---

## 驗收

- [ ] 網站上線：`https://<帳號>.github.io/vue3-training-rwd-web/`
- [ ] 在子頁面按 F5 重新整理，畫面正常
- [ ] 履歷 PDF 下載正常
- [ ] PR 會自動跑 CI，故意寫錯時會失敗
- [ ] `main` 分支已設定保護
- [ ] 用手機實際打開網站看過（不只是 DevTools 模擬）
- [ ] 把網站網址加到 GitHub repo 右上角的 **About → Website**
