---
name: fpga-vivado-workflow
description: Use when adding or modifying RTL in a Xilinx/AMD Vivado+Vitis FPGA project - guides the full design-to-board flow (plan, simulate, synthesize, board-verify) with mandatory gates; invoke before touching src/*.v
---

# FPGA Vivado+Vitis 開發工作流程

## 階段 0：先規劃再動手（強制）
動任何 RTL 前，先用 superpowers `brainstorming` 設計、`writing-plans` 出計畫。有 spec 與 plan 才寫 code。

## 前置檢查
若專案無 agent 指令檔（`AGENTS.md` / `CLAUDE.md`）→ 先 invoke `init-fpga-agents` 建檔，再回到本流程。路徑請查該專案指令檔的路徑對照（IP 名稱、板子用該專案實際值）。

## 階段一：Branch + RTL + 門1
1. 確認不在 main，開 branch：`git checkout -b feat/<名稱>` 或 `fix/<名稱>`
2. 只改 `src/*.v`
3. cp 到 Vivado 兩目錄（ipdefs/hdl + ipshared 快取；路徑見指令檔）
4. **[門1] iverilog 模擬必須 pass**：`iverilog -o sim/sim_out sim/<tb>.v src/<module>.v && vvp sim/sim_out`
5. commit（`sim:` / `feat:` / `fix:`）

## 階段二：Vivado + 門2 + Vitis + 門3
1. IP 右鍵 → Reset Output Products → Generate Output Products
2. Generate Bitstream
3. **[門2] Timing WNS ≥ 0**（負值先修；MIPI D-PHY domain 例外見 fpga-pitfalls）
4. **回填結果模板**：資源使用量 / 時序 WNS / 功耗
5. Export Hardware (Include Bitstream) → 覆蓋 XSA
6. Vitis → Update Hardware Specification → Build Platform → Build App → Run
7. **[門3] 板上功能驗證通過**
8. **回填結果模板**：功能驗證結果
9. 告知使用者「板上驗證通過，請確認是否可以合併」→ 停下等回覆

## 階段三：收尾（等使用者明說「可以合併」才執行）
1. `git checkout main && git merge --squash <branch> && git commit -m "feat: ..."`
2. `git branch -d <branch> && git push`
3. **回填結果模板**：結論
4. 更新筆記（TODO / 開發紀錄）

## 遇錯處理
- 階段二任何 Vivado/Vitis 錯誤 → invoke `fpga-pitfalls` 查解法
- **查無此坑且自行解決** → 用 fpga-pitfalls 條目模板草擬新條目，依其「新增條目規範」回寫（PR / 回報維護者）

## 不可違反
- 不得在板上驗證後自動合併；合併三動作（squash/刪 branch/push）須等使用者明確同意
- 不得以「小改動」跳過門1 模擬
