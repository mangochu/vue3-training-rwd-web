# 02. 環境與專案初始化

## 目標

- 用 `pnpm create vue` 建立 Vue 3 + TS 專案
- 看懂每個自動產生的檔案在做什麼
- 建立 Git repo，推上 GitHub

---

## 1. 確認環境

在終端機（PowerShell 或 VS Code 內建終端機）執行：

```powershell
node -v    # 需要 v20.19+ 或 v22.12+（Vite 7 的要求），建議用 LTS 版
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

create-vue 的問題會隨版本有一點不同，原則如下：

| 問題 | 選擇 | 理由 |
| --- | --- | --- |
| TypeScript | ✅ | 本專案重點 |
| JSX | ❌ | 用 Vue 模板就好 |
| Router（SPA 路由） | ✅ | 第 06 章要用 |
| Pinia（狀態管理） | ❌ | 這個網站的狀態很少，先不用；第 09 章想練再加 |
| Vitest（單元測試） | ✅ | 第 07 章要用 |
| End-to-End 測試 | ❌ | 先不碰 |
| ESLint | ✅ | 程式碼品質 |
| Prettier | ✅ | 程式碼排版 |
| 實驗性功能（例如 Oxlint、Rolldown） | ❌ | 練習專案先用穩定的工具 |
| 是否建立不含範例程式碼的空白專案 | ❌ **先選否** | 留著範例可以學它怎麼寫，第 03 章再刪 |

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
| `env.d.ts` | 型別宣告 | 讓 TS 認得 `import.meta.env`、`.vue` 檔 |
| `index.html` | 網頁入口 | Vite 從這裡開始，`<div id="app">` 是 Vue 掛載點 |
| `src/main.ts` | JS 入口 | `createApp(App).use(router).mount('#app')` |
| `src/App.vue` | 根元件 | `<RouterView />` 是路由頁面的顯示位置 |
| `src/router/index.ts` | 路由設定 | 第 06 章詳細說明 |
| `eslint.config.ts`（或 `.js`） | ESLint 設定（Flat Config 格式） | |
| `.prettierrc.json` | Prettier 設定 | 可以在這裡改分號、引號等偏好 |

### 看一下 `package.json` 的 scripts

```jsonc
{
  "scripts": {
    "dev": "vite",                       // 開發伺服器
    "build": "run-p type-check \"build-only {@}\" --",  // 型別檢查 + 打包（平行執行）
    "preview": "vite preview",           // 預覽打包後的結果
    "test:unit": "vitest",               // 單元測試
    "build-only": "vite build",
    "type-check": "vue-tsc --build",     // 只做型別檢查
    "lint": "eslint . --fix",            // 檢查並自動修正
    "format": "prettier --write src/"    // 格式化
  }
}
```

（實際內容可能因版本略有不同，以你產生的為準。）

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
