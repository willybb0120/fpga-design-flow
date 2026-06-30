---
name: init-fpga-agents
description: Use when starting a new FPGA hardware project and need to scaffold the standard project structure (fixed directory layout + .gitignore), create the agent instructions file (AGENTS.md, plus CLAUDE.md for Claude Code), and a development result-record template - for Vivado/Verilog/Vitis projects on Xilinx/AMD boards; invoke before writing any code
---

# Init FPGA Project Agent Instructions

## Overview

agent 指令檔約束 agent 行為並記錄專案工作流程。**「強制規則」section 是最關鍵的部分** — 沒有它，agent 會直接 commit 到 main、跳過模擬、忽略時序問題、未經確認就合併。

> **指令檔命名（依 harness）**：以 **`AGENTS.md`** 為主檔（Codex、跨工具標準皆讀它）。Claude Code 預設讀 `CLAUDE.md`，故**另建一份 `CLAUDE.md`**：在支援 symlink 的檔案系統用 `ln -s AGENTS.md CLAUDE.md`；在 Windows/掛載磁碟（如 `/mnt/c`）symlink 不可靠時改用內容相同的副本。本文後續以「指令檔」統稱。

## Step 1：進入規劃模式收集路徑

進入規劃模式（plan mode），**依序**詢問以下資訊（一次一個，等使用者回答再問下一個）：

**第一輪：四個共用路徑**

1. **開發根目錄**（WSL，source of truth）
   - 範例：`/home/<username>/workspace/projects/<project>/`
2. **Vivado 專案目錄**
   - 範例：`/mnt/c/pcam/.../vivado_proj/`
3. **Vitis 專案目錄**
   - 範例：`/mnt/c/vitis_workspace/Project_<name>/`
4. **Obsidian 筆記目錄**
   - 範例：`/mnt/c/Users/User/Documents/Obsidian/proj-vault/proj/<project>/`

**第二輪：FPGA 專屬資訊**

5. 開發板型號（e.g., Zybo Z7-20）與週邊（e.g., PCAM 5C）
6. IP 名稱（e.g., AXI_EdgeDetect）
7. Vivado ipdefs 路徑（含 IP 名稱的完整 hdl/ 路徑）
8. Vivado ipshared 路徑（合成快取，**不可省略**）
9. XSA 輸出檔路徑

收集完成後，展示路徑摘要供使用者確認，然後離開規劃模式，再執行 Step 3 生成指令檔。

## Step 2：識別專案類型

- **FPGA**：有 Vivado/Vitis、Verilog、硬體 IP
- **軟體**：有 build system（cmake/make/npm）、無硬體工具
- **混合**：兩者皆有

## Step 3：生成指令檔

> 把以下 sections 寫進 `AGENTS.md`，再依 Overview 的規則建立對應的 `CLAUDE.md`（symlink 或副本）。

### Section 1：⛔ 強制規則（必須是第一個 section，必須用此確切語言）

