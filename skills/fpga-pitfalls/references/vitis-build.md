# Vitis 2025.2 建置踩坑（SDT mode）

### 坑：linker_files/lscript_a9.ld.in 不存在
- **錯誤訊息**：`CMake Error: File .../linker_files/lscript_a9.ld.in does not exist`
- **原因**：Vitis 2025.2 CMake 流程需要 linker script 模板
- **解法**：從 platform 複製 `zynq_fsbl/linker_files/lscript_a9.ld.in` 到 app 的 `src/linker_files/`。注意 `linker_gen()` 執行後會自動刪此資料夾，是設計行為
- **適用版本**：Vitis 2025.2

### 坑：USER_LINKER_SCRIPT 未定義
- **錯誤訊息**：`set_target_properties called with incorrect number of arguments`
- **原因**：`linker_gen()` 生成 lscript.ld 但未設 USER_LINKER_SCRIPT
- **解法**：在 `src/UserConfig.cmake` 加 `set(USER_LINKER_SCRIPT "${CMAKE_SOURCE_DIR}/lscript.ld")`
- **適用版本**：Vitis 2025.2

### 坑：子目錄的 .c/.cpp 沒被編譯
- **錯誤訊息**：`undefined reference to '<func>'`
- **原因**：CMake 只掃頂層 src/，子目錄來源需手動加
- **解法**：在 `UserConfig.cmake` 設 `USER_COMPILE_SOURCES` 與 `USER_INCLUDE_DIRECTORIES` 列出子目錄檔案與 include 路徑
- **適用版本**：Vitis 2025.2

### 坑：XPAR_* 常數全部改名（SDT mode）
- **錯誤訊息**：大量 `<name> was not declared in this scope`
- **原因**：Vitis 2025.2 啟用 SDT，LookupConfig 改用 base address 而非整數 device ID
- **解法**：建 `src/xparameters_compat.h`，用 `#define 舊名 新名` 逐一對照（下表），main 改 include 它
  | 舊名（2019.1） | 新名（2025.2） |
  |---|---|
  | `XPAR_PS7_SCUGIC_0_DEVICE_ID` | `XPAR_XSCUGIC_0_BASEADDR` |
  | `XPAR_PS7_GPIO_0_DEVICE_ID` | `XPAR_XGPIOPS_0_BASEADDR` |
  | `XPAR_PS7_I2C_0_DEVICE_ID` | `XPAR_XIICPS_0_BASEADDR` |
  | `XPAR_AXIVDMA_0_DEVICE_ID` | `XPAR_AXI_VDMA_0_BASEADDR` |
  | `XPAR_VTC_0_DEVICE_ID` | `XPAR_XVTC_0_BASEADDR` |
  | `XPAR_DDR_MEM_BASEADDR` | `XPAR_PS7_DDR_0_BASEADDRESS` |
  | （中斷號）`*_INTR` | 改用 GIC SPI 號（硬體不變，如 52U/57U） |
- **適用版本**：Vitis 2025.2（SDT mode）

### 坑：Digilent wrapper 型別溢位
- **錯誤訊息**：`conversion from 'unsigned int' to 'uint16_t' changes value`，執行期 LookupConfig 失敗
- **原因**：wrapper constructor 參數是 uint16_t，但新版要傳 32-bit base address
- **解法**：把相關 header 的 `uint16_t dev_id` 改成 `UINTPTR dev_id`（中斷 id 改 uint32_t）
- **適用版本**：Vitis 2025.2

### 坑：ps7_init 找不到
- **錯誤訊息**：`invalid command name "ps7_init"`
- **原因**：Vitis 沒自動找到 PS 初始化 TCL
- **解法**：Run Configurations → Target Setup → PS Initialization file，填入 `_ide/psinit/ps7_init.tcl` 路徑
- **適用版本**：Vitis 2025.2
