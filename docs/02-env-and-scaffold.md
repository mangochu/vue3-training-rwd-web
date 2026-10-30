# 02. 環境與專案初始化

## 目標

- 用 `pnpm create vue` 建立 Vue 3 + TS 專案
- 看懂每個自動產生的檔案在做什麼
- 建立 Git repo，推上 GitHub

---

## 1. 確認環境

在終端機（PowerShell 或 VS Code 內建終端機）執行：

```powershell
node -v    # 需要 v22.18+ 或 v24.12+（看產生的 package.json 裡的 engines 欄位），建議用 LTS 版
pnpm -v    # 有版本號就好
git --version
```

還沒有 pnpm 的話，可以用以下任一種方式安裝：

```powershell
# 方式 A：用 Node 內建的 corepack（建議）
corepack enable pnpm

# 方式 B：官方安裝腳本（PowerShell）
Invoke-WebRequest https://get.pnpm.io/install.ps1 -UseBasicParsing | Invoke-Expression
```

### VS Code 擴充套件

| 套件 | 用途 |
| --- | --- |
| **Vue - Official**（`Vue.volar`） | Vue 語法、TS 型別提示（**必裝**） |
| **ESLint** | 即時顯示 lint 錯誤 |
| **Prettier - Code formatter** | 存檔時自動排版 |

> 如果有裝舊的 **Vetur**，請停用，它和 Vue - Official 會衝突。

---

## 2. 建立專案

專案資料夾裡已經有 `README.md`、`docs/`、`CLAUDE.md`，**不是空資料夾**。
如果直接在這裡建立專案，create-vue 會問你要不要清空資料夾，選「是」的話**這些教學文件會被刪掉**。

所以先在暫存資料夾建立，再把檔案搬過來：

```powershell
cd D:\ClaudeCode
pnpm create vue@latest vue-scaffold-tmp
```

> ⚠️ 一定要先 `cd` 到 `D:\ClaudeCode`（專案的**上一層**）再執行。
> 如果在專案資料夾裡面執行，會變成 `vue3-training-rwd-web\vue-scaffold-tmp\` 這樣多一層。
> 不小心建錯位置也沒關係，用下面的搬移指令把檔案搬到根目錄就好。

問答選項見下一節。建立完成後，把檔案搬進專案資料夾：

```powershell
# 範本產生的 README.md 先改名，避免覆蓋掉教學用的 README
Rename-Item D:\ClaudeCode\vue-scaffold-tmp\README.md README.scaffold.md

# 搬移所有檔案（-Force 會一起搬隱藏檔，例如 .gitignore、.vscode）
Get-ChildItem D:\ClaudeCode\vue-scaffold-tmp -Force | Move-Item -Destination D:\ClaudeCode\vue3-training-rwd-web

# 刪除已經空了的暫存資料夾
Remove-Item D:\ClaudeCode\vue-scaffold-tmp
```

最後打開 `package.json`，把 `"name"` 改成 `"vue3-training-rwd-web"`。
`README.scaffold.md` 讀完之後可以刪掉。

### 互動式問答怎麼選

操作方式：`↑` / `↓` 移動，**空白鍵**勾選，`Enter` 確認。
create-vue 的問題會隨版本有一點不同，以下是 2026 年版本的畫面，原則都一樣。

**Features（功能）：勾 TypeScript、Router、Vitest、Linter、Prettier**

| 選項 | 選擇 | 理由 |
| --- | --- | --- |
| TypeScript | ✅ | 本專案重點（如果清單裡沒有，往上捲看看，或是會另外問） |
| JSX Support | ❌ | JSX 是用 JS 函式寫畫面的另一種方式，Vue 用 `<template>` 就夠了 |
| Router (SPA development) | ✅ | 第 06 章要用 |
| Pinia (state management) | ❌ | 這個網站的狀態很少，先不用；第 09 章想練再加 |
| Vitest (unit testing) | ✅ | 第 07 章要用 |
| End-to-End Testing | ❌ | 先不碰 |
| Linter (error prevention) | ✅ | 會**同時安裝 ESLint 和 Oxlint**，見第 3 節的說明 |
| Prettier (code formatting) | ✅ | 程式碼排版 |

**Experimental features（實驗性功能）：全部不勾，直接按 Enter**

| 選項 | 是什麼 | 為什麼不選 |
| --- | --- | --- |
| Replace Prettier with Oxfmt | 用 Rust 寫的 Oxfmt 取代 Prettier，速度更快 | Prettier 是業界標準，資料最多 |
| Vue 3.6 (Release Candidate) | Vue 3.6 的候選版（含 Vapor Mode 等新功能） | 還不是正式版，可能有 bug |
| Replace TypeScript with typescript-native-bridge (tsgo) | 用 Go 重寫的新版 TS 編譯器做型別檢查 | 和 `vue-tsc`、編輯器的整合還在實驗階段 |

> 學習時要先用穩定的工具，出錯時才能確定是自己的程式有問題。

**是否建立不含範例程式碼的空白專案**：先選 **No**，留著範例可以學它怎麼寫，第 03 章再刪。

### 安裝與啟動

```powershell
cd D:\ClaudeCode\vue3-training-rwd-web
pnpm install
pnpm dev
```

打開終端機顯示的網址（通常是 `http://localhost:5173`），看到 Vue 的歡迎頁就成功了。