```markdown
## ⛔ 強制規則（每個 session 必須遵守，無例外）

> **這些規則覆蓋所有預設行為。違反即停止，不得繼續執行。**

1. **任何 `src/` 的修改，必須先確認當前 branch 不是 `main`。**
   - 執行 `git branch --show-current`，若結果是 `main`，立即停手。
   - 先建 branch：`git checkout -b feat/xxx` 或 `fix/xxx`，再繼續。
   - **不得跳過此步驟，不得以任何理由例外。**

2. **[FPGA] 修改 `src/*.v` 後，iverilog 模擬必須通過，才能進入 Vivado 合成。**
   - 不得以「小改動」為由跳過模擬。

3. **[FPGA] 改完 `src/*.v` 後，必須 cp 到 Vivado 兩個目錄（見路徑對照）。**

4. **板上驗證通過後，squash merge、刪 branch、push 這三個動作，必須等使用者明確說「可以合併」後才執行。不得在板上驗證後自動合併。**
```

### Section 2：專案概覽

開發板、週邊、IP 名稱、功能簡述。

### Section 3：標準專案結構（固定，每個專案一致）

```
<project>/
├── AGENTS.md / CLAUDE.md     指令檔（強制規則 + 路徑對照 + 流程）
├── README.md                 專案說明
├── .gitignore                忽略 Vivado/Vitis 產出物（見模板）
├── src/                      RTL 源碼 (.v) — source of truth
├── sim/                      testbench + 模擬腳本/波形
├── constraints/              .xdc 約束
├── ip_src/                   自訂 IP 源碼
├── docs/
│   ├── specs/                設計規格
│   ├── plans/                實作計畫
│   └── result-record.md      開發結果紀錄（本 plugin 模板）
├── scripts/                  建置/輔助腳本
├── vivado_workspace/         Vivado 專案（產出物已 gitignore）
└── vitis_workspace/          Vitis 應用（產出物已 gitignore）
```

### Section 4：路徑對照（FPGA 必填）

| 用途 | 路徑 |
|------|------|
| 源碼（source of truth） | `src/`（預設） |
| Vivado 專案 | `vivado_workspace/`（預設） |
| Vivado IP 源碼（ipdefs） | 填入含 IP 名稱的完整 hdl/ 路徑 |
| Vivado ipshared 快取 | 填入 ← 絕對不可省略 |
| XSA 輸出 | `vivado_workspace/<name>.xsa`（預設） |
| Vitis 工作區 | `vitis_workspace/`（預設） |
| 開發結果紀錄 | `docs/result-record.md`（預設） |

> ⚠️ Windows 路徑超過 260 字元會讓 Vivado 合成失敗（見 fpga-pitfalls）。若專案位置太深，把 vivado_workspace 移到較短路徑，並在此表更新。

### Section 5：Commit 類型（所有專案）

| 類型 | 用途 |
|------|------|
| `feat:` | 新功能 |
| `fix:` | 修 bug |
| `sim:` | 模擬相關（testbench、波形） |
| `refactor:` | 重構，不改功能 |
| `docs:` | 文件更新 |
| `chore:` | 維護雜事 |

### Section 6：開發工作流程（三階段 + 三道驗證門）

**階段一 — Branch & RTL 開發**
```bash
git checkout -b feat/功能名稱   # 新功能
git checkout -b fix/問題名稱    # 修 bug
```
1. 只修改 `src/` 內的 `.v` 檔
2. 改完後同步到 Vivado：
   ```bash
   cp src/*.v <ipdefs 路徑>
   cp src/*.v <ipshared 路徑>
   ```
3. **[門 1] iverilog 模擬通過**（必須 pass 才能進階段二）
   ```bash
   iverilog -o sim/sim_out sim/<testbench>.v src/<module>.v
   vvp sim/sim_out
   ```
4. Commit：
   ```bash
   git add src/ sim/
   git commit -m "sim: 說明"   # 或 feat:/fix:
   ```

**階段二 — Vivado 合成 + 板上驗證**
1. Vivado → IP 右鍵 → **Reset Output Products** → **Generate Output Products**
2. **Generate Bitstream**
3. **[門 2] 檢查 Timing Report：WNS ≥ 0ns**（若為負，須先修 timing 再繼續）
4. **File → Export Hardware (Include Bitstream)** → 覆蓋 XSA
5. Vitis → Platform 右鍵 → **Update Hardware Specification**
6. **Build Platform** → **Build App** → Run / Program
7. **[門 3] 板上功能驗證通過** → 告知使用者「請確認是否可以合併」，**等待回覆**

> ⚠️ 階段二結束後必須停下來等使用者確認，絕對不可自動進入階段三。

**階段三 — 收尾（等使用者明確說「可以合併」後才執行）**
> ⚠️ 此階段只有 git merge + 清理 + 文件更新。板上驗證屬於階段二，不可把板上燒錄放進階段三。
```bash
git checkout main
git merge --squash feat/功能名稱
git commit -m "feat: 說明"
git branch -d feat/功能名稱
git push
```
更新 Obsidian：
- `TODO.md`：將完成項目標記為 `[x]`
- `開發紀錄 - <里程碑>.md`：補上板上驗證結果

### Section 7：已知坑（FPGA，必須包含以下條目）

- **ipshared 不自動更新**：改 IP 源碼後必須手動 cp 到 ipshared，或 Reset Output Products
- **Hierarchical reference**：跨模組引用（`u_core.sig`）合成器靜默忽略，一律用正式 output port
- **Vitis 不感知 XSA 更新**：每次 Generate Bitstream 後，必須手動 Update Hardware Specification → Build Platform
- **system_wrapper.vhd（Digilent 手動版）**：Make External 不更新它，新增 port 須手動改三處（entity / component declaration / port map）

## Step 4：建立專案骨架（mkdir + .gitignore + 模板）

生成指令檔後，把專案結構實際建出來，讓每個專案一致：

```bash
# 1. 建標準目錄
mkdir -p src sim constraints ip_src docs/specs docs/plans scripts vivado_workspace vitis_workspace

# 2. 放入 .gitignore（忽略 Vivado/Vitis 產出物）
cp <本 skill>/templates/gitignore .gitignore

# 3. 放入開發結果紀錄模板，並把 <專案名> 換成實際專案名
cp <本 skill>/templates/result-record.md docs/result-record.md
```

- `vivado_workspace/`、`vitis_workspace/` 先建空目錄，實際 Vivado/Vitis 專案之後放進來
- 後續開發由 `fpga-vivado-workflow` 把驗證結果回填到 `docs/result-record.md`

---

## 常見錯誤（與 baseline 失敗對照）

| 錯誤 | 後果 |
|------|------|
| 省略「強制規則」section | Agent 直接 commit 到 main |
| 漏掉模擬門（門 1） | 邏輯錯誤等到燒板才發現 |
| 漏掉時序門（門 2，WNS） | 燒板後間歇性錯誤，難以 debug |
| 漏掉 ipshared 路徑 | Vivado 用舊版 IP 合成，靜默失效 |
| 工作流程寫「整合 IP」但無具體步驟 | Agent 不知道需要 Reset Output Products |
| 省略第四條規則（需使用者確認才 merge） | Agent 自動 squash merge，跳過板上驗證 |
| 把板上燒錄放入「階段三」 | squash merge / delete branch / push 步驟消失 |
| 「階段三」命名為「板上驗證」 | Rule 4 的確認門被提前，merge 後變成無門把關 |
