# Practical Notes — Source of Truth

這個 repo 是 **[Practical-Starlight](https://github.com/Beny942k6xu4/Practical-Starlight)** 網站的**內容唯一來源**。Starlight 那邊只負責渲染；任何筆記新增或修改都在這裡進行。

## 為什麼分兩個 repo

| 關注點 | 放哪 |
|---|---|
| 筆記內容（markdown） | **這裡（Practical-Notes）** |
| 站台外觀、sidebar 規則、build 工具 | Practical-Starlight |
| 推送後重新部署 | Practical-Starlight CI 自動拉取本 repo 並重 build |

好處：寫筆記時不會看到一堆 build/CI 雜訊，git log 也乾淨；Starlight 結構大改不會污染筆記歷史。

## 目錄結構

```
.
├── index.md                       # 首頁（Starlight splash hero）
├── meta/
│   └── principles.md              # 知識組織原則（5 層模型）
└── Server/                        # Domain（首字大寫表達邏輯層級）
    └── PowerShell/                # Subdomain
        └── basic-learn/           # Topic
            └── rename/            # Skill
                ├── index.md       # skill landing
                └── path.md        # Concept（一個 concept = 一頁）
```

對應的線上 URL（sync 時會自動小寫並加 `domains/` 前綴）：

| repo 路徑 | 線上 URL |
|---|---|
| `index.md` | `/` |
| `meta/principles.md` | `/meta/principles/` |
| `Server/index.md` | `/domains/server/` |
| `Server/PowerShell/basic-learn/rename/path.md` | `/domains/server/powershell/basic-learn/rename/path/` |

## 新增筆記的最小流程

1. 在對應層級建立 `.md`（看 `meta/principles.md` 與既有檔案的 frontmatter 與寫法）。
2. `git add . && git commit -m "docs(rename): add NewName concept"`
3. `git push`
4. GitHub Actions（`.github/workflows/notify-starlight.yml`）會 dispatch 給 Practical-Starlight，幾分鐘後線上會更新。

## 寫作格式

每個 concept 頁面採用：

- frontmatter：`title` / `description` / `sidebar.order`
- 「定位」一行（Domain > Subdomain > Topic > Skill > Concept）
- Summary（一句話結論）
- 參數屬性表（若適用）
- 典型用法（可複製執行）
- 與相鄰 concept 對照
- 常見錯誤
- 觀察
- Related（內部相對連結）
- External references（官方文件）

完整 template 與寫作慣例見 Practical-Starlight repo 的 `.vscode/ai/coding/Note_templates.md` 與 `.vscode/ai/coding/Writing_style.md`。

## 不需放在這 repo 的東西

- build 工具、`package.json`、`node_modules`
- Astro / Starlight 設定
- CI deploy workflow（部署是 Starlight 的工作）

這裡只放：markdown、圖片、`README.md`、`.github/workflows/notify-starlight.yml`。
