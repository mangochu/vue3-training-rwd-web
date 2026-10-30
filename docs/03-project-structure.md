# 03. 專案結構與型別設計

## 目標

- 清掉範例程式碼，建立自己的資料夾結構
- 用 TypeScript `interface` 定義資料格式
- **資料與畫面分離**：作品、履歷寫在 `data/`，元件只負責顯示

---

## 1. 清掉範例

刪除前，先花 5 分鐘讀一下 `src/components/HelloWorld.vue` 和 `TheWelcome.vue`，
看 `defineProps` 和 `<slot>` 怎麼寫。讀完再刪：

- `src/components/` 底下所有範例元件（包含 `icons/`、`__tests__/`）
- `src/views/AboutView.vue`
- `src/assets/` 底下的 `logo.svg`、`base.css`、`main.css`

然後：

- `src/App.vue` 只留一個 `<RouterView />`
- `src/views/HomeView.vue` 只留一個 `<h1>Hello</h1>`
- `src/router/index.ts` 移除 about 路由
- `src/main.ts` 移除 `import './assets/main.css'`

執行 `pnpm dev` 確認畫面只剩 Hello，再執行 `pnpm type-check` 確認沒有錯誤。

```powershell
git add .
git commit -m "chore: remove scaffold example code"
```

---

## 2. 建立資料夾結構

目標結構如下（現在先建資料夾，檔案之後的章節再慢慢補）：

```
src/
├── assets/
│   ├── images/          # 頭像、作品截圖
│   └── styles/
│       ├── reset.css     # 第 04 章
│       ├── tokens.css    # 第 04 章：Design Tokens（CSS 變數）
│       └── global.css    # 第 04 章：全域樣式、工具 class
├── components/
│   ├── layout/          # 版面元件：AppHeader、AppFooter
│   ├── sections/        # 首頁區塊：HeroSection、AboutSection……
│   └── ui/              # 可重複使用的小元件：BaseButton、TagBadge、SectionTitle
├── composables/         # 可重用的邏輯：useScrollSpy、useTheme……
├── data/                # 網站內容：projects.ts、experiences.ts、profile.ts
├── types/               # TS 型別：index.ts
├── router/
│   └── index.ts
├── views/               # 路由頁面：HomeView、ProjectDetailView、NotFoundView
├── App.vue
└── main.ts
public/
└── resume.pdf           # 不經過打包、原樣複製的檔案（履歷 PDF、favicon）
```

### 命名規則

| 種類 | 規則 | 範例 |
| --- | --- | --- |
| 元件檔 | PascalCase、**至少兩個單字**（ESLint 會檢查） | `ProjectCard.vue` ✅ `Card.vue` ❌ |
| 只出現一次的版面元件 | 加 `App` 或 `The` 前綴 | `AppHeader.vue` |
| 基礎 UI 元件 | 加 `Base` 前綴 | `BaseButton.vue` |
| composable | `use` 開頭、camelCase | `useScrollSpy.ts` |
| 路由頁面 | `View` 結尾 | `HomeView.vue` |

