# 05. 元件開發

## 目標

- 熟練 `defineProps`、`defineEmits`、`<slot>` 搭配 TypeScript
- 用 `computed` 做作品篩選
- 寫出第一個 composable
- 完成首頁五個區塊

> 本章只給**規格、提示與自我檢查**，程式碼請自己寫。先求能動，再求漂亮。

---

## 0. Vue 3 + TS 語法速查

先把這些語法練熟，後面每個元件都會用到。

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'
import type { Project } from '@/types'

// ① Props：用泛型定義型別
const props = defineProps<{
  project: Project
  size?: 'sm' | 'md'          // 可選
}>()

// ② Props 預設值（Vue 3.5+ 可以直接解構並給預設值）
// const { size = 'md' } = defineProps<{ size?: 'sm' | 'md' }>()

// ③ Emits：定義事件名稱與參數型別
const emit = defineEmits<{
  select: [slug: string]       // 事件 select，帶一個 string 參數
}>()

// ④ 響應式狀態
const count = ref(0)
const double = computed(() => count.value * 2)
</script>
```

| 想傳遞的東西 | 用什麼 |
| --- | --- |
| 父 → 子：資料 | `props` |
| 子 → 父：通知某件事發生了 | `emit` |
| 父 → 子：一段 HTML 結構 | `<slot>` |
| 雙向綁定 | `defineModel()`（Vue 3.4+） |

> 官方文件：[元件基礎](https://cn.vuejs.org/guide/essentials/component-basics.html)、[TS 與組合式 API](https://cn.vuejs.org/guide/typescript/composition-api.html)

---

## 1. 基礎 UI 元件（先做這些）

### 1-1. `ui/SectionTitle.vue`

| 規格 | |
| --- | --- |
| 用途 | 每個區塊的標題 |
| Props | `subtitle?: string` |
| Slot | 預設 slot 放標題文字 |
| 使用方式 | `<SectionTitle subtitle="我做過的東西">Projects</SectionTitle>` |

練習重點：**slot**。

### 1-2. `ui/BaseButton.vue`

| 規格 | |
| --- | --- |
| Props | `variant?: 'primary' \| 'outline'`（預設 `primary`）、`href?: string` |
| 行為 | 有 `href` 時渲染成 `<a>`，沒有時渲染成 `<button>` |
| Slot | 按鈕文字 |

練習重點：**動態元件** `<component :is="...">`，以及 props 預設值。

自我檢查：

- [ ] `href` 是外部網址時，有自動加上 `target="_blank"` 和 `rel="noopener noreferrer"` 嗎？（查一下為什麼要加 `noopener`）
- [ ] hover 和 `:focus-visible` 都有樣式嗎？

### 1-3. `ui/TagBadge.vue`

| 規格 | |
| --- | --- |
| Props | `label: string`、`active?: boolean` |
| Emits | `click`（篩選按鈕會用到） |

想一想：這個元件應該用 `<span>` 還是 `<button>`？如果「有時候可以點、有時候不行」，要怎麼設計？

---

## 2. 首頁區塊

在 `views/HomeView.vue` 依序放入五個區塊，每個區塊的根元素都要有對應的 `id`（`about`、`projects`……），讓 Header 的錨點連結可以跳過去。

### 2-1. `sections/HeroSection.vue`

| 規格 | |
| --- | --- |
| 資料來源 | `data/profile.ts` |
| 內容 | 打招呼、名字、職稱、一句話介紹、兩個 `BaseButton`（看作品 → `#projects`、下載履歷 → PDF） |
| RWD | 手機置中、單欄；`lg` 以上左文右圖（用 Grid） |
| 字級 | 名字用 `--fs-hero`（clamp） |

> 頁面上應該只有**一個** `<h1>`，就放在這裡。

### 2-2. `sections/AboutSection.vue`

| 規格 | |
| --- | --- |
| 內容 | 頭像、自我介紹、技能標籤列（`TagBadge`） |
| RWD | 手機：圖上文下；`md` 以上：圖左文右（`grid-template-columns: 200px 1fr` 之類） |
| 圖片 | 用 `import` 引入 `assets/images/` 裡的頭像，`alt` 不能空白 |

練習重點：圖片 RWD。頭像想做成圓形的話，查一下 `aspect-ratio` 和 `object-fit: cover`。

### 2-3. `sections/ProjectsSection.vue` + `ProjectCard.vue`（本章重點）

**ProjectCard 規格**

| 規格 | |
| --- | --- |
| Props | `project: Project` |
| 內容 | 封面圖、標題、簡介、技術標籤、Demo / GitHub 連結（**有才顯示**，用 `v-if`） |
| 樣式 | hover 時卡片稍微上浮（`transform` + `box-shadow` + `transition`） |
| 圖片 | 固定比例（`aspect-ratio: 16 / 9`），圖片加 `loading="lazy"` |

**ProjectsSection 規格**

