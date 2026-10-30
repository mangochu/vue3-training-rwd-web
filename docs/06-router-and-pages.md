# 06. 路由與多頁面

## 目標

- 新增作品詳細頁 `/projects/:slug` 和 404 頁
- 理解 `createWebHistory` 與 `createWebHashHistory` 的差別，以及它們和 GitHub Pages 的關係
- 處理錨點捲動、頁面標題、路由懶載入

---

## 1. 路由模式：先做決定

| | `createWebHistory` | `createWebHashHistory` |
| --- | --- | --- |
| 網址樣子 | `/projects/weather-app` | `/#/projects/weather-app` |
| 好看程度 | 好看 | 有 `#`，比較不好看 |
| 在 GitHub Pages 上重新整理 | ❌ 會 404，**需要額外處理**（第 08 章） | ✅ 不會有問題 |
| 和頁內錨點 `#about` 的相容性 | ✅ 正常 | ⚠️ **會衝突**，因為 `#` 已經被路由用掉了 |

**本專案建議用 `createWebHistory`**，因為首頁用了很多 `#about` 這種錨點。
GitHub Pages 的 404 問題在第 08 章用 `404.html` 解決。

確認 `src/router/index.ts` 是這樣寫的：

```ts
const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [ /* ... */ ],
})
```

> `import.meta.env.BASE_URL` 會讀取 `vite.config.ts` 的 `base` 設定。部署到 `/<repo>/` 子路徑時，路由才會正確。

---

## 2. 定義路由

### 任務：完成這三條路由

| path | name | 元件 | 備註 |
| --- | --- | --- | --- |
| `/` | `home` | `HomeView` | 直接 import |
| `/projects/:slug` | `project-detail` | `ProjectDetailView` | **懶載入** |
| `/:pathMatch(.*)*` | `not-found` | `NotFoundView` | 懶載入，要放在**最後** |

懶載入的寫法：

```ts
component: () => import('@/views/ProjectDetailView.vue')
```

> **觀察**：執行 `pnpm build` 後打開 `dist/assets/`，會發現 `ProjectDetailView` 被拆成獨立的 js 檔，
> 只有使用者進入那個頁面時才會下載。

---

## 3. 作品詳細頁 `ProjectDetailView.vue`

### 規格

1. 從網址取得 `slug`，到 `projects` 資料裡找對應的作品
2. 找不到時導向 404 頁（或顯示「找不到這個作品」）
3. 顯示：大圖、標題、完整說明、技術標籤、Demo / GitHub 按鈕、「← 回作品列表」連結
4. 首頁的 `ProjectCard` 加一個連結 `<RouterLink :to="...">` 進到詳細頁

### 取得路由參數的兩種方式

**方式 A：`useRoute()`**

```ts
import { useRoute } from 'vue-router'
const route = useRoute()
// route.params.slug 的型別是 string | string[]，要自己處理
```

**方式 B：`props: true`（建議）**

在路由設定加上 `props: true`，`slug` 就會變成元件的 prop：

```ts
{ path: '/projects/:slug', component: ..., props: true }
```

```ts
// ProjectDetailView.vue
const props = defineProps<{ slug: string }>()
```

> 方式 B 讓元件不依賴路由，比較好測試，也是比較好的設計。**請用方式 B 實作**，再想想為什麼。

### 思考題

在 A 作品的詳細頁上，如果有一個「下一個作品」連結到 B 作品，點擊後畫面**會更新嗎**？
提示：同一個路由元件只改參數時，Vue Router 會**重用**元件實例，`onMounted` 不會再執行一次。
你的寫法如果用 `computed` 依賴 `props.slug`，就沒問題；如果只在 `onMounted` 裡查一次資料，就會出錯。

---

## 4. 從其他頁面回到首頁錨點

在作品詳細頁點 Header 的「About」時，要能回到首頁並捲到 `#about`。

### 步驟

1. Header 的連結改成 `<RouterLink :to="{ name: 'home', hash: '#about' }">`
2. 在 `createRouter` 加上 `scrollBehavior`：

```ts
scrollBehavior(to, from, savedPosition) {
  // TODO：
  // 1. 瀏覽器上一頁 / 下一頁 → 回到 savedPosition
  // 2. 有 to.hash → 捲動到那個元素（記得扣掉 Header 高度，可以用 top 參數）
  // 3. 其他 → 回到頁面最上方 { top: 0 }
}
```

> 官方文件：[滾動行為](https://router.vuejs.org/zh/guide/advanced/scroll-behavior.html)

---

## 5. 動態頁面標題

每個頁面的 `<title>` 應該不一樣（對 SEO 和瀏覽器分頁都有幫助）。

1. 在路由加上 `meta: { title: 'Projects' }`
2. 在 `router.afterEach` 裡設定 `document.title`
3. 詳細頁的標題要包含作品名稱，想想看要在哪裡設定

**TS 加分題**：讓 `meta.title` 有型別提示。建立 `src/router/types.d.ts`：

```ts
import 'vue-router'

declare module 'vue-router' {
  interface RouteMeta {
    title?: string
  }
}
```

這叫做**模組擴充（Module Augmentation）**，是 TS 很常用的技巧。

---

## 6. 404 頁 `NotFoundView.vue`

自由發揮：一個大大的 404、一句幽默的話、一個回首頁的按鈕。

---

## 驗收

- [ ] 從首頁卡片可以進入詳細頁，網址是 `/projects/<slug>`
- [ ] 亂打網址（例如 `/abc`）會顯示 404 頁
- [ ] 打不存在的作品 slug（例如 `/projects/xxx`）會顯示 404 或「找不到作品」
- [ ] 在詳細頁點 Header 的 About，會回到首頁並捲到 About 區塊，標題沒有被 Header 擋住
- [ ] 瀏覽器「上一頁」會回到原本的捲動位置
- [ ] 每個頁面的分頁標題都不一樣
- [ ] `pnpm build` 後確認 `dist/assets/` 有被拆出來的 js 檔
- [ ] 已 commit：`feat: add project detail and not found pages`

<details>
<summary>卡關提示：scrollBehavior 的 hash 捲動</summary>

```ts
if (to.hash) {
  return { el: to.hash, top: 80, behavior: 'smooth' }  // top 就是要預留的偏移量
}
```

從其他頁面跳回首頁時，元素可能還沒渲染出來，可以回傳一個 Promise，延遲一下再捲動。

</details>
