# 07. 程式品質與開發流程

## 目標

- 搞懂 lint、format、type-check、test 各自在檢查什麼
- 用 Vitest 寫純函式和元件的測試
- 建立「分支 → PR → 合併」的開發流程（就算只有自己一個人也要練）

---

## 1. 四種檢查的差別

| 指令 | 工具 | 檢查什麼 | 例子 |
| --- | --- | --- | --- |
| `pnpm lint` | ESLint | 程式碼**寫法**有沒有問題 | 宣告了沒用的變數、`v-for` 沒寫 `key` |
| `pnpm format` | Prettier | 程式碼**長相**（縮排、引號、換行） | 單引號還是雙引號 |
| `pnpm type-check` | vue-tsc | **型別**對不對 | 把 `number` 傳給 `string` 的 prop |
| `pnpm test:unit` | Vitest | **行為**對不對 | `filterProjects` 傳 `'Vue'` 有沒有只回傳 Vue 的作品 |

> **任務**：四個指令都跑一次，修掉所有錯誤。

### 設定 VS Code 存檔時自動格式化

建立 `.vscode/settings.json`（這個檔案**要 commit**，讓專案設定一致）：

```jsonc
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  }
}
```

---

## 2. 單元測試（Vitest）

### 2-1. 測試純函式

第 05 章抽出來的 `filterProjects` 很適合當第一個測試對象。
建立 `src/utils/__tests__/filterProjects.spec.ts`：

```ts
import { describe, it, expect } from 'vitest'
import { filterProjects } from '../filterProjects'
import type { Project } from '@/types'

// 測試用的假資料：只放測試需要的欄位，其他用預設值補上
const make = (slug: string, tags: string[]): Project => ({
  slug,
  title: slug,
  summary: '',
  tags,
  cover: '',
  featured: false,
})

const projects = [
  make('a', ['Vue', 'TS']),
  make('b', ['React']),
  make('c', ['Vue']),
]

describe('filterProjects', () => {
  it('傳入 All 時回傳全部作品', () => {
    expect(filterProjects(projects, 'All')).toHaveLength(3)
  })

  // TODO：自己寫下面這些測試
  // it('只回傳包含指定標籤的作品', ...)
  // it('沒有符合的標籤時回傳空陣列', ...)
  // it('不會修改原本的陣列', ...)
})
```

```powershell
pnpm test:unit            # 監看模式，改檔案會自動重跑
pnpm test:unit --run      # 只跑一次（CI 用這個）
```

> **任務**：再幫第 05 章的 `formatPeriod(start, end?)` 寫測試，至少包含「有結束日期」、「沒有結束日期（顯示至今）」兩種情況。

### 2-2. 測試元件

用 `@vue/test-utils`（create-vue 選 Vitest 時已經裝好）測試 `ProjectCard`：

```ts
import { describe, it, expect } from 'vitest'
import { mount } from '@vue/test-utils'
import ProjectCard from '../ProjectCard.vue'

describe('ProjectCard', () => {
  it('顯示作品標題', () => {
    const wrapper = mount(ProjectCard, {
      props: { project: /* TODO：準備一筆假資料 */ },
    })
    expect(wrapper.text()).toContain(/* TODO */)
  })

  // TODO：
  // it('沒有 demoUrl 時不顯示 Demo 連結', ...)
  // it('有 repoUrl 時，GitHub 連結的 href 正確', ...)
})
```

> 如果 `ProjectCard` 裡面用了 `<RouterLink>`，測試時會報錯，因為測試環境沒有路由。
> 查一下 `global.stubs` 這個選項怎麼用。

### 什麼值得測？

- ✅ 有邏輯的純函式（篩選、排序、日期格式化）
- ✅ 元件的**條件渲染**（有 / 沒有某個欄位時的顯示差異）
- ❌ 純樣式（顏色、間距），這部分用眼睛和 DevTools 檢查比較有效率
- ❌ Vue 本身的功能（例如「`v-for` 會渲染出列表」），框架已經測過了

---

## 3. Git 開發流程

從這章開始，**不要再直接 commit 到 `main`**，改用以下流程：

```
main ─────●──────────────●──────────────●───
           \            /  \            /
 feature/   ●──●──●────●    ●──●──●────●
 dark-mode                  feature/scroll-animation
```

### 步驟

```powershell
# 1. 從最新的 main 開分支
git switch main
git pull
git switch -c feature/dark-mode

# 2. 開發、commit（可以有很多個 commit）
git add .
git commit -m "feat: add theme toggle button"

# 3. 推上 GitHub
git push -u origin feature/dark-mode
```

4. 到 GitHub 頁面上點 **Compare & pull request**，建立 PR
5. PR 說明寫清楚：做了什麼、怎麼測試、有沒有截圖（RWD 的改動建議附上手機和桌機截圖）
6. 等第 08 章的 CI 檢查通過（綠勾勾）後，按 **Squash and merge**
7. 回到本機：

```powershell
git switch main
git pull
git branch -d feature/dark-mode
```

### 分支命名

| 前綴 | 用途 |
| --- | --- |
| `feature/` | 新功能 |
| `fix/` | 修 bug |
| `docs/` | 文件 |
| `chore/` | 設定、依賴 |

### 保護 main 分支（做完第 08 章再設定）

GitHub repo → **Settings** → **Rules** → **Rulesets**（或舊版的 Branches → Branch protection rules）：

- 目標分支：`main`
- 勾選 **Require a pull request before merging**
- 勾選 **Require status checks to pass**，選第 08 章的 CI job

這樣就**無法直接 push 到 main**，CI 沒過也無法合併。

---

## 4. Git Hooks（選做）

在 commit 之前自動跑 lint 和 format，避免把有問題的程式碼 commit 進去：

```powershell
pnpm add -D husky lint-staged
pnpm exec husky init
```

`husky init` 會建立 `.husky/pre-commit`，把裡面的內容改成：

```sh
pnpm exec lint-staged
```

在 `package.json` 加上：

```jsonc
{
  "lint-staged": {
    "*.{ts,vue}": ["eslint --fix", "prettier --write"],
    "*.{css,json,md}": ["prettier --write"]
  }
}
```

> lint-staged 只檢查**這次有改動的檔案**，所以速度很快。

---

## 驗收

- [ ] `pnpm lint`、`pnpm type-check`、`pnpm test:unit --run`、`pnpm build` 全部通過
- [ ] 至少有 2 個純函式測試檔、1 個元件測試檔
- [ ] 我故意改壞 `filterProjects`，確認測試會失敗，然後再改回來
- [ ] 至少用 PR 流程合併過 1 個功能
- [ ] `.vscode/settings.json` 已 commit

<details>
<summary>卡關提示：測試裡 @/types 找不到</summary>

確認 `vitest.config.ts` 是用 `mergeConfig(viteConfig, ...)` 繼承 `vite.config.ts` 的設定
（create-vue 預設就是這樣），別名 `@` 才會生效。

</details>
