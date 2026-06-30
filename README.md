# fpga-design-flow

FPGA 電路設計 workflow + 踩坑知識庫的 Claude Code plugin（Xilinx/AMD Vivado+Vitis）。

## 安裝
1. `/plugin marketplace add <github-repo-url>`
2. `/plugin install fpga-design-flow`

## 四個 skill 何時用
| skill | 何時用 |
|-------|--------|
| init-fpga-claudemd | 開新 FPGA 專案，建 CLAUDE.md + 結果模板 |
| fpga-vivado-workflow | 改 RTL 走完整 Vivado+Vitis 流程（含三道門） |
| rtl-iverilog-flow | 只做 RTL+iverilog 驗證（其他平台/簡單專案） |
| fpga-pitfalls | 遇 Vivado/Vitis/時序錯誤查解法 |

## 更新
- 取得新版：`/plugin marketplace update fpga-design-flow` → 更新 plugin
- ⚠️ 本機 cache 的改動不會自動上傳、且更新時會被覆蓋

## 貢獻新踩坑（成長知識庫）
1. 遇到新坑、解決 → agent 用 fpga-pitfalls 條目模板草擬條目
2. 把條目 commit 到 fork/branch → 開 PR
3. 維護者審核 diff → 合併 → 版本 +1 → push
4. 其他人更新 plugin 即取得新條目