> **可能遇到的訊息**：pnpm 10 之後，預設**不會自動執行**套件的安裝腳本，可能會看到
> `Ignored build scripts: esbuild ...` 這類警告。執行 `pnpm approve-builds`，勾選它列出的套件即可。
> 這是 pnpm 的安全機制，目的是防止惡意套件在安裝時執行程式。

---

## 3. 看懂產生的檔案

**不要跳過這一節。** 打開每個檔案看一下，對照下表：

| 檔案 | 作用 | 你需要知道的事 |
| --- | --- | --- |
| `package.json` | 專案資訊、依賴、指令 | 看 `scripts` 區塊，`pnpm <名稱>` 就是執行對應的指令 |
| `pnpm-lock.yaml` | 鎖定每個套件的確切版本 | **要 commit**，CI 會依照它安裝 |
| `vite.config.ts` | Vite 設定 | `@` 別名指向 `src/`；第 08 章會在這裡加 `base` |
| `tsconfig.json` | TS 設定入口 | 透過 `references` 分成下面幾份 |
| `tsconfig.app.json` | `src/` 底下程式碼的 TS 設定 | 瀏覽器環境 |
| `tsconfig.node.json` | `vite.config.ts` 等設定檔的 TS 設定 | Node 環境 |
| `tsconfig.vitest.json` | 測試檔的 TS 設定 | |
| `vitest.config.ts` | Vitest 設定 | 用 `mergeConfig` 繼承 `vite.config.ts`，所以測試裡也能用 `@` 別名 |
| `env.d.ts` | 型別宣告 | 讓 TS 認得 `import.meta.env`、`.vue` 檔 |
| `index.html` | 網頁入口 | Vite 從這裡開始，`<div id="app">` 是 Vue 掛載點 |
| `src/main.ts` | JS 入口 | `createApp(App).use(router).mount('#app')` |
| `src/App.vue` | 根元件 | `<RouterView />` 是路由頁面的顯示位置 |
| `src/router/index.ts` | 路由設定 | 第 06 章詳細說明 |
| `eslint.config.ts` | ESLint 設定（Flat Config 格式） | 裡面用 `eslint-plugin-oxlint` 關掉和 Oxlint 重複的規則 |
| `.oxlintrc.json` | Oxlint 設定 | 見下方說明 |
| `.prettierrc.json` | Prettier 設定 | 可以在這裡改分號、引號等偏好 |
| `.editorconfig` | 編輯器共用設定 | 縮排、換行符號等，各種編輯器都看得懂 |
| `.gitattributes` | Git 設定 | 統一換行符號為 LF，避免 Windows / Mac 混用時出現整個檔案都被改到的 diff |

### 看一下 `package.json` 的 scripts

```jsonc
{
  "scripts": {
    "dev": "vite",                                   // 開發伺服器
    "build": "run-p type-check \"build-only {@}\" --", // 型別檢查 + 打包（run-p = 平行執行）
    "preview": "vite preview",                       // 預覽打包後的結果
    "test:unit": "vitest",                           // 單元測試（監看模式）
    "build-only": "vite build",                      // 只打包、不做型別檢查
    "type-check": "vue-tsc --build",                 // 只做型別檢查
    "lint": "run-s \"lint:*\"",                      // 依序執行所有 lint: 開頭的指令（run-s = 依序執行）
    "lint:oxlint": "oxlint . --fix",                 // 先用 Oxlint 檢查並修正
    "lint:eslint": "eslint . --fix --cache",         // 再用 ESLint 檢查並修正
    "format": "prettier --write --experimental-cli src/" // 格式化
  }
}
```

（實際內容可能因版本略有不同，以你產生的為準。）