| 規格 | |
| --- | --- |
| 篩選列 | 從所有作品的 `tags` **自動收集**不重複的標籤，最前面加一個「All」 |
| 篩選邏輯 | 點擊標籤後只顯示有這個標籤的作品 |
| 列表 | `v-for` 渲染 `ProjectCard`，**記得加 `:key`**（用什麼當 key 最好？） |
| RWD | 卡片列表用 Grid（第 04 章的 `auto-fit` 或 media query） |
| 手機篩選列 | 標籤太多時，橫向捲動而不是換行（`overflow-x: auto` + `flex-wrap: nowrap`） |

**實作步驟**

1. 先不做篩選，直接把全部作品渲染出來
2. 用 `computed` 算出 `allTags: string[]`
3. 用 `ref` 存目前選到的標籤 `activeTag`
4. 用 `computed` 算出 `filteredProjects`
5. 把篩選邏輯抽成一個**純函式** `filterProjects(projects, tag)`，放到 `src/utils/filterProjects.ts`。第 07 章會幫它寫單元測試

<details>
<summary>卡關提示：收集不重複的標籤</summary>

```ts
const allTags = computed(() => {
  const tags = projects.flatMap(p => p.tags)   // 把二維陣列攤平
  return ['All', ...new Set(tags)]             // Set 會自動去除重複
})
```

</details>

<details>
<summary>卡關提示：切換篩選時想要動畫</summary>

用 Vue 內建的 `<TransitionGroup>` 包住 `v-for`，再搭配 `.xxx-move`、`.xxx-enter-from` 等 class。
官方文件：[TransitionGroup](https://cn.vuejs.org/guide/built-ins/transition-group.html)

</details>

### 2-4. `sections/ResumeSection.vue`

| 規格 | |
| --- | --- |
| 資料來源 | `data/experiences.ts` |
| 內容 | 時間軸：每筆有圓點、日期、名稱、職稱、說明條列 |
| 排序 | 新的在上面（用 `computed` 排序，**不要直接改原始陣列**，想想為什麼） |
| 日期顯示 | `'2024-03'` 顯示成 `2024.03`；沒有結束日期就顯示「至今」 |
| 下載 | 「下載 PDF 履歷」按鈕 |
| RWD | 手機：時間軸線在左側；`lg` 以上：可以挑戰左右交錯（選做，用 `:nth-child(even)`） |

**時間軸 CSS 提示**：

- 垂直線：外層容器用 `::before` 畫一條 `width: 2px` 的線
- 圓點：每一筆用 `::before` 畫一個圓，`position: absolute` 對齊到線上

可以把「日期格式化」抽成一個函式 `formatPeriod(start, end?)`，第 07 章一起測試。

### 2-5. `sections/ContactSection.vue`

| 規格 | |
| --- | --- |
| 內容 | 一段邀請聯絡的文字、Email 按鈕（`mailto:`）、社群連結列表 |
| Icon | 選一種：① 自己放 SVG 檔 ② 安裝 icon 套件（例如 `pnpm add @iconify/vue`） |
| a11y | 只有 icon 沒有文字的連結，一定要加 `aria-label` |

---

## 3. 第一個 composable：`useScrollSpy`

**需求**：捲動頁面時，Header 導覽列上「目前所在區塊」的連結要高亮。

**觀念**：composable 就是「把有狀態的邏輯抽成一個 `use` 開頭的函式」，可以在多個元件之間重用。

**規格**：

```ts
// src/composables/useScrollSpy.ts
export function useScrollSpy(sectionIds: string[]) {
  // 回傳目前可見的區塊 id
  // return { activeId }   // activeId: Ref<string>
}
```

**實作提示**：

1. 用 `IntersectionObserver` 觀察每個 section（**不要**用 `scroll` 事件去算位置，效能較差）
2. 在 `onMounted` 開始觀察，在 `onUnmounted` 呼叫 `observer.disconnect()`
3. 調整 `rootMargin`（例如 `'-50% 0px -50% 0px'`），讓「畫面正中央」碰到的區塊算作 active
4. 在 `AppHeader` 使用：`const { activeId } = useScrollSpy(['about', 'projects', ...])`，再用 `:class` 加上 active 樣式

> MDN：[Intersection Observer API](https://developer.mozilla.org/zh-TW/docs/Web/API/Intersection_Observer_API)

---

## 驗收

- [ ] 五個區塊都完成，內容都來自 `data/`，**template 裡沒有寫死的個人資料**
- [ ] 所有元件的 props 都有 TS 型別，`pnpm type-check` 通過
- [ ] 作品篩選正常，「All」可以顯示全部
- [ ] 320px ～ 1920px 都沒有跑版
- [ ] 所有圖片都有有意義的 `alt`
- [ ] 外部連結都有 `rel="noopener noreferrer"`
- [ ] 捲動時 Header 會高亮目前區塊
- [ ] 每完成一個區塊就 commit 一次（應該會有 5 個以上的 `feat:` commit）

<details>
<summary>卡關提示：子元件可以直接改 props 嗎？</summary>

不行。props 是**單向資料流**，子元件不能修改。如果子元件需要改變資料，要 `emit` 事件讓父元件去改，
或使用 `defineModel()`。試著在子元件寫 `props.project.title = 'x'`，看看 ESLint 和 Vue 會怎麼警告你。

</details>
