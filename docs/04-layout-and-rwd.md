# 04. 全域樣式與 RWD 佈局

## 目標

- 建立 CSS Reset、Design Tokens、全域樣式
- 理解並實作 **Mobile First**
- 完成 `AppHeader`（含手機漢堡選單）與 `AppFooter`
- 學會用 DevTools 測試 RWD

---

## 1. RWD 核心觀念

### 1-1. viewport meta

打開 `index.html`，確認有這行（create-vue 預設就有）：

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
```

> **小實驗**：暫時刪掉這行，用 DevTools 手機模式看看會變怎樣，看完再加回去。
> 沒有這行的話，手機瀏覽器會用約 980px 的寬度渲染，再整頁縮小，你寫的 media query 都不會生效。

### 1-2. Mobile First

**先寫手機版樣式，再用 `min-width` 往大螢幕加樣式。**

```css
/* ✅ Mobile First：預設就是手機版 */
.cards {
  display: grid;
  grid-template-columns: 1fr;        /* 手機：1 欄 */
}
@media (min-width: 768px) {
  .cards { grid-template-columns: repeat(2, 1fr); }  /* 平板：2 欄 */
}
@media (min-width: 1024px) {
  .cards { grid-template-columns: repeat(3, 1fr); }  /* 桌機：3 欄 */
}
```

```css
/* ❌ Desktop First：先寫桌機，再用 max-width 覆蓋回來 */
.cards { grid-template-columns: repeat(3, 1fr); }
@media (max-width: 1023px) { ... }
@media (max-width: 767px) { ... }
```

為什麼 Mobile First 比較好：

- 手機版通常比較簡單（單欄），往上加樣式比往下拆容易
- 手機瀏覽器不用先讀一堆桌機樣式再覆蓋掉
- 逼你先想清楚「最重要的內容是什麼」

### 1-3. 什麼時候用 Flexbox？什麼時候用 Grid？

| | Flexbox | Grid |
| --- | --- | --- |
| 維度 | 一維（一排或一列） | 二維（行與列同時控制） |
| 適合 | 導覽列、按鈕群組、標籤列、卡片內部排列 | 卡片列表、整頁版面、圖文左右排列 |
| 本專案用在 | Header、Footer、技能標籤、按鈕組 | Projects 卡片列表、About 圖文區、Hero |

> 練習網站：[Flexbox Froggy](https://flexboxfroggy.com/#zh-tw)、[Grid Garden](https://cssgridgarden.com/#zh-tw)，各花 30 分鐘玩完再繼續。

### 1-4. 減少 media query 的技巧

```css
/* 技巧 1：clamp() 讓字級平滑縮放 */
h1 { font-size: clamp(2rem, 1.2rem + 4vw, 4rem); }
/* 最小 2rem，最大 4rem，中間依螢幕寬度線性變化 */

/* 技巧 2：auto-fit + minmax() 讓 Grid 自己決定欄數，完全不用 media query */
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: var(--space-4);
}

/* 技巧 3：用 max-width + margin-inline: auto 限制內容最大寬度 */
.container {
  width: 100%;
  max-width: 1120px;
  margin-inline: auto;
  padding-inline: var(--space-3);
}
```

> **任務**：Projects 卡片用「技巧 2」和「1-2 的三個 media query」各寫一次，比較兩者的差別。
> 想想：什麼情況下你會**需要**明確控制欄數？

---

## 2. 建立全域樣式

### 2-1. `src/assets/styles/reset.css`

瀏覽器的預設樣式不一致，Reset 用來把它們統一。這份可以直接使用，但**每一行都要讀懂**：

```css
*, *::before, *::after {
  box-sizing: border-box;   /* width 包含 padding 和 border，算尺寸比較直覺 */
}

* {
  margin: 0;
}

html {
  -webkit-text-size-adjust: 100%;  /* 防止 iOS 橫向時自動放大字 */
}

body {
  min-height: 100vh;
  line-height: 1.6;
  -webkit-font-smoothing: antialiased;
}

img, picture, svg, video {
  display: block;
  max-width: 100%;          /* RWD 關鍵：圖片不會撐破容器 */
}

input, button, textarea, select {
  font: inherit;            /* 表單元素預設不繼承字型 */
}

