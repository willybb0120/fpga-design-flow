---
name: rtl-iverilog-flow
description: Use when verifying Verilog RTL with iverilog on any platform or a simple project without the Vivado flow - a lightweight plan, RTL, simulate, commit loop
---

# 通用 RTL + iverilog 驗證流程

## 階段 0：先規劃（強制）
動任何 RTL 前，先用 superpowers `brainstorming` 釐清設計、出計畫，再寫 code。

## 階段一：開發 + 驗證門
1. 確認不在 main，開 branch：`git checkout -b feat/<名稱>`
2. 寫 `src/*.v` 與 testbench `sim/tb_*.v`
3. **[門] iverilog 模擬必須 pass**：
   `iverilog -o sim/sim_out sim/tb_<module>.v src/<module>.v && vvp sim/sim_out`
   確認 testbench 印出全 PASS
4. commit（`sim:` / `feat:` / `fix:`）

## 特性
- 平台無關，不碰 Vivado/Vitis
- 需要合成/板上流程時，改用 fpga-vivado-workflow
