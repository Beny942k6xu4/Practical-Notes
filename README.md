# Practical Notes

實作型學習筆記的 **source of truth**。所有 concept 內容只在這個 repo 維護，部署站台另由 [Practical-Starlight](https://github.com/Beny942k6xu4/Practical-Starlight) 從這裡拉內容後建置。

- 上線網址：<https://Beny942k6xu4.github.io/Practical-Starlight/>
- 規則：**內容只在本地寫、push 上來**；不要直接在 GitHub web 介面上加檔，避免本地與遠端內容歧異。

## 知識組織模型

筆記用五層邏輯定位每一份 concept：

```
Domain  >  Subdomain  >  Topic  >  Skill  >  Concept
```

- **Domain**：穩定、邊界明確的大領域（例：`Server`）。
- **Subdomain**：Domain 下的主要分區（例：`PowerShell`、`Bash`）。
- **Topic**：學習主題（例：`basic-learn`、`networking`）。
- **Skill**：能力（例：`rename`、`grep`）。
- **Concept**：能單獨成立、可獨立解釋的最小知識單元（例：`Path`、`NewName`）。

不確定屬於哪裡的素材，先放 `Topics/`（待補的 staging 區），穩定後再升級進 Domain。

## 目錄結構

```
.
├── index.md                  首頁（Starlight 會渲染成 splash hero）
├── meta/
│   └── principles.md         組織原則、長期維護準則
└── Server/                   Domain
    └── PowerShell/           Subdomain
        └── basic-learn/      Topic
            └── rename/       Skill
                ├── index.md  skill landing（含 concept 清單）
                ├── path.md   Concept（一檔一個 concept）
                └── …
```

命名規則：

- Domain / Subdomain 沿用品牌書寫（`Server`、`PowerShell`、`Bash`）。
- Topic / Skill / Concept 一律 kebab-case 小寫（`basic-learn`、`pass-thru`、`literal-path`）。
- Concept 為扁平 `.md`，不開子資料夾。

對應的 URL（由 Starlight 渲染）：

| 檔案                                            | URL                                                   |
| ----------------------------------------------- | ----------------------------------------------------- |
| `index.md`                                      | `/`                                                   |
| `meta/principles.md`                            | `/meta/principles/`                                   |
| `Server/index.md`                               | `/domains/server/`                                    |
| `Server/PowerShell/basic-learn/rename/index.md` | `/domains/server/powershell/basic-learn/rename/`      |
| `Server/PowerShell/basic-learn/rename/path.md`  | `/domains/server/powershell/basic-learn/rename/path/` |

## Concept 頁面結構

每一份 concept 至少含：

- frontmatter：`title` / `description` / `sidebar.order`
- 定位（五層階層）
- Summary（一句話結論）
- 參數屬性或核心要點（表格）
- 典型用法（可複製貼上執行）
- 與相鄰 concept 的對照
- 常見錯誤
- 觀察（這份筆記獨有的論點層）
- Related（同層或相鄰連結）
- External references（只放官方來源）

範例可參考 `Server/PowerShell/basic-learn/rename/path.md`。

## 新增內容流程

```powershell
Set-Location 'C:\project\Practical-Notes'

# 1. 編輯／新增 .md
# 2. 看一下變更
git status
git diff

# 3. commit。訊息格式：<type>(<scope>): <summary>
#    type:  docs | refactor | chore
#    scope: 最深層的 skill 或 topic 名（例：rename、basic-learn）
git add .
git commit -m "docs(rename): add NewName concept"

# 4. push
git push
```

commit 訊息只寫「內容做了什麼」，不寫所用工具。

push 後會由 `.github/workflows/notify-starlight.yml` 透過 `repository_dispatch` 通知 Practical-Starlight 重新建置；2–3 分鐘後上線。

## 驗證部署

1. <https://github.com/Beny942k6xu4/Practical-Notes/actions> — `Notify Practical-Starlight to rebuild` 應為成功。
2. <https://github.com/Beny942k6xu4/Practical-Starlight/actions> — 1 分鐘內應出現 `repository_dispatch` 觸發的 `Deploy to GitHub Pages`。
3. <https://Beny942k6xu4.github.io/Practical-Starlight/> — Ctrl + Shift + R 強制重新整理確認新內容已上線。

## 規則底線

- **不**直接在 GitHub web 介面編輯內容。
- **不** `git push --force`（除非明確需要重寫歷史並有共識）。
- **不**把編輯器設定（`.obsidian/`、`.vscode/` 個人偏好）commit 進來。
- 一個 PR / commit 不混雜對 Practical-Starlight repo 的修改 — 是分開的 repo。

## 相關專案

- [Practical-Starlight](https://github.com/Beny942k6xu4/Practical-Starlight) — Astro + Starlight 部署站台
