---
name: init-fpga-claudemd
description: Use when starting a new FPGA hardware project and need to create CLAUDE.md plus a development result-record template - for Vivado/Verilog/Vitis projects on Xilinx/AMD boards; invoke before writing any code
---

# Init Project CLAUDE.md

## Overview

CLAUDE.md 約束 agent 行為並記錄專案工作流程。**「強制規則」section 是最關鍵的部分** — 沒有它，agent 會直接 commit 到 main、跳過模擬、忽略時序問題、未經確認就合併。

## Step 1：進入 Plan Mode 收集路徑

使用 `EnterPlanMode` 工具進入規劃模式，**依序**詢問以下資訊（一次一個，等使用者回答再問下一個）：

**第一輪：四個共用路徑**

1. **開發根目錄**（WSL，source of truth）
   - 範例：`/home/wsl_user/workspace/projects/<project>/`
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

收集完成後，展示路徑摘要供使用者確認，然後使用 `ExitPlanMode` 離開規劃模式，再執行 Step 3 生成 CLAUDE.md。

## Step 2：識別專案類型

- **FPGA**：有 Vivado/Vitis、Verilog、硬體 IP
- **軟體**：有 build system（cmake/make/npm）、無硬體工具
- **混合**：兩者皆有

## Step 3：生成 CLAUDE.md

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

### Section 3：目錄結構

```
FPGA:     src/, sim/, constraints/, hdl/, docs/
軟體:     src/, tests/, docs/, build/
```

### Section 4：路徑對照（FPGA 必填）

| 用途 | 路徑 |
|------|------|
| WSL 源碼（source of truth） | 填入 |
| Vivado 專案 | 填入 |
| Vivado IP 源碼（ipdefs） | 填入 |
| Vivado ipshared 快取 | 填入 ← 絕對不可省略 |
| XSA 輸出 | 填入 |
| Vitis 工作區 | 填入 |
| Obsidian 專案筆記 | 填入 |
| 開發結果紀錄 | docs/result-record.md（預設） |

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

### Section 8：建立開發結果紀錄模板
建檔同時，把本 skill 的 `templates/result-record.md` 複製到專案的結果紀錄路徑（預設 `docs/result-record.md`），並把 `<專案名>` 換成實際專案名。後續開發由 fpga-vivado-workflow 回填。

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
