# 個人作品集網站：Vue 3 + TypeScript 練習專案

> 這是一個「自己動手做」的練習專案。`docs/` 裡的每一章都是**指導手冊**，不是完成品。
> 照著步驟自己打程式碼、自己除錯，卡關時再看每章最後的「卡關提示」。

## 最終目標

- 一個 RWD 個人網站，內容有自我介紹、作品集、履歷時間軸、聯絡方式與 GitHub 連結
- 技術：**pnpm + Vite + Vue 3（Composition API / `<script setup>`）+ TypeScript + Vue Router**
- 品質：ESLint + Prettier + `vue-tsc` 型別檢查 + Vitest 單元測試
- 部署：**GitHub Actions**（CI 檢查）並自動部署到 **GitHub Pages**（CD）

## 學習路線

| 章節 | 主題 | 你會學到 | 預估時間 |
| --- | --- | --- | --- |
| [00](docs/00-overview.md) | 專案總覽 | 規劃範圍、技術選型的理由 | 0.5 天 |
| [01](docs/01-design-reference.md) | 設計參考與線框圖 | 參考網站、Wireframe、Design Tokens | 1 天 |
| [02](docs/02-env-and-scaffold.md) | 環境與專案初始化 | pnpm、create-vue、Git / GitHub | 0.5 天 |
| [03](docs/03-project-structure.md) | 專案結構與型別 | 資料夾規劃、TS interface、資料與畫面分離 | 0.5 天 |
| [04](docs/04-layout-and-rwd.md) | 全域樣式與 RWD 佈局 | Mobile First、CSS 變數、Flex / Grid、漢堡選單 | 2 天 |
| [05](docs/05-components.md) | 元件開發 | props / emits / slots、computed、composable | 3 天 |
| [06](docs/06-router-and-pages.md) | 路由與多頁面 | Vue Router、動態路由、404、錨點捲動 | 1 天 |
| [07](docs/07-quality-and-workflow.md) | 程式品質與開發流程 | Lint、型別檢查、Vitest、Git 分支與 PR | 1 天 |
| [08](docs/08-deploy-github-pages.md) | CI/CD 與 GitHub Pages | GitHub Actions、`base` 設定、SPA 404 問題 | 1 天 |
| [09](docs/09-advanced.md) | 進階挑戰（選做） | 深色模式、無障礙、Lighthouse、SEO | 不限 |

## 進度檢查表

把 `[ ]` 改成 `[x]` 來記錄自己的進度。

- [ ] 00 讀完總覽，寫好自己的「網站內容清單」
- [ ] 01 選好參考網站，畫出手機 / 平板 / 桌機三種線框圖
- [ ] 02 `pnpm dev` 能跑起來，第一個 commit 已推上 GitHub
- [ ] 03 完成資料夾結構與 `types/`、`data/`
- [ ] 04 完成全域樣式、Header（含手機漢堡選單）、Footer
- [ ] 05 完成 Hero / About / Projects / Resume / Contact 五個區塊
- [ ] 06 完成作品詳細頁與 404 頁
- [ ] 07 `pnpm lint`、`pnpm type-check`、`pnpm test:unit` 全部通過
- [ ] 08 網站上線於 `https://<你的帳號>.github.io/<repo 名稱>/`
- [ ] 09 至少完成一項進階挑戰

## 每章的閱讀方式

每一章都分成這幾塊：

1. **目標**：這章做完會得到什麼
2. **觀念**：先搞懂為什麼
3. **步驟**：照著做，指令可以直接複製，元件程式碼則要**自己寫**
4. **驗收**：用檢查清單確認自己真的完成
5. **卡關提示**：先自己試 15 分鐘，真的卡住再展開看