> 這些規則來自 [Vue 官方風格指南](https://vuejs.org/style-guide/)，有空可以讀一遍。

### `assets/` 和 `public/` 的差別

| | `src/assets/` | `public/` |
| --- | --- | --- |
| 會經過 Vite 處理嗎 | 會（壓縮、檔名加 hash） | 不會，原樣複製 |
| 怎麼引用 | `import avatar from '@/assets/images/avatar.jpg'` | 用 `import.meta.env.BASE_URL + 'resume.pdf'` |
| 適合放 | 元件裡用到的圖片 | PDF、favicon、`robots.txt` |

> ⚠️ 引用 `public/` 的檔案**不要直接寫 `/resume.pdf`**，部署到 GitHub Pages 子路徑時會 404。原因在第 08 章說明。

---

## 3. 定義型別

建立 `src/types/index.ts`。以下是 `Project` 的範例，**其他型別請你自己寫**。

```ts
// src/types/index.ts

/** 作品集中的單一作品 */
export interface Project {
  /** 網址用的唯一識別，例如 'weather-app'，會用在 /projects/:slug */
  slug: string
  title: string
  /** 卡片上的一句話簡介 */
  summary: string
  /** 詳細頁的完整說明（可選） */
  description?: string
  /** 技術標籤，用來做篩選 */
  tags: string[]
  /** 封面圖片網址 */
  cover: string
  /** 線上 Demo 網址（可選） */
  demoUrl?: string
  /** GitHub repo 網址（可選） */
  repoUrl?: string
  /** 是否在首頁精選 */
  featured: boolean
}
```

### 任務：自己定義下面這些型別

1. **`Experience`**（工作或學歷經歷）
   - 需要：公司 / 學校名稱、職稱 / 科系、開始日期、結束日期（**還在職的話可能沒有**）、說明條列、類型（工作或學歷）
   - 提示：「類型」只有兩種可能，想想除了 `string` 之外，有沒有更精確的型別寫法

2. **`SocialLink`**（社群連結）
   - 需要：平台名稱、網址、icon 名稱
   - 提示：平台名稱可以限定成 `'github' | 'linkedin' | 'email' | ...`

3. **`Profile`**（個人資訊）
   - 需要：名字、職稱、一句話介紹、自我介紹段落、技能列表、頭像、社群連結（用上面的 `SocialLink`）

<details>
<summary>卡關提示：Experience 的「類型」和「結束日期」怎麼寫</summary>

```ts
type ExperienceType = 'work' | 'education'   // 聯合型別（Union Type）

export interface Experience {
  // ...
  type: ExperienceType
  endDate?: string   // 加 ? 代表可以沒有；也可以寫成 string | null，兩種寫法的差別可以查一下
}
```

日期建議存成 `'2024-03'` 這種字串，顯示時再格式化。

</details>

---

## 4. 撰寫資料檔

建立 `src/data/projects.ts`：

```ts
import type { Project } from '@/types'

export const projects: Project[] = [
  {
    slug: 'vue-portfolio',
    title: '個人作品集網站',
    summary: '使用 Vue 3 + TypeScript 打造的 RWD 個人網站',
    tags: ['Vue', 'TypeScript', 'CSS'],
    cover: 'https://placehold.co/600x400',
    repoUrl: 'https://github.com/<你的帳號>/vue3-training-rwd-web',
    featured: true,
  },
  // TODO：用你在第 00 章寫的 content-draft.md，再補至少 2 個作品
]
```

> `import type` 代表只匯入型別，打包時會被完全移除。

### 任務

1. 完成 `projects.ts`（至少 3 筆）
2. 建立 `experiences.ts`（至少 2 筆，一筆工作或實習、一筆學歷）
3. 建立 `profile.ts`
4. **故意打錯一個欄位名稱**（例如把 `title` 打成 `titel`），看 VS Code 和 `pnpm type-check` 怎麼報錯，再改回來

> 圖片還沒準備好的話，可以先用 [placehold.co](https://placehold.co) 產生佔位圖。

---

## 驗收

- [ ] 範例程式碼已清除，`pnpm type-check` 通過
- [ ] 資料夾結構建立完成
- [ ] `types/index.ts` 有 `Project`、`Experience`、`SocialLink`、`Profile`
- [ ] `data/` 底下有三個資料檔，內容是我自己的
- [ ] 我體驗過打錯欄位時 TS 怎麼報錯
- [ ] 已 commit：`feat: add types and content data`

<details>
<summary>卡關提示：import '@/types' 報錯找不到模組</summary>

- 確認檔案路徑是 `src/types/index.ts`（`index.ts` 可以省略不寫）
- 確認 `tsconfig.app.json` 裡有 `"paths": { "@/*": ["./src/*"] }`（create-vue 預設就有）
- 重新啟動 VS Code 的 TS 服務：`Ctrl + Shift + P` → `Vue: Restart Vue and TS servers`

</details>
