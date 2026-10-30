# 09. 進階挑戰（選做）

每一項都獨立，挑自己有興趣的做。**每一項都用一個 feature 分支 + PR 來完成**（第 07 章的流程）。

---

## A. 深色模式 ★★☆

**練習重點**：composable、`localStorage`、CSS 變數切換、`prefers-color-scheme`

### 規格

1. 第一次造訪時，跟隨作業系統設定（`prefers-color-scheme`）
2. Header 有切換按鈕，使用者手動切換後，記住他的選擇（`localStorage`）
3. 切換方式：在 `<html>` 加上 `data-theme="dark"`，CSS 用 `[data-theme="dark"] { --color-bg: ...; }` 覆蓋變數

### 實作提示

```ts
// src/composables/useTheme.ts
type Theme = 'light' | 'dark'

export function useTheme() {
  // 1. 讀 localStorage，沒有的話用 window.matchMedia('(prefers-color-scheme: dark)').matches
  // 2. 用 ref<Theme> 存狀態
  // 3. 用 watch 監聽變化：更新 document.documentElement.dataset.theme 和 localStorage
  // 4. 回傳 { theme, toggleTheme }
}
```

### 進階問題

- 重新整理時，畫面會先閃一下淺色再變深色（**FOUC**），為什麼？
  提示：Vue 還沒載入之前，`<html>` 上還沒有 `data-theme`。可以在 `index.html` 的 `<head>` 放一小段**同步執行**的 inline script 先設定好。
- 多個元件都呼叫 `useTheme()` 時，狀態會共用嗎？怎麼讓它共用？（提示：把 `ref` 宣告在函式**外面**，或改用 Pinia）

---

## B. 捲動進場動畫 ★★☆

**練習重點**：自訂指令（Custom Directive）、IntersectionObserver

### 規格

- 寫一個 `v-reveal` 指令，元素進入畫面時淡入並往上移動
- 只觸發一次
- 使用者開啟「減少動態效果」時不播放動畫（`prefers-reduced-motion`）

```ts
// src/directives/reveal.ts
import type { Directive } from 'vue'

export const vReveal: Directive<HTMLElement> = {
  mounted(el) { /* TODO */ },
  unmounted(el) { /* TODO：別忘了清理 observer */ },
}
```

在 `<script setup>` 裡，以 `v` 開頭的變數可以直接當指令使用：`<section v-reveal>`。

---

## C. 無障礙（a11y）檢查 ★☆☆

逐項檢查並修正：

- [ ] 只用鍵盤（`Tab`、`Shift+Tab`、`Enter`、`Esc`）能操作整個網站
- [ ] 所有可互動元素都有清楚的 `:focus-visible` 樣式
- [ ] 加上「跳到主要內容」連結（Skip Link）：平常隱藏，按 Tab 時第一個出現
- [ ] 標題層級正確：只有一個 `h1`，`h2` 底下才有 `h3`，不跳級
- [ ] 顏色對比通過 4.5:1（深色模式也要檢查）
- [ ] 用 Windows 內建的「朗讀程式」（`Ctrl + Win + Enter`）聽一遍首頁

工具：Chrome 擴充套件 **axe DevTools**、DevTools 的 **Lighthouse → Accessibility**。

---

## D. Lighthouse 跑到 90+ ★★☆

DevTools → **Lighthouse** → 勾選 Mobile → **Analyze page load**。
**請用部署後的網址測**（`pnpm dev` 的開發模式沒有壓縮，分數不準）。

常見的改善項目：

| 問題 | 解法 |
| --- | --- |
| 圖片太大 | 轉成 WebP / AVIF、壓縮（[Squoosh](https://squoosh.app)）、設定 `width` / `height` 避免版面跳動（CLS） |
| 字型阻塞渲染 | Google Fonts 網址加 `&display=swap`、加 `<link rel="preconnect">` |
| 首屏外的圖片 | `loading="lazy"`（但 Hero 圖**不要**加） |
| 缺少 meta description | 見下一項 |

---

## E. SEO 與社群分享卡片 ★☆☆

在 `index.html` 加上：

```html
<meta name="description" content="..." />
<!-- Open Graph：分享到 Facebook / LINE / Slack 時顯示的卡片 -->
<meta property="og:title" content="..." />
<meta property="og:description" content="..." />
<meta property="og:image" content="https://<帳號>.github.io/<repo>/og-image.png" />
<meta property="og:url" content="https://<帳號>.github.io/<repo>/" />
<meta name="twitter:card" content="summary_large_image" />
```

- `og:image` 建議 1200×630，放在 `public/`，網址要寫**完整的絕對網址**
- 驗證工具：把網址貼到 Facebook 的 [Sharing Debugger](https://developers.facebook.com/tools/debug/)

---

## F. 多語系 ★★★

**練習重點**：第三方套件整合、TS 型別安全的翻譯 key

```powershell
pnpm add vue-i18n
```

- 支援中文 / 英文切換
- `data/` 裡的內容也要能翻譯：想想資料結構要怎麼改（例如 `title: { zh: '...', en: '...' }`）
- 進階：讓翻譯 key 打錯時 TS 會報錯

---

## G. 自訂網域 ★☆☆

1. 買一個網域（Cloudflare、Namecheap、Gandi 等）
2. GitHub repo → Settings → Pages → **Custom domain** 填入網域
3. 到網域商設定 DNS（GitHub 文件：[Managing a custom domain](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site)）
4. ⚠️ 改成自訂網域後，網站就在根目錄了，`vite.config.ts` 的 `base` 要改回 `'/'`
5. 勾選 **Enforce HTTPS**

---

## H. 從 GitHub API 讀取 repo 資料 ★★★

**練習重點**：`fetch`、非同步狀態（loading / error）、API 回應的型別定義

- 呼叫 `https://api.github.com/users/<帳號>/repos?sort=updated`，顯示最近更新的 repo
- 處理三種狀態：載入中（skeleton）、錯誤、成功
- 為 API 回應定義 interface（只定義你用到的欄位就好）
- 抽成 composable：`useGithubRepos(username)`
- 注意：未登入的 API 每小時有呼叫次數上限，想想看要怎麼處理（快取到 `sessionStorage`？）

---

## 做完之後

1. 更新 `README.md`：加上網站截圖、線上網址、技術棧、你學到了什麼
2. 把這個 repo 釘選（Pin）到你的 GitHub 個人頁面
3. 回頭看第 05 章寫的元件，用現在的眼光重構一次，你會發現自己進步很多
