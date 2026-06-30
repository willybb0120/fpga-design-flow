---
name: fpga-pitfalls
description: Use when a Vivado synthesis, Vitis build, or FPGA timing error appears (e.g. "Path length exceeds 260", linker_gen, undefined XPAR_*, negative WNS), or before a tool-version migration - a categorized knowledge base of Xilinx/AMD toolchain pitfalls and their fixes
---

# FPGA 踩坑知識庫

## 用法
1. 判斷錯誤屬哪一類 → 開對應 reference 檔
2. 在檔內比對症狀（錯誤訊息）
3. 套用「解法」；注意「適用版本」是否相符

## 分類索引
- `references/vivado-migration.md` — Vivado 開專案 / 版本移植的坑
- `references/vitis-build.md` — Vitis CMake 建置 / SDT mode 的坑
- `references/timing-and-known-issues.md` — 時序違反與其他已知行為

## 新增條目
- **格式**：一律用 `templates/pitfall-entry.md` 的條目模板
- **歸類**：先找符合的現有 reference 檔；都不符再開新類別檔，並在上方「分類索引」補一行
- **觸發**：在開發中查無此坑、最後自行解決時，用模板草擬條目
- **回寫（重要）**：plugin 為版本鎖定，本機 cache 的改動更新時會被覆蓋、也不會傳給別人。新條目必須回到 **source repo** 才算數：
  - 維護者：改 reference 檔 → commit → plugin 版本 +1 → push
  - 成員：草擬條目 → 開 PR / 回報維護者 → 由維護者併入
