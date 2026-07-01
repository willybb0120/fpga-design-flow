# fpga-design-flow

> 讓全實驗室用**同一套流程、同一套專案結構**開發 FPGA。
> 從建檔（標準目錄 + .gitignore）、規劃、RTL 模擬到合成與板上驗證，每一步都有把關；踩過的坑寫成知識庫，下一個人不必重踩。
> harness 中立，支援 Claude Code 與 Codex。

## 為什麼需要它
- **流程不統一**：每個人開發步驟不同，交接、review、品質都難一致
- **專案結構各搞各的**：檔案放哪沒共識，接手別人的專案要重新摸索
- **驗證靠自律**：模擬/時序/板上沒有強制把關，容易漏到燒板才爆
- **經驗綁個人**：踩坑紀錄散落各處，人一走就沒了

## 誰需要使用它

**適合**
- 用 **Xilinx/AMD Vivado + Vitis** 做 FPGA 開發的實驗室成員
- **新進成員**：一鍵拿到標準流程與專案結構，不必從零摸索
- **接手他人專案的人**：每個專案結構一致，好上手
- **知識庫維護者**：集中管理踩坑經驗、審 PR
- 使用 **Claude Code 或 Codex** 的人

**不適合（誠實說明）**
- 非 Xilinx 工具鏈（Intel/Quartus 等）——目前不涵蓋
- 不用 agent（純手動）開發者——skills 需要 agent 觸發才有作用

## 用起來像什麼
開新專案時，agent 先幫你**建出標準目錄結構**（src/sim/constraints/docs/… + .gitignore）和指令檔，全實驗室專案長得一致。之後你要改一個模組，它不會直接寫 code——先帶你釐清設計、出計畫；接著開 branch、改 RTL，**iverilog 沒過不准進合成**；合成後**時序 WNS 沒過不准燒板**；板上驗證通過後，**它停下來等你說「可以合併」才收尾**。撞到 Vivado/Vitis 錯誤，它自動翻踩坑知識庫找解法——查不到、你自己解掉後，再把新坑回寫進知識庫給下一個人。

## 開發流程
![開發流程](docs/flowchart.png)

## 安裝

### Claude Code
1. `/plugin marketplace add https://github.com/willybb0120/fpga-design-flow`
2. `/plugin install fpga-design-flow`

### Codex App
- 側欄 Plugins → 加入此 repo → 安裝 `fpga-design-flow`

### Codex CLI
1. `codex plugin marketplace add https://github.com/willybb0120/fpga-design-flow`
2. `codex plugin install fpga-design-flow`

> Codex 讀 `AGENTS.md`、Claude Code 讀 `CLAUDE.md`；`init-fpga-agents` 以 AGENTS.md 為主檔並另建 CLAUDE.md，兩邊通用。

## 四個 skill 何時用
| skill | 何時用 |
|-------|--------|
| init-fpga-agents | 開新 FPGA 專案，建標準目錄 + .gitignore + AGENTS.md（+ CLAUDE.md）+ 結果模板 |
| fpga-vivado-workflow | 改 RTL 走完整 Vivado+Vitis 流程（含三道門） |
| rtl-iverilog-flow | 只做 RTL+iverilog 驗證（其他平台/簡單專案） |
| fpga-pitfalls | 遇 Vivado/Vitis/時序錯誤查解法 |

## 架構概述

四個 skill 分成兩類——**程序**（照著走的流程）與**資料**（查詢用的知識），加上隨附模板：

| 類別 | skill | 角色 |
|------|-------|------|
| 程序 | init-fpga-agents | 建檔＋建標準結構（一次性） |
| 程序 | fpga-vivado-workflow | 完整開發 SOP：規劃→模擬→合成→板上（反覆） |
| 程序 | rtl-iverilog-flow | 輕量驗證：只做 RTL+iverilog（其他平台/簡單專案） |
| 資料 | fpga-pitfalls | 踩坑知識庫，遇錯查解、隨 PR 成長 |

**串接關係**
```
init-fpga-agents ──建好專案──▶ fpga-vivado-workflow ──遇錯──▶ fpga-pitfalls
（新專案）                      （每次開發）              （查解法/回寫）
                                rtl-iverilog-flow
                                （不走 Vivado 時的輕量版）
```

**設計取捨**
- **程序與資料分離**：知識庫獨立成長，不污染工作流程
- **模板隨 skill**：結果紀錄、`.gitignore`、踩坑條目模板各放在用它的 skill 底下 `templates/`
- **harness 中立**：skills 不綁單一工具，Claude Code / Codex 共用

## 標準專案結構（init-fpga-agents 自動建立）
```
<project>/
├── AGENTS.md / CLAUDE.md   指令檔（強制規則 + 路徑對照 + 流程）
├── .gitignore              忽略 Vivado/Vitis 產出物
├── src/                    RTL 源碼 (.v)
├── sim/                    testbench + 模擬
├── constraints/            .xdc
├── ip_src/                 自訂 IP 源碼
├── docs/                   specs/ + plans/ + result-record.md
├── scripts/                建置/輔助腳本
├── vivado_workspace/       Vivado 專案
└── vitis_workspace/        Vitis 應用
```

## 設計原則
- **先規劃再動手**：先 brainstorm 出 spec 與 plan，不直接跳進 code
- **三道門紀律**：模擬 / 時序 / 板上，每道沒過不前進
- **資料與程序分離**：踩坑知識庫獨立成長，不污染工作流程
- **分發中立**：不綁任何人的私有路徑與單一 harness

## 環境需求
- Vivado / Vitis（合成與板上驗證）
- iverilog（RTL 模擬）
- Claude Code 或 Codex
- 開發環境：WSL / Linux（路徑範例以此為準）

## 更新
- 取得新版：`/plugin marketplace update fpga-design-flow` → 更新 plugin
- ⚠️ 本機 cache 的改動不會自動上傳、且更新時會被覆蓋

## 理想的小團隊使用方法

以一個實驗室（1 位維護者 + 幾位成員）為例：

**角色分工**
| 角色 | 做什麼 |
|------|--------|
| 維護者 | 擁有 repo、審 PR、合併踩坑、發新版 |
| 成員 | 用 skills 開發、把新坑以 PR 回報 |

**運作流程**
1. **設定（一次）**：維護者建好這個 repo，成員各自 `/plugin install`
2. **上手**：成員開新專案用 `init-fpga-agents`，立刻拿到一致的目錄結構與指令檔——不用問「檔案放哪」
3. **日常**：所有人走同一套 `fpga-vivado-workflow`，流程、commit 慣例、三道驗證門都一致，review 與互相接手都無痛
4. **共建知識庫**：任何人踩到新坑、解掉 → agent 用模板草擬條目 → 開 PR → 維護者審核合併
5. **同步**：維護者發新版後，成員 `/plugin marketplace update` → 全體拿到最新踩坑庫
6. **交接**：成員畢業離開，經驗已沉澱在知識庫與標準結構裡，不隨人流失

**小團隊的好處**
- 新人第一天就照標準做，不必等人帶
- 踩過的坑只踩一次，第二個人直接查
- 專案結構統一，跨人 review／接手成本低

## 貢獻新踩坑（知識庫隨實驗室成長）
1. 遇到新坑、解決 → agent 用 fpga-pitfalls 條目模板草擬條目
2. commit 到 fork/branch → 開 PR
3. 維護者審核 diff → 合併 → 版本 +1 → push
4. 其他人更新 plugin 即取得新條目

## License
MIT