### 為什麼有兩個 Linter？

| | Oxlint | ESLint |
| --- | --- | --- |
| 寫成的語言 | Rust | JavaScript |
| 速度 | 非常快（快 50～100 倍） | 較慢 |
| 負責 | 常見的 JS / TS 錯誤 | Vue 模板相關規則（`eslint-plugin-vue`）等 Oxlint 還不支援的部分 |

兩者分工合作：Oxlint 先快速抓掉大部分問題，ESLint 再處理 Vue 專屬的規則。
`eslint.config.ts` 裡的 `eslint-plugin-oxlint` 會把 Oxlint 已經檢查過的規則在 ESLint 裡關掉，避免重複檢查。
**兩個都保留，不用刪。**

> **小練習**：分別執行 `pnpm build` 和 `pnpm preview`，看看 `dist/` 資料夾裡產生了什麼。
> 這個 `dist/` 就是最後要部署到 GitHub Pages 的東西。

---

## 4. 鎖定 pnpm 版本

在 `package.json` 加上 `packageManager` 欄位，讓本機和 CI 使用同一個 pnpm 版本：

```powershell
pnpm -v   # 例如 12.14.0
```

```jsonc
{
  "name": "vue3-training-rwd-web",
  "packageManager": "pnpm@12.14.0",  // ← 換成你的版本
  // ...
}
```

> 第 08 章的 GitHub Actions 會讀這個欄位來安裝對應的 pnpm 版本。

---

## 5. Git 與 GitHub

### 5-1. 初始化與第一個 commit

> 本專案在加入教學文件時已經執行過 `git init`，前兩行可以跳過，從 `git status` 開始。

```powershell
git init
git branch -M main
git status          # 確認 node_modules、dist 沒有出現（.gitignore 已經排除）
git add .
git commit -m "chore: init project with create-vue"
```

### 5-2. 建立 GitHub repo

1. 到 GitHub → 右上角 `+` → **New repository**
2. Repository name：`vue3-training-rwd-web`（之後的網址會是 `https://<帳號>.github.io/vue3-training-rwd-web/`）
   - 如果想要網址是 `https://<帳號>.github.io/`，repo 名稱要取成 **`<帳號>.github.io`**，第 08 章的 `base` 設定也要跟著改
3. 選 **Public**（免費帳號的 GitHub Pages 需要 public repo）
4. **不要**勾選 Add README / .gitignore / license（本機已經有了，勾了會衝突）
5. 建立後，照著頁面上 "push an existing repository" 的指令執行：

```powershell
git remote add origin https://github.com/<你的帳號>/vue3-training-rwd-web.git
git push -u origin main
```

### 5-3. Commit 訊息規範（Conventional Commits）

從現在開始，commit 訊息用這種格式：

```
<類型>: <簡短說明>
```

| 類型 | 用途 | 範例 |
| --- | --- | --- |
| `feat` | 新功能 | `feat: add project card component` |
| `fix` | 修 bug | `fix: close mobile menu after navigation` |
| `style` | 樣式調整（不影響邏輯） | `style: adjust hero spacing on mobile` |
| `refactor` | 重構 | `refactor: extract useScrollSpy composable` |
| `docs` | 文件 | `docs: update README` |
| `test` | 測試 | `test: add filterProjects unit test` |
| `chore` | 雜項（設定、依賴） | `chore: add github actions workflow` |
| `ci` | CI 設定 | `ci: cache pnpm store` |

---

## 驗收

- [ ] `pnpm dev` 能看到 Vue 歡迎頁
- [ ] `pnpm build` 成功，產生 `dist/`
- [ ] 我能說出 `tsconfig.app.json` 和 `tsconfig.node.json` 的差別
- [ ] `package.json` 有 `packageManager` 欄位
- [ ] GitHub 上看得到我的程式碼

<details>
<summary>卡關提示：push 時要求登入 / 失敗</summary>

- GitHub 已經不支援用密碼 push，需要用 **Personal Access Token** 或 **Git Credential Manager**（Windows 版 Git 通常內建，會跳出瀏覽器登入視窗）。
- 也可以安裝 GitHub CLI 後執行 `gh auth login`。

</details>

<details>
<summary>卡關提示：VS Code 在 .vue 檔裡一直出現紅線</summary>

1. 確認裝的是 **Vue - Official** 而不是 Vetur
2. `Ctrl + Shift + P` → `Developer: Reload Window`
3. 確認 VS Code 開啟的是**專案根目錄**（看得到 `package.json` 的那層）

</details>
