# Vivado 移植 / 開專案踩坑

### 坑：開舊版 .xpr 跳出 IP Upgrade 對話框
- **錯誤訊息**：（對話框）IP upgrade required
- **原因**：用新版 Vivado 開舊版（如 2019.1）專案
- **解法**：直接點 OK，讓 Vivado 自動升級 IP
- **適用版本**：Vivado 2025.2 開 2019.1 專案

### 坑：詢問是否遷移為新版目錄結構
- **錯誤訊息**：（對話框）migrate to new directory structure
- **原因**：新版 Vivado 想把專案改成 srcs/gen 分離結構；遷移不可逆，且自訂 IP 不會被包含
- **解法**：選 **No**，維持原始目錄結構
- **適用版本**：Vivado 2025.2

### 坑：Windows 路徑超過 260 字元
- **錯誤訊息**：`ERROR: [Common 17-680] Path length exceeds 260-Byte maximum`
- **原因**：專案路徑太深，合成寫 .dcp checkpoint 失敗
- **解法**：Tcl Console 執行 `save_project_as <name> C:/<short_path> -force`
- **適用版本**：Vivado on Windows（各版皆可能）

### 坑：Board files 未安裝
- **錯誤訊息**：`WARNING: Board part '' set for the project is not found`
- **原因**：Vivado 找不到該開發板的 board 定義
- **解法**：`xhub::install [xhub::get_xitems *<vendor>:boards:<board>*]`，或手動複製 vivado-boards repo 對應資料夾到 board store
- **適用版本**：Vivado 2025.2