p, h1, h2, h3, h4 {
  overflow-wrap: break-word; /* 長網址或長英文單字自動換行，避免水平捲軸 */
}

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

> 參考：Josh Comeau 的 [A Modern CSS Reset](https://www.joshwcomeau.com/css/custom-css-reset/)，裡面有每一行的詳細解釋。

### 2-2. `src/assets/styles/tokens.css`

把第 01 章的 Design Tokens 寫成 CSS 變數，**這份請你自己完成**：

```css
:root {
  /* 顏色 */
  --color-bg: #ffffff;
  --color-text: #0f172a;
  --color-primary: #4f46e5;
  /* TODO：補完第 01 章表格裡的其他顏色 */

  /* 字型 */
  --font-sans: "Noto Sans TC", system-ui, sans-serif;
  /* TODO：字級 --fs-sm ～ --fs-hero */

  /* 間距 */
  /* TODO：--space-1 ～ --space-6 */

  /* 其他 */
  --radius: 0.75rem;
  --header-height: 64px;
  --container-max: 1120px;
  --transition: 200ms ease;
}
```

> ⚠️ **CSS 變數不能用在 media query 的條件裡**：`@media (min-width: var(--bp-md))` 無效。
> 斷點只能直接寫數字（`768px`），或改用 SCSS 變數 / mixin（見本章最後的選讀）。

### 2-3. `src/assets/styles/global.css`

**請你自己寫**，至少包含：

- `body` 套用 `--color-bg`、`--color-text`、`--font-sans`
- `a` 的顏色與 hover 樣式
- `.container`（1-4 的技巧 3）
- `.section`：每個區塊上下的間距（手機小一點，桌機大一點）
- `html { scroll-behavior: smooth; }`，讓錨點連結平滑捲動
- `html { scroll-padding-top: var(--header-height); }`，避免錨點捲動時標題被固定的 Header 擋住
- `.visually-hidden`：畫面上看不到，但螢幕閱讀器讀得到（漢堡按鈕的文字會用到，可以搜尋 "visually hidden css"）

### 2-4. 在 `main.ts` 引入

```ts
import './assets/styles/reset.css'
import './assets/styles/tokens.css'
import './assets/styles/global.css'
```

> 順序很重要：reset → tokens → global。

### 2-5. 中文字型（選做）

到 [Google Fonts](https://fonts.google.com/noto/specimen/Noto+Sans+TC) 選 Noto Sans TC，
把它給的 `<link>` 貼到 `index.html` 的 `<head>`。
中文字型檔案很大，**只選 400 和 700 兩種字重**就好。

---

## 3. 實作 AppHeader

### 需求規格

| 寬度 | 行為 |
| --- | --- |
| 所有寬度 | 固定在頂部（`position: sticky` 或 `fixed`），左邊 Logo / 名字，右邊導覽 |
| < 768px | 導覽連結隱藏，顯示漢堡按鈕 ≡；點擊後展開選單，按鈕變成 ✕ |
| ≥ 768px | 導覽連結橫向排列，隱藏漢堡按鈕 |

選單互動：

1. 點擊選單裡的連結後，選單要**自動關閉**
2. 按 `Esc` 鍵要關閉選單
3. 選單打開時，背景頁面**不能捲動**（選做）

無障礙（a11y）：

- 漢堡按鈕要用 `<button>`，**不要用 `<div>`**
- 按鈕要有 `aria-expanded="true/false"` 和 `aria-controls="選單的id"`
- 按鈕要有文字說明（例如用 `.visually-hidden` 包住「開啟選單」）
- 導覽區要用 `<nav aria-label="主要導覽">`

### 實作步驟

**Step 1**：建立 `src/components/layout/AppHeader.vue`，先寫 HTML 結構，**完全不寫 CSS**，確認結構語意正確。

**Step 2**：在 `<script setup lang="ts">` 宣告一個 `ref` 控制選單開關，寫 `toggleMenu`、`closeMenu` 兩個函式。

**Step 3**：導覽項目不要寫死在 template 裡，在 script 裡用一個陣列定義，再用 `v-for` 渲染：

```ts
interface NavItem {
  label: string
  href: string   // 例如 '#about'
}
const navItems: NavItem[] = [ /* TODO */ ]
```

**Step 4**：寫手機版 CSS（Mobile First）：
- Header 用 Flex，`justify-content: space-between`
- 選單預設隱藏，`isOpen` 為 `true` 時加上一個 class 讓它顯示（用 `:class="{ 'is-open': isOpen }"`）

**Step 5**：加 `@media (min-width: 768px)`：隱藏漢堡按鈕，選單永遠顯示並橫向排列。

**Step 6**：處理 `Esc` 鍵。提示：在 `onMounted` 加上 `keydown` 監聽，在 `onUnmounted` **移除**它。
想想看：如果忘記移除會發生什麼事？

**Step 7**：在 `App.vue` 放上 `<AppHeader />`、`<main><RouterView /></main>`、`<AppFooter />`。

### AppFooter

自己完成：版權文字（年份用 `new Date().getFullYear()` 自動產生）+ 社群連結（從 `data/profile.ts` 讀取）。

---

## 4. 用 DevTools 測試 RWD

1. `F12` → `Ctrl + Shift + M` 開啟裝置模式
2. 上方選 **Responsive**，手動拖拉寬度，**從 320px 慢慢拉到 1920px**，觀察哪裡開始跑版
3. 至少檢查這些寬度：`320`（最小的手機）、`375`、`768`、`1024`、`1440`
4. 檢查有沒有**水平捲軸**（最常見的 RWD bug）

<details>
<summary>卡關提示：出現水平捲軸，找不到是哪個元素撐破的</summary>

在 DevTools 的 Console 貼上這段，會印出所有比視窗還寬的元素：

```js
document.querySelectorAll('*').forEach(el => {
  if (el.offsetWidth > document.documentElement.clientWidth) console.log(el)
})
```

常見兇手：寫死 `width` 的圖片、長網址、`100vw`（它包含捲軸寬度）、負的 margin。

</details>

---

## 驗收

- [ ] 三份 CSS 都已建立並引入，`reset.css` 每一行我都懂
- [ ] Header 在 320px ～ 1920px 都正常，沒有水平捲軸
- [ ] 漢堡選單：點擊開關、點連結關閉、Esc 關閉都正常
- [ ] 用鍵盤 `Tab` 可以走遍所有導覽連結，而且看得到 focus 框
- [ ] DevTools Elements 面板裡，按鈕的 `aria-expanded` 會跟著切換
- [ ] 已 commit：`feat: add global styles and responsive header`

<details>
<summary>卡關提示：Step 2 的 ref 怎麼寫</summary>

```ts
import { ref } from 'vue'

const isOpen = ref(false)   // TS 會自動推導成 Ref<boolean>
const toggleMenu = () => { isOpen.value = !isOpen.value }
```

在 template 裡不用寫 `.value`：`:aria-expanded="isOpen"`。

</details>

<details>
<summary>卡關提示：Step 6 的 Esc 鍵</summary>

```ts
import { onMounted, onUnmounted } from 'vue'

const onKeydown = (e: KeyboardEvent) => {
  if (e.key === 'Escape') closeMenu()
}
onMounted(() => window.addEventListener('keydown', onKeydown))
onUnmounted(() => window.removeEventListener('keydown', onKeydown))
```

忘記移除的話，元件被銷毀後監聽器還在（記憶體洩漏），重新掛載時還會重複綁定。
進階：之後可以改用 [VueUse](https://vueuse.org/) 的 `onKeyStroke`，但現在先自己寫一次。

</details>

---

## 選讀：要不要用 SCSS？

原生 CSS 已經有變數和巢狀語法（現代瀏覽器都支援 CSS Nesting），這個專案**不一定需要 SCSS**。
如果想練 SCSS 的 mixin 來管理斷點：

```powershell
pnpm add -D sass-embedded
```

```scss
// src/assets/styles/_breakpoints.scss
@mixin md { @media (min-width: 768px) { @content; } }
@mixin lg { @media (min-width: 1024px) { @content; } }
```

然後在 `vite.config.ts` 的 `css.preprocessorOptions.scss.additionalData` 自動引入，細節請查 Vite 文件的 CSS 章節。
